# Development Guide

## Prerequisites

| Requirement | Version | Check Command |
|-------------|---------|---------------|
| .NET SDK | 10.0.100+ | `dotnet --version` |
| Git | 2.x | `git --version` |
| PowerShell | 7.x | `$PSVersionTable.PSVersion` |
| VS Code | Latest | Recommended IDE |

### Recommended VS Code Extensions

- C# Dev Kit (`ms-dotnettools.csdevkit`)
- .NET Extension Pack (`ms-dotnettools.vscode-dotnet-runtime`)
- EditorConfig (`EditorConfig.EditorConfig`)

## Initial Setup

### 1. Clone Repository

```powershell
git clone https://github.com/amitjoshi-ms/gametime-dotnet-quiz.git
cd gametime-dotnet-quiz
```

### 2. Verify SDK Version

```powershell
# Should match version in global.json
dotnet --version
# Expected: 10.0.100 or higher
```

### 3. Restore and Build

```powershell
# Using build script (recommended)
.\build.ps1

# Or using dotnet CLI directly
dotnet restore
dotnet build
```

### 4. Run Tests

```powershell
.\build.ps1 -Target Test

# Or directly
dotnet test
```

### 5. Run Application

```powershell
.\build.ps1 -Run

# Or directly
dotnet run --project src/QuizApp
```

Open browser to **https://localhost:5001**

## Build Script Reference

The `build.ps1` script provides a unified build experience:

```powershell
# Full build (clean, restore, build, test, lint, publish)
.\build.ps1

# Specific targets
.\build.ps1 -Target Clean      # Remove all build outputs
.\build.ps1 -Target Restore    # Restore NuGet packages
.\build.ps1 -Target Build      # Compile solution
.\build.ps1 -Target Test       # Run all tests
.\build.ps1 -Target Lint       # Check code formatting
.\build.ps1 -Target Publish    # Create deployment package

# Options
.\build.ps1 -Configuration Release  # Release build (default: Debug)
.\build.ps1 -SkipTests              # Skip test execution
.\build.ps1 -Run                    # Build and launch dev server

# Combine options
.\build.ps1 -Configuration Release -SkipTests
```

## Project Structure

```
src/
├── Shared/                     # Shared class library
│   ├── Models/
│   │   ├── Question.cs         # Quiz question record
│   │   ├── QuestionChunk.cs    # Chunk container
│   │   ├── Manifest.cs         # Question index
│   │   ├── CategorySummary.cs  # Category stats
│   │   └── ChunkReference.cs   # Chunk pointer
│   └── Shared.csproj
│
├── QuizApp/                    # Blazor WASM application
│   ├── wwwroot/
│   │   ├── index.html          # SPA entry point
│   │   ├── css/app.css         # Custom styles
│   │   ├── _headers            # Cloudflare caching
│   │   └── service-worker.js   # Offline support
│   ├── Layout/
│   │   └── MainLayout.razor    # App shell
│   ├── Pages/
│   │   ├── Home.razor          # Landing page
│   │   ├── Quiz.razor          # Game page
│   │   └── Results.razor       # Score page
│   ├── Components/
│   │   ├── QuestionCard.razor  # Question display
│   │   └── ProgressIndicator.razor
│   ├── Services/
│   │   ├── IQuizService.cs     # Service interface
│   │   ├── QuizService.cs      # Data fetching
│   │   └── QuizState.cs        # Game state
│   ├── Program.cs              # App configuration
│   └── QuizApp.csproj
│
└── QuestionGenerator/          # Console app for data generation
    ├── Services/
    │   ├── PageScraper.cs      # HTML scraping
    │   ├── ContentHasher.cs    # SHA256 hashing
    │   ├── GitHubModelsClient.cs # LLM API
    │   └── QuestionManager.cs  # Chunk management
    ├── Program.cs
    └── QuestionGenerator.csproj

tests/
├── Shared.Tests/
│   └── Models/
│       ├── QuestionTests.cs
│       └── ManifestTests.cs
└── QuizApp.Tests/
    └── Services/
        ├── QuizServiceTests.cs
        └── QuizStateTests.cs
```

## Development Workflow

### Feature Development

1. **Create branch**
   ```powershell
   git checkout -b feature/my-feature
   ```

2. **Make changes**
   - Write tests first (TDD per Constitution)
   - Implement feature
   - Ensure tests pass

3. **Format and lint**
   ```powershell
   dotnet format
   .\build.ps1 -Target Lint
   ```

4. **Commit**
   ```powershell
   git add .
   git commit -m "feat(scope): description"
   ```

5. **Push and create PR**
   ```powershell
   git push -u origin feature/my-feature
   # Create PR via GitHub
   ```

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting, no code change
- `refactor`: Code restructuring
- `test`: Adding/updating tests
- `chore`: Maintenance

**Examples:**
```
feat(quiz): add countdown timer for questions
fix(scoring): correct off-by-one in final score
docs(readme): add deployment instructions
test(state): add QuizState reset tests
chore(deps): update FluentUI to 5.0.1
```

## Hot Reload

For faster development, use watch mode:

```powershell
dotnet watch --project src/QuizApp
```

Changes to `.razor` and `.cs` files trigger automatic rebuild and browser refresh.

## Debugging

### VS Code

1. Open project in VS Code
2. Press `F5` or select **Run > Start Debugging**
3. Select **.NET Core** if prompted
4. Browser opens automatically

### Debug Configuration

`.vscode/launch.json`:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch QuizApp",
      "type": "blazorwasm",
      "request": "launch",
      "cwd": "${workspaceFolder}/src/QuizApp"
    }
  ]
}
```

### Browser DevTools

- **F12**: Open DevTools
- **Application > Service Workers**: Manage offline cache
- **Network**: Monitor API calls
- **Console**: View Blazor errors

## Testing

### Run All Tests

```powershell
dotnet test
```

### Run Specific Project

```powershell
dotnet test tests/QuizApp.Tests
dotnet test tests/Shared.Tests
```

### Run with Coverage

```powershell
dotnet test --collect:"XPlat Code Coverage"
```

Coverage reports generated in `tests/*/TestResults/*/coverage.cobertura.xml`

### Test Naming Convention

```csharp
[Fact]
public async Task MethodName_Scenario_ExpectedResult()
{
    // Arrange
    // Act
    // Assert
}
```

## Code Style

### EditorConfig

Project includes `.editorconfig` with rules for:
- 4-space indentation
- LF line endings
- UTF-8 encoding
- Trim trailing whitespace

### Format Check

```powershell
# Check formatting (CI uses this)
dotnet format --verify-no-changes

# Auto-fix formatting
dotnet format
```

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

## Troubleshooting

### Build Errors

```powershell
# Clean and rebuild
.\build.ps1 -Target Clean
dotnet restore
dotnet build
```

### Package Restore Issues

```powershell
# Clear NuGet cache
dotnet nuget locals all --clear
dotnet restore
```

### Service Worker Issues

If changes aren't appearing:
1. Open DevTools (F12)
2. Go to **Application > Service Workers**
3. Click **Unregister**
4. Hard refresh (Ctrl+Shift+R)

### Port Already in Use

```powershell
# Find process using port 5001
netstat -ano | findstr :5001

# Kill process (replace PID)
taskkill /PID <pid> /F
```
