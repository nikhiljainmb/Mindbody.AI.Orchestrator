# tasks/ — durable per-task records

One folder per task: `tasks/YYYY-MM-DD-<slug>/` (slug = 3–5 kebab-case words from the task,
e.g. `tasks/2026-08-31-autopay-retry-policy/`).

## Contents

| File | Written by | When |
|---|---|---|
| `prd.md` | orchestrator (from `templates/prd.md`) | `/ac` only, before approval |
| `plan.md` | orchestrator (from `templates/plan.md`) | after task breakdown |
| `status.md` | orchestrator | one line, updated at each phase: `planning \| implementing \| reviewing \| done \| blocked` + timestamp |
| `final-status.md` | orchestrator | archived copy of FINAL_STATUS.md at run end |
| archived run files | orchestrator | previous run's shared-workspace/ files, moved here when a new run starts |

## Observing a run

```
ls shared-workspace/    # what's happening right now
ls tasks/               # every task ever started
cat tasks/<id>/status.md
```

## Lifecycle

Task folders are gitignored by default (local runtime records). Teams that want a durable
audit trail in git: delete the three `tasks/*` lines in `.gitignore`. The protocol already
supports it — run files are archived, never overwritten destructively.
