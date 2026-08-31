<!-- TEMPLATE — instantiate by copying as one subtask section inside tasks/<id>/plan.md.
     Never edit this template in place. One subtask = one coder delegation. -->

## Subtask <NN>: <imperative title>

status: [todo|doing|verified|blocked|escalated]
Depends on: <subtask NNs, or none>
Files:
- <exact/path/one.ext> (new)
- <exact/path/two.ext> (modify)
- <exact/path/three.ext> (read)
parallel-safe: yes|no

`verified` requires evidence in CODING_PROGRESS.md (AGENTS.md invariant rule 6) — a coder
claim alone never moves status past `doing`. `parallel-safe: yes` is allowed only per the
task-breakdown skill's disjointness rule (Step 6).

### Goal
<1-2 lines: what exists once this subtask is verified.>

### Acceptance criteria
Observable behavior only — never implementation steps.
- <given/when/then, or a condition checkable by command>

### Verification
Exact commands — the coder runs them, the test-runner re-runs them.
- Build: `<command>`
- Test: `<command>`
