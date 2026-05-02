# Extending feature-dev

The plugin is intentionally small — 1 command + 11 sub-agents + 1 shared snippet. Forking and modifying is encouraged.

## Common extensions

### Add a new sub-agent (e.g. `fd-review-accessibility`)

1. Create `agents/fd-review-accessibility.md` with frontmatter:
   ```yaml
   ---
   name: fd-review-accessibility
   description: Phase 7d of /feature-dev:run. Reviews UI changes for WCAG 2.2 AA compliance.
   tools: Read, Grep, Bash
   model: sonnet
   ---
   ```
2. Body: same shape as other reviewers — verdict, findings (severity-tagged), output to `07d-review-accessibility.md`.
3. Wire it into the orchestrator: edit `commands/run.md` "Review" section to dispatch the agent in parallel with the others (only when stack is `react` / `nextjs` / `angular`).
4. Update `docs/agents.md` and `docs/architecture.md`.

### Add a new variant (e.g. `prototype` — even lighter than `quick`)

1. Add a column to the variant matrix in `commands/run.md` and `docs/variants.md`.
2. Adjust the orchestrator's variant-branching logic.
3. Add an example under `docs/examples/`.

### Add a new stack (e.g. Rust)

See [`docs/stack-detection.md`](stack-detection.md) → "Adding a new stack".

### Replace a reviewer's prompt

Just edit the agent's `.md` file. Frontmatter must stay valid.

### Change models

Edit the `model:` line in the agent's frontmatter. Use a full model ID like `claude-opus-4-7` (latest Opus) for analysis-heavy agents and `sonnet` (alias to latest Sonnet) for producing agents. `haiku` works if you want to cut cost on lightweight agents (clarify, claudemd). When a new Opus version ships, search-and-replace `claude-opus-4-7` across `agents/*.md` to upgrade.

## Forking and republishing

```bash
git clone https://github.com/<original>/feature-dev my-feature-dev
cd my-feature-dev
# edit .claude-plugin/plugin.json — change name + repository
# make your changes
git remote set-url origin git@github.com:<your-user>/my-feature-dev.git
git push -u origin main
```

Users install your fork with:
```
/plugin marketplace add <your-user>/my-feature-dev
/plugin install my-feature-dev
```

## Don't extend by adding upstream code in user repos

The plugin's job is to orchestrate generic agents that adapt to the project. If you find yourself adding repo-specific logic to the agents, put it in your project's `CLAUDE.md` instead — discovery will pick it up.
