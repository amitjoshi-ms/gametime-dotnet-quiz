<!--
SYNC IMPACT REPORT
==================
Version change: 1.0.0 (initial)
Modified principles: N/A (new constitution)
Added sections: Core Principles (5), Technology Stack, Git Workflow, Governance
Removed sections: N/A
Templates requiring updates:
  - .specify/templates/plan-template.md: ⚠ pending review
  - .specify/templates/spec-template.md: ⚠ pending review
  - .specify/templates/tasks-template.md: ⚠ pending review
Follow-up TODOs: None
-->

# GameTime .NET Quiz Constitution

## Core Principles

### I. Zero Cost Infrastructure
All infrastructure MUST use free tiers exclusively. No paid services permitted.
- Cloudflare Pages for hosting (unlimited bandwidth)
- GitHub Actions for CI/CD (2000 minutes/month)
- GitHub Models for LLM (openai/gpt-4o-mini free tier)
- **Rationale**: Learning project with no budget; validates that quality apps can be built at zero cost.

### II. Static-First Architecture
The application MUST compile to purely static assets with no server-side runtime.
- Blazor WebAssembly in standalone mode (no ASP.NET Core host)
- All data served as static JSON from Cloudflare Pages
- No databases, no server functions, no backend APIs
- **Rationale**: Simplifies deployment, eliminates cold starts, maximizes caching efficiency.

### III. Immutable Content Caching
Generated content MUST use content-addressed filenames for aggressive caching.
- Question chunks named `{category}-{seq}-{hash8}.json`
- Chunks receive `Cache-Control: immutable, max-age=31536000`
- Manifest receives `Cache-Control: no-cache, must-revalidate`
- **Rationale**: Users get instant loads after first visit; bandwidth costs stay zero.

### IV. Additive-Only Content
Questions MUST only be added, never deleted or modified in place.
- New questions go into new chunk files
- Existing chunks are immutable once deployed
- Content hash tracking detects source changes for regeneration
- **Rationale**: Preserves cache validity; simplifies state management; no data loss risk.

### V. Test-First Development
All non-trivial code MUST have tests written before implementation.
- Minimum 80% line coverage, 70% branch coverage
- Critical paths (scoring, data loading, hashing) require 100% coverage
- Use xUnit + FluentAssertions + NSubstitute
- **Rationale**: Prevents regressions; enables confident refactoring; documents intent.

## Technology Stack

| Category | Technology | Version | Locked |
|----------|------------|---------|--------|
| Runtime | .NET | 10.0 | ✅ |
| Framework | Blazor WebAssembly | Standalone | ✅ |
| UI Library | Microsoft.FluentUI.AspNetCore.Components | 4.13.2 | ✅ |
| Testing | xUnit | 2.9.0 | ✅ |
| Assertions | FluentAssertions | 7.0.0 | ✅ |
| Mocking | NSubstitute | 5.3.0 | ✅ |
| Scraping | AngleSharp | 1.2.0 | ✅ |
| LLM | GitHub Models (openai/gpt-4o-mini) | - | ✅ |
| Hosting | Cloudflare Pages | Free tier | ✅ |

Technology changes require constitution amendment with migration plan.

## Git Workflow

### Two-Repository Structure
- **gametime-dotnet-quiz** (CODE): Blazor app + QuestionGenerator; branch protection REQUIRED
- **gametime-dotnet-quiz-data** (DATA): Generated JSON; no branch protection (bot commits)

### Commit Message Format
All commits MUST follow conventional commit format:
```
<type>(<scope>): <subject>
```
Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Pull Request Requirements
- All PRs to `main` MUST pass: build, test, lint
- Self-merge permitted for solo development
- Squash merge preferred for clean history

## Governance

This constitution supersedes all other project documentation in case of conflict.

### Amendment Process
1. Open PR with proposed changes to this file
2. Document rationale and impact in PR description
3. Update version number following semver:
   - MAJOR: Principle removal or backward-incompatible change
   - MINOR: New principle or section added
   - PATCH: Clarifications, typos, non-semantic changes
4. Update `LAST_AMENDED_DATE` to current date
5. Merge after review (or self-merge for solo projects)

### Compliance
- All PRs MUST verify compliance with constitution principles
- AI agents MUST read this file before making architectural decisions
- Deviations require explicit justification in PR description

**Version**: 1.0.0 | **Ratified**: 2026-01-16 | **Last Amended**: 2026-01-16
