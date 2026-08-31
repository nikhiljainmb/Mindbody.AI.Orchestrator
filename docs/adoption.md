# Adoption

Three ways to put this repo in front of your code. All three end the same way: the assistant reads
`AGENTS.md`/`CLAUDE.md` at the project root (Cursor: `.cursor/rules/orchestrator.mdc`) and the
pipeline runs against your source.

## A. Clone and work inside it (fastest trial)

Clone this repo, open Claude Code in it, drop or scaffold some code inside, run
`/coordinate "<task>"`. Zero setup — use it to evaluate the pipeline before committing to B or C.
Not for real projects: your code ends up living inside a config repo's history.

## B. Copy into an existing project (recommended for real repos)

Copy exactly this set into the project root:

- `.claude/` (agents, commands, skills)
- `.cursor/` (only if the team uses Cursor)
- `AGENTS.md`
- `CLAUDE.md` — if the project already has one, merge: keep the `@AGENTS.md` import and the
  Claude-specific rules alongside your existing content
- `CLAUDE.local.md.example`
- `templates/`
- `patterns/` (optionally trim to the languages your project uses)
- `shared-workspace/` (README.md + .gitkeep — the contract, committed)
- `tasks/` (README.md + .gitkeep)

Then append this repo's `.gitignore` entries to yours: the `CLAUDE.local.md` /
`.claude/settings.local.json` lines and the `shared-workspace/*` and `tasks/*` blocks with their
`!` exceptions. Skipping this commits runtime coordination state to your history.

## C. Git submodule plus copied .claude/ (single upstream, many repos)

```
git submodule add <this-repo-url> orchestrator
cp -r orchestrator/.claude .claude
mkdir -p shared-workspace tasks
```

Claude Code reads `.claude/` only at the project root — a submodule copy alone is invisible, hence
the copy step. Create a root `CLAUDE.md` that imports `@orchestrator/AGENTS.md` and mirrors the
lazy-load table with `orchestrator/`-prefixed paths, and add the same `.gitignore` entries as
option B. Everything except `.claude/` is then referenced from the submodule, so `git submodule
update --remote` pulls upstream changes in one step.

## Trade-offs

| | A: clone | B: copy set | C: submodule + copy |
|---|---|---|---|
| Setup cost | git clone | copy 8 items + .gitignore | submodule + copy .claude/ |
| Best for | trying the pipeline | one production repo | many repos, one upstream |
| Upstream updates | git pull | re-copy on release | pull submodule, re-copy .claude/ |
| Local customization | edit in place | diverge freely | keep edits outside the submodule |
| Main drawback | code inside a config repo | manual upgrades | two-step update; .claude/ duplicated |

## Upgrading (B and C)

On a new release: fetch the new version, then re-copy the option-B set (B) or run
`git submodule update --remote` and re-copy `.claude/` (C). Before overwriting, diff your local
edits against the incoming version — `git diff --no-index .claude/ <new>/.claude/` — and re-apply
deliberately. Keep machine- and person-specific changes in `CLAUDE.local.md` (gitignored, never
shipped by a release), not in the copied files, and upgrades stay a clean overwrite.
