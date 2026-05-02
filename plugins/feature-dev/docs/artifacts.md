# Artifacts

Every run of `/feature-dev:run` creates a directory in your project:

```
<your-repo>/.agents/feature-dev/<slug>/
```

`<slug>` is `kebab-case(first 40 chars of description)-YYYYMMDD`. Example: `add-user-avatar-upload-20260502/`.

## Contents

```
meta.json                   # variant, description, slug, stack, test_framework, scope, status, timestamps
01-discovery.md             # stack, code map, reusable utilities, blockers
02-clarify.md               # numbered questions + user's answers (optional, skipped in `quick`)
03-solutions.md             # 2-3 solution proposals + CHOSEN: #N (skipped in `quick`)
04-architecture.md          # 2-3 architecture options + refactor list + CHOSEN: #N
decisions.md                # running ADR log (created/appended by fd-architecture)
05-implementation.md        # files changed, deps added, test summary, test run output
06-integration.md           # boundaries covered, tests added, fixtures (optional)
07a-review-code.md          # code review verdict + findings
07b-review-security.md      # security review verdict + findings (skipped in `quick`)
07c-review-perf.md          # perf review verdict + findings (only `full` or hot path)
08-docs.md                  # what docs were written / updated and where
09-claudemd.md              # what was added to CLAUDE.md (skipped in `quick`)
```

Every `NN-*.md` file ends with a `## SUMMARY` block — that's what the orchestrator extracts to show you at gates and in the final summary.

## meta.json schema

```json
{
  "variant": "quick | standard | full",
  "description": "free-text feature description from user",
  "slug": "kebab-case-slug-YYYYMMDD",
  "created_at": "2026-05-02T15:07:00Z",
  "stack": "dotnet | go | react | nextjs | angular | unknown",
  "test_framework": "xunit | mstest | nunit | testing | testify | vitest | jest | jasmine | playwright | unknown",
  "scope": "<path>",          // only set if monorepo
  "status": "discovery | clarify | solutions | architecture | implementation | integration | review | docs | claudemd | complete",
  "completed_at": "2026-05-02T16:32:00Z"   // when status = complete
}
```

## Should I commit these?

Two valid choices:

**Yes, commit them.** Treats artifacts as a record of how the feature was built. `decisions.md` becomes a living ADR log. Reviewers can see why a particular architecture was chosen. Future Claude Code sessions can reference past discoveries to avoid re-researching the same areas.

**No, gitignore them.** Treats them as scratch. Add to `.gitignore`:
```
.agents/
```

Recommended: commit at least `decisions.md` and the artifacts of any `full` run. Scratch the rest if they're noisy.

## Resuming an interrupted run

Today: manual. Open `meta.json`, find `status`, re-invoke `/feature-dev:run` and tell the orchestrator which phase to resume from in the description.

Roadmap: `/feature-dev:resume <slug>` to auto-continue.
