<!-- TEMPLATE — instantiate by copying to tasks/<YYYY-MM-DD-slug>/plan.md. Never edit this
     template in place. Written by the orchestrator after breakdown (skill: task-breakdown). -->

# Plan — <task-id>: <title>

<One paragraph restating the ask in the orchestrator's own words: what will exist when this
plan is done, and for whom. If a PRD exists (tasks/<id>/prd.md), the plan must not
contradict it.>

Out of scope:
- <what agents must not touch — must agree with ORCHESTRATOR_CONSTRAINTS.md scope>

---

Repeat the section below once per subtask. Field shape and semantics are exactly
`templates/task.md` — do not add, drop, or rename fields.

## Subtask <NN>: <imperative title>

status: [todo|doing|verified|blocked|escalated]
Depends on: <subtask NNs, or none>
Files:
- <exact/path> (new)
- <exact/path> (modify)
- <exact/path> (read)
parallel-safe: yes|no

### Goal
<1-2 lines>

### Acceptance criteria
- <observable behavior>

### Verification
- Build: `<command>`
- Test: `<command>`
