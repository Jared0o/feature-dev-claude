# Example: .NET — `/feature-dev:run full "add audit logging across all services"`

A `full` run in a multi-project .NET solution.

## Setup assumed

```
MyCompany.sln
├── src/
│   ├── MyCompany.Orders/        (ASP.NET Core)
│   ├── MyCompany.Billing/       (ASP.NET Core)
│   ├── MyCompany.Shared/        (class lib)
│   └── MyCompany.Workers/       (worker service)
├── tests/...
└── CLAUDE.md
```

## Invocation

```
/feature-dev:run full "add audit logging across all services — must capture user, action, target entity, before/after for mutations; queryable by SOC team"
```

## What happens

1. **Discovery** — `stack: dotnet`, `test_framework: xunit`. Detects monorepo (multiple `*.csproj`); orchestrator asks which project to scope to. You answer "all of them — this is solution-wide". `meta.json.scope` set to solution root. Discovery maps existing logging infrastructure (Serilog), existing `MyCompany.Shared/Logging` namespace, existing middleware pipeline.
2. **Clarify** (always in `full`) — asks 4 questions: storage backend, retention policy, PII redaction rules, async vs sync emission. Answered.
3. **Solutions** [GATE] — proposes:
   - **A**: Cross-cutting middleware + ActionFilter that emits audit events to a Kafka topic
   - **B**: Source generator that decorates marked methods with audit emission
   - **C**: EF Core SaveChanges interceptor + manual emission for non-EF actions
   You pick A.
4. **Architecture** [GATE] — proposes 3 placement options. You pick: new `MyCompany.Shared.Auditing` library + middleware in each ASP.NET host, IAuditEmitter abstraction, Kafka producer in DI. Refactor list: 4 existing controllers need `[Audit]` attribute, `MyCompany.Shared/Logging/LogContext.cs` needs to expose UserId. ADR appended.
5. **Implementation** — adds the library, wires middleware in all 3 hosts, decorates the 4 controllers, writes xUnit tests for the library + each host's middleware integration. `dotnet test` — green.
6. **Integration tests** (always in `full`) — uses `WebApplicationFactory<T>` for each host, testcontainers for Kafka. Verifies audit events appear in Kafka end-to-end with correct payload, correct user identity, redaction applied.
7. **Review** [GATE] — runs in parallel:
   - `fd-review-code`: APPROVE
   - `fd-review-security`: APPROVE WITH MITIGATIONS — recommends Kafka topic ACLs (note for ops team), no inline payload of secrets
   - `fd-review-perf`: APPROVE WITH RECOMMENDATIONS — flags sync emission on hot order path, recommends batching producer
   You ask "address the perf finding". Re-runs implementation + reviewers. All APPROVE.
8. **Docs** — writes `docs/architecture/audit-logging.md` (dev), `docs/handoff/audit-events-v1.md` (for SOC team — schema, topic name, ACL setup), `docs/release-notes/2026-05-02-audit-logging.md` (non-tech, plain-language summary for stakeholders).
9. **CLAUDE.md update** — adds the `MyCompany.Shared.Auditing` module to the project structure section, adds "All new state-mutating endpoints must be decorated with `[Audit]`" to the Conventions section.

## Final state

```
.agents/feature-dev/add-audit-logging-across-all-services-20260502/
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
├── 07c-review-perf.md
├── 08-docs.md
└── 09-claudemd.md
```

Three gates plus monorepo-scope question. This is the kind of feature where the gated, documented workflow pays for itself — you have an audit trail of decisions, a security review on record, and three forms of documentation for three audiences.
