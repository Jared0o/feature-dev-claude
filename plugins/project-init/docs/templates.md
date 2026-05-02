# Templates

Templates are plain-text files with `{{Placeholder}}` markers, located under `${CLAUDE_PLUGIN_ROOT}/templates/`. The `.tmpl` suffix is dropped when rendered.

## Placeholders

### Solution-level

| Placeholder | Substituted with | Example |
|---|---|---|
| `{{SolutionName}}` | PascalCase solution name | `MyShop` |
| `{{SolutionNameLower}}` | lowercase solution name | `myshop` |

### Module-level

| Placeholder | Substituted with | Example |
|---|---|---|
| `{{ModuleName}}` | PascalCase module name | `Catalog` |
| `{{ModuleNameLower}}` | lowercase module name (used as Postgres schema name) | `catalog` |

### Entity-level

| Placeholder | Substituted with | Example (Product entity with `string Name`, `decimal Price`) |
|---|---|---|
| `{{Entity}}` | PascalCase entity name | `Product` |
| `{{EntityLower}}` | lowercase | `product` |
| `{{EntityCamel}}` | camelCase | `product` |
| `{{EntityProperties}}` | block of `    public <Type> <Name> { get; set; }` lines, ONE PER PROPERTY (excluding `Id`, `CreatedAt`, `UpdatedAt`) | `    public string Name { get; set; }`<br>`    public decimal Price { get; set; }` |
| `{{EntityPropertiesCtor}}` | comma-separated `<Type> <Name>` (excluding `Id`, `CreatedAt`, `UpdatedAt`); empty if none | `string Name, decimal Price` |
| `{{EntityPropertiesCtorPrefixComma}}` | same as above, prefixed with `, ` when non-empty; empty otherwise | `, string Name, decimal Price` |
| `{{EntityPropertiesMapperPrefixComma}}` | comma-prefixed `e.<Name>` mapping expressions for query handlers; empty if none | `, e.Name, e.Price` |

## Path mapping

Template path → workspace path:

| Template | Workspace destination |
|---|---|
| `templates/solution/<X>` | `<X>` (repo root) |
| `templates/solution/api/<X>` | `src/Api/{{SolutionName}}.Api/<X>` |
| `templates/shared/abstraction/<X>` | `src/Shared/{{SolutionName}}.Shared.Abstraction/<X>` |
| `templates/shared/infrastructure/<X>` | `src/Shared/{{SolutionName}}.Shared.Infrastructure/<X>` |
| `templates/module/core/<X>` | `src/Modules/{{ModuleName}}/{{SolutionName}}.Modules.{{ModuleName}}.Core/<X>` |
| `templates/module/infrastructure/<X>` | `src/Modules/{{ModuleName}}/{{SolutionName}}.Modules.{{ModuleName}}.Infrastructure/<X>` |
| `templates/module/api/<X>` | `src/Modules/{{ModuleName}}/{{SolutionName}}.Modules.{{ModuleName}}.Api/<X>` |
| `templates/module/tests/unit/<X>` | `tests/{{SolutionName}}.Modules.{{ModuleName}}.Tests.Unit/<X>` |
| `templates/module/tests/integration/<X>` | `tests/{{SolutionName}}.Modules.{{ModuleName}}.Tests.Integration/<X>` |
| `templates/claude/CLAUDE.md.tmpl` | `CLAUDE.md` (repo root) |
| `templates/claude/<X>.md.tmpl` (other) | `.claude/<X>.md` |
| `templates/claude/specs/{{ModuleName}}Spec_1.md.tmpl` | `.claude/specs/{{ModuleName}}/{{ModuleName}}Spec_1.md` |

The agent substitutes placeholders in BOTH the destination path AND the file body. Placeholders inside file paths exist literally inside the plugin (e.g. the file `templates/module/core/Domain/{{Entity}}.cs.tmpl`) and are resolved at scaffold time.

## Extending

To add a new template:
1. Drop the `.tmpl` file under `templates/` at the right subpath.
2. Use existing placeholders, or add a new one and document it both here and in the relevant agent's frontmatter (`pi-scaffold-solution.md` / `pi-scaffold-module.md` / `pi-claudemd.md`).
3. Bump `version` in `.claude-plugin/plugin.json`.
