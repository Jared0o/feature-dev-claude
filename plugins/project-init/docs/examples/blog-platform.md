# Example: blog platform

Walkthrough of `/project-init:init "Platforma blogowa z postami, komentarzami i moderacją"` in an empty directory.

## Phase 1 — Discovery (opus)

`pi-discovery` proposes:
- **SolutionName:** `BlogPlatform`
- **Domain language:** post, comment, author, moderator, draft, published, flagged
- **Actors:** anonymous reader, registered author, moderator
- **Open questions:** are comments tied to user accounts or guest-allowed? auto-publish or moderation queue?

[GATE] User confirms `BlogPlatform`.

## Phase 2 — Modules (opus)

`pi-modules` proposes:
1. **Users** — auth, profile (entities: `User`)
2. **Posts** — authored content (entities: `Post`); publishes `PostPublishedEvent`
3. **Comments** — comments under posts (entities: `Comment`); subscribes `PostPublishedEvent` (auto-create comment thread)
4. **Moderation** — flag/review posts and comments (entities: `Flag`)

[GATE] User accepts.

## Phase 3 — Architecture (opus)

`pi-architecture` writes per-module designs. Example for `Posts`:
- Entity `Post`: `Title`, `Slug`, `BodyMarkdown`, `Status` (Draft / Published / Archived), `AuthorId`, `PublishedAt?`
- Commands: `CreatePostCommand`, `UpdatePostCommand`, `PublishPostCommand`, `ArchivePostCommand`
- Queries: `GetPostByIdQuery`, `GetPostsQuery`
- Events: publishes `PostPublishedEvent(Guid PostId, Guid AuthorId, string Title)`
- Errors: `PostNotFoundError`, `PostAlreadyPublishedError`

[GATE] User accepts (or asks to drop `ArchivePostCommand`, etc.).

## Phase 4 — Scaffold solution (sonnet)

Writes `BlogPlatform.slnx`, `Directory.Build.props`, `Directory.Packages.props`, `global.json`, `.gitignore`, `src/Api/BlogPlatform.Api/*`, `src/Shared/BlogPlatform.Shared.Abstraction/*`, `src/Shared/BlogPlatform.Shared.Infrastructure/*`.

## Phase 5 — Scaffold modules (sonnet, looped)

`pi-scaffold-module` runs once per module. For `Posts`, primary entity is `Post`, so it creates:
- `src/Modules/Posts/BlogPlatform.Modules.Posts.Core/{Domain/Post.cs, Commands/{Create,Update,Delete}Post.cs, Queries/{GetPostById,GetPosts}.cs, Dtos/{PostDto, PagedResult}.cs, Errors/PostNotFoundError.cs, Interfaces/IPostRepository.cs, Extensions.cs, *.csproj}`
- `src/Modules/Posts/BlogPlatform.Modules.Posts.Infrastructure/{Persistence/{PostsDbContext, Configurations/PostConfiguration}.cs, Migrations/ApplyPostsMigrations.cs, Repositories/PostRepository.cs, Extensions.cs, *.csproj}`
- `src/Modules/Posts/BlogPlatform.Modules.Posts.Api/{PostsModule.cs, Endpoints/PostEndpoints.cs, *.csproj}`
- `tests/BlogPlatform.Modules.Posts.Tests.Unit/{Commands/CreatePostCommandHandlerTests.cs, GlobalUsings.cs, *.csproj}`
- `tests/BlogPlatform.Modules.Posts.Tests.Integration/{Fixtures/PostsDbFixture.cs, Repositories/PostRepositoryTests.cs, GlobalUsings.cs, *.csproj}`

For `Publish`/`Archive` and any extra entities/commands beyond the templated CRUD, the agent duplicates-and-renames the matching command file.

## Phase 6 — Wiring (sonnet)

Adds module entries to `BlogPlatform.slnx`, `<ProjectReference>` in `BlogPlatform.Api.csproj`, connection strings in `appsettings.json` (`UsersContext`, `PostsContext`, `CommentsContext`, `ModerationContext`).

## Phase 7 — Verify (sonnet)

Runs `dotnet restore && dotnet build BlogPlatform.slnx`. Build should be green. `dotnet test` may have placeholder failures (unit tests use stubs); integration tests need Docker.

## Phase 8 — CLAUDE.md (sonnet)

Writes root `CLAUDE.md` plus `.claude/{architecture,conventions,modules,testing,decisions}.md` and `.claude/specs/<Module>/<Module>Spec_1.md` per module.

## Output

Final layout:

```
BlogPlatform.slnx
Directory.Build.props
Directory.Packages.props
global.json
CLAUDE.md
.claude/...
.gitignore
src/Api/BlogPlatform.Api/...
src/Shared/...
src/Modules/{Users,Posts,Comments,Moderation}/...
tests/...
```

Suggested next steps:
```bash
git init && git add -A && git commit -m "feat: initial scaffold"
docker compose up -d postgres   # or run a local Postgres on :5432
dotnet run --project src/Api/BlogPlatform.Api
```
