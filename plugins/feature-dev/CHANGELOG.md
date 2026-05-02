# Changelog

## [0.1.0] - 2026-05-02

Initial release.

- Orchestrator command `/feature-dev:run <quick|standard|full> "<description>"`
- 11 sub-agents (`fd-discovery`, `fd-clarify`, `fd-solutions`, `fd-architecture`, `fd-implementation`, `fd-integration-tests`, `fd-review-code`, `fd-review-security`, `fd-review-perf`, `fd-docs`, `fd-claudemd`)
- Stack auto-detection for .NET, Go, React, Next.js, Angular
- Phase artifacts persisted in `<project>/.agents/feature-dev/<slug>/`
- Three execution variants with gated phases
- Documentation in `docs/`
