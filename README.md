# feature-dev-claude — a Claude Code plugin marketplace

A personal marketplace of Claude Code plugins maintained by [@Jared0o](https://github.com/Jared0o).

## Install the marketplace

```
/plugin marketplace add Jared0o/feature-dev-claude
```

Then install any plugin from the table below:

```
/plugin install <plugin-name>@feature-dev-claude
```

To pull updates later:

```
/plugin marketplace update feature-dev-claude
```

## Plugins in this marketplace

| Plugin | Description | Docs |
|--------|-------------|------|
| [`feature-dev`](plugins/feature-dev/) | Orchestrator workflow with gated phases for designing, implementing, reviewing, and documenting features (.NET / Go / React / Next.js / Angular). | [README](plugins/feature-dev/README.md) · [docs/](plugins/feature-dev/docs/) |

## Repo layout

```
.
├── .claude-plugin/
│   └── marketplace.json     # marketplace manifest (lists every plugin below)
├── plugins/
│   └── feature-dev/         # one directory per plugin
│       ├── .claude-plugin/plugin.json
│       ├── commands/
│       ├── agents/
│       ├── _shared/
│       ├── docs/
│       ├── README.md
│       └── CHANGELOG.md
├── LICENSE                   # marketplace-wide license (per-plugin LICENSE allowed if different)
└── README.md
```

## Adding a new plugin

1. Create `plugins/<your-plugin>/` with the standard plugin layout (`.claude-plugin/plugin.json`, `commands/`, `agents/`, etc.).
2. Append an entry to `.claude-plugin/marketplace.json`:
   ```json
   {
     "name": "<your-plugin>",
     "source": "./plugins/<your-plugin>",
     "description": "..."
   }
   ```
3. Add a row to the table above.
4. Commit and push. Existing users pick it up via `/plugin marketplace update feature-dev-claude`.

## Local development

Test a single plugin without going through the marketplace:

```bash
claude --plugin-dir ~/projekty/feature-dev/plugins/<plugin-name>
```

Or test the marketplace itself locally:

```
/plugin marketplace add C:\Users\jaroslaw.przybyl\projekty\feature-dev
/plugin install <plugin-name>@feature-dev-claude
```

## License

MIT — see [LICENSE](LICENSE). Individual plugins may add their own LICENSE if different.
