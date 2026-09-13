# rust-development-playbook — a skill for agentic Rust work

A user-level [agent skill](https://code.claude.com/docs/en/skills) that
installs a battle-tested session playbook for **any** Rust work: writing,
editing, building, testing, debugging, benchmarking, and reviewing code in
cargo workspaces.

## What it encodes

What repeatedly worked, what repeatedly failed, and the mechanisms
behind both. It is organized as two explicit lists plus stop signals.

Highlights:

- **Proven strategies (Do)** — adversarial plan review before executing;
  RED-first tests with bug classes pinned as tests; dump-and-prove-the-mechanism
  before patching; worktree isolation with green baselines; measurement before
  optimization; small compile-checkable increments; externalized state at every
  green point; honest verification and determinism by construction.
- **Anti-patterns (Avoid)** — the proven session-burners: blind patching
  (beyond two attempts), placeholder drafts, attribution before evidence,
  experimental churn that masks bugs, hand-tracing instead of running,
  assumed library/harness conventions, caches on unstable invariants,
  verification that can't fail loudly, stale artifacts, chasing pre-existing
  drift, prose-instead-of-scripts for operational checks, scripted file
  writes, unsolicited actions, unisolated parallel verification, overbroad
  staging.
- **Red flags** — five self-check signals ("third fix attempt without new
  evidence", "green but can't name the artifact version", ...) that trigger
  stop-and-instrument instead of another guess.

Core principle: **prove the mechanism before touching the code.** Fast
sessions went evidence → minimal fix; slow sessions patched, guessed, or
assumed.

## Install

The skill is installed user-wide at:

```sh
~/.agents/skills/rust-development-playbook/
```

`~/.agents/skills/` is the cross-runtime user-level skills directory
(Claude Code, Codex, Copilot CLI, and Gemini CLI all recognize it). For
[opencode](https://opencode.ai), copy the same folder to
`~/.config/opencode/skills/rust-development-playbook/` if your setup prefers
that path — the format is compatible.

Or copy `SKILL.md` into the skills directory of any agent platform that
supports the SKILL.md format.

The skill triggers automatically whenever an agent has to do Rust work —
touching `.rs` files or `Cargo.toml`, running `cargo`/`rustc`/`clippy`,
debugging borrow-checker or build issues, or planning multi-task Rust
implementation work.

## Pairing

For faster feedback loops, run `rust-analyzer`
locally and keep `cargo fmt`/`cargo clippy` invocations in the project's
documented form (the playbook's rules reference checking house conventions
before relying on them).

## Files

- `SKILL.md` — the skill itself (frontmatter trigger + Do/Avoid lists + red flags).
- `README.md` — this file.

## License

MIT
