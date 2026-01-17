# Data Repository Setup Prompt

Use this prompt to set up the companion data repository for GameTime .NET Quiz.

---

## Prompt for AI Assistant

I need to create the **gametime-dotnet-quiz-data** repository - the companion data repo for my quiz application. This repo stores AI-generated quiz questions as JSON files and is deployed to Cloudflare Pages as a static data endpoint.

### Repository Purpose

- Stores generated quiz questions in chunked JSON format
- Runs weekly GitHub Actions workflow to scrape .NET docs and generate new questions
- Deploys to Cloudflare Pages as a CORS-enabled JSON API
- Allows bot commits without branch protection (separating code from data)

### Architecture Context

This is the **DATA** repo in a two-repo architecture:

| Repository | Purpose | Branch Protection |
|------------|---------|-------------------|
| `gametime-dotnet-quiz` | Blazor WASM app (CODE) | ✅ Required |
| `gametime-dotnet-quiz-data` | Generated questions (DATA) | ❌ None |

The quiz app (hosted separately) fetches questions from this repo's Cloudflare Pages deployment.

### Required Folder Structure

```
gametime-dotnet-quiz-data/
├── data/
│   ├── state/                       # Tracking state (NOT deployed)
│   │   ├── target-pages.json        # URLs to scrape
│   │   └── content-manifest.json    # Last scrape state with hashes
│   └── questions/                   # Generated questions (NOT deployed directly)
│       ├── manifest.json            # Question index
│       └── chunks/                  # Question chunk files
│           └── {category}-{seq}-{hash8}.json
│
├── publish/                         # Cloudflare Pages deploy root
│   ├── _headers                     # CORS + caching rules
│   ├── manifest.json                # Copied from data/questions/
│   └── chunks/                      # Copied from data/questions/chunks/
│       └── *.json
│
├── .github/
│   └── workflows/
│       └── generate-questions.yml   # Weekly question generation
│
├── .gitignore
└── README.md
```

### File Contents Required

#### 1. `data/state/target-pages.json`

```json
{
  "version": "1.0",
  "pages": [
    {
      "url": "https://dotnet.microsoft.com/en-us/learn/csharp",
      "category": "csharp-basics",
      "enabled": true
    },
    {
      "url": "https://dotnet.microsoft.com/en-us/learn/aspnet",
      "category": "aspnet-basics",
      "enabled": true
    },
    {
      "url": "https://dotnet.microsoft.com/en-us/learn/dotnet",
      "category": "dotnet-basics",
      "enabled": true
    },
    {
      "url": "https://dotnet.microsoft.com/en-us/learn/maui",
      "category": "maui-basics",
      "enabled": true
    },
    {
      "url": "https://dotnet.microsoft.com/en-us/learn/blazor",
      "category": "blazor-basics",
      "enabled": true
    }
  ]
}
```

#### 2. `data/state/content-manifest.json`

```json
{
  "version": "1.0",
  "lastRun": null,
  "pages": {}
}
```

#### 3. `data/questions/manifest.json`

```json
{
  "version": "1.0",
  "generatedAt": null,
  "totalQuestions": 0,
  "categories": [],
  "chunks": []
}
```

#### 4. `publish/_headers`

```
/*
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: GET, OPTIONS
  Access-Control-Allow-Headers: Content-Type

/manifest.json
  Cache-Control: no-cache, must-revalidate

/chunks/*
  Cache-Control: public, max-age=31536000, immutable
```

#### 5. `.github/workflows/generate-questions.yml`

