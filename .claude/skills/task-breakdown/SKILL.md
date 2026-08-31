---
name: task-breakdown
description: >-
  Decompose a development task into small, independently verifiable subtasks with explicit
  dependency edges and observable acceptance criteria. Use when planning any implementation,
  and whenever /coordinate or /ac begins execution — the plan this produces is what every
  downstream agent works from.
---

# task-breakdown

## Step 1 — Fix the scope before slicing

Read the task statement (or, under `/ac`, the approved PRD in `tasks/<id>/prd.md`). List what
is in scope, what is explicitly out, and which files/dirs the work touches. If you cannot name
the touched files, the task is under-specified: record the gap and stop (AGENTS.md, context
discipline) rather than slicing blind.

## Step 2 — Slice by the sizing rule

One subtask = one concern, 30–90 minutes of work for a focused agent. One concern means one
behavior, one layer, or one seam — not one file. Too small wastes delegation overhead; too big
hides failures until review. If describing a slice needs "and", split it.

## Step 3 — Fill every required field

Each subtask carries exactly the fields defined in `templates/task.md` — no more, no fewer.
Do not invent your own shape; the coder and test-runner parse that shape.

## Step 4 — Write observable acceptance criteria

Criteria describe what an outside observer can check, never what the coder should do.

- Observable: "POST /payments over the rate limit returns 429 with Retry-After honored."
- Procedural (reject): "Edit the handler to add rate limiting."

Every criterion must be checkable by the subtask's verify command(s) or by reading the diff.
If you cannot say how it will be checked, it is not a criterion yet.

## Step 5 — Tests ship inside the subtask

Every subtask that changes code includes its own tests, written in the same coder invocation
(gate mechanics: `.claude/skills/verification-protocol/`). Never emit a trailing "write tests"
subtask — it decouples tests from the code they cover and is always the slice that gets
squeezed.

## Step 6 — Draw the dependency graph

Give every subtask explicit `depends-on` edges (subtask IDs). No implicit ordering: if B needs
A's interface, say so. Mark a subtask parallel-safe ONLY when its file set is disjoint from
every other parallel-safe subtask's file set. Shared file = sequential, no exceptions.

## Step 7 — Emit the two outputs

1. `tasks/<id>/plan.md`, instantiated from `templates/plan.md`: subtask list, dependency
   graph, verify commands.
2. `shared-workspace/ORCHESTRATOR_CONSTRAINTS.md`, per the schema in
   `shared-workspace/README.md` — created before any agent is spawned, immutable during the
   run (lifecycle lives in that contract).

## Anti-patterns

- A final "wire everything up" subtask. Integration was never planned; every subtask must
  leave the system integrated.
- Vague criteria: "works correctly", "handles errors gracefully" — not observable, not
  criteria.
- Subtasks over ~90 minutes. Split by behavior or layer until under the cap.
- Hidden coupling between parallel-safe subtasks: a shared helper, config file, or DI
  registration both will edit. Disjoint means disjoint.

## Gotchas

- Subagents inherit nothing. A subtask that only makes sense with this conversation open is
  wrong: its plan.md entry plus its context packet
  (`.claude/skills/context-management/`) must stand alone.
- `depends-on` covers knowledge, not just files. B can touch different files and still depend
  on the interface decision A makes — model that edge.
- Verify commands must be resolvable at plan time. "Run the tests" is not a command;
  `dotnet test --filter FullyQualifiedName~PaymentLimits` is.
- Never renumber mid-run. Subtask IDs are referenced from CODING_PROGRESS.md and review
  findings; append new subtasks at the end.
