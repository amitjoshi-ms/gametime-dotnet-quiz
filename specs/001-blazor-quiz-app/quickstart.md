# Quickstart: Blazor Quiz App

**Feature**: 001-blazor-quiz-app  
**Date**: 2026-01-16

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- Git
- VS Code with C# Dev Kit (recommended)

## Getting Started

### 1. Clone and Build

```powershell
# Clone the repository
git clone https://github.com/amitjoshi-ms/gametime-dotnet-quiz.git
cd gametime-dotnet-quiz

# Restore and build
dotnet restore
dotnet build
```

### 2. Run Locally

```powershell
# Run the quiz app
dotnet run --project src/QuizApp

# Or with hot reload
dotnet watch --project src/QuizApp
```

Open browser to `https://localhost:5001`

### 3. Run Tests

```powershell
dotnet test
```

## Project Structure

```
src/
├── Shared/          # Shared models (Question, Manifest, etc.)
├── QuizApp/         # Blazor WASM application
└── QuestionGenerator/ # Console app for data generation

tests/
├── Shared.Tests/
└── QuizApp.Tests/
```

## Key Components

### Models (src/Shared/Models/)

```csharp
// Question.cs
public sealed record Question(
    string Id,
    string QuestionText,
    IReadOnlyList<string> Options,
    int CorrectIndex,
    string Explanation);

// Manifest.cs
public sealed record Manifest(
    string Version,
    DateTimeOffset GeneratedAt,
    int TotalQuestions,
    IReadOnlyList<CategorySummary> Categories,
    IReadOnlyList<ChunkReference> Chunks);
```

### Services (src/QuizApp/Services/)

```csharp
// IQuizService.cs
public interface IQuizService
{
    Task<IReadOnlyList<Question>> GetRandomQuestionsAsync(int count = 10);
}

// QuizState.cs (scoped service)
public sealed class QuizState
{
    public IReadOnlyList<Question> Questions { get; private set; }
    public int CurrentIndex { get; private set; }
    public int Score { get; }
    
    public void StartGame(IReadOnlyList<Question> questions);
    public void SubmitAnswer(int selectedIndex);
}
```

### Pages (src/QuizApp/Pages/)

| Page | Route | Purpose |
|------|-------|---------|
| Home.razor | `/` | Landing page with Start button |
| Quiz.razor | `/quiz` | Question display and answer selection |
| Results.razor | `/results` | Final score and replay option |

## Configuration

### Data Endpoint

Configure in `Program.cs`:

```csharp
builder.Services.AddHttpClient("DataApi", client =>
{
    client.BaseAddress = new Uri("https://gametime-dotnet-quiz-data.pages.dev/");
});
```

### Fluent UI Setup

```csharp
// Program.cs
builder.Services.AddFluentUIComponents();
```

```razor
<!-- App.razor -->
<FluentToastProvider />
<FluentDialogProvider />
<Router AppAssembly="typeof(App).Assembly">
    ...
</Router>
```

## Development Workflow

1. **Create feature branch**: `git checkout -b feature/my-feature`
2. **Make changes**: Edit code, add tests
3. **Test locally**: `dotnet test`
4. **Format code**: `dotnet format`
5. **Commit**: `git commit -m "feat(scope): description"`
6. **Push and PR**: `git push -u origin feature/my-feature`

## Build for Deployment

```powershell
# Windows
.\build.ps1

# Output in publish/ directory
```

## Common Tasks

### Add a New Page

1. Create `Pages/MyPage.razor`
2. Add `@page "/my-route"` directive
3. Add navigation link if needed

### Add a New Service

1. Create interface in `Services/IMyService.cs`
2. Create implementation in `Services/MyService.cs`
3. Register in `Program.cs`: `builder.Services.AddScoped<IMyService, MyService>()`

### Update Fluent UI Version

1. Edit `Directory.Packages.props`
2. Run `dotnet restore`
3. Test for breaking changes

## Troubleshooting

### CORS Errors

Data endpoint must have `Access-Control-Allow-Origin: *` header. Check `publish/_headers` in data repo.

### Service Worker Issues

Clear browser cache and service worker:
- Chrome DevTools → Application → Service Workers → Unregister
- Hard refresh (Ctrl+Shift+R)

### Build Failures

```powershell
# Clean and rebuild
dotnet clean
dotnet restore
dotnet build
```