```yaml
name: Generate Questions

on:
  schedule:
    # Every Sunday at 2:00 AM UTC
    - cron: '0 2 * * 0'
  workflow_dispatch:
    inputs:
      force_regenerate:
        description: 'Force regenerate all questions (ignore content hashes)'
        required: false
        default: 'false'
        type: boolean

permissions:
  contents: write

env:
  DOTNET_VERSION: '10.0.x'
  DOTNET_NOLOGO: true
  DOTNET_CLI_TELEMETRY_OPTOUT: true

jobs:
  generate:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout data repo
        uses: actions/checkout@v4
        with:
          path: data-repo

      - name: Checkout code repo
        uses: actions/checkout@v4
        with:
          repository: YOUR_USERNAME/gametime-dotnet-quiz
          path: code-repo

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Restore dependencies
        working-directory: code-repo
        run: dotnet restore

      - name: Build QuestionGenerator
        working-directory: code-repo
        run: dotnet build src/QuestionGenerator -c Release --no-restore

      - name: Run QuestionGenerator
        working-directory: code-repo
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          DATA_REPO_PATH: ${{ github.workspace }}/data-repo
          FORCE_REGENERATE: ${{ inputs.force_regenerate }}
        run: |
          dotnet run --project src/QuestionGenerator -c Release --no-build -- \
            --data-path "$DATA_REPO_PATH/data" \
            --publish-path "$DATA_REPO_PATH/publish" \
            $(if [ "$FORCE_REGENERATE" = "true" ]; then echo "--force"; fi)

      - name: Check for changes
        id: check_changes
        working-directory: data-repo
        run: |
          git add -A
          if git diff --staged --quiet; then
            echo "has_changes=false" >> $GITHUB_OUTPUT
          else
            echo "has_changes=true" >> $GITHUB_OUTPUT
          fi

      - name: Commit and push
        if: steps.check_changes.outputs.has_changes == 'true'
        working-directory: data-repo
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          
          # Get summary of changes
          NEW_QUESTIONS=$(git diff --staged --numstat | grep chunks/ | awk '{sum+=$1} END {print sum+0}')
          
          git commit -m "chore: generate questions [skip ci]" \
            -m "New/updated questions: ${NEW_QUESTIONS}" \
            -m "Generated at: $(date -u +"%Y-%m-%dT%H:%M:%SZ")"
          
          git push

      - name: Summary
        run: |
          echo "## Question Generation Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          if [ "${{ steps.check_changes.outputs.has_changes }}" = "true" ]; then
            echo "✅ New questions generated and committed" >> $GITHUB_STEP_SUMMARY
          else
            echo "ℹ️ No content changes detected" >> $GITHUB_STEP_SUMMARY
          fi
```

#### 6. `.gitignore`

```gitignore
# OS files
.DS_Store
Thumbs.db

# IDE
.idea/
.vscode/
*.swp
*.swo

# Logs
*.log
```

#### 7. `README.md`

Create a comprehensive README explaining:
- This is the data companion to gametime-dotnet-quiz
- Folder structure and purpose
- How the workflow runs
- Caching strategy (manifest = no-cache, chunks = immutable)
- How to manually trigger question generation
- Link to the main code repository

### Cloudflare Pages Configuration

After creating the repo, connect to Cloudflare Pages with these settings:

| Setting | Value |
|---------|-------|
| Project name | `gametime-dotnet-quiz-data` |
| Production branch | `main` |
| Build command | *(leave empty)* |
| Build output directory | `publish` |
| Root directory | `/` |

No build step needed - Cloudflare just serves the `publish/` folder as static files.

### Key Design Decisions

1. **Content-Addressed Chunks**: Filenames include hash (`csharp-basics-001-a1b2c3d4.json`) for immutable caching
2. **Manifest = No Cache**: Always fresh to detect new chunks
3. **CORS Enabled**: App hosted on different Cloudflare Pages project needs cross-origin access
4. **Additive Only**: Questions are never deleted, only added
5. **SHA256 Hashing**: Detect doc changes without relying on unreliable ETags

### Question Chunk Schema

```json
{
  "id": "csharp-basics-001-a1b2c3d4",
  "generatedAt": "2026-01-16T02:00:00Z",
  "category": "csharp-basics",
  "sourceUrl": "https://dotnet.microsoft.com/en-us/learn/csharp",
  "questions": [
    {
      "id": "q-uuid-here",
      "questionText": "What keyword declares an immutable variable in C#?",
      "options": ["const", "var"],
      "correctIndex": 0,
      "explanation": "The 'const' keyword creates a compile-time constant that cannot be changed."
    }
  ]
}
```

### Manifest Schema

```json
{
  "version": "1.0",
  "generatedAt": "2026-01-16T02:00:00Z",
  "totalQuestions": 150,
  "categories": [
    { "name": "csharp-basics", "count": 50 },
    { "name": "aspnet-basics", "count": 40 },
    { "name": "blazor-basics", "count": 60 }
  ],
  "chunks": [
    { "id": "csharp-basics-001-a1b2c3d4", "category": "csharp-basics", "count": 10 },
    { "id": "csharp-basics-002-e5f6g7h8", "category": "csharp-basics", "count": 10 }
  ]
}
```

---

Please create all the files listed above with the correct folder structure. Initialize as a Git repository ready to push to GitHub.
