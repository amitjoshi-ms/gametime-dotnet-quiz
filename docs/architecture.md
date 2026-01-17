# Architecture

## Overview

GameTime .NET Quiz follows a static-first architecture with complete separation between code and data.

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER'S BROWSER                          │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Blazor WebAssembly                     │  │
│  │  ┌─────────┐  ┌─────────────┐  ┌───────────────────────┐  │  │
│  │  │  Home   │  │    Quiz     │  │       Results         │  │  │
│  │  │  Page   │──│    Page     │──│        Page           │  │  │
│  │  └─────────┘  └─────────────┘  └───────────────────────┘  │  │
│  │                      │                                     │  │
│  │              ┌───────▼───────┐                             │  │
│  │              │  QuizService  │                             │  │
│  │              └───────────────┘                             │  │
│  │                      │                                     │  │
│  │              ┌───────▼───────┐                             │  │
│  │              │  QuizState    │                             │  │
│  │              └───────────────┘                             │  │
│  └───────────────────────────────────────────────────────────┘  │
│                         │ HTTPS                                  │
└─────────────────────────┼───────────────────────────────────────┘
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
        ▼                                   ▼
┌───────────────────┐             ┌───────────────────┐
│  CLOUDFLARE CDN   │             │  CLOUDFLARE CDN   │
│  (Quiz App)       │             │  (Data Endpoint)  │
│                   │             │                   │
│  - index.html     │             │  - manifest.json  │
│  - _framework/*   │             │  - chunks/*.json  │
│  - css/app.css    │             │                   │
└───────────────────┘             └───────────────────┘
        │                                   │
        │                                   │
┌───────▼───────────┐             ┌────────▼──────────┐
│  gametime-dotnet  │             │  gametime-dotnet  │
│  -quiz (GitHub)   │             │  -quiz-data       │
│                   │             │  (GitHub)         │
│  CODE REPOSITORY  │             │  DATA REPOSITORY  │
└───────────────────┘             └───────────────────┘
```

## Two-Repository Design

### Why Two Repositories?

1. **Branch Protection**: GitHub free plan allows branch protection only if no automated commits. Code repo is protected; data repo allows bot commits.

2. **Separation of Concerns**: Code changes require PR review; data regeneration is automated weekly.

3. **Independent Caching**: App assets and data have different caching strategies.

| Repository | Purpose | Protection | Updates |
|------------|---------|------------|---------|
| `gametime-dotnet-quiz` | Application code | Branch protection, PR required | Manual (developer) |
| `gametime-dotnet-quiz-data` | Generated questions | None | Automated (weekly) |

### Code Repository Structure

```
gametime-dotnet-quiz/
├── src/
│   ├── Shared/                 # CLASS LIBRARY
│   │   └── Models/             # Question, Manifest, etc.
│   │
│   ├── QuizApp/                # BLAZOR WASM
│   │   ├── Pages/              # Home, Quiz, Results
│   │   ├── Components/         # QuestionCard, ProgressIndicator
│   │   ├── Services/           # QuizService, QuizState
│   │   └── wwwroot/            # Static assets
│   │
│   └── QuestionGenerator/      # CONSOLE APP
│       └── Services/           # Scraper, Hasher, LLM Client
│
├── tests/
│   ├── Shared.Tests/
│   └── QuizApp.Tests/
│
├── docs/                       # Documentation
├── .github/                    # Workflows, Copilot config
└── .specify/                   # Speckit tool
```

### Data Repository Structure

```
gametime-dotnet-quiz-data/
├── data/
│   ├── state/                  # Scraping state (not deployed)
│   │   ├── target-pages.json
│   │   └── content-manifest.json
│   └── questions/              # Generated questions
│       ├── manifest.json
│       └── chunks/
│           └── {category}-{seq}-{hash8}.json
│
└── publish/                    # Cloudflare deployment root
    ├── _headers
    ├── manifest.json
    └── chunks/
```

## Data Flow

### Question Loading

```
1. App loads → QuizService.GetRandomQuestionsAsync()
          │
          ▼
2. Fetch manifest.json from data endpoint
          │
          ▼
3. Parse manifest → Get list of chunk references
          │
          ▼
4. Select random chunks (enough for 10+ questions)
          │
          ▼
5. Fetch selected chunk files in parallel
          │
          ▼
6. Flatten all questions → Shuffle → Take 10
          │
          ▼
7. Store in QuizState → Display first question
```

### Answer Flow

```
1. User selects answer option
          │
          ▼
2. QuizState.SubmitAnswer(selectedIndex)
          │
          ▼
3. Compare with Question.CorrectIndex
          │
          ▼
4. Update Answers list, increment CurrentIndex
          │
          ▼
5. Fire OnStateChanged event
          │
          ▼
6. UI re-renders with feedback
          │
          ▼
7. If CurrentIndex >= 10 → Navigate to Results
```

## Component Architecture

### Services

```
┌─────────────────────────────────────────────────────────┐
│                    Program.cs (DI)                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  builder.Services.AddScoped<IQuizService, QuizService>  │
│  builder.Services.AddScoped<QuizState>                  │
│  builder.Services.AddHttpClient("DataApi", ...)        │
│  builder.Services.AddFluentUIComponents()               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

| Service | Lifetime | Purpose |
|---------|----------|---------|
| `IQuizService` | Scoped | Fetches and caches questions |
| `QuizState` | Scoped | Manages game state within session |
| `HttpClient` | Named | Configured for data endpoint |

### Pages

| Page | Route | Purpose |
|------|-------|---------|
| `Home.razor` | `/` | Landing page with Start button |
| `Quiz.razor` | `/quiz` | Question display and answering |
| `Results.razor` | `/results` | Final score and replay options |

### Components

| Component | Purpose |
|-----------|---------|
| `MainLayout.razor` | App shell with FluentUI layout |
| `QuestionCard.razor` | Displays question, options, feedback |
| `ProgressIndicator.razor` | Shows "Question X of 10" |

## Caching Strategy

### App Assets (Code Repo CDN)

| Asset | Cache-Control | Rationale |
|-------|---------------|-----------|
| `index.html` | `no-cache` | Entry point, must be fresh |
| `_framework/*.dll` | `immutable, max-age=31536000` | Content-hashed by Blazor |
| `_framework/*.wasm` | `immutable, max-age=31536000` | Content-hashed by Blazor |
| `css/app.css` | `max-age=86400` | Daily refresh acceptable |

### Data Assets (Data Repo CDN)

| Asset | Cache-Control | Rationale |
|-------|---------------|-----------|
| `manifest.json` | `no-cache, must-revalidate` | Must check for new chunks |
| `chunks/*.json` | `immutable, max-age=31536000` | Filename includes hash |

### Service Worker

```
Strategy by asset type:
┌─────────────────────┬──────────────────┐
│  Asset Pattern      │  Strategy        │
├─────────────────────┼──────────────────┤
│  _framework/*       │  Cache-first     │
│  chunks/*.json      │  Cache-first     │
│  manifest.json      │  Network-first   │
│  index.html         │  Network-first   │
└─────────────────────┴──────────────────┘
```

## Security Model

### No Authentication

- All users are anonymous
- No personal data collected
- No session persistence

### Content Security

- Questions are pre-generated (no runtime LLM calls)
- All content from trusted source (Microsoft Learn)
- No user-generated content

### CORS Configuration

Data endpoint allows cross-origin requests:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, OPTIONS
```

## Performance Targets

| Metric | Target | How Measured |
|--------|--------|--------------|
| First Contentful Paint | < 2s | Lighthouse |
| Time to Interactive | < 4s | Lighthouse |
| Total Bundle Size | < 5MB | Network waterfall |
| Question Fetch | < 500ms | After WASM loaded |
| Lighthouse Accessibility | > 90 | Lighthouse audit |
