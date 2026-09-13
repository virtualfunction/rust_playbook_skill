---
name: rust-development-playbook
description: "Use when writing, editing, building, testing, debugging, benchmarking, or reviewing Rust code — any task touching .rs files, Cargo.toml, or workspaces; cargo/rustc/clippy failures; borrow-checker, linker, or build-cache surprises; slow test/build iteration; or planning multi-task Rust implementation work. Invoke at the start of any Rust session."
---

# Rust Development Playbook

Two lists: **proven strategies** (these demonstrably converged fast) and **anti-patterns** (these demonstrably burned sessions). Apply the first list; treat the second as stop signals.

## Core Principle

**Prove the mechanism before touching the code.** Every fast session followed evidence → minimal fix. Every slow session patched, guessed, or assumed — then re-derived the same lesson painfully. When in doubt, instrument before editing.

## Do — Proven Strategies

1. **Review plans adversarially before executing.** Self-review and independent review of written plans catch verification steps that can never pass, wrong expected exit codes, lock-scope gaps, and stack-size/harness assumption errors — before they cost a debug arc.
2. **Write the failing test first (RED), then the minimal fix (GREEN).** Confirm the test fails for the *diagnosed* reason, not incidentally.
3. **Pin every bug class as a test** once fixed, so the class can never silently return.
4. **Instrument and dump intermediate state** (IR, AST, traces, logs) and confirm the root cause from the dump *before* writing the fix. One dump beats ten patches.
5. **Isolate work**: fresh branch or worktree per task, green baseline confirmed at setup, tests/lints green at every commit.
6. **Measure before optimizing.** Collect timings, split compile cost from test cost, find the dominant term, fix the cheapest option first, and confirm the approach before implementing.
7. **Land small, compile-checkable increments.** Verified small slices go green immediately; big-bang batches take an order of magnitude more debug cycles.
8. **Externalize state at every green point**: WIP commits whose messages are written for the *next reader*, status notes, and a gotcha ledger updated as you go — not only at session end. This is what survives context loss, timeouts, and model switches.
9. **Write handoff notes containing hard-won process notes** (build gotchas, harness conventions, tool quirks), not just task state. They measurably improve the next session.
10. **Verify honestly.** Determine whether lint/format failures are yours or pre-existing; only fix what your diff owns. Review unexpected results before accepting them, and retract wrong conclusions explicitly.
11. **Make silent fallbacks honest.** After proving nothing depends on a silent default, replace it with a loud error.
12. **Determinism by construction.** Sort anything hash-ordered that feeds generated output or decisions; add run-twice-diff checks where nondeterminism would be a bug.
13. **Audit first-use coverage.** When a feature lands, identify which existing test exercises the *first realistic use* of each new code path; add one if none does.
14. **Stop-at-green is a discipline.** When an investigation exceeds its budget, checkpoint: commit verified state, write where things stand, resume fresh. This converts botched arcs into minutes of recovery.

## Avoid — Proven Anti-Patterns

1. **Blind patching.** More than two fix attempts without re-reading actual state makes things worse. After two failures: stop editing, dump state, prove the mechanism, write the fix once.
2. **Placeholder/draft implementations "to be filled in later."** They pass exhaustiveness, do nothing at runtime, and get thrown away anyway. Write the real thing once or trap honestly.
3. **Attributing root cause before the mechanism is proven.** Premature bisect narratives get published, then retracted, costing more than starting over. Evidence before blame.
4. **Experimental churn on a live system.** Poking extra instances reshuffles internal state (ids, caches, ordering) and *masks* the bug instead of isolating it. Freeze the repro, then instrument.
5. **Hand-tracing code in your head instead of running it.** Mental simulation of nontrivial logic produces wrong conclusions at scale; dump the intermediate representation or run the binary and assert the output.
6. **Assuming library/binding API capabilities or harness conventions.** Check signatures, versions, and docs *before* writing code that depends on them — missing wrappers, default stack limits, and exit-code contracts differ from expectations.
7. **Caching or skip logic keyed on unstable invariants** (fresh ids, per-run numbering, iteration order). Document the invariant or don't cache on it.
8. **Verification commands that can't fail loudly.** `&& echo SUCCESS || echo whatever` patterns and exit-code-agnostic checks report success on path errors. Assertions or explicit exit-code checks only.
9. **Trusting stale artifacts.** "Patch didn't take" is a first-class hypothesis: rebuild the exact binary, clean the suspect crate, print the artifact identity in test output.
10. **Chasing pre-existing drift** (formatting, lints) outside your diff. Note it, don't fix it.
11. **Continuing past a stated stopping point.** This reliably produces degenerate fix loops. The stop-at-green commit is the exit.
12. **Grinding on a stalled problem without changing approach.** A long no-progress turn is the signal to simplify the sub-problem or weaken the intermediate goal — not to try harder on the same shape.
13. **Relying on prose notes to enforce operational checks.** Ledger entries get ignored under pressure; known violations recurred despite warnings. Script the check (e.g., `git status --porcelain` before staging, CWD assertion before committing).
14. **Scripted file writes** (Python/echo/heredoc for source files). Recurring silent corruption: dropped lines, mangled characters. Use file tools; if a scripted transform is unavoidable, re-read and rebuild immediately.
15. **Unsolicited actions on in-flight tasks.** Edits made before the user's direction lands force reverts and erode trust. The cheapest action is the one not taken without a nod.
16. **Running verification concurrently with in-flight work** without per-run isolation (colliding output dirs, shared state, port/filesystem conflicts). Parallelism needs isolation or it serializes you the hard way.
17. **Overbroad staging.** Never `git add -A` (or path-globbed equivalents) without a status pre-check; untracked files from parallel sessions get swept into commits.

## Red Flags — Stop and Apply the DO List

- You are on your third fix attempt without new evidence.
- A fix "should" work but the behavior didn't change.
- You are explaining behavior to yourself without running anything.
- A build/test pass is green but you can't name the artifact version.
- You are about to stage files you did not personally list.

**Any of these: stop, instrument, prove the mechanism, then fix once.**
