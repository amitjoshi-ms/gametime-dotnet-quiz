# Contributing Guide

Thank you for your interest in contributing to GameTime .NET Quiz!

## Code of Conduct

Be respectful, inclusive, and constructive. We're all here to learn and build something fun.

## Getting Started

1. Fork the repository
2. Clone your fork
3. Create a feature branch
4. Make your changes
5. Submit a pull request

See [Development Guide](development.md) for detailed setup instructions.

## Development Process

### 1. Find or Create an Issue

- Check existing issues for something to work on
- Create an issue for new features or bugs
- Wait for confirmation before starting large changes

### 2. Create a Branch

```powershell
git checkout main
git pull origin main
git checkout -b feature/your-feature-name
```

Branch naming conventions:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation
- `refactor/` - Code restructuring

### 3. Make Changes

Follow these guidelines:

- **Test First**: Write tests before implementation (per Constitution)
- **Small Commits**: One logical change per commit
- **Clear Messages**: Follow commit message format
- **Format Code**: Run `dotnet format` before committing

### 4. Test Your Changes

```powershell
# Run all tests
dotnet test

# Check formatting
dotnet format --verify-no-changes

# Build to ensure no errors
dotnet build
```

### 5. Submit Pull Request

1. Push your branch to your fork
2. Create PR against `main` branch
3. Fill out the PR template
4. Wait for review

## Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `refactor` | Code restructuring |
| `test` | Adding/updating tests |
| `chore` | Maintenance |

### Examples

```
feat(quiz): add countdown timer for questions

Adds a 30-second countdown timer to each question.
Timer is optional and can be disabled in settings.

Closes #42
```

```
fix(scoring): correct off-by-one error in final score

The score was showing 9/10 when all answers were correct.
Fixed by adjusting the count calculation.

Fixes #57
```

## Pull Request Guidelines

### PR Title

Use the same format as commit messages:
```
feat(quiz): add countdown timer
```

### PR Description

Include:
- **What**: Brief description of changes
- **Why**: Motivation and context
- **How**: Implementation approach
- **Testing**: How you tested the changes

### PR Checklist

- [ ] Code follows project conventions
- [ ] Tests added/updated for changes
- [ ] All tests pass (`dotnet test`)
- [ ] Code formatted (`dotnet format`)
- [ ] Documentation updated if needed
- [ ] No compiler warnings

### Review Process

1. Automated checks run (build, test, lint)
2. Maintainer reviews code
3. Address feedback with new commits
4. Squash merge when approved

## Code Standards

### Constitution Principles

All contributions must follow the [Constitution](.specify/memory/constitution.md):

1. **Zero Cost**: No paid services
2. **Static-First**: No server runtime
3. **Immutable Caching**: Content-addressed filenames
4. **Additive-Only**: Questions never deleted
5. **Test-First**: 80% minimum coverage

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Namespace | PascalCase | `GameTime.Quiz.Models` |
| Class/Record | PascalCase | `QuestionChunk` |
| Interface | IPascalCase | `IQuizService` |
| Method | PascalCase | `GetRandomQuestions` |
| Property | PascalCase | `QuestionText` |
| Private Field | _camelCase | `_httpClient` |
| Parameter | camelCase | `questionCount` |

### File Organization

```csharp
// 1. File-scoped namespace
namespace GameTime.Quiz.Services;

// 2. Using statements (if needed beyond implicit)

// 3. XML documentation
/// <summary>
/// Service for fetching quiz questions.
/// </summary>
public sealed class QuizService : IQuizService
{
    // 4. Constants
    private const int DefaultQuestionCount = 10;

    // 5. Fields
    private readonly HttpClient _httpClient;

    // 6. Constructor
    public QuizService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    // 7. Properties

    // 8. Public methods

    // 9. Private methods
}
```

### Required Patterns

- Use records for DTOs
- Use dependency injection
- Use async/await properly
- Handle errors gracefully

See [Constitution](.specify/memory/constitution.md) for detailed patterns.

## Testing Guidelines

### Test Naming

```csharp
[Fact]
public async Task MethodName_Scenario_ExpectedResult()
{
    // Arrange
    // Act  
    // Assert
}
```

### Test Structure (AAA)

```csharp
[Fact]
public async Task GetRandomQuestionsAsync_WithValidCount_ReturnsRequestedQuestions()
{
    // Arrange
    var service = CreateService();
    
    // Act
    var questions = await service.GetRandomQuestionsAsync(5);
    
    // Assert
    questions.Should().HaveCount(5);
}
```

### Coverage Requirements

| Metric | Minimum |
|--------|---------|
| Line Coverage | 80% |
| Branch Coverage | 70% |
| Critical Paths | 100% |

## Documentation

### When to Update Docs

- Adding new components or services
- Changing public APIs
- Adding configuration options
- Fixing common issues

### Documentation Files

| File | Content |
|------|---------|
| `README.md` | Project overview |
| `docs/architecture.md` | System design |
| `docs/development.md` | Dev setup |
| `docs/deployment.md` | Deploy instructions |
| `docs/components.md` | Component reference |
| `docs/data-contracts.md` | JSON schemas |

### XML Documentation

All public APIs should have XML docs:

```csharp
/// <summary>
/// Fetches random questions from the data endpoint.
/// </summary>
/// <param name="count">Number of questions to return (default: 10).</param>
/// <returns>A randomized list of questions.</returns>
/// <exception cref="HttpRequestException">When endpoint is unreachable.</exception>
public async Task<IReadOnlyList<Question>> GetRandomQuestionsAsync(int count = 10)
```

## Questions?

- Open a discussion on GitHub
- Check existing issues and PRs
- Review the documentation

Thank you for contributing! 🎉
