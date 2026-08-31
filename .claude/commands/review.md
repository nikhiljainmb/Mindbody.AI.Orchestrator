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
   review run's files may simply be discarded). Then write a minimal
   `shared-workspace/ORCHESTRATOR_CONSTRAINTS.md` with `Mode: REVIEW_ONLY`, the resolved target
   spec, and the reviewers list, per the schema in `shared-workspace/README.md`. No task folder
   is created for a review run — its outputs are ephemeral by design.

3. **Review.** Launch `reviewer-architecture`, `reviewer-testing`, `reviewer-quality`, and
   `reviewer-security` in ONE message — four parallel Task calls — each with the same target
   spec.

4. **Synthesize.** Spawn `review-synthesizer`; it writes `REVIEW_CHECKLIST.md`. State
   `Mode: REVIEW_ONLY` in its prompt — no test-runner ran, so `TEST_RESULTS.md` is absent and
   the coverage cross-check is N/A, not MISSING.

5. **Report.** Print `REVIEW_CHECKLIST.md` verbatim and stop.

Hard rules for this pipeline:

- Zero modifications outside `shared-workspace/` — no source edits, no fixes, no task folder.
- If findings warrant fixes, offer `/coordinate` to fix them — but do not start it yourself.
