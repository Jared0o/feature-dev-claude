---
description: Add a new module to an existing project-init / Ecommerce-style modular monolith (Core + Infrastructure + Api + Tests, wired into .slnx and appsettings).
argument-hint: "<module name and short description>"
---

You are the orchestrator of `/project-init:add-module`. The user invoked you with `$ARGUMENTS` — a quoted module name plus short description.

## 0. Setup and pre-checks

1. Resolve `arg = $ARGUMENTS` (strip quotes; reject if empty).
2. **Layout check:** verify the current directory looks like a project-init / Ecommerce solution:
   - exactly one `*.slnx` at root → infer `solution_name` from its filename
   - `src/Api/<SolutionName>.Api/<SolutionName>.Api.csproj` exists
   - `src/Modules/` directory exists (create if missing — fine)
   - `src/Shared/<SolutionName>.Shared.Abstraction/` exists

   If any check fails, stop and tell the user: "this directory does not look like a project-init solution; run `/project-init:init` in an empty directory first."
3. Compute `slug = kebab-case(arg first 40 chars) + "-" + YYYYMMDD`. `art_dir = ./.agents/project-init/add-module-<slug>/`. Create it. Write `meta.json`:
   ```json
   {
     "kind": "add-module",
     "arg": "<arg>",
     "slug": "<slug>",
     "solution_name": "<inferred>",
     "module_name": null,
     "entities": [],
     "status": "design"
   }
   ```

## Phases

### 1. Design (`pi-add-module`, opus) — GATE

Pass: `art_dir`, `arg`, `solution_name`. Agent writes `01-design.md`:
- Proposed `module_name` (PascalCase)
- Entities (with properties)
- Commands, queries, events published, events subscribed
- Errors
- Connection-string key (`<ModuleName>Context`)
- Schema name (lowercase `<modulename>`)

GATE on the design block. After approval, persist `module_name` and `entities` into `meta.json`.

### 2. Scaffold (`pi-scaffold-module`, sonnet)

Pass: `art_dir`, `solution_name`, `module_name`, entity list, architecture from `01-design.md`. Agent renders module templates from `${CLAUDE_PLUGIN_ROOT}/templates/module/` into:
- `src/Modules/<ModuleName>/<SolutionName>.Modules.<ModuleName>.Core/`
- `src/Modules/<ModuleName>/<SolutionName>.Modules.<ModuleName>.Infrastructure/`
- `src/Modules/<ModuleName>/<SolutionName>.Modules.<ModuleName>.Api/`
- `tests/<SolutionName>.Modules.<ModuleName>.Tests.Unit/`
- `tests/<SolutionName>.Modules.<ModuleName>.Tests.Integration/`

Output: `02-scaffold.md`.

### 3. Wiring (`pi-wiring`, sonnet)

Pass: `art_dir`, `solution_name`, single-module list (just the new one). Agent:
- Adds the five new project paths to `<SolutionName>.slnx`
- Adds `<ProjectReference Include="..\..\Modules\<ModuleName>\<SolutionName>.Modules.<ModuleName>.Api\...csproj" />` to `<SolutionName>.Api.csproj`
- Adds `"<ModuleName>Context"` to `appsettings.json` + `appsettings.Development.json`
- Adds any new `PackageVersion` entries to `Directory.Packages.props` if missing

Output: `03-wiring.md`.

### 4. Verify (`pi-verify`, sonnet, has Bash)

`dotnet restore`, `dotnet build <SolutionName>.slnx`. If green, `dotnet test --no-build --filter "FullyQualifiedName~<ModuleName>"`. Output: `04-verify.md`. If build fails → GATE: re-run scaffold/wiring or stop.

### 5. CLAUDE.md update (`pi-claudemd`, sonnet)

Mode: `update`. Agent appends the new module to `.claude/modules.md` and creates `.claude/specs/<ModuleName>/<ModuleName>Spec_1.md`. Does NOT rewrite root `CLAUDE.md` — only adds a `@.claude/specs/<ModuleName>/<ModuleName>Spec_1.md` reference at the end of the existing module section if appropriate.

Output: `05-claudemd.md`.

## Closing

- Set `meta.json.status = "complete"`.
- Print final summary: module name, files created, build/test verdict, suggested follow-ups:
  - Add a real EF migration: `dotnet ef migrations add Initial<ModuleName> --project src/Modules/<ModuleName>/<SolutionName>.Modules.<ModuleName>.Infrastructure --startup-project src/Api/<SolutionName>.Api --context <ModuleName>DbContext`
  - Replace placeholder entities/commands with real domain logic.

## Notes

- Never proceed past a gate without confirmation.
- If scaffolding into a path that already exists and is non-empty → stop and ask user (do not overwrite an existing module silently).
