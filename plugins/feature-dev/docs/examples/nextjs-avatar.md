# Example: Next.js — `/feature-dev:run standard "let users upload a profile avatar"`

A typical-feature run in a Next.js app with App Router.

## Setup assumed

```
my-app/
├── package.json   (next, react, tailwind, drizzle, vitest)
├── app/
│   ├── (dashboard)/profile/page.tsx
│   └── api/...
├── lib/
└── CLAUDE.md      (mentions Drizzle for DB, Clerk for auth, S3 for media)
```

## Invocation

```
/feature-dev:run standard "let users upload a profile avatar"
```

## What happens

1. **Discovery** — `stack: nextjs`, `test_framework: vitest`. Reads CLAUDE.md, follows `@lib/s3.ts` and `@lib/auth.ts`. Notes existing utilities: `lib/s3.ts:uploadObject()`, `lib/auth.ts:requireUser()`.
2. **Clarify** — asks 2 questions:
   - "Max file size and allowed MIME types?"
   - "Replace existing avatar or version-history?"
   You answer: 2 MB, jpg/png/webp; replace.
3. **Solutions** [GATE] — proposes:
   - **A**: Direct upload to S3 from browser via presigned URL (recommended)
   - **B**: Upload to Next.js route handler, route forwards to S3
   - **C**: Use a third-party widget (Uploadcare/Bytescale)
   You pick A.
4. **Architecture** [GATE] — proposes:
   - **Option 1**: New `/api/avatar/presign` route + client component using existing `lib/s3.ts:presignPutUrl()` helper (recommended)
   - **Option 2**: Server Action `requestAvatarUpload()`
   - **Option 3**: Edge route for lower latency
   You pick 1. ADR appended.
5. **Implementation** — writes route, client `<AvatarUploader />` component, updates profile page, adds Vitest tests for the route + component (with msw mocking S3). Runs `vitest run` — green.
6. **Integration tests** — auto-detected: implementation crosses HTTP boundary (new API route). Adds Playwright test that mocks S3 and exercises the upload UI end-to-end.
7. **Review** [GATE] — runs in parallel:
   - `fd-review-code`: APPROVE WITH NITS
   - `fd-review-security`: APPROVE WITH MITIGATIONS — flags missing MIME validation server-side (browser checks aren't enough), recommends content-type sniffing
   You ask "address the security finding" → orchestrator re-runs `fd-implementation` with the feedback. Re-runs reviewers. Both APPROVE.
8. **Docs** — writes `docs/features/avatar-upload.md` (dev), updates README's "Features" list, adds an entry to existing `docs/api/v1/README.md` for the presign endpoint (handoff).
9. **CLAUDE.md update** — adds `lib/s3.ts:presignPutUrl()` to the "Common helpers" section so future agents reuse it.

## Final state

```
.agents/feature-dev/let-users-upload-a-profile-avatar-20260502/
├── meta.json
├── 01-discovery.md
├── 02-clarify.md
├── 03-solutions.md
├── 04-architecture.md
├── decisions.md
├── 05-implementation.md
├── 06-integration.md
├── 07a-review-code.md
├── 07b-review-security.md
├── 08-docs.md
└── 09-claudemd.md
```

Three gates (solutions, architecture, review). Total time depends on you — agents run in 10-20 minutes total.
