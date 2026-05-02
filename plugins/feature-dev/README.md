# feature-dev

A Claude Code plugin that turns "I want to add a feature" into a structured, gated workflow: discover → propose → architect → implement → test → review → document. Supports **.NET, Go, React, Next.js, and Angular** projects with auto stack detection.

## Why

Vibe-coding a feature works for 30 minutes. For anything bigger, you want:
- the agent to **read your CLAUDE.md and existing code** before proposing anything,
- a chance to **pick between alternatives** instead of accepting the first thing it produces,
- **gates** where you can course-correct before more code gets written,
- **parallel review** (code + security + perf) at the end,
- **docs and a CLAUDE.md update** so the next session has the context.

`feature-dev` does all of that as a single slash command with three "size" variants.

## Install

### Local (development / try-before-publish)

```bash
claude --plugin-dir ~/projekty/feature-dev/plugins/feature-dev
```

### From GitHub

```
/plugin marketplace add Jared0o/feature-dev-claude
/plugin install feature-dev@feature-dev-claude
```

The first command registers this repo as a Claude Code plugin marketplace (it ships with `.claude-plugin/marketplace.json`). The second installs the `feature-dev` plugin from that marketplace.

To update later:

```
/plugin marketplace update feature-dev-claude
/plugin install feature-dev@feature-dev-claude
```

After install, restart Claude Code. Verify with `/plugin` (should list `feature-dev`) and `/agents` (should list 11 `feature-dev:fd-*` agents).

## Quick start

```
/feature-dev:run quick "add /healthz endpoint"
/feature-dev:run standard "let users upload an avatar"
/feature-dev:run full "add audit logging across all services"
```

Each run creates a directory `<your-repo>/.agents/feature-dev/<slug>/` containing one Markdown artifact per phase plus a running `decisions.md` ADR log. Commit it (or add it to `.gitignore` if you don't want the artifacts in your repo).

## Variants — when to use which

| Variant   | Use it for                              | Phases skipped vs full         |
|-----------|-----------------------------------------|--------------------------------|
| `quick`   | <1h tweaks, small endpoints, bug fixes  | clarify, solutions gate, integration tests, security/perf review, CLAUDE.md update |
| `standard`| Typical features (default choice)       | perf review (unless hot path), some non-tech docs |
| `full`    | Cross-cutting / risky / multi-service   | nothing — runs every phase     |

Full breakdown in [docs/variants.md](docs/variants.md).

## What runs

```
discovery → clarify → solutions [GATE] → architecture [GATE]
  → implementation → integration tests → review (code+security+perf) [GATE]
  → docs → CLAUDE.md update
```

Each phase is a separate sub-agent. See [docs/agents.md](docs/agents.md) for what each one does, which model it uses, and what tools it has.

## Documentation

- [Architecture](docs/architecture.md) — how the orchestrator sequences sub-agents and enforces gates
- [Agents](docs/agents.md) — one section per sub-agent
- [Variants](docs/variants.md) — `quick` vs `standard` vs `full`
- [Stack detection](docs/stack-detection.md) — how the plugin detects your stack and how to add a new one
- [Artifacts](docs/artifacts.md) — what gets written to `.agents/feature-dev/<slug>/`
- [Extending](docs/extending.md) — fork it, add your own phase
- Examples: [Go quick](docs/examples/go-healthz.md) · [Next.js standard](docs/examples/nextjs-avatar.md) · [.NET full](docs/examples/dotnet-audit.md)

## Requirements

- Claude Code with plugin support
- Project should ideally have a `CLAUDE.md` at the root — the discovery phase reads it and follows transitively-linked files

## License

MIT — see [LICENSE](LICENSE).
