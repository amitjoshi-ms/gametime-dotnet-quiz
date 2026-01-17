# Implementation Plan: Blazor Quiz App

**Branch**: `001-blazor-quiz-app` | **Date**: 2026-01-16 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-blazor-quiz-app/spec.md`

## Summary

Build a static Blazor WebAssembly quiz application that presents 10 random multiple-choice questions (2 options each) sourced from pre-generated JSON data. Uses Fluent UI for modern UX, fetches questions from a remote Cloudflare Pages endpoint, and supports offline play via service worker caching.

## Technical Context

**Language/Version**: C# 13 / .NET 10.0  
**Primary Dependencies**: Blazor WASM (standalone), Microsoft.FluentUI.AspNetCore.Components 4.13.2  
**Storage**: Static JSON files fetched from remote endpoint (no database)  
**Testing**: xUnit 2.9.0, FluentAssertions 7.0.0, NSubstitute 5.3.0  
**Target Platform**: Browser (WASM), deployed to Cloudflare Pages  
**Project Type**: Web (Blazor WASM standalone - no backend in this repo)  
**Performance Goals**: TTI < 4s on 3G, question fetch < 500ms post-load  
**Constraints**: Total bundle < 5MB compressed, offline-capable after first load  
**Scale/Scope**: Single-user browser app, ~100-1000 questions in data pool

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Requirement | Status | Notes |
|-----------|-------------|--------|-------|
| I. Zero Cost | Free tiers only | ✅ PASS | Cloudflare Pages, GitHub Actions |
| II. Static-First | No server runtime | ✅ PASS | Blazor WASM standalone, static JSON |
| III. Immutable Caching | Content-addressed files | ✅ PASS | Chunks use `{category}-{seq}-{hash8}.json` |
| IV. Additive-Only | No deletion/modification | ✅ PASS | Questions only added, never removed |
| V. Test-First | 80% coverage minimum | ⏳ PENDING | Tests required during implementation |

**Gate Status**: ✅ PASS - All constitution principles satisfied by design.

**Post-Design Verification** (2026-01-16):
- ✅ Data model uses immutable records
- ✅ JSON schemas enforce 2-option constraint
- ✅ No server endpoints in design
- ✅ All state is client-side only
- ✅ Caching headers defined for all asset types

## Project Structure

### Documentation (this feature)

```text
specs/001-blazor-quiz-app/
├── plan.md              # This file
├── research.md          # Phase 0 output (minimal - tech stack locked)
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output (JSON schemas)
└── tasks.md             # Phase 2 output (created by /speckit.tasks)
```

### Source Code (repository root)

```text
src/
├── Shared/                          # CLASS LIBRARY - shared models
│   ├── Models/
│   │   ├── Question.cs
│   │   ├── QuestionChunk.cs
│   │   └── Manifest.cs
│   └── Shared.csproj
│
├── QuizApp/                         # BLAZOR WASM - main application
│   ├── wwwroot/
│   │   ├── index.html
│   │   ├── css/app.css
│   │   ├── _headers                 # Cloudflare caching rules
│   │   └── service-worker.js        # Offline support
│   ├── Layout/
│   │   └── MainLayout.razor
│   ├── Pages/
│   │   ├── Home.razor
│   │   ├── Quiz.razor
│   │   └── Results.razor
│   ├── Components/
│   │   ├── QuestionCard.razor
│   │   └── ProgressIndicator.razor
│   ├── Services/
│   │   ├── IQuizService.cs
│   │   ├── QuizService.cs
│   │   └── QuizState.cs
│   ├── Program.cs
│   └── QuizApp.csproj
│
└── QuestionGenerator/               # CONSOLE APP - for data repo workflow
    ├── Services/
    │   ├── PageScraper.cs
    │   ├── ContentHasher.cs
    │   ├── GitHubModelsClient.cs
    │   └── QuestionManager.cs
    ├── Program.cs
    └── QuestionGenerator.csproj

tests/
├── Shared.Tests/
│   └── Models/
│       └── QuestionTests.cs
└── QuizApp.Tests/
    └── Services/
        ├── QuizServiceTests.cs
        └── QuizStateTests.cs

# Root configuration files
Directory.Build.props                # Central build properties
Directory.Build.targets              # Custom build targets
Directory.Packages.props             # Central package management
global.json                          # SDK version lock
nuget.config                         # NuGet sources
.editorconfig                        # Code style
build.ps1                            # Windows build script
build.sh                             # Cloudflare Linux build script
.gitignore
```

**Structure Decision**: Web application with shared library. QuizApp is standalone Blazor WASM (no backend). QuestionGenerator is separate console app used by data repo's GitHub Actions workflow.

## Complexity Tracking

No constitution violations. Design follows all principles without exceptions.
