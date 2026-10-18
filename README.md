# rust_playbook_skill — agentic Rust playbook skills

Two user-level [agent skills](https://code.claude.com/docs/en/skills) for agentic
Rust sessions, born from one pre-split playbook and maintained as siblings:

| subfolder | skill | covers |
|---|---|---|
| [`rust-session-playbook/`](rust-session-playbook/SKILL.md) | session discipline | running/supervising builds, test gates, benches, measurement legs; dispatching and reviewing subagent work; long-leg protocol, sentinels, checkpoints, verdict hygiene |
| [`rust-development-playbook/`](rust-development-playbook/SKILL.md) | development strategy | proven strategies and anti-patterns for writing, editing, building, testing, debugging, and reviewing code in cargo workspaces |

## Install

Copy a subfolder's `SKILL.md` into your runtime's user skills directory under the
skill's own name (e.g. `~/.agents/skills/rust-session-playbook/SKILL.md`).

## Layout convention

Each subfolder is one skill: `SKILL.md` at the subfolder root, named for the
skill's frontmatter `name`. The two skills split from a single pre-split
playbook (preserved in this repo's early history); the session playbook carries
the harness/session discipline, the development playbook carries the code-level
strategy. Sources of truth are the per-skill local repos; this collection is the
published snapshot — sync by copying a changed `SKILL.md` into its subfolder.
