# payments/ — payment-domain knowledge

Shared context for the Mindbody/Playlist payments domain: the repo catalog and the AutoPay
design docs. Like `patterns/`, this folder is **loaded on demand only** — an agent reads it
when the task at hand is payment-domain work, never "for context" (AGENTS.md context
discipline). Orchestrators: point a subagent at the specific file its subtask needs, not at
the folder.

| File | What it is | Read it when |
|---|---|---|
| [repos.md](repos.md) | Catalog of every payment-related repo: purpose, stack, integrations, AutoPay role, key docs | You need to know where something lives or what a service does |
| [autopay-hld.md](autopay-hld.md) | High-level design of the EXISTING AutoPay system — architecture, gates, money path, data stores, retry/idempotency semantics, grounded gaps | Any AutoPay/recurring-billing task |
| [autopay-lld-scenarios.md](autopay-lld-scenarios.md) | The LLD: code-verified end-to-end data-flow scenarios (S1–S20) with file:line anchors | Tracing a specific flow (charge night, declines, contract changes, suspension) |
| [autopay-rewrite-hld.md](autopay-rewrite-hld.md) | Rewrite target architecture — bounded contexts, canonical flows, migration plan | Designing new AutoPay work or judging it against the target state |

Canonical source of the AutoPay docs is Notion —
[AutoPay — High-Level Design (Existing System)](https://app.notion.com/p/playlist/AutoPay-High-Level-Design-Existing-System-3c2dda30e231815789b0f09e1b17ca99)
and its child pages. These copies were synced 2026-08-31; when they disagree with Notion,
Notion wins — re-sync rather than editing here.

## Adding to this folder

One file per subject, provenance header at the top (canonical link + sync date), and a row in
the table above. Domain facts belong here; coding patterns belong in `patterns/`; process
rules belong in AGENTS.md and the skills.
