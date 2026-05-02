# Architecture

`feature-dev` is one slash command (`/feature-dev:run`) that orchestrates 11 sub-agents through a fixed sequence of phases. Each phase writes a Markdown artifact and ends with a `## SUMMARY` block the orchestrator extracts to show you at gates.

## Phase flow

```
┌──────────────────────────────────────────────────────────────────┐
│ 0. Setup — parse args, create .agents/feature-dev/<slug>/        │
│    write meta.json                                               │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────┐   ┌──────────────┐   ┌──────────────────────────┐
│ 1 Discovery  │ → │ 2 Clarify    │ → │ 3 Solutions  [GATE]      │
│ fd-discovery │   │ fd-clarify   │   │ fd-solutions             │
└──────────────┘   └──────────────┘   └──────────────────────────┘
                                                 ↓
┌──────────────────────┐    ┌─────────────────┐    ┌────────────────────┐
│ 4 Architecture [GATE]│ →  │ 5 Implementation│ →  │ 6 Integration tests│
│ fd-architecture      │    │ fd-implementation│    │ fd-integration-tests│
└──────────────────────┘    └─────────────────┘    └────────────────────┘
                                                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ 7 Review [GATE] — runs in PARALLEL                              │
│   fd-review-code | fd-review-security | fd-review-perf          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────┐    ┌──────────────────┐
│ 8 Docs       │ →  │ 9 CLAUDE.md upd. │
│ fd-docs      │    │ fd-claudemd      │
└──────────────┘    └──────────────────┘
```

## Gates

Three explicit gates: **Solutions**, **Architecture**, **Review**.

At every gate the orchestrator:
1. Prints the SUMMARY of the just-finished phase.
2. Asks: `OK to continue, or what to change?`
3. On approval — proceeds.
4. On change request — re-invokes the same agent with your feedback appended; loops until you approve.

For solutions and architecture, the chosen option is recorded as `CHOSEN: #N` at the bottom of the artifact. The orchestrator updates this if you pick a different option than the agent recommended.

## Why sub-agents

Each phase has a different shape (read-only vs writes code, broad survey vs tight diff review, etc.) and benefits from different prompts, tools, and models. Sub-agents also isolate context — the security reviewer doesn't need to remember the brainstorming from phase 3, just look at the diff.

`fd-architecture` and `fd-review-security` use the Opus model because they reward depth; the rest use Sonnet to keep cost reasonable.

## State

All state lives in `<your-repo>/.agents/feature-dev/<slug>/`:
- `meta.json` — variant, description, stack, status (which phase is current)
- `NN-<phase>.md` — one file per phase, numbered for ordering
- `decisions.md` — running ADR-style log of architectural decisions

This means you can interrupt a run, come back later, and the orchestrator can resume by reading `meta.json.status` (manual today; auto-resume is on the roadmap).

## Stack awareness

The discovery phase calls the routine in [`_shared/stack-detect.md`](../_shared/stack-detect.md) which writes a `STACK_DETECT:` block. Implementation, integration-tests, security, and perf agents read `meta.json.stack` and `meta.json.test_framework` and adapt their behavior (e.g. xUnit vs vitest, ASP.NET-specific OWASP checks vs Next.js-specific ones).

## Parallelism

The Review phase (7) runs reviewers in parallel — the orchestrator dispatches `fd-review-code`, `fd-review-security`, and (in `full`) `fd-review-perf` in a single message with multiple Task tool calls. The orchestrator then aggregates verdicts before showing the gate.

Other phases are strictly sequential because each depends on the prior phase's artifact.
