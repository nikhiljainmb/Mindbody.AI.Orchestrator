# Claude Code wiring

@AGENTS.md

## Claude-specific rules

- Workflows start via `/coordinate`, `/ac`, or `/review` — do not improvise a pipeline.
- Delegate implementation to the `coder` subagent. You (the main instance) are the
  orchestrator: AGENTS.md invariant rule 2 applies.
- The parallel-launch rule: launch the four reviewers in ONE message (four parallel Task
  calls) — a single multi-Task message is what makes them run concurrently.
- Subagents inherit nothing from this conversation. Build each Task prompt as a context
  packet per `.claude/skills/context-management/`.

## Load on demand — never preemptively

| When you need to… | Load |
|---|---|
| Break a task into subtasks | skill: `task-breakdown` |
| Declare anything "done" / judge test sufficiency | skill: `verification-protocol` |
| Delegate to a subagent / trim bloated context | skill: `context-management` |
| Build a PRD or ask clarifying questions | skill: `prd-building` |
| See coordination file schemas | `shared-workspace/README.md` |

Personal overrides live in `CLAUDE.local.md` (gitignored — copy `CLAUDE.local.md.example`).
