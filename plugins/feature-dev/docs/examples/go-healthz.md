# Example: Go — `/feature-dev:run quick "add /healthz endpoint"`

A quick run in a Go HTTP service.

## Setup assumed

```
my-go-service/
├── go.mod
├── cmd/server/main.go
├── internal/http/router.go
└── CLAUDE.md
```

## Invocation

```
/feature-dev:run quick "add /healthz endpoint that returns 200 OK with build version"
```

## What happens

1. **Setup** — orchestrator creates `.agents/feature-dev/add-healthz-endpoint-that-returns-20260502/`, writes `meta.json` with `variant: quick`.
2. **Discovery** (`fd-discovery`) — reads `CLAUDE.md`, runs stack-detect → `stack: go`, `test_framework: testing`. Maps `internal/http/router.go` as the place where routes live.
3. **Clarify** — skipped (quick).
4. **Solutions** — skipped, default option used.
5. **Architecture** — runs but no gate. Decides: handler in `internal/http/healthz.go`, registered in `router.go`. Build version read from existing `internal/buildinfo` package found by discovery.
6. **Implementation** — writes `internal/http/healthz.go`, edits `internal/http/router.go`, adds `internal/http/healthz_test.go` with `httptest`. Runs `go test ./internal/http/...` — green.
7. **Integration tests** — skipped (quick).
8. **Review** — `fd-review-code` only. Returns APPROVE WITH NITS (suggests using `httptest.NewRequest` directly instead of building manually). Auto-approved (no REQUEST CHANGES).
9. **Docs** — inline comment only (variant=quick).
10. **CLAUDE.md update** — skipped.

## Final state

```
.agents/feature-dev/add-healthz-endpoint-that-returns-20260502/
├── meta.json
├── 01-discovery.md
├── 03-solutions.md       # stub with CHOSEN: #1
├── 04-architecture.md
├── decisions.md          # 1 ADR entry
├── 05-implementation.md
└── 07a-review-code.md
```

Total time: ~3-5 minutes. One gate (final summary).
