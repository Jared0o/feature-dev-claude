# Sub-agents

All agents are invoked by the orchestrator (`/feature-dev:run`) — never call them directly.

Each agent reads `<art_dir>/meta.json` first, writes an artifact named `NN-<phase>.md`, and ends that artifact with a `## SUMMARY` block.

All **analysis** agents (read-only — discovery, clarify, solutions, architecture, code/security/perf review) use **`claude-opus-4-7`** (latest Opus) for deepest reasoning. **Producing** agents (implementation, integration-tests, docs, CLAUDE.md update) use **`sonnet`** because the work is more mechanical and Opus would significantly raise cost per run.

| # | Agent | Model | Tools | Writes | Skipped in |
|---|-------|-------|-------|--------|------------|
| 1 | fd-discovery       | claude-opus-4-7 | Read, Glob, Grep                | `01-discovery.md`        | never  |
| 2 | fd-clarify         | claude-opus-4-7 | Read, Grep                      | `02-clarify.md`          | quick  |
| 3 | fd-solutions       | claude-opus-4-7 | Read, Glob, Grep                | `03-solutions.md`        | quick  |
| 4 | fd-architecture    | claude-opus-4-7 | Read, Glob, Grep                | `04-architecture.md`, `decisions.md` | — |
| 5 | fd-implementation  | sonnet          | Read, Write, Edit, Bash, Glob, Grep | `05-implementation.md` + code | — |
| 6 | fd-integration-tests | sonnet        | Read, Write, Edit, Bash, Glob, Grep | `06-integration.md` | quick + when not crossing boundaries |
| 7a| fd-review-code     | claude-opus-4-7 | Read, Grep, Bash                | `07a-review-code.md`     | — |
| 7b| fd-review-security | claude-opus-4-7 | Read, Grep, Bash                | `07b-review-security.md` | quick |
| 7c| fd-review-perf     | claude-opus-4-7 | Read, Grep, Bash                | `07c-review-perf.md`     | quick + standard (unless hot path) |
| 8 | fd-docs            | sonnet          | Read, Write, Edit, Glob, Grep   | `08-docs.md` + project docs | — |
| 9 | fd-claudemd        | sonnet          | Read, Edit, Glob, Grep          | `09-claudemd.md` + CLAUDE.md | quick |

## fd-discovery

Reads `CLAUDE.md` (root + nested), follows `@file` references transitively (cap 30 files), runs stack detection, maps the code areas relevant to the feature, lists existing utilities the implementer should reuse. Updates `meta.json` with detected stack and test framework. If the project is a monorepo, surfaces the package list so the orchestrator can ask which one to scope to.

## fd-clarify

Lists numbered clarification questions when the feature description is ambiguous. Will explicitly say "no clarifications needed" rather than invent questions. The orchestrator relays the questions to the user and appends answers to the artifact.

## fd-solutions

Produces 2-3 distinct solution approaches (genuinely different in shape, not minor variations). Each with: approach paragraph, pros, cons, S/M/L effort, risks. Recommends one. Output ends with `CHOSEN: #N` which the orchestrator updates based on user input. **Gated.**

## fd-architecture

Given the chosen solution, proposes 2-3 architecture options for fitting it into the existing codebase. For each: file/module placement, public API signatures (no implementation), data flow, **explicit refactorings needed in existing files**, tradeoffs vs codebase conventions. Appends an ADR entry to `decisions.md`. **Gated.**

## fd-implementation

Implements the chosen architecture + unit tests. Reuses utilities listed in discovery. Matches existing code style. Writes tests in the framework from `meta.json.test_framework` (xUnit / `testing` / Vitest / Jest / Jasmine). Runs the test suite; iterates until green. Runs linter/formatter if configured.

## fd-integration-tests

Triggered when the implementation crosses a module/service boundary (touches >1 top-level dir, or adds an HTTP/gRPC/queue/DB boundary) or when the variant is `full`. Sets up minimal real fixtures (in-memory DB, mock external HTTP) — **does not mock the system under test**. Covers golden path end-to-end + at least one failure mode per boundary.

## fd-review-code

Diff-only review focused on correctness, reuse (did the implementer use what discovery flagged?), readability, dead/half-finished code, test quality. Does not duplicate linter checks. Verdict: APPROVE / APPROVE WITH NITS / REQUEST CHANGES.

## fd-review-security

OWASP-style review tailored to the detected stack. Always checks input validation, AuthZ (not just AuthN), secrets handling, crypto choices, dependency risk. Adds web checks (SQLi, XSS, CSRF, SSRF, CORS, rate limits) and stack-specific checks (e.g. dotnet deserialization, `dangerouslySetInnerHTML`, `NEXT_PUBLIC_*` env exposure). Verdict: APPROVE / APPROVE WITH MITIGATIONS / REQUEST CHANGES.

## fd-review-perf

Only in `full` (or when the diff touches a documented hot path). Backend: N+1 queries, missing indexes, async misuse, hot-path allocations. Frontend: re-render storms, bundle size delta, blocking work, data-fetch waterfalls. Verdict: APPROVE / APPROVE WITH RECOMMENDATIONS / REQUEST CHANGES.

## fd-docs

Discovers where docs live in the project (existing `docs/`, README sections, mkdocs/docusaurus). Writes dev-facing docs always; handoff for downstream consumers when an external API was added; non-tech summary in `full` variant. Updates index/README links. Code examples must run as-is.

## fd-claudemd

Updates the project's `CLAUDE.md` (root or closest nested) with novel modules, conventions, gotchas, commands, or dependencies introduced by the feature. Refuses to add information already derivable from code/git or one-off implementation details.
