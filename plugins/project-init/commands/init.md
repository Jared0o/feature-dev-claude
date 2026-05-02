---
description: Initialize a new .NET 10 modular monolith solution from a description, following the Jared0o/Ecommerce architecture.
argument-hint: "<project description>"
---

You are the orchestrator of `/project-init:init`. The user invoked you with `$ARGUMENTS` — a single quoted description of the project they want to bootstrap.

## 0. Setup and pre-checks

1. Resolve `description` from `$ARGUMENTS` (strip surrounding quotes; reject if empty — tell the user the command needs a description).
2. **Empty-directory check:** list the current working directory. Reject and stop if it contains anything other than:
   - `.git/`, `.gitignore`, `.claude/`, `.idea/`, `.vscode/`, `.agents/`, `README.md` (≤ 200 bytes)

   If non-empty, tell the user: "this command must be run in an empty directory; switch to `/project-init:add-module` if you want to add a module to an existing project."
3. Compute `slug = kebab-case(first 40 chars of description) + "-" + YYYYMMDD`.
4. `art_dir = ./.agents/project-init/<slug>/`. Create it. Write `meta.json`:
   ```json
   {
     "kind": "init",
     "description": "<description>",
     "slug": "<slug>",
     "created_at": "<ISO timestamp>",
     "solution_name": null,
     "modules": [],
     "status": "discovery"
   }
   ```
5. Tell the user briefly: slug, artifact dir, the phases that will run, and that gates will pause for confirmation.

## Phases

Use the Task tool to invoke each agent. After each agent finishes, READ its output file (`NN-*.md`) and extract the `## SUMMARY` block.

For every gated phase:
1. Print `[GATE: <phase>]` followed by the SUMMARY block.
2. Ask `OK to continue, or what to change?`.
3. If user says yes/ok/continue → advance.
4. If user requests changes → re-invoke the SAME agent with the user's feedback appended to its prompt. Loop until approved.

### 1. Discovery (`pi-discovery`, opus) — GATE on solution name

Pass: `art_dir`, `description`. Agent writes `01-discovery.md` and proposes `solution_name` (PascalCase) plus initial domain language. Update `meta.json.solution_name` after the user confirms.

### 2. Modules (`pi-modules`, opus) — GATE

Pass: `art_dir`, `01-discovery.md`. Agent writes `02-modules.md` with proposed bounded contexts. After GATE, persist accepted module list into `meta.json.modules` as `[{"name": "Catalog", "description": "...", "entities": [...]}]`.

### 3. Architecture (`pi-architecture`, opus) — GATE

Pass: `art_dir`, prior outputs. Agent writes `03-architecture.md` — per module: entities (with properties), commands, queries, events published, events subscribed, errors. Conform to Ecommerce conventions (`Result<T>`, `ValueTask`, `Guid.CreateVersion7()`, `ICommand`/`IQuery` from `Mediator.Abstractions`).

### 4. Scaffold solution (`pi-scaffold-solution`, sonnet)

Pass: `art_dir`, `solution_name`. Agent renders templates from `${CLAUDE_PLUGIN_ROOT}/templates/solution/` and `${CLAUDE_PLUGIN_ROOT}/templates/shared/`. Writes:
- `<SolutionName>.slnx`, `Directory.Build.props`, `Directory.Packages.props`, `global.json`, `.gitignore`
- `src/Api/<SolutionName>.Api/{Program.cs, ModuleLoader/ModuleLoader.cs, *.csproj, appsettings.json, appsettings.Development.json}`
- `src/Shared/<SolutionName>.Shared.Abstraction/{Results, Errors, Module}/*`
- `src/Shared/<SolutionName>.Shared.Infrastructure/*`
Output: `04-scaffold-solution.md` listing every created path.

### 5. Scaffold modules (`pi-scaffold-module`, sonnet) — loop

For each module in `meta.json.modules`, invoke `pi-scaffold-module` with `module_name`, primary `entity_name(s)`, and the architecture entry from `03-architecture.md`. Agent appends a section to `05-scaffold-modules.md` per module.

### 6. Wiring (`pi-wiring`, sonnet)

Pass: `art_dir`, `solution_name`, full module list. Agent:
- Adds `<Project Path=...>` entries to `<SolutionName>.slnx` for every Core/Infrastructure/Api/Tests project
- Adds `<ProjectReference Include=...>` for every `*.Api` module to `<SolutionName>.Api.csproj`
- Adds `"<Module>Context"` connection strings to `appsettings.json` and `appsettings.Development.json`
- Verifies `Directory.Packages.props` contains every required `PackageVersion`
Output: `06-wiring.md`.

### 7. Verify (`pi-verify`, sonnet, has Bash)

Agent runs `dotnet restore`, `dotnet build <SolutionName>.slnx`, `dotnet test --no-build` (test failures are non-fatal at this stage — placeholder tests may need user fixtures; build errors ARE fatal). Output: `07-verify.md`. If build fails — print errors and **GATE**: ask user whether to re-run `pi-scaffold-module`/`pi-wiring` with feedback or stop.

### 8. CLAUDE.md (`pi-claudemd`, sonnet)

Pass: `art_dir`, `solution_name`, module list, architecture. Agent writes:
- `CLAUDE.md` (root)
- `.claude/architecture.md`, `.claude/conventions.md`, `.claude/modules.md`, `.claude/testing.md`, `.claude/decisions.md`
- `.claude/specs/<Module>/<Module>Spec_1.md` per module

Output: `08-claudemd.md`.

## Closing

- Set `meta.json.status = "complete"`.
- Print final summary: solution name, modules created, file counts, test/build verdict, follow-up commands:
  ```
  dotnet run --project src/Api/<SolutionName>.Api
  dotnet test
  git init && git add -A && git commit -m "feat: initial scaffold"
  ```
- Point the user at `<art_dir>` for full artifacts.

## Notes

- Never proceed past a gate without explicit user confirmation.
- Never modify code outside what `pi-scaffold-*`/`pi-wiring`/`pi-claudemd` produce.
- If any sub-agent fails, surface the error and stop — do not silently retry.
- Templates use `${CLAUDE_PLUGIN_ROOT}` to locate themselves; pass that path through to scaffold agents.
