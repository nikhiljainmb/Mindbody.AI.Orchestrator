---
name: prd-building
description: >
  Build a PRD through structured clarifying questions and get explicit human approval before
  any implementation. Use for /ac, or whenever requirements are ambiguous enough that
  implementation could plausibly go two different directions.
---

# PRD building

Goal: turn a vague request into an approved `tasks/YYYY-MM-DD-<slug>/prd.md` that the pipeline
can execute without guessing. The human approval gate is the point of /ac — nothing downstream
starts until it is passed.

## Step 1 — Find the real questions

Walk the taxonomy; only ask where the answer changes what gets built. Two example questions
per category:

- **Scope** — "Does this apply to all payment types, or only autopay?" / "Is the CLI part of
  this task or a follow-up?"
- **Users/consumers** — "Who calls this endpoint — the mobile app, internal services, or
  both?" / "Do existing integrations depend on the current response shape?"
- **Constraints** — "Must this stay backward compatible with the v1 API?" / "Is there a
  performance budget, e.g. a p99 latency ceiling?"
- **Non-goals** — "Is refactoring the existing retry logic explicitly out of scope?" /
  "Should the admin UI stay untouched even where it displays this data?"
- **Success metrics** — "What number proves this worked — error rate, conversion, support
  tickets?" / "Is 'all existing tests pass' sufficient, or is there a business metric?"
- **Edge cases** — "What happens when the gateway times out mid-charge?" / "How do we handle
  a user with zero saved payment methods?"

## Step 2 — Ask in batches

3–5 questions per round, max 2 rounds. Rank by impact and ask the highest-impact questions
first; whatever does not make the cut becomes a recorded assumption (Step 3). Never drip
questions one at a time, and never open a third round.

## Step 3 — Decide and record

When the human answers "you decide", or does not answer a question within the two rounds:
make the call yourself and record it in the PRD's Assumptions section — the decision plus a
one-line rationale. Do not re-ask. An assumption on the record is revisable at the approval
gate; a silent guess is not.

## Step 4 — Draft the PRD

Instantiate `templates/prd.md` into `tasks/YYYY-MM-DD-<slug>/prd.md`. PRD sections map 1:1 to
that template — do not invent, drop, or reorder sections; if a section has nothing to say,
write "None" rather than deleting it. Answers from Step 2 fill the sections; Step 3 calls go
in Assumptions.

## Step 5 — Make acceptance criteria testable

Every acceptance criterion is either given/when/then over observable behavior, or a checkable
command with its expected result. If no agent could later prove it true or false, it is not a
criterion.

Good:
> Given a saved card that is expired, when autopay runs, then the charge is skipped and an
> AutopayFailed event is emitted. Verify: `dotnet test --filter AutopayExpiredCard` → PASS.

Bad:
> Autopay should handle expired cards gracefully.

"Gracefully" names no observable behavior and no command — a reviewer cannot fail it, so it
verifies nothing.

## Step 6 — The approval gate

Present the PRD and ask for approval. Proceed ONLY on an explicit affirmative: "approved",
"yes", "go". Silence, "looks reasonable", or a yes with attached changes is not approval.

- Change request → revise, re-present, ask again. Unlimited loops — the gate is the point
  of /ac, and there is no cycle cap here.
- On approval → mark the PRD `Status: Approved (<YYYY-MM-DD>)`, then start the pipeline with
  ORCHESTRATOR_CONSTRAINTS.md Mode: PRD_APPROVED.

## Gotchas

- The 2-round cap limits questions, not approval loops. Do not confuse the two: questions are
  bounded, gate iterations are not.
- Do not start any implementation "while waiting" for approval — the pipeline begins strictly
  after the gate (AGENTS.md invariant rule 2 has nothing to delegate before an approved PRD).
- A qualified yes ("yes, but rename the endpoint") is a change request. Revise and re-present.
- Approval binds to the presented version. Any edit after approval — including "small" ones —
  reopens the gate.
- If both possible answers to a question lead to the same implementation, do not ask it. Ask
  decisions, not preferences.
- "You decide" is an answer, not an invitation to re-ask later with different wording. Record
  it once in Assumptions and move on.
