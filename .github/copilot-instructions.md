# gamtime-dotnet-quiz Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-01-16

## Active Technologies

- C# 13 / .NET 10.0 + Blazor WASM (standalone), Microsoft.FluentUI.AspNetCore.Components 5.0.0 (001-blazor-quiz-app)

## Project Structure

```text
src/
├── Shared/                 # Shared class library (models)
├── QuizApp/                # Blazor WASM application
└── QuestionGenerator/      # Console app for data generation
tests/
├── Shared.Tests/
├── QuizApp.Tests/
└── QuestionGenerator.Tests/
docs/                       # Project documentation
```

## Commands

```powershell
# Build
.\build.ps1                           # Full build (clean, restore, build, test, lint, publish)
.\build.ps1 -Target Build             # Build only
.\build.ps1 -Configuration Release    # Release build

# Test
dotnet test                           # Run all tests
dotnet test --collect:"XPlat Code Coverage"  # With coverage
.\build.ps1 -Target Test              # Via build script

# Lint
dotnet format --verify-no-changes     # Check formatting (CI uses this)
dotnet format                         # Auto-fix formatting
.\build.ps1 -Target Lint              # Via build script

# Run
.\build.ps1 -Run                      # Build and launch dev server
dotnet run --project src/QuizApp      # Direct run
dotnet watch --project src/QuizApp    # Hot reload mode
```

## Code Style

- Use records for DTOs: `public sealed record Question(...)`
- File-scoped namespaces: `namespace GameTime.Quiz.Models;`
- Primary constructors for DI
- Async suffix for async methods: `GetRandomQuestionsAsync`
- Private fields: `_camelCase`
- 4-space indentation, UTF-8, LF line endings

## Testing Requirements

- Minimum 80% line coverage (per Constitution)
- Test naming: `MethodName_Scenario_ExpectedResult`
- Use xUnit 2.9.0, FluentAssertions 7.0.0, NSubstitute 5.3.0

## Documentation

See `docs/` folder:
- [docs/README.md](docs/README.md) - Documentation index
- [docs/architecture.md](docs/architecture.md) - System design
- [docs/development.md](docs/development.md) - Dev setup
- [docs/deployment.md](docs/deployment.md) - Cloudflare Pages
- [docs/data-contracts.md](docs/data-contracts.md) - JSON schemas
- [docs/components.md](docs/components.md) - Blazor components
- [docs/contributing.md](docs/contributing.md) - Contribution guide

## Recent Changes

- 001-blazor-quiz-app: Added C# 13 / .NET 10.0 + Blazor WASM (standalone), Microsoft.FluentUI.AspNetCore.Components 5.0.0

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
