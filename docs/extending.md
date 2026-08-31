# Extending

Every extension follows the same discipline as the core: add detail where it belongs, register it
where it is discovered, and never restate a rule that already lives elsewhere (AGENTS.md
precedence rule).

## Add an agent

1. Copy the closest existing definition in `.claude/agents/` — a new reviewer starts from an
   existing `reviewer-*.md`, a new worker from `coder.md`.
2. Keep the body structure: Persona, Inputs, Outputs, Protocol, Context contract. The persona
   block *is* the persona mechanism (see `docs/architecture.md`).
3. Give it exactly one writable coordination file and add that file's row and schema to
   `shared-workspace/README.md`. One writer per file is the invariant — no exceptions for new
   agents.
4. Register it in the AGENTS.md roles table and in the loading matrix of
   `.claude/skills/context-management/`.
5. Wire it into whichever command launches it, as a delegation step with a context packet.

## Add a skill

Create `.claude/skills/<name>/SKILL.md`. Write the description trigger-oriented — "when you need
to X" — because the description is all the orchestrator sees when deciding whether to load it.
Then add a row to the CLAUDE.md lazy-load table; a skill missing from that table is never loaded.

## Add a language to patterns/

One folder per language, structured per `patterns/README.md`. Pattern folders are loaded only when
the target project's language matches, so adding one costs nothing for other projects.

## Add a command

Create `.claude/commands/<name>.md`: frontmatter (description, argument hint) plus a numbered
step-by-step protocol. Commands are call graphs, skills are the knowledge — reference the skills
and the shared-workspace contract at each step, never restate their contents.

## Add a hook

v1 deliberately ships no hooks: the protocol is enforced by instruction — agents obey the files
because the files say so. When your team wants a rule enforced by mechanism instead, register a
hook in `.claude/settings.json`. This recipe is how any item in the v2 roadmap below graduates
from instruction to mechanism. Example — run affected tests after every source edit (the first
roadmap item):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "pwsh -File scripts/run-affected-tests.ps1" }
        ]
      }
    ]
  }
}
```

`PostToolUse` fires after the matched tool succeeds; the matcher is a regex over tool names, so
`Edit|Write` covers both write paths. The script lives in your project and exits non-zero to
surface a failure to the agent. The same shape covers the other roadmap items — a coverage gate
as a `PostToolUse` hook on the test-runner's commands, or a `Stop` hook that refuses to end the
session while plan.md has unfinished subtasks. Team-wide hooks go in `.claude/settings.json`;
personal ones in `.claude/settings.local.json` (gitignored), the same split as CLAUDE.local.md.

## v2 roadmap (deliberate omissions)

v1 enforces the protocol by instruction: agents obey the files because the files say so.
Mechanical enforcement is the next layer, omitted deliberately to keep v1 markdown-only:

- **Hooks.** A PostToolUse hook that runs affected tests after every source edit; a coverage-gate
  script that parses Cobertura XML and fails the run when new-code coverage is below the mandate,
  instead of trusting reviewer arithmetic; a Stop hook that refuses to end the session while
  plan.md still has unfinished subtasks.
- **CI coverage gates.** The same new/changed-line coverage bar enforced in CI, so the number in
  FINAL_STATUS.md is reproduced by a machine on every PR.
- **More language pattern folders.** Grow `patterns/` per its README as teams adopt the repo for
  new stacks.
- **True multi-instance via git worktrees.** One orchestrator terminal plus N coder terminals,
  each in its own worktree and branch. The single-writer file protocol already tolerates this
  (see `docs/architecture.md`); what is missing is a branch-naming convention and depends-on
  edges in plan.md so parallel coders never touch the same files.
- **Plugin packaging.** Ship `.claude/` as a Claude Code plugin, collapsing adoption options B
  and C into a single install command.
