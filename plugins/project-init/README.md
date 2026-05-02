# project-init

`.NET 10` modular monolith scaffolder for Claude Code. Initializes a brand-new solution (or adds new modules to an existing one) following the architecture of [Jared0o/Ecommerce](https://github.com/Jared0o/Ecommerce):

- **.NET 10**, central package management (`Directory.Packages.props`)
- **Modular monolith** with reflection-based auto-discovery (`IModule` + `ModuleLoader`)
- **Mediator** (`Mediator.SourceGenerator` + `Mediator.Abstractions`) — not MediatR
- `Result<T>` pattern, no exceptions for business errors
- **EF Core + PostgreSQL**, per-module `DbContext` + schema, migrations applied on startup
- **xunit** + **NSubstitute** (unit) + **Testcontainers.PostgreSQL** (integration)
- `Guid.CreateVersion7()`, `ValueTask<T>` handlers, FluentValidation

## Commands

### `/project-init:init "<description>"`

Greenfield. Run in an empty directory. Walks gated phases:

1. **Discovery** (opus) — proposes solution name, extracts domain language.
2. **Modules** (opus) — proposes bounded contexts. **GATE**.
3. **Architecture** (opus) — entities, commands, queries, events per module. **GATE**.
4. **Scaffold solution** (sonnet) — `.slnx`, `Directory.*.props`, `global.json`, Api host, `Shared.Abstraction`, `Shared.Infrastructure`.
5. **Scaffold modules** (sonnet) — Core/Infrastructure/Api + Tests for every module.
6. **Wiring** (sonnet) — `.slnx` entries, `ProjectReference`s in Api, `appsettings.json` connection strings.
7. **Verify** (sonnet) — `dotnet restore`/`build`/`test`.
8. **CLAUDE.md** (sonnet) — root `CLAUDE.md` + `.claude/{architecture,conventions,modules,testing,decisions}.md` + `specs/`.

### `/project-init:add-module "<name and short description>"`

Adds a new module to an existing solution. Detects solution name from `*.slnx`, scaffolds Core/Infrastructure/Api + Tests, wires `.slnx` and `appsettings.json`, runs `dotnet build` + `dotnet test`, updates `.claude/modules.md` + `specs/`.

## Artifact directory

Per-run state lives in `./.agents/project-init/<slug>/`:
- `meta.json` — variant, solution_name, modules, status per phase
- `01-discovery.md` … `08-claudemd.md` — phase outputs

Re-running a phase (after a GATE) is idempotent — agent overwrites its phase file.

## Templates

Plain-text templates with `{{Placeholder}}` substitution live under `templates/`. See `docs/templates.md` for the placeholder list and how to extend them.

## License

MIT
