# patterns/ — proven reference implementations

Minimal, language-specific reference patterns that agents check code against. Nothing here
loads automatically — read a pattern file only when the current step requires it, and only
for the target project's language (AGENTS.md, context discipline).

Who uses these:

- **coder** consults the matching language folder before implementing a subtask and follows
  its patterns unless ORCHESTRATOR_CONSTRAINTS.md scopes otherwise.
- **reviewer-architecture** and **reviewer-testing** check conformance against these files;
  an unexplained deviation is a finding, recorded with the usual ID prefix and severity.

Patterns are generic per language — no project names, internal URLs, or team-specific
conventions belong here. When the target repo's own conventions (its AGENTS.md/CLAUDE.md)
conflict with a pattern, the target repo wins; the reviewer notes the divergence instead of
flagging it.

## Index

| File | Covers |
|---|---|
| `dotnet/xunit-test-patterns.md` | AAA layout, naming, theories, fixtures, mocking, negative/edge cases, async tests |
| `dotnet/service-patterns.md` | Constructor DI, options pattern, result types, layering, thin controllers, cancellation |

## Adding a language

1. Create `patterns/<lang>/` with one file per topic (test patterns, service patterns, ...).
2. Keep every file under 150 lines. These load into agent context — brevity is the feature.
3. One pattern per `##` heading: state the rule in one or two lines, then show a minimal
   fenced good example. Add a bad example only when the mistake is common enough to name.
4. Update the index above.
