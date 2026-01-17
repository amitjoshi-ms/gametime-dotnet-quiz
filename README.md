# 🎮 GameTime .NET Quiz

A free, static single-page application that generates quiz questions from official Microsoft .NET documentation. Built with Blazor WebAssembly and Fluent UI, hosted on Cloudflare Pages.

[![Build Status](https://github.com/YOUR_USERNAME/gametime-dotnet-quiz/actions/workflows/validate-pr.yml/badge.svg)](https://github.com/YOUR_USERNAME/gametime-dotnet-quiz/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 🎯 Features

- **10 Questions Per Game** - Quick, focused learning sessions
- **Multiple Choice Format** - 2 options per question for fast decision-making
- **AI-Generated Questions** - Fresh content from GitHub Models (GPT-4o-mini)
- **Offline Support** - Progressive Web App with service worker caching
- **Zero Cost** - All free tiers (Cloudflare Pages, GitHub Actions, GitHub Models)
- **Modern UI** - Microsoft Fluent UI design system

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLOUDFLARE PAGES                         │
├────────────────────────────┬────────────────────────────────────┤
│   gametime-dotnet-quiz     │     gametime-dotnet-quiz-data      │
│   (Blazor WASM App)        │     (JSON Data Endpoint)           │
│                            │                                    │
│   ┌──────────────────┐     │     ┌──────────────────┐           │
│   │   index.html     │     │     │   manifest.json  │ no-cache  │
│   │   _framework/*   │     │     │   chunks/*.json  │ immutable │
│   │   css/app.css    │     │     └──────────────────┘           │
│   └──────────────────┘     │                                    │
└────────────────────────────┴────────────────────────────────────┘
              │                              │
              │         FETCH                │
              └──────────────────────────────┘
```

### Two-Repository Design

| Repository | Purpose | Branch Protection |
|------------|---------|-------------------|
| `gametime-dotnet-quiz` | Blazor app source code | ✅ Required (PR reviews) |
| `gametime-dotnet-quiz-data` | Generated questions JSON | ❌ None (bot commits) |

This separation enables branch protection on the code repo while allowing automated commits to the data repo on GitHub's free plan.

## 🛠️ Technology Stack

| Category | Technology | Version |
|----------|------------|---------|
| **Runtime** | .NET | 10.0 |
| **Framework** | Blazor WebAssembly | Standalone |
| **UI Library** | Microsoft.FluentUI.AspNetCore.Components | 4.13.2 |
| **Hosting** | Cloudflare Pages | Free tier |
| **LLM** | GitHub Models | openai/gpt-4o-mini |
| **Testing** | xUnit + FluentAssertions + NSubstitute | Latest |
| **CI/CD** | GitHub Actions | - |

## 📁 Project Structure

```
gametime-dotnet-quiz/
├── src/
│   ├── Shared/                      # Shared models and utilities
│   │   ├── Models/
│   │   │   ├── Question.cs          # Quiz question model
│   │   │   ├── QuestionChunk.cs     # Chunk container
│   │   │   ├── Manifest.cs          # Questions manifest
│   │   │   └── ContentManifest.cs   # Scraping state
│   │   └── Shared.csproj
│   │
│   ├── QuizApp/                     # Blazor WASM application
│   │   ├── wwwroot/
│   │   │   ├── _headers             # Cloudflare caching rules
│   │   │   ├── css/app.css          # Fluent UI customizations
│   │   │   └── index.html           # SPA entry point
│   │   ├── Pages/
│   │   │   ├── Home.razor           # Landing page
│   │   │   ├── Quiz.razor           # Game play page
│   │   │   └── Results.razor        # Score display
│   │   ├── Components/
│   │   │   ├── QuestionCard.razor   # Question display
│   │   │   └── ProgressBar.razor    # Quiz progress
│   │   ├── Services/
│   │   │   ├── QuizService.cs       # Data fetching & caching
│   │   │   └── QuizState.cs         # Game state management
│   │   ├── Program.cs               # App configuration
│   │   └── QuizApp.csproj
│   │
│   └── QuestionGenerator/           # GitHub Actions console app
│       ├── Services/
│       │   ├── PageScraper.cs       # AngleSharp web scraping
│       │   ├── ContentHasher.cs     # SHA256 change detection
│       │   ├── GitHubModelsClient.cs # LLM API integration
│       │   └── QuestionManager.cs   # Chunk management
│       ├── Program.cs
│       └── QuestionGenerator.csproj
│
├── tests/
│   ├── Shared.Tests/
│   └── QuestionGenerator.Tests/
│
├── .github/
│   └── workflows/
│       └── validate-pr.yml          # PR build/lint/test
│
├── Directory.Build.props            # Central build properties
├── Directory.Build.targets          # Custom build targets
├── Directory.Packages.props         # Central package versions
├── global.json                      # SDK version pinning
├── nuget.config                     # NuGet configuration
├── build.ps1                        # Windows build script
├── build.sh                         # Cloudflare build script
├── .editorconfig                    # Code style rules
├── .gitignore
├── CONSTITUTION.md                  # Project standards
└── README.md                        # This file
```

## 🚀 Getting Started

### Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- [Git](https://git-scm.com/)
- [Visual Studio Code](https://code.visualstudio.com/) (recommended)

### Clone & Build

```powershell
# Clone the repository
git clone https://github.com/YOUR_USERNAME/gametime-dotnet-quiz.git
cd gametime-dotnet-quiz

# Build and run
.\build.ps1 -Run
```

### Build Script Commands

```powershell
# Windows (PowerShell)
.\build.ps1                    # Full build (clean, restore, build, test, lint, publish)
.\build.ps1 -Run               # Build and launch dev server
.\build.ps1 -SkipTests         # Skip test execution
.\build.ps1 -Configuration Release  # Release build

# Available targets
.\build.ps1 -Target Clean      # Clean all outputs
.\build.ps1 -Target Restore    # Restore packages
.\build.ps1 -Target Build      # Compile solution
.\build.ps1 -Target Test       # Run tests
.\build.ps1 -Target Lint       # Run code analysis
.\build.ps1 -Target Publish    # Create deployment package
```

### Running Locally

```powershell
# Option 1: Using build script
.\build.ps1 -Run

# Option 2: Direct dotnet commands
dotnet run --project src/QuizApp

# Option 3: Watch mode (auto-reload)
dotnet watch --project src/QuizApp
```

The app will be available at `https://localhost:5001` or `http://localhost:5000`.

## 🧪 Testing

```powershell
# Run all tests
dotnet test

# Run with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test project
dotnet test tests/QuestionGenerator.Tests
```

### Test Structure

| Project | Tests |
|---------|-------|
| `Shared.Tests` | Model validation, serialization |
| `QuestionGenerator.Tests` | Scraping, hashing, LLM integration |

## 🎨 Fluent UI Components

The app uses Microsoft's Fluent UI design system for a modern, accessible experience:

```razor
@* Example: Question Card Component *@
<FluentCard>
    <FluentStack Orientation="Orientation.Vertical" VerticalGap="16">
        <FluentLabel Typo="Typography.H4">
            @Question.QuestionText
        </FluentLabel>
        
        @foreach (var (option, index) in Question.Options.Select((o, i) => (o, i)))
        {
            <FluentButton Appearance="@GetAppearance(index)"
                          OnClick="@(() => SelectAnswer(index))"
                          Style="width: 100%">
                @option
            </FluentButton>
        }
    </FluentStack>
</FluentCard>
```

### Key Components Used

| Component | Usage |
|-----------|-------|
| `FluentCard` | Question containers, result cards |
| `FluentButton` | Answer options, navigation |
| `FluentProgress` | Quiz progress indicator |
| `FluentBadge` | Score display, category tags |
| `FluentIcon` | Correct/incorrect feedback |
| `FluentToast` | Notifications |
| `FluentDialog` | Confirmations |

## 📦 Deployment

### Cloudflare Pages Setup

1. **Connect Repository**
   - Go to Cloudflare Dashboard → Pages → Create a project
   - Connect your GitHub repository

2. **Configure Build Settings**
   ```
   Build command: chmod +x build.sh && ./build.sh
   Build output directory: publish
   Root directory: /
   ```

3. **Environment Variables**
   ```
   DOTNET_VERSION: 10.0.100
   ```

### Caching Strategy

| Asset Type | Cache Control | Rationale |
|------------|---------------|-----------|
| `_framework/*.dll` | 1 year, immutable | Content-hashed by Blazor |
| `_framework/*.wasm` | 1 year, immutable | Content-hashed by Blazor |
| `manifest.json` | no-cache | Must check for updates |
| `chunks/*.json` | 1 year, immutable | Filename contains hash |

## 🔄 Question Generation

Questions are generated weekly by the companion data repository:

```
┌──────────────────┐     ┌─────────────────┐     ┌──────────────────┐
│   GitHub Actions │────▶│  PageScraper    │────▶│ GitHub Models    │
│   (Weekly Cron)  │     │  (AngleSharp)   │     │ (GPT-4o-mini)    │
└──────────────────┘     └─────────────────┘     └──────────────────┘
                                                          │
                                                          ▼
                                                 ┌──────────────────┐
                                                 │  Question Chunks │
                                                 │  (JSON Files)    │
                                                 └──────────────────┘
```

### Content Sources

Questions are generated from:
- https://dotnet.microsoft.com/en-us/learn
- Tutorial pages, concept guides, and documentation

### Question Schema

```json
{
  "id": "csharp-basics-001-a1b2c3d4",
  "category": "csharp-basics",
  "questionText": "What keyword declares an immutable variable in C#?",
  "options": ["const", "var"],
  "correctIndex": 0,
  "explanation": "The 'const' keyword creates a compile-time constant.",
  "sourceUrl": "https://dotnet.microsoft.com/en-us/learn/csharp",
  "generatedAt": "2026-01-16T00:00:00Z"
}
```

## 🤝 Contributing

### Development Workflow

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Make** your changes
4. **Test** locally (`.\build.ps1 -Target Test`)
5. **Commit** with conventional commits (`git commit -m 'feat: add amazing feature'`)
6. **Push** to your fork (`git push origin feature/amazing-feature`)
7. **Open** a Pull Request

### Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Code Style

- Follow `.editorconfig` rules
- Run `dotnet format` before committing
- All public APIs must have XML documentation
- Maintain test coverage for new features

## 📋 Project Standards

See [CONSTITUTION.md](CONSTITUTION.md) for complete project standards including:
- Architecture decisions
- Coding conventions
- Testing requirements
- Security guidelines
- Performance targets

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Microsoft Learn](https://dotnet.microsoft.com/en-us/learn) - Content source
- [Fluent UI](https://www.fluentui-blazor.net/) - UI components
- [GitHub Models](https://github.com/marketplace/models) - AI question generation
- [Cloudflare Pages](https://pages.cloudflare.com/) - Free hosting

---

<p align="center">
  Made with ❤️ for the .NET community
</p>
