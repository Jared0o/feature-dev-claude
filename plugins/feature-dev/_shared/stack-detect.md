# Stack detection

Run this routine from any sub-agent that needs to know the project stack.

## Algorithm

1. **Scan root** and 2 directory levels deep for these signals:

   | Signal file (glob)              | Stack       | Default test framework                   |
   |---------------------------------|-------------|------------------------------------------|
   | `*.csproj`, `*.sln`, `*.fsproj` | dotnet      | xUnit (detect: MSTest / NUnit if present)|
   | `go.mod`                        | go          | `testing` + `testify` if imported        |
   | `package.json` containing `next`| nextjs      | Vitest/Jest (detect) + Playwright for e2e|
   | `package.json` containing `react` (no `next`) | react | Vitest/Jest (detect) + React Testing Library |
   | `angular.json`                  | angular     | Jasmine/Karma (default) or Jest (detect) |

2. **Test-framework detection** — once stack is identified, look at:
   - dotnet: `<PackageReference Include="xunit"|"MSTest.TestFramework"|"NUnit" />` in any `*.csproj`
   - go: imports of `github.com/stretchr/testify` in any `*_test.go`
   - js/ts: `devDependencies` of `package.json` for `vitest`, `jest`, `@playwright/test`
   - angular: `test.builder` field in `angular.json` or `jest` in `package.json`

3. **Multiple matches → monorepo**:
   - If 2+ different stack signals found, report the matrix (which stack lives where) and ASK the user which package to target before proceeding.
   - If multiple instances of the same stack (e.g. several `*.csproj`), list them and ask which solution/project is in scope.

4. **No match** → report explicitly: `stack: unknown — please confirm`. Do not guess.

## Output format

Return JSON-like block at the end of your reasoning so the orchestrator can parse it:

```
STACK_DETECT:
  stack: <dotnet|go|react|nextjs|angular|unknown>
  test_framework: <xunit|mstest|nunit|testing|testify|vitest|jest|jasmine|playwright|unknown>
  monorepo: <true|false>
  packages: [<path>, ...]   # only if monorepo=true
  scope: <chosen path>       # filled after user confirms in monorepo case
```

This block is also persisted into `meta.json` of the current feature artifact directory by the orchestrator.

## Adding a new stack

To extend (e.g. Rust, Python):
1. Add a row to the table above with the signal file and default test framework
2. Add a detection rule in step 2 (which manifest entry indicates which framework)
3. Update `docs/stack-detection.md` and bump plugin minor version
