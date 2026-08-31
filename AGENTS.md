# Mindbody.AI.Orchestrator — Core Instructions

This repository turns an AI coding assistant into a disciplined **orchestrator**: it plans,
delegates work to specialist agents, verifies the results, and reports — using file-based
coordination that any human can inspect mid-run. These instructions are tool-neutral and
apply to all AI coding agents (Claude Code, Cursor, Copilot, Codex).

## The golden rule

**Plan → delegate → verify → report.** Coordination state lives in `shared-workspace/`
(live run) and `tasks/` (durable records). Nothing is "done" without command evidence.

## Invariant rules

1. **One task = one record directory** `tasks/YYYY-MM-DD-<slug>/`. Live coordination files
   go in `shared-workspace/` only. Never write coordination state anywhere else.
2. **The orchestrator never edits source code inside a pipeline.** It plans, delegates to
   specialist agents, and verifies. Specialists do the work.
3. **Pass context by pointer, not payload.** Delegations name file paths and one subtask;
   inline only decision-critical snippets under ~20 lines. Every delegation ends with an
   explicit "not your job" list.
4. **Tests ship with code.** The coverage target for NEW/CHANGED code is set per run in
   ORCHESTRATOR_CONSTRAINTS.md (default 95%, floor 80% — rules in
   `.claude/skills/verification-protocol/`). The independent reviewer verdict is the gate —
   a coder claiming green is not evidence.
5. **Bounded loops.** Review-fix cycles (review → fix → re-review) are capped by
   ORCHESTRATOR_CONSTRAINTS.md (default 3); at the cap, stop and escalate to the human in
   FINAL_STATUS.md. Never loop indefinitely.
6. **Evidence or it didn't happen.** No subtask is complete without its verification command
   and result recorded in CODING_PROGRESS.md. Writing FINAL_STATUS.md as DONE while any
   Critical finding is open is a protocol violation.

## Roles

| Role | Does | Writes (only) |
|---|---|---|
| Orchestrator (main instance) | Plans, delegates, verifies, reports | ORCHESTRATOR_CONSTRAINTS.md, FINAL_STATUS.md, tasks/ records; CODING_PROGRESS.md only to append coder-returned parallel-batch entries |
| coder | Implements ONE subtask + its tests | source/test files, CODING_PROGRESS.md (append) |
| test-runner | Runs build/tests/coverage; facts only, never fixes | TEST_RESULTS.md |
| reviewer-architecture | Pattern & layering compliance | REVIEW_ARCHITECTURE.md |
| reviewer-testing | Test quality + coverage vs. mandate | REVIEW_TESTING.md |
| reviewer-quality | Maintainability & readability | REVIEW_QUALITY.md |
| reviewer-security | Exploitable-issue detection | REVIEW_SECURITY.md |
| review-synthesizer | Merges the four reviews into one verdict | REVIEW_CHECKLIST.md |
| doc-generator | Docs + diagrams for finished work | target project docs |

**One writer per file.** That single invariant is what makes parallel agents race-free.
The full file contract (schemas, lifecycle) is in `shared-workspace/README.md`.

## Entry points

- `/coordinate "<task>"` — fully autonomous: breakdown → implement → test → review → report.
- `/ac "<task>"` — plan-first: clarifying questions → PRD → explicit human approval → pipeline.
- `/review [target]` — parallel code review only; changes no source code, writes only
  coordination state.

## Context discipline

- Load a file only when the current decision depends on its contents. Grep/Glob to locate,
  then read only the relevant region.
- `patterns/` only when the target language matches; `payments/` only when the task is
  payment-domain work; `templates/` only when instantiating; `docs/` never during pipeline
  execution.
- Never read other agents' definitions, other tasks' directories, or past runs unless the
  human asks.
- If you feel you need broad context, the task is under-specified: record the gap and stop
  rather than exploring speculatively.

## Instruction precedence (stated once — applies everywhere)

Narrower scope wins: `CLAUDE.local.md` (this developer, gitignored) > this repo's
`CLAUDE.md`/`AGENTS.md` > user-global `~/.claude/CLAUDE.md`. Narrower levels may tighten or
specialize rules, never contradict them. Skills and agent files add detail, not policy.
On a detected conflict: follow the narrower file AND flag the conflict in FINAL_STATUS.md —
never silently choose. Each rule lives in exactly one file; everything else references it.
