<!-- TEMPLATE — instantiate by copying to tasks/<YYYY-MM-DD-slug>/prd.md. Never edit this
     template in place. Written by the orchestrator during /ac (skill: prd-building). -->

# PRD — <task-id>: <title>

Status: Draft | Approved (<YYYY-MM-DD>)

## Problem
<What is broken or missing today, and for whom. Facts, not solutions.>

## Goals
- <Outcome 1 — measurable where possible>

## Non-goals
- <Explicitly excluded work — anything a reasonable coder might otherwise assume is in scope>

## Users / consumers
- <Who or what consumes the result: end users, services, downstream teams>

## Functional requirements
- FR1: <behavior the system must exhibit>

## Non-functional requirements
- NFR1: <performance, security, compatibility, or operability constraint>

## Acceptance criteria
Every criterion is testable: given/when/then, or a command whose output can be checked.
- AC1: Given <precondition>, when <action>, then <observable result>
- AC2: `<command>` → <expected output>

## Assumptions
Decide-and-record: when the human answers "you decide", record the decision here with a
one-line rationale. Recorded decisions bind the pipeline.
- <assumption or recorded decision — rationale>

## Open questions
Must be empty before Status moves to Approved.
- <question blocking approval>

## Approval
Approval requires an explicit affirmative from the human ("yes", "approved") — silence,
or absence of objection, is not approval.
Approved by: <name> | Date: <YYYY-MM-DD>
Verbatim reply: "<the human's approval message>"
