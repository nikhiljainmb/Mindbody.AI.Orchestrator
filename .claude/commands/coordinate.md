---
description: fully autonomous pipeline — break down, implement, test, review, fix, report; no approval gates
argument-hint: "<task description>"
---

Run the full pipeline on the task in $ARGUMENTS, end to end, with no approval gates. You are the
orchestrator; the roles table in AGENTS.md defines who does what.

1. **Init.** Create `tasks/YYYY-MM-DD-<slug>/` (slug: 3-5 kebab-case words from the task). If
   `shared-workspace/` holds files from a previous run, archive them into that run's task folder
   first, then start clean (lifecycle per `shared-workspace/README.md`). Write
   `tasks/.../status.md` with state `planning`.

2. **Breakdown.** Apply the `task-breakdown` skill to $ARGUMENTS: write `tasks/.../plan.md`
   (instantiated from `templates/plan.md`), then `shared-workspace/ORCHESTRATOR_CONSTRAINTS.md`
   with `Mode: AUTONOMOUS` per the schema in `shared-workspace/README.md` — written exactly
   once, after breakdown, before any agent is spawned; immutable from here on.

3. **Implement.** For each subtask in plan.md, spawn the `coder` subagent with a context packet
   built per the `context-management` skill: only that subtask plus a pointer to
   ORCHESTRATOR_CONSTRAINTS.md — nothing else. Run subtasks sequentially by default; run in
   parallel ONLY those the plan marks parallel-safe. Set `status.md` to `implementing`.

4. **Test.** Spawn `test-runner`; it writes `TEST_RESULTS.md`. On failures, hand the failing test
   names back to `coder` for a fix — these fix cycles count toward the max-3 budget in
   ORCHESTRATOR_CONSTRAINTS.md. If tests still fail after the budget, mark the run BLOCKED and go
   to step 8.

5. **Review.** Launch `reviewer-architecture`, `reviewer-testing`, `reviewer-quality`, and
   `reviewer-security` IN A SINGLE MESSAGE — four parallel Task calls; that is what makes them
   run concurrently. Give each the same diff spec. Set `status.md` to `reviewing`.

6. **Synthesize.** Spawn `review-synthesizer`; it writes `REVIEW_CHECKLIST.md`. On an overall
   verdict of REQUEST_CHANGES: send the Must-fix items inline to `coder`, re-run `test-runner`,
   re-run ONLY the reviewers whose verdict was REQUEST_CHANGES (approving reviewers' reports
   carry forward), then re-synthesize. After 3 cycles without APPROVE or APPROVE_WITH_NITS,
   go to step 8 with Status: ESCALATED.

7. **Document.** Only if the public surface changed (API, CLI, config, exported types): spawn
   `doc-generator` with the changed-file list and a shipped summary in its prompt. It reports
   the doc files it touched in its final response. Otherwise skip.

8. **Report.** Write `shared-workspace/FINAL_STATUS.md` per the schema in
   `shared-workspace/README.md` (including the `## Documentation` list from step 7, if any),
   copy it to `tasks/.../final-status.md`, and set `status.md` to `done`, `blocked`, or
   `escalated`. Print a summary of at most 10 lines, ending with the two inspection commands:
   `ls shared-workspace/` and `ls tasks/`.

Hard rules for this pipeline:

- You never edit source code here — AGENTS.md invariant rule 2. Every code change goes through
  `coder`.
- Writing FINAL_STATUS.md as DONE while any Critical finding is open is a protocol violation —
  AGENTS.md invariant rule 6.
