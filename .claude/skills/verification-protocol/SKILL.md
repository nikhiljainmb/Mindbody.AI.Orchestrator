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
   its narrow-run Verification line) to CODING_PROGRESS.md — necessary, not sufficient.
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

Target: 95% line coverage of NEW/CHANGED code. Override via CLAUDE.local.md; floor 80%.

- **Tier 1 — tooling exists** (e.g. `dotnet test --collect:"XPlat Code Coverage"` +
  `reportgenerator`): test-runner records the % for NEW/CHANGED code only. Whole-repo
  measurement is instantly impossible in legacy codebases and teaches agents to ignore the
  mandate — never gate on it. reviewer-testing gates on the recorded %; below target =
  Critical finding.
- **Tier 2 — no tooling** (TEST_RESULTS.md reports TOOLING_ABSENT): reviewer-testing audits
  the diff branch-by-branch and must list untested branches by method name. "Coverage looks
  fine" is not a finding.
- **Tier 3 — human waiver**: recorded in ORCHESTRATOR_CONSTRAINTS.md (Special instructions),
  echoed in FINAL_STATUS.md under Waivers. Only a human grants Tier 3.

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

## v2 enforcement points (documented, deliberately not built)

Today this protocol is enforced by instruction. When it graduates to mechanism:

- PostToolUse hook that runs the affected tests after every source edit.
- Coverage-gate script parsing Cobertura XML, failing below the effective target.
- Stop hook refusing to end the session while plan.md has unfinished subtasks.
- CI gate re-running the final gate on push, so local discipline is not the last line.
