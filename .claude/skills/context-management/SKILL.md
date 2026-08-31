---
name: context-management
description: >
  Rules for what context each role loads and what it must never load. Use when delegating to
  any subagent, when the current context feels bloated, or when deciding whether to read
  patterns/, templates/, or docs/. Defines the context packet every delegation must be built from.
---

# Context management

Context is a budget. Every file loaded that the current decision does not depend on makes the
agent slower and dumber. The matrix below is the orchestrator's summary for composing packets;
each agent's own context contract (in its `.claude/agents/` file) is authoritative for that
agent — on any difference, the agent file wins and this matrix gets fixed.

## The loading matrix

| Role | LOADS | NEVER LOADS |
|---|---|---|
| orchestrator | AGENTS.md + CLAUDE.md (auto); skills per the CLAUDE.md lazy-load table; shared-workspace/README.md when schemas are needed; all coordination files (read); tasks/<id>/; templates/ at instantiation only; source via Grep/Glob to scope the plan | source files to edit them; patterns/; docs/; other tasks' directories; agent definition files |
| coder | its context packet; ORCHESTRATOR_CONSTRAINTS.md; FILES IN SCOPE; patterns/ for the target language; last CODING_PROGRESS.md entry | REVIEW_*.md and REVIEW_CHECKLIST.md (fix-cycle findings arrive inline in the packet); TEST_RESULTS.md (the orchestrator summarizes failures into the packet); templates/; docs/; source outside FILES IN SCOPE |
| test-runner | its context packet; ORCHESTRATOR_CONSTRAINTS.md (test + coverage commands and target) | source file contents beyond command output; every REVIEW_*.md; CODING_PROGRESS.md; patterns/; docs/ |
| reviewer-architecture | its context packet; ORCHESTRATOR_CONSTRAINTS.md; the diff and files in scope; patterns/ for the target language | other reviewers' reports; REVIEW_CHECKLIST.md; CODING_PROGRESS.md; TEST_RESULTS.md; docs/ |
| reviewer-testing | its context packet; ORCHESTRATOR_CONSTRAINTS.md; the diff, test files, and TEST_RESULTS.md; patterns/dotnet/xunit-test-patterns.md (only for .NET diffs) | other reviewers' reports; REVIEW_CHECKLIST.md; CODING_PROGRESS.md; patterns/ beyond the xunit test reference; docs/ |
| reviewer-quality | its context packet; ORCHESTRATOR_CONSTRAINTS.md; the diff and files in scope | other reviewers' reports; REVIEW_CHECKLIST.md; CODING_PROGRESS.md; TEST_RESULTS.md; patterns/; docs/ |
| reviewer-security | its context packet; ORCHESTRATOR_CONSTRAINTS.md; the diff and files in scope | other reviewers' reports; REVIEW_CHECKLIST.md; CODING_PROGRESS.md; TEST_RESULTS.md; patterns/; docs/ |
| review-synthesizer | the four REVIEW_*.md; TEST_RESULTS.md (cycle facts arrive via its prompt) | ORCHESTRATOR_CONSTRAINTS.md; source code and diffs (it merges verdicts, never re-reviews); CODING_PROGRESS.md; patterns/; docs/ |
| doc-generator | the changed-file list in its prompt (or FINAL_STATUS.md when the prompt points there); the shipped source files; the target project's existing docs | shared-workspace REVIEW_*.md and TEST_RESULTS.md; CODING_PROGRESS.md; patterns/; templates/ |

Reviewer independence is deliberate: four reviewers who cannot see each other's reports produce
four uncorrelated opinions. Do not "help" a reviewer by pasting another's findings.

## The delegation rule

Subagents inherit nothing — not this conversation, not files you have open, not decisions made
three messages ago. A subagent's world is exactly its prompt plus the files that prompt names.
Therefore: pass the MINIMUM prompt that lets the agent succeed, built from the packet below.
Anything the packet omits does not exist for that agent.

## The context packet

Every delegation prompt uses these fields, in this order:

```
ROLE: coder
RUN DIR: shared-workspace/
YOUR SUBTASK: Subtask 03 — add retry policy to AutopayService.Charge
CONSTRAINTS: read ORCHESTRATOR_CONSTRAINTS.md first; it is binding.
FILES IN SCOPE (the only source files to open unprompted):
  src/Billing/AutopayService.cs (modify)
  src/Billing/RetryPolicy.cs (read)
  tests/Billing/AutopayServiceTests.cs (extend)
PRIOR STATE (<=5 lines; durable detail is in CODING_PROGRESS.md):
  Subtasks 01-02 DONE. RetryPolicy exists; decision: exponential backoff, max 3 attempts.
DELIVERABLE: implementation + tests green locally; append one CODING_PROGRESS.md entry
  with the verification command and result.
NOT YOUR JOB: migrations, the IPaymentGateway contract, other subtasks, running reviews.
```

Markers: `(modify)` = agent changes it, `(read)` = reference only, `(extend)` = append/add to it.

## Composition rules

- Pointers over payloads. Name paths; inline only decision-critical snippets under ~20 lines
  (AGENTS.md invariant rule 3).
- Summarize history, never replay it. Five lines of PRIOR STATE, not a transcript.
- Always name the negative space. NOT YOUR JOB is mandatory — agents fill silence with scope.
- Do not restate constraints in the packet. Point at ORCHESTRATOR_CONSTRAINTS.md; a paraphrase
  drifts from the original and creates two versions of the law.

## Lazy-loading rules

- patterns/ — only when the target language matches, and only by roles the matrix allows.
- templates/ — only at the moment of instantiating a file from one.
- docs/ — never during pipeline execution. Docs explain the system to humans, not to agents
  mid-run.

## The starved-agent rule

A subagent that needs a file its packet did not name records the gap — coder: a `Blocked on:`
line in its CODING_PROGRESS.md entry with Status: BLOCKED; other roles: a BLOCKED note in
their output file — and stops. Speculative exploration ("let me just look around the repo") is
a contract violation, not initiative. The orchestrator fixes the packet and re-delegates.

## Conflicts

Instruction conflicts resolve by the AGENTS.md precedence rule — apply it, do not restate it.
For facts about the current task (what changed, what passed, what is blocked), shared-workspace/
files beat memory files: CLAUDE.md and CLAUDE.local.md carry policy, never run state.

## Gotchas

- A packet that will not fit in ~30 lines means the subtask is too big — go back to the
  task-breakdown skill instead of writing a longer packet.
- Do not paste a file's contents when the agent can open it — the pointer is the packet.
- PRIOR STATE summarizes decisions, not diffs. The next agent needs "backoff is exponential,
  max 3", not the code that implements it.
- "Load more context to be safe" is the failure mode this skill exists to stop. If broad
  context feels necessary, the task is under-specified — record the gap and stop (AGENTS.md).
- The matrix binds the orchestrator too: needing docs/ mid-pipeline means the plan is wrong,
  not that the rule has an exception.
