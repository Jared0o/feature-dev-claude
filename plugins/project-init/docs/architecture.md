# project-init — internal architecture

`/project-init:init` and `/project-init:add-module` are orchestrator commands. The orchestrator is the slash-command body itself; it sequences sub-agents through the Task tool, reads each agent's output file from the artifact directory, and gates on user feedback at fixed checkpoints.

## Phase graph

```
init:
  pi-discovery (opus)         01-discovery.md           [GATE: solution name]
    │
    ▼
  pi-modules (opus)           02-modules.md             [GATE: module list]
    │
    ▼
  pi-architecture (opus)      03-architecture.md        [GATE: per-module design]
    │
    ▼
  pi-scaffold-solution (sonnet) 04-scaffold-solution.md
    │
    ▼
  pi-scaffold-module (sonnet) 05-scaffold-modules.md   (one invocation per module)
    │
    ▼
  pi-wiring (sonnet)          06-wiring.md
    │
    ▼
  pi-verify (sonnet, Bash)    07-verify.md             [GATE only on build failure]
    │
    ▼
  pi-claudemd (sonnet)        08-claudemd.md

add-module:
  pi-add-module (opus)        01-design.md              [GATE]
    │
    ▼
  pi-scaffold-module (sonnet) 02-scaffold.md
    │
    ▼
  pi-wiring (sonnet)          03-wiring.md
    │
    ▼
  pi-verify (sonnet, Bash)    04-verify.md             [GATE only on build failure]
    │
    ▼
  pi-claudemd (sonnet, mode=update) 05-claudemd.md
```

## Artifact directory

`./.agents/project-init/<slug>/` (init) or `./.agents/project-init/add-module-<slug>/` (add-module).

- `meta.json` — running state (current phase, solution_name, modules, ...).
- `NN-<phase>.md` — one file per phase, with a `## SUMMARY` block the orchestrator extracts.

Re-running a phase (after a GATE rejection) overwrites the same `NN-<phase>.md` — phases are idempotent.

## Why `claude-opus-4-7` for design phases and `claude-sonnet-4-6` for scaffolding

- Design phases (discovery, modules, architecture, add-module design) require domain reasoning and judgment about boundaries — `opus` is worth it.
- Scaffolding/wiring/verify/claudemd are mostly mechanical text emission — `sonnet` is faster and cheaper, with no quality loss.

## Templates

See `docs/templates.md` for the placeholder list and how the scaffold agents map template paths → workspace paths.

## Why a custom plugin instead of `dotnet new`

`dotnet new` is one-shot and stack-anonymous. This plugin produces a working skeleton AND co-generated `CLAUDE.md`/`.claude/*` docs that immediately make subsequent Claude Code work coherent — it knows the conventions because the same agent that wrote the code also wrote the docs.
