---
description: plan-first pipeline — clarifying questions, PRD, explicit approval, then autonomous execution
argument-hint: "<task description>"
---

Plan first, execute only after explicit human approval. You are the orchestrator.

1. **Load the skill.** Apply the `prd-building` skill to the task in $ARGUMENTS.

2. **Question rounds.** Run the skill's clarifying-question rounds. Its batching and
   decide-and-record rules apply — do not restate them here.

3. **Draft the PRD.** Create `tasks/YYYY-MM-DD-<slug>/prd.md` from `templates/prd.md` and
   present the PRD inline in chat for approval.

4. **APPROVAL GATE.** Proceed only on an explicit affirmative from the human ("approved",
   "yes, go", or equivalent). A change request means: revise the PRD and re-present — as many
   rounds as it takes, no limit. Silence, ambiguity, or a partial answer is not approval. Do not
   begin implementation under any other condition.

5. **Execute.** On approval: mark prd.md `Status: Approved (<YYYY-MM-DD>)`. Then follow
   `.claude/commands/coordinate.md` from step 1 onward — do not duplicate the pipeline here —
   with two adjustments: the task folder already exists (step 1's archive-and-clean and
   `status.md` init still apply), and when step 2 writes ORCHESTRATOR_CONSTRAINTS.md, set
   `Mode: PRD_APPROVED` and fold the approved acceptance criteria into the scope and quality
   bar.

Exit criteria: same as `/coordinate`, plus an approved `prd.md` in the task folder.
