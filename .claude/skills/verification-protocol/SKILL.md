---
name: verification-protocol
description: >-
  The definition of done: the coverage mandate, evidence rules, and the gates a subtask and a
  run must pass. Use whenever declaring a subtask or task complete, and whenever judging test
  sufficiency or a coverage report.
---

# verification-protocol

## Step 1 — Apply the evidence rule

No completion claim without the exact command that was run AND its output: test names,
pass/fail counts, coverage %. "Tests pass" is a sentence, not evidence. This is how every
agent satisfies AGENTS.md invariant 6 — it applies to every status flip, by anyone.

## Step 2 — Run the per-subtask gate

1. coder writes tests WITH the code, in the same invocation, and appends its entry (including
   its narrow-run Verification line) to CODING_PROGRESS.md — in a parallel batch, returned for
   the orchestrator to append. Necessary, not sufficient.
2. test-runner independently runs the build and the FULL suite (its contract forbids subsets)
   at the implement-phase checkpoint and after every fix cycle, writing TEST_RESULTS.md. A
   coder claiming green is not evidence; only test-runner output counts at the gate.
3. Orchestrator checks each acceptance criterion against TEST_RESULTS.md and the diff, then
   flips the subtask's status in the plan. Any criterion unmet → the subtask stays open.

## Step 3 — Run the final gate

1. test-runner: full suite + coverage across all new/changed code → TEST_RESULTS.md.
2. Parallel review — four reviewers, then review-synthesizer → REVIEW_CHECKLIST.md.
3. The merged verdict must be APPROVE (or APPROVE_WITH_NITS) before FINAL_STATUS.md may say
   DONE. DONE with an open Critical finding is a protocol violation (AGENTS.md invariant 6).

## Step 4 — Enforce the coverage mandate (three tiers)

Target: the % of NEW/CHANGED line coverage recorded in ORCHESTRATOR_CONSTRAINTS.md at run
start — default 95%, floor 80%; a CLAUDE.local.md `coverage_mandate` changes the default and
the orchestrator transcribes the effective value into the constraints. Measurement is against
the constraints' Diff baseline.

- **Tier 1 — tooling exists** (e.g. `dotnet test --collect:"XPlat Code Coverage"` +
  `reportgenerator`): test-runner records the % for NEW/CHANGED code only. Whole-repo
  measurement is instantly impossible in legacy codebases and teaches agents to ignore the
  mandate — never gate on it. reviewer-testing gates on the recorded %; below target =
  Critical finding.
- **Tier 2 — no tooling** (TEST_RESULTS.md reports TOOLING_ABSENT): reviewer-testing audits
  the diff branch-by-branch and must list untested branches by method name. "Coverage looks
  fine" is not a finding.
- **Tier 3 — human waiver**: only a human grants it. Granted before the run → recorded in
  ORCHESTRATOR_CONSTRAINTS.md (Special instructions) at creation. Granted mid-run → recorded
  verbatim in FINAL_STATUS.md under Waivers (the constraints file is immutable). Either way it
  is echoed under Waivers in FINAL_STATUS.md — a waiver is never silent.

## Step 5 — Instantiate the checklist

At the final gate, instantiate `templates/verification-checklist.md`. Tick an item only with
its evidence attached. An unticked item blocks DONE; a ticked item without evidence is worse.

## Anti-patterns

- Trusting the coder's "all green" and skipping test-runner because "it's a small change".
- Evidence-free ticks: a status flipped with no command output recorded anywhere.
- Gating on whole-repo coverage, or reporting it as if it were the new-code number.
- Coverage theater: assertion-free tests, happy-path-only tests, or tests skipped or deleted
  to make the number.
- Downgrading a below-target coverage finding from Critical to dodge a fix cycle.

## Gotchas

- TOOLING_ABSENT is a fact, not a waiver. It moves the run to Tier 2 — never past the
  mandate.
- A new-code % is meaningless without its baseline: the exact command recorded in
  TEST_RESULTS.md must name the commit/diff range it measured against.
- NO_TESTS_FOUND on a code-changing subtask is a gate FAIL, whatever the build says.
- A flaky rerun that flips FAIL→PASS is a finding, not a pass: record both outcomes and the
  failing test names.

## v2 enforcement points

Today this protocol is enforced by instruction. The mechanized gates that graduate it — hooks,
coverage-gate scripts, CI — are catalogued in `docs/extending.md` (v2 roadmap and the
add-a-hook recipe).
