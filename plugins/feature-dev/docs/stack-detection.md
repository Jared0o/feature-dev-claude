# Stack detection

Implemented in [`_shared/stack-detect.md`](../_shared/stack-detect.md). Discovery (`fd-discovery`) runs it first and writes the result to `meta.json`.

## Supported stacks

| Stack    | Signal files               | Default test framework                        |
|----------|----------------------------|-----------------------------------------------|
| dotnet   | `*.csproj`, `*.sln`, `*.fsproj` | xUnit (detects MSTest / NUnit if present)|
| go       | `go.mod`                    | `testing` + testify if imported              |
| nextjs   | `package.json` containing `next` | Vitest/Jest + Playwright for e2e        |
| react    | `package.json` containing `react` (no `next`) | Vitest/Jest + React Testing Library |
| angular  | `angular.json`              | Jasmine/Karma or Jest (detects)              |

Detection scans the repo root + 2 levels deep.

## Monorepos

If multiple stack signals are found:
1. Discovery flags `monorepo: true` in the `STACK_DETECT:` block.
2. The orchestrator pauses and asks the user which package to scope to.
3. The chosen path is recorded as `meta.json.scope`. All subsequent agents only act within `scope`.

If multiple instances of the same stack are found (e.g. several `*.csproj`), the same flow applies — pick a project.

## Test framework detection

Once the stack is identified, the routine looks for explicit indicators:
- **dotnet**: `<PackageReference Include="xunit"|"MSTest.TestFramework"|"NUnit" />` in any `*.csproj`
- **go**: imports of `github.com/stretchr/testify` in any `*_test.go`
- **js/ts**: `devDependencies` of `package.json` for `vitest`, `jest`, `@playwright/test`
- **angular**: `test.builder` in `angular.json` or `jest` in `package.json`

If indicators conflict, the most recently used one wins (most files importing it). Agents prefer the existing convention over introducing a new framework.

## When detection returns "unknown"

Discovery reports `stack: unknown` rather than guessing. The orchestrator asks the user to confirm. This is intentional — wrong stack assumption corrupts every later phase.

## Adding a new stack

To extend (e.g. Rust, Python):

1. Add a row to the table in `_shared/stack-detect.md`:
   - signal file pattern
   - default test framework
2. Add a test-framework detection rule (which manifest entry → which framework).
3. Update agents that branch on stack:
   - `fd-implementation` — add the test framework's syntax / conventions
   - `fd-integration-tests` — add the integration test layer (e.g. testcontainers + framework's test runner)
   - `fd-review-security` — add stack-specific OWASP checks
   - `fd-review-perf` — add stack-specific perf concerns
4. Add an example under `docs/examples/`.
5. Update this page's table.
6. Bump the plugin minor version in `.claude-plugin/plugin.json`.
