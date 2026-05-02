# Variants

`feature-dev` ships three variants because not every feature needs the same level of process.

```
/feature-dev:run quick    "..."
/feature-dev:run standard "..."
/feature-dev:run full     "..."
```

## Phase matrix

| Phase                     | quick                    | standard                          | full                              |
|---------------------------|--------------------------|-----------------------------------|-----------------------------------|
| 1 Discovery               | ✓                        | ✓                                 | ✓                                 |
| 2 Clarify                 | ✗                        | only if ambiguous                 | always                            |
| 3 Solutions [GATE]        | ✗ (auto option 1)        | ✓                                 | ✓                                 |
| 4 Architecture [GATE]     | inline, no gate          | ✓                                 | ✓                                 |
| 5 Implementation          | ✓                        | ✓                                 | ✓                                 |
| 6 Integration tests       | ✗                        | auto-detect (cross-boundary)      | always                            |
| 7 Review [GATE]           | code only, auto-pass     | code + security                   | code + security + perf            |
| 8 Docs                    | inline comments only     | dev docs (+handoff if API added)  | dev + handoff + non-tech          |
| 9 CLAUDE.md update        | ✗                        | ✓                                 | ✓                                 |

## When to use which

### `quick` — < 1 hour of work
Use for:
- Bug fixes with an obvious root cause
- Adding a single small endpoint (`/healthz`, `/version`)
- Renames, small refactors confined to one file
- Copy/text changes
- Config tweaks

You skip a lot of process. The agent picks the obvious approach without offering alternatives. There's only one gate (final approval). Don't use this for anything you'd want a teammate to review carefully.

### `standard` — typical features
Use for:
- A new screen, endpoint group, or worker
- Features that touch ~3-10 files in 1-2 modules
- Anything where you want to choose between approaches but don't need a perf review

This is the default. You get gates at solutions and architecture so you can steer, plus code + security review at the end.

### `full` — cross-cutting, risky, or compliance-relevant
Use for:
- Auth/authz changes
- Cross-service features (audit logging, tracing, multi-tenant isolation)
- Anything touching payments, PII, or auth tokens
- Features with strict perf requirements (hot path)
- Anything that needs a non-technical handoff (release notes, stakeholder communication)

Adds: clarify always runs, integration tests always run, perf review runs, non-tech docs are written.

## Gates in detail

A gate is the orchestrator pausing and asking `OK to continue, or what to change?`.

- **Quick** — only the final summary asks for confirmation
- **Standard** — gates at: solutions, architecture, review (3 gates)
- **Full** — same 3 gates; review gate aggregates 3 reviewers' verdicts

If you say "what to change ..." the orchestrator re-invokes the same agent with your feedback. No phases are re-run further upstream automatically — if you change architecture after seeing review findings, you do that explicitly.

## Choosing programmatically

Rules of thumb (the orchestrator does not auto-pick — you choose):
- "Just do X for me, I trust you" → `quick`
- "Help me design and build X" → `standard`
- "X needs to be solid" / "X is regulated" / "X is multi-team" → `full`
