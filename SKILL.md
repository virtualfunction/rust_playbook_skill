---
name: rust-session-playbook
description: "Use when running or supervising Rust builds, test gates, benches, or measurement legs that may outlive a single tool call; dispatching, resuming, or reviewing subagent work; a session starts echoing no-op commands, backgrounding legs, or climbing in cost per step; writing measurement numbers or DONE/SHIP/merge verdicts. Invoke at the start of any Rust session expected to run gates, measurements, or delegation."
---

# Rust Session Playbook

The harness, not the language, is what burns sessions. Two laws:

1. **Every command either completes inside the turn or follows the long-leg protocol. There is no third state.** A command whose only purpose is to wait (`echo ok`, bare `sleep`) resumes you into the same turn — it cannot end it — so a session that believes otherwise poll-loops until it is killed.
2. **Evidence must outlive the session.** A claim that rests on an artifact that dies with the session (tmp scratch, another agent's console) is not evidence.

**Applies to subagents equally.** Being dispatched for one specific task is not an exemption — in the reference corpus the only skill that fired inside a runaway session was self-waived on exactly that ground, at step 801 of 816. These protocols bind parent and child alike; they work only if followed from step 1, not after a late skill load.

## Do — Protocols (each has required slots; missing one is a violation)

1. **Short commands and gates (≤ ~2 min): foreground, explicit tool timeout, serial, one writer per log.** The harness kills foreground calls at a default cap (~120 s) *silently* — the in-command `timeout 900` does NOT raise it. Pass the tool-call timeout parameter explicitly on anything that might exceed the cap. This one parameter prevents most background-cascade failures.
2. **Long legs (> ~2 min expected duration — the real leg's wall time, not a stand-in's): the notified-background single.** One leg at a time, strictly serial, one writer per log. Launch it, write a completion sentinel at leg end (`cmd && touch sentinel` or an equivalent trap), then end the turn or issue **one real status check per message** (`test -f sentinel && tail -n 3 log`). Never a sleep-poll series, never a filler command. Ending the turn needs no command at all; if your model insists on a final tool call, ONE `true` after launching the leg is the sanctioned yield — a series of no-op commands is the runaway in disguise. Ending the turn with a leg in flight assumes the harness resumes you on completion — if it does not, use ONE blocking wait with a generous tool timeout instead; still never a poll series. Every leg's launch command writes the manifest into its own log's first lines (`echo "manifest: leg=<id> sha=$(git rev-parse -short HEAD) start=$(date -Is)" && <cmd>`; the gate binary may stamp its own output) — "did two legs run concurrently?" becomes a grep, not forensics.
3. **Checkpoint before every long leg.** A file (not memory) recording: task, command running, its log path, sentinel path, what is done, next step. On ANY resume — interruption, restart, handoff — read the checkpoint first and salvage existing artifacts before re-running anything. After a suspected crash, verify tree state on disk (`git status` / diff) before trusting any report — one silent session-store crash was caught only this way, by the dispatcher, not by any error signal.
4. **Evidence lives under the worktree** in a git-ignored dir. tmp/ephemeral scratch is wiped by restarts and invalidated baselines cost full re-measurement. Every claimed number cites a surviving artifact path.
5. **Frozen numbers:** a number enters a document of record only after its producing command exited 0. A placeholder table in a spec/perf doc is fabrication even if you revert it — the violation is the write.
6. **Single writer per worktree.** Before dispatching help onto a tree, confirm no live agent holds it — an agent with parked or in-flight legs counts as live even if its turn ended. Never commit another agent's uncommitted work. Concurrent writers commit each other's half-state and fabricate attribution. Shared scratch and report files take a single named writer too — concurrent appends to one report are how "someone else modified my file" mysteries are manufactured.
7. **Incremental reports.** Append findings to the report file as produced; write the full report to a file BEFORE returning it in chat (long single-message payloads get truncated or garbled in transit; a crashed session with no file leaves nothing).
8. **Verification floor:** a verdict that gates a merge re-runs the primary fast gate in-session. Log-reading verifies provenance, not correctness — an audit-only review says so in its verdict. Every green/ship claim names the command run in the same session. A reviewed claim cites the one-command probe that verifies it — a reviewer forced to re-derive a claim instead of re-running its probe files that as a feedback entry in the project's feedback ledger (the 6–10× reasoning:output ratio is the signature of reviews without probes). When adjudicating failures, use `systematic-debugging`; before any DONE/SHIP claim, `verification-before-completion` — these were the two skills never invoked in 214 reference sessions, both at exactly these moments.
9. **Context economy.** Every step re-reads the whole context; cost grows quadratically with session length. When a session passes ~200 messages or per-step input visibly climbs, summarize state to the checkpoint file and continue fresh. Reviews and fixes belong in fresh small-brief subagents — fresh-context-per-subagent was the cheapest reliable structure found.
10. **Persist corrections immediately.** A standing preference the human states (model choice, protocol, scope) goes into AGENTS.md or the plan in the same turn. Chat-only corrections decay within a day.
11. **Briefs carry pointers, not vibes.** Every dispatch names the exact files, sections, and artifact paths to read, and states what is already done or already known — every subagent re-reads the world from scratch (one 21K-line file was read 552 times across 70 sessions), and rescue dispatches that omit known state force expensive re-derivation. Briefs also carry the pre-answered facts so no subagent re-litigates them: the next free D-id, operative ruling clauses quoted verbatim, reverted-commit shas, pre-registered defaults for ambiguous dispositions — and when the task touches classification or corpus claims, the census/probe command is a pre-registered step-0 whose output the brief cites. A precise brief is the cheapest tokens you ever spend.
12. **Probe before derivation.** A hypothesis about what the system does (runtime behavior, compiled-code behavior, corpus content, tool flags, semantics) is answered by its cheapest runnable probe — one grep, one log read, one run — inside one tool call. If no probe exists, file the gap in the report file, take the pre-registered conservative branch, move on — never loop on derivation. Deriving a fact that a grep/log/run answers is the same violation as polling. At every wave gate, the speculation retrospective mines where probes were missing and converts each episode into a pin, comment, brief fact, or ledger entry (the frozen recipe: the wave's spec, Appendix A; the gate recipe step: `perf.md`).

## Avoid — Rationalizations (observed verbatim, all costly)

| Excuse | Reality |
|---|---|
| "I'll end my turn here and get resumed when it finishes" (after `echo`) | A filler command never ends a turn. This exact belief produced 750+ no-op steps and zero output in one session. |
| "It's only a ~20 s command, backgrounding is harmless" | Backgrounded legs outlive suspends and restarts; an unattended failure costs a rescue session and races the re-run. |
| "I put `timeout 1500` in the command" | That does not raise the harness tool cap. The tool-call timeout parameter is the only lever. |
| "The contract allows polling with one check per message" | If your transcript shows `sleep N; tail` three times running, you are looping — stop, checkpoint, end turn. |
| "They're placeholders; measurement comes later" | Placeholder numbers in a doc of record are fabrication. Measure first or write "unmeasured". |
| "The log says pass; no need to re-run" | A log is the writer's claim about itself. Verdicts ride on in-session reproduction. |
| "The skill contract is binding, that's enough" | Prose does not enforce. Assume the rule will be violated at the worst moment; make the check mechanical where you can (sentinel files, timeout params, writer checks). |
| "I'll just emit `true` to end my turn" (repeatedly) | One yield per leg launch, then silence. A series of no-op commands is the runaway wearing a different token — loop-watch discriminates by cadence, and a dense no-op run gets interrupted. |

## Red Flags — STOP and checkpoint now

- Your last three commands were echoes, sleeps, or tail-with-no-verdict.
- You are about to write a number whose command has not exited yet.
- You are about to claim DONE while the gate ran in a different session, a background shell, or a log you did not create.
- Another agent is editing the same worktree.
- Your per-step context is several hundred K tokens and climbing.
- A resume is needed and no checkpoint file exists.
- Your tool call went out empty or truncated and the session stalls — re-emit it once; if still stuck, the checkpoint file is the recovery path for whoever resumes you.
- Your replies are being torn down at the output-token limit, mid-sentence — stop emitting one long message; append to the report file in chunks instead.
- You are re-deriving session state from memory for the second time — write/read the checkpoint instead.
- You are predicting a pending leg's outcome in prose — read the log when the sentinel lands.
- The same tool-parameter misuse has fired twice — switch to a different tool form.
- You are deliberating whether to invoke a skill — fire or skip in one call.

All of these mean: write the checkpoint, run the one real check, end the turn cleanly.

## Scope

This skill governs the session around the work — long commands, delegation, evidence, verdicts. It does not replace build/test strategy (see `rust-development-playbook`) or any project's ratified measurement protocol; where a project documents a different long-leg convention, that document wins.
