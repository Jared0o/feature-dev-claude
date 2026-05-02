# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **Claude Code plugin marketplace** named `jared0o-plugins` (defined in `.claude-plugin/marketplace.json`). The repo ships **two plugins** under `plugins/`:

- `feature-dev` — gated multi-agent workflow for designing/implementing/reviewing features in existing projects (.NET / Go / React / Next.js / Angular).
- `project-init` — `.NET 10` modular monolith scaffolder that initializes solutions or adds modules following the architecture of [Jared0o/Ecommerce](https://github.com/Jared0o/Ecommerce).

Note: `marketplace.json.name` is `jared0o-plugins` — do **not** rename it back to anything that looks like an official Anthropic/Claude marketplace (was rejected in commit `1db5d4e` for impersonation). The GitHub repo is still `Jared0o/claude-plugins`; only the marketplace's logical name was changed. Some user-facing docs in `plugins/feature-dev/README.md` still reference `@claude-plugins` — that's stale; the marketplace name is `jared0o-plugins`.

## Architecture pattern (shared by both plugins)

Both plugins follow the same **orchestrator + sub-agents + gates** pattern. Understanding it once means you understand both:

1. A single slash command (`/feature-dev:run`, `/project-init:init`, `/project-init:add-module`) is the **orchestrator**. The command body itself sequences work — there is no separate orchestrator file.
2. The orchestrator invokes **sub-agents** through the Task tool. Each sub-agent is a single Markdown file under `plugins/<plugin>/agents/` with YAML frontmatter (`name`, `description`, `tools`, `model`).
3. Each sub-agent writes ONE numbered output file (`01-<phase>.md`, `02-<phase>.md`, …) under `./.agents/<plugin>/<slug>/`. Every output file ends with a `## SUMMARY` block — that's the only thing the orchestrator extracts.
4. After **gated** phases, the orchestrator prints `[GATE: <phase>]` plus the SUMMARY and waits for user confirmation. If the user requests changes, the orchestrator re-invokes the SAME agent with the user's feedback appended; phase outputs are **idempotent** (same filename, overwritten).
5. State across phases lives in `meta.json` inside the artifact dir.

**Model split convention:** `claude-opus-4-7` for design/reasoning agents (discovery, clarify, solutions, architecture, reviews); `claude-sonnet-4-6` (or 4.6 family) for mechanical code/file emission (implementation, scaffolding, docs). Match this when adding new agents.

**Agent prefix convention:** prefix every agent name with the plugin's short tag — `fd-*` for `feature-dev`, `pi-*` for `project-init`. New plugins should pick a 2-letter tag and stick to it.

## Plugin layout convention

Every plugin under `plugins/<name>/` has:

```
.claude-plugin/plugin.json     # name, version, description, author, repository, license
commands/<command>.md          # slash command with YAML frontmatter (description, argument-hint)
agents/<prefix>-<phase>.md     # one file per sub-agent
docs/                          # plugin's own docs (architecture, examples, ...)
README.md                      # user-facing docs
CHANGELOG.md
```

Plugin-specific extras:
- `feature-dev` has `_shared/stack-detect.md` — the stack detection routine that `fd-discovery` runs.
- `project-init` has `templates/` — `.tmpl` files with `{{Placeholder}}` substitution. The scaffold agents (`pi-scaffold-solution`, `pi-scaffold-module`, `pi-claudemd`) read these and render to a target workspace. **Filenames inside `templates/` literally contain placeholders** (e.g. `{{SolutionName}}.slnx.tmpl`); Glob-listings show them with the braces. Placeholders are resolved in BOTH the destination path and the file body. Full placeholder reference: `plugins/project-init/docs/templates.md`.

## Adding a new plugin

1. Create `plugins/<your-plugin>/` matching the layout above.
2. Append an entry to `.claude-plugin/marketplace.json`'s `plugins[]` array.
3. Add a row to the table in the root `README.md`.
4. Update the repo-layout tree in the root `README.md`.

There is no build step — the marketplace is plain files. Users pick up changes via `/plugin marketplace update jared0o-plugins`.

## Editing existing plugins

- **Commands and agents are Markdown** with YAML frontmatter. No tests to run, no build to verify — the only "verification" is loading the plugin in Claude Code and exercising the workflow.
- **When you change an agent's prompt**, also update its `description` frontmatter if the contract changed.
- **When you add a new placeholder to `project-init` templates**, document it in BOTH `plugins/project-init/docs/templates.md` AND the relevant scaffold agent's frontmatter (typically `pi-scaffold-module.md` or `pi-scaffold-solution.md`). Agents can't infer placeholders they aren't told about.
- **Bump the plugin's `version`** in `plugin.json` for any user-visible behaviour change.

## Local testing

Test a single plugin without the marketplace:

```bash
claude --plugin-dir ~/projekty/feature-dev/plugins/<plugin-name>
```

Test the marketplace itself locally before pushing:

```
/plugin marketplace add C:\Users\jaroslaw.przybyl\projekty\feature-dev
/plugin install <plugin-name>@jared0o-plugins
```

## Git workflow

`main` is the published branch — pushes to `main` are visible to every user who has installed the marketplace as soon as they run `/plugin marketplace update jared0o-plugins`. Treat `main` accordingly: don't commit broken plugin manifests, don't rename `marketplace.json.name` casually (caching + impersonation check), and prefer one focused commit per logical change so users can read the changelog from `git log`.
