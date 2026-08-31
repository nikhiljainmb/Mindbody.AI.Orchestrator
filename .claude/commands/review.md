---
description: parallel code review only — no implementation, no fixes; reviews the working-tree diff by default
argument-hint: "[path | branch | omit for uncommitted diff]"
---

Review only. Nothing gets implemented, nothing gets fixed. You are the orchestrator.

1. **Resolve the target** from $ARGUMENTS:
   - No argument → `git diff` of uncommitted work; if the tree is clean, fall back to the last
     commit.
   - A path → review those files.
   - A branch name → diff the current branch against it.

2. **Constraints.** If `shared-workspace/` holds files from a previous pipeline run, archive
   them into that run's task folder first (lifecycle per `shared-workspace/README.md`; a prior
   review run's files may simply be discarded). Then write
   `shared-workspace/ORCHESTRATOR_CONSTRAINTS.md` using the schema's REVIEW_ONLY variant in
   `shared-workspace/README.md`: `Mode: REVIEW_ONLY`, a review-id header, the resolved target
   spec as scope, and the reviewers list — the plan and quality-bar fields are omitted. No task
   folder is created for a review run — its outputs are ephemeral by design.

3. **Review.** Launch `reviewer-architecture`, `reviewer-testing`, `reviewer-quality`, and
   `reviewer-security` per the CLAUDE.md parallel-launch rule: all four in ONE message. Each
   prompt carries the review-id, cycle 1, the same target spec, and `Mode: REVIEW_ONLY` — no
   test-runner ran, so `TEST_RESULTS.md` is absent by design (reviewer-testing runs its branch
   audit instead; the others are unaffected).

4. **Synthesize.** Spawn `review-synthesizer`; it writes `REVIEW_CHECKLIST.md`. State
   `Mode: REVIEW_ONLY` in its prompt — no test-runner ran, so `TEST_RESULTS.md` is absent and
   the coverage cross-check is N/A, not MISSING.

5. **Report.** Print `REVIEW_CHECKLIST.md` verbatim and stop.

Hard rules for this pipeline:

- No source edits, no fixes, no task folder for this run. The only writes are the
  `shared-workspace/` coordination files — plus step 2's archiving of a PREVIOUS pipeline
  run's files into that run's task folder, which is lifecycle bookkeeping, not review output.
- If findings warrant fixes, offer `/coordinate` to fix them — but do not start it yourself.
