# shared-workspace/ — the coordination contract

Live coordination files for the **current run**. Contents are gitignored runtime state;
this README (the contract) is committed. Durable records are archived to `tasks/` — at the
start of every new run the orchestrator moves the previous run's files into that run's
`tasks/YYYY-MM-DD-<slug>/` folder, then starts clean. Nothing is ever destructively wiped —
with one exception: a `REVIEW_ONLY` run (`/review`) creates no task folder, so its files are
ephemeral and may be discarded when the next run starts.

**The one invariant that makes parallel agents safe: every file has exactly one writer.**

| File | Writer (only) | Readers | Lifecycle |
|---|---|---|---|
| `ORCHESTRATOR_CONSTRAINTS.md` | orchestrator | all agents except review-synthesizer (its prompt carries run facts) | created before any agent is spawned; **immutable** during the run — scope changes mean a new run |
| `CODING_PROGRESS.md` | coder (**append-only**; in a parallel batch, coders return entries and the orchestrator appends them in subtask order) | orchestrator; coder (last entry) | grows during implementation; never edit prior entries |
| `TEST_RESULTS.md` | test-runner | orchestrator, reviewer-testing, synthesizer | overwritten each run — latest results are the truth |
| `REVIEW_ARCHITECTURE.md` | reviewer-architecture | synthesizer | overwritten when this reviewer re-runs; an approving reviewer's report carries forward across fix cycles |
| `REVIEW_TESTING.md` | reviewer-testing | synthesizer | same as above |
| `REVIEW_QUALITY.md` | reviewer-quality | synthesizer | same as above |
| `REVIEW_SECURITY.md` | reviewer-security | synthesizer | same as above |
| `REVIEW_CHECKLIST.md` | review-synthesizer | orchestrator | rebuilt each review cycle |
| `FINAL_STATUS.md` | orchestrator | human; doc-generator (standalone doc runs only) | written exactly once, last; archived to tasks/ |

Every writer may additionally load the schema section below for the file it writes — that read
is always in-contract.

## Schemas

### ORCHESTRATOR_CONSTRAINTS.md — the run's law

```markdown
# Run Constraints — <task-id>: <title>
Created: <YYYY-MM-DD HH:mm> | Mode: AUTONOMOUS | PRD_APPROVED | REVIEW_ONLY
Immutable after creation.

## Scope
In scope: <dirs/files agents may modify>
Out of scope (do not touch): <migrations, public API contracts, ...>
Allowed paths: <globs>

## Quality bar
Coverage target: <n>% of new/changed lines   (defaults and floor: verification-protocol skill;
  a CLAUDE.local.md coverage_mandate, when set, is transcribed here at creation)
Coverage command: <e.g. dotnet test --collect:"XPlat Code Coverage">
Diff baseline: <commit/branch the "new/changed" measurement is taken against>
Test command: <exact command; a CLAUDE.local.md test_command, when set, is transcribed here>
Style: <linters/analyzers that must stay green>

## Run rules
Max review-fix cycles: 3
Reviewers for this run: architecture, testing, quality, security
Plan: tasks/<id>/plan.md
Special instructions from human: <verbatim, or "none">
```

REVIEW_ONLY variant (`/review`): header uses a review-id (`YYYY-MM-DD-review-<slug>`), Scope
holds the resolved target spec, Run rules keeps only the reviewers list — the Quality bar and
Plan fields are omitted (no tests run, no task folder exists).

### CODING_PROGRESS.md — append-only journal

```markdown
# Coding Progress — <task-id>
(Coder: append entries only. Never edit prior entries.)

---
## [<YYYY-MM-DD HH:mm>] Subtask <NN> — <title>
Status: DONE | BLOCKED
Changed: <files, with (new) markers and test counts>
Decisions: <1-3 bullets the next agent must not contradict>
Verification: <command> → <result: build PASS, tests 14/14 PASS, coverage 96.8% new-code>
Blocked on: <only if BLOCKED — the specific missing input, per your context contract>
---
```

Fix-cycle and test-fix entries use the same fields with the header
`## [<timestamp>] Fix cycle <N> — findings <IDs>` or `## [<timestamp>] Test fix <N> —
<failing tests>`.

### TEST_RESULTS.md — facts from the test-runner

```markdown
# Test Results — <task-id>
Overall: PASS | FAIL | NO_TESTS_FOUND
Build: PASS | FAIL
Tests: <total> total, <passed> passed, <failed> failed, <skipped> skipped
Coverage (new/changed code): <%> vs baseline <ref from constraints> | TOOLING_ABSENT
| File | New-line coverage |
|---|---|
Failing tests:
- <Fully.Qualified.TestName>: <first assertion message line>
Commands run:
- <exact command 1>
```

### REVIEW_<ROLE>.md — one per specialist reviewer

```markdown
# Review (<role>) — <task-id>, cycle <N>
Reviewed: <commit/diff range>

## Verdict: APPROVE | APPROVE_WITH_NITS | REQUEST_CHANGES

## Findings
| ID | Severity | File:Line | Issue | Suggested fix |
|----|----------|-----------|-------|---------------|
| A1 | Critical/Major/Minor | src/x.cs:88 | ... | ... |

## Coverage check (reviewer-testing only)
Reported: <%> vs target <%> → PASS | FAIL | N/A (REVIEW_ONLY)
(If TOOLING_ABSENT or REVIEW_ONLY: Tier-2 branch list per the verification-protocol skill.)

## Summary
<max 3 lines>
```

Finding ID prefixes: A=architecture, T=testing, Q=quality, S=security.
Per-reviewer verdict rule: any Critical or Major finding → REQUEST_CHANGES; only Minors →
APPROVE_WITH_NITS; no findings → APPROVE.

### REVIEW_CHECKLIST.md — the synthesizer's merged verdict

```markdown
# Review Checklist — <task-id>, cycle <N> of <max>
| Reviewer | Verdict | Blocking IDs | Report |
|----------|---------|--------------|--------|

## Overall verdict: APPROVE | APPROVE_WITH_NITS | REQUEST_CHANGES
(the worst individual verdict; any Critical finding forces REQUEST_CHANGES)

## Must fix before done
- [ ] <ID> (<severity>, <reviewer>) <file:line> — <issue> → <fix>

## Deferred (Minor/nits — never trigger a fix cycle)
- <ID> ...
```

### FINAL_STATUS.md — the orchestrator's final report

```markdown
# Final Status — <task-id>: <title>
Status: DONE | ESCALATED | BLOCKED
Completed: <timestamp> | Review cycles used: <n> of <max>

## Shipped
- Subtask <NN>: <one-liner — what changed, where>

## Quality
Tests: <passed>/<total> PASS | Coverage (new code): <%> vs target <%> (Tier 1|2|3)
Waivers: <none, or the waiver + reason — transcribed from constraints, or recorded verbatim
  here when granted mid-run (verification-protocol Tier 3)>

## Documentation (only when doc-generator ran; from its final response)
- <doc file touched> — <what changed>

## Open items (non-blocking)
- <deferred Minor findings, with file refs>

## Escalation (only if ESCALATED or BLOCKED)
Unresolved: <finding IDs + summary, or the blocking conflict>
Attempts: <what each fix cycle or unblock attempt tried>
Recommendation: <the orchestrator's honest opinion for the human>
```
