---
description: Run the gated feature-dev workflow (discovery → solutions → architecture → implementation → tests → review → docs → CLAUDE.md update).
argument-hint: <quick|standard|full> "<feature description>"
---

You are the orchestrator of the `feature-dev` workflow. The user invoked you with `$ARGUMENTS`. The first token is the variant (`quick` | `standard` | `full`); the remainder (quoted) is the feature description.

## 0. Setup

1. Parse `$1` → `variant` (validate; if missing or invalid, default to `standard` and tell the user).
2. Parse `$2` → `description` (everything after the variant; strip surrounding quotes).
3. Compute `slug = kebab-case(first 40 chars of description) + "-" + YYYYMMDD`.
4. Compute `art_dir = ./.agents/feature-dev/<slug>/` (relative to current repo root).
5. Create `art_dir`. Write `meta.json`:
   ```json
   {
     "variant": "<variant>",
     "description": "<description>",
     "slug": "<slug>",
     "created_at": "<ISO timestamp>",
     "stack": null,
     "test_framework": null,
     "scope": null,
     "status": "discovery"
   }
   ```
6. Tell the user briefly: variant chosen, slug, artifact dir, what phases will run.

## Variant matrix

| Phase                 | quick | standard | full |
|-----------------------|-------|----------|------|
| 1 Discovery           | ✓     | ✓        | ✓    |
| 2 Clarify             | ✗     | only if ambiguous | ✓ |
| 3 Solutions [GATE]    | ✗ (default option 1) | ✓ | ✓ |
| 4 Architecture [GATE] | inline, no gate | ✓ | ✓ |
| 5 Implementation      | ✓     | ✓        | ✓    |
| 6 Integration tests   | ✗     | auto-detect | ✓ |
| 7 Review [GATE]       | code only, auto-pass | code+security | code+security+perf |
| 8 Docs                | inline | dev | dev+handoff+non-tech |
| 9 CLAUDE.md update    | ✗     | ✓        | ✓    |

## How to invoke a sub-agent

Use the Task tool with the agent's name (e.g. `fd-discovery`). Pass:
- `art_dir` absolute path
- The variant
- Any prior-phase summaries it needs

After each sub-agent completes, READ its output file (`NN-*.md`) and extract the `## SUMMARY` block.

## Gates

For every gated phase:
1. Print to user: `[GATE: <phase name>]` followed by the SUMMARY block from the artifact.
2. Ask: `OK to continue, or what to change?`
3. If user says yes/ok/continue → update `meta.json.status` and proceed.
4. If user requests changes → re-invoke the SAME sub-agent with the user's feedback appended to the prompt; loop until user approves.
5. For `solutions` and `architecture` gates, also confirm the `CHOSEN: #N` line — if user picks a different option, edit the file before continuing.

## Phase sequence

1. Run `fd-discovery`. If it reports a monorepo, ask user which package to scope to before continuing; update `meta.json.scope`.
2. **Clarify decision:**
   - quick → skip
   - standard → run `fd-clarify`; if it produces questions, ask user, append answers to `02-clarify.md`, then continue
   - full → always run `fd-clarify`
3. **Solutions:**
   - quick → write a stub `03-solutions.md` with `CHOSEN: #1` containing the obvious approach, no gate
   - standard/full → run `fd-solutions`, GATE
4. **Architecture:**
   - quick → run `fd-architecture` but skip the gate (proceed with its recommendation)
   - standard/full → run `fd-architecture`, GATE
5. Run `fd-implementation`.
6. **Integration tests decision:**
   - quick → skip
   - standard → parse `05-implementation.md` "Files changed" — if changes span 2+ top-level dirs OR cross an API boundary (HTTP route added, new gRPC method, new queue handler), run `fd-integration-tests`
   - full → always run
7. **Review:** invoke reviewers IN PARALLEL via a single message with multiple Task tool calls:
   - quick → only `fd-review-code`, auto-approve unless it returns REQUEST CHANGES
   - standard → `fd-review-code` + `fd-review-security`
   - full → `fd-review-code` + `fd-review-security` + `fd-review-perf`

   Aggregate verdicts. Print combined SUMMARY. GATE (skip gate in `quick` if no REQUEST CHANGES).

   If any reviewer returns REQUEST CHANGES — ask user whether to re-run `fd-implementation` with feedback or proceed.
8. Run `fd-docs`.
9. **CLAUDE.md update:**
   - quick → skip
   - standard/full → run `fd-claudemd`

## Closing

After all phases:
- Update `meta.json.status = "complete"`
- Print final summary: phases run, files changed, tests added, review verdicts, docs written
- Point user at the artifact directory

## Notes

- Never proceed past a gate without explicit user confirmation
- Never modify code outside what `fd-implementation` / `fd-integration-tests` / `fd-docs` / `fd-claudemd` produce
- If any sub-agent fails or returns an error, surface it to the user and stop — do not silently retry
