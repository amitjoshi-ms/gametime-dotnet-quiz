# Documentation Index

Welcome to the GameTime .NET Quiz documentation.

## Quick Links

| Document | Description |
|----------|-------------|
| [Architecture](architecture.md) | System design, two-repo structure, data flow |
| [Development](development.md) | Local setup, build commands, workflow |
| [Deployment](deployment.md) | Cloudflare Pages configuration |
| [Data Contracts](data-contracts.md) | JSON schemas for manifest and chunks |
| [Components](components.md) | Blazor components and services |
| [Contributing](contributing.md) | How to contribute, code standards |

## Project Overview

GameTime .NET Quiz is a static single-page application that:

- Presents 10 random multiple-choice questions per game
- Uses AI-generated questions from .NET documentation
- Runs entirely in the browser (Blazor WebAssembly)
- Works offline after first load

## Technology Stack

| Category | Technology |
|----------|------------|
| Runtime | .NET 10.0 |
| Framework | Blazor WebAssembly (standalone) |
| UI Library | Microsoft Fluent UI 5.0.0 |
| Hosting | Cloudflare Pages |
| Data | Static JSON (separate data repository) |

## Repository Structure

```
gametime-dotnet-quiz/           # This repository (CODE)
├── src/
│   ├── Shared/                 # Shared models
│   ├── QuizApp/                # Blazor WASM app
│   └── QuestionGenerator/      # Console app for data generation
├── tests/
├── docs/                       # This documentation
└── .specify/                   # Speckit tool (internal)

gametime-dotnet-quiz-data/      # Separate repository (DATA)
├── data/                       # Generated content
└── publish/                    # Cloudflare deployment
```

## Getting Started

1. **Clone**: `git clone https://github.com/amitjoshi-ms/gametime-dotnet-quiz.git`
2. **Build**: `.\build.ps1`
3. **Run**: `.\build.ps1 -Run`
4. **Open**: https://localhost:5001

See [Development](development.md) for detailed setup instructions.
