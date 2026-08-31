---
description: fully autonomous pipeline — break down, implement, test, review, fix, report; no approval gates
argument-hint: "<task description>"
---

Run the full pipeline on the task in $ARGUMENTS, end to end, with no approval gates. You are the
orchestrator; the roles table in AGENTS.md defines who does what.

1. **Init.** Create `tasks/YYYY-MM-DD-<slug>/` (naming per `tasks/README.md`). If
   `shared-workspace/` holds files from a previous run, archive them into that run's task folder
   first, then start clean (lifecycle per `shared-workspace/README.md`). Write
   `tasks/.../status.md` with state `planning`.

2. **Breakdown.** Apply the `task-breakdown` skill to $ARGUMENTS: write `tasks/.../plan.md`
   (instantiated from `templates/plan.md`), then `shared-workspace/ORCHESTRATOR_CONSTRAINTS.md`
   with `Mode: AUTONOMOUS` per the schema in `shared-workspace/README.md` — written exactly
   once, after breakdown, before any agent is spawned; immutable from here on.

3. **Implement.** For each subtask in plan.md, spawn the `coder` subagent with a full context
   packet per the `context-management` skill — subtask, constraints pointer, files in scope,
   prior state, deliverable, NOT YOUR JOB — and nothing beyond it. Run subtasks sequentially by
   default; run in parallel ONLY those the plan marks parallel-safe, and tell each parallel
   coder to RETURN its CODING_PROGRESS.md entry in its final response instead of writing it —
   you append the returned entries in subtask order, so the file keeps one writer at a time.
   If a coder returns BLOCKED: a packet gap → fix the packet and re-delegate
   (context-management skill); a scope conflict with the constraints' Allowed paths →
   constraints are immutable, so go to step 8 with Status: BLOCKED, recording the conflict and
   your recommendation (widening scope means a new run). Set `status.md` to `implementing`.

4. **Test.** Spawn `test-runner`; it writes `TEST_RESULTS.md`. On failures, hand the failing
   test names back to `coder` — at most 2 test-fix attempts here, separate from the review-fix
   cycle cap. Still failing → mark the run BLOCKED and go to step 8.

5. **Review.** Launch `reviewer-architecture`, `reviewer-testing`, `reviewer-quality`, and
   `reviewer-security` per the CLAUDE.md parallel-launch rule: all four in ONE message. Each
   prompt carries the task-id, the cycle number, and the same diff spec. Set `status.md` to
   `reviewing`.

6. **Synthesize.** Spawn `review-synthesizer` (prompt: task-id, cycle number, max cycles,
   mode); it writes `REVIEW_CHECKLIST.md`. A missing or malformed reviewer report is not a
   verdict — re-spawn that reviewer and re-synthesize before acting on the checklist. On an
   overall verdict of REQUEST_CHANGES: send the Must-fix items inline to `coder`; re-run
   `test-runner`; re-run ONLY the reviewers whose verdict was REQUEST_CHANGES (reports carry
   forward per the contract), each re-run prompt carrying the new cycle number, that
   reviewer's open findings copied inline, and a diff spec extended to include the fix
   commits — the reviewer re-checks those findings AND sweeps the fix diff; if a fix touched
   files outside the open findings' scope, re-run all four reviewers instead. Then
   re-synthesize. After the max review-fix cycles set in ORCHESTRATOR_CONSTRAINTS.md without
   APPROVE or APPROVE_WITH_NITS, go to step 8 with Status: ESCALATED.

7. **Document.** Only if the public surface changed (API, CLI, config, exported types): spawn
   `doc-generator` with the changed-file list and a shipped summary in its prompt. It reports
   the doc files it touched in its final response. Otherwise skip.

8. **Report.** Instantiate `templates/verification-checklist.md` into
   `tasks/.../verification-checklist.md` and fill it per verification-protocol Step 5 — every
   tick with its evidence. Then write `shared-workspace/FINAL_STATUS.md` per the schema in
   `shared-workspace/README.md` (including the `## Documentation` list from step 7, if any),
   copy it to `tasks/.../final-status.md`, and set `status.md` to `done`, `blocked`, or
   `escalated`. Print a summary of at most 10 lines, ending with the two inspection commands:
   `ls shared-workspace/` and `ls tasks/`.

Hard rules for this pipeline:

- You never edit source code here — AGENTS.md invariant rule 2. Every code change goes through
  `coder`.
- Writing FINAL_STATUS.md as DONE while any Critical finding is open is a protocol violation —
  AGENTS.md invariant rule 6.
