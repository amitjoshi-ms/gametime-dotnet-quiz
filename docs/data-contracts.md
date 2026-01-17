# Data Contracts

This document describes the JSON data formats used by the quiz application.

## Overview

Questions are stored as static JSON files:

```
data-endpoint/
├── manifest.json           # Index of all chunks
└── chunks/
    ├── csharp-basics-001-a1b2c3d4.json
    ├── csharp-basics-002-e5f6g7h8.json
    ├── aspnet-basics-001-12345678.json
    └── ...
```

## Manifest

The `manifest.json` file indexes all available question chunks.

### Schema

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

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | ✅ | Schema version (e.g., "1.0") |
| `generatedAt` | string | ✅ | ISO 8601 timestamp |
| `totalQuestions` | integer | ✅ | Total questions across all chunks |
| `categories` | array | ✅ | Summary per category |
| `chunks` | array | ✅ | References to chunk files |

### CategorySummary

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Category identifier (lowercase, hyphenated) |
| `count` | integer | Number of questions in category |

### ChunkReference

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Chunk identifier (matches filename without `.json`) |
| `category` | string | Category of questions in chunk |
| `count` | integer | Number of questions in chunk |

## Question Chunk

Each chunk file contains a batch of questions.

### Filename Convention

```
{category}-{sequence}-{hash8}.json

Examples:
  csharp-basics-001-a1b2c3d4.json
  aspnet-basics-003-f9e8d7c6.json
```

- `category`: Topic category (lowercase, hyphenated)
- `sequence`: 3-digit sequential number (001, 002, ...)
- `hash8`: First 8 characters of content hash

### Schema

```json
{
  "id": "csharp-basics-001-a1b2c3d4",
  "generatedAt": "2026-01-16T02:00:00Z",
  "category": "csharp-basics",
  "sourceUrl": "https://dotnet.microsoft.com/en-us/learn/csharp",
  "questions": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "questionText": "What keyword declares an immutable variable in C#?",
      "options": ["const", "var"],
      "correctIndex": 0,
      "explanation": "The 'const' keyword creates a compile-time constant."
    }
  ]
}
```

### Chunk Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Chunk identifier (matches filename) |
| `generatedAt` | string | ✅ | ISO 8601 timestamp |
| `category` | string | ✅ | Category of questions |
| `sourceUrl` | string | ✅ | Source documentation URL |
| `questions` | array | ✅ | Array of Question objects |

### Question Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier (UUID format) |
| `questionText` | string | ✅ | The question being asked |
| `options` | array | ✅ | Exactly 2 answer options |
| `correctIndex` | integer | ✅ | Index of correct option (0 or 1) |
| `explanation` | string | ✅ | Why the correct answer is correct |

### Validation Rules

- `id`: Non-empty, valid UUID format
- `questionText`: Minimum 10 characters
- `options`: Exactly 2 elements, each non-empty
- `correctIndex`: Must be 0 or 1
- `explanation`: Minimum 10 characters

## C# Models

### Question.cs

```csharp
namespace GameTime.Quiz.Models;

public sealed record Question(
    string Id,
    string Category,
    string QuestionText,
    IReadOnlyList<string> Options,
    int CorrectIndex,
    string Explanation,
    string SourceUrl);
```

### QuestionChunk.cs

```csharp
namespace GameTime.Quiz.Models;

public sealed record QuestionChunk(
    string Id,
    DateTimeOffset GeneratedAt,
    string Category,
    string SourceUrl,
    IReadOnlyList<Question> Questions);
```

### Manifest.cs

```csharp
namespace GameTime.Quiz.Models;

public sealed record Manifest(
    string Version,
    DateTimeOffset GeneratedAt,
    int TotalQuestions,
    IReadOnlyList<CategorySummary> Categories,
    IReadOnlyList<ChunkReference> Chunks);

public sealed record CategorySummary(
    string Name,
    int Count);

public sealed record ChunkReference(
    string Id,
    string Category,
    int Count);
```

## JSON Serialization

### Configuration

```csharp
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    PropertyNameCaseInsensitive = true
};
```

### Fetching Data

```csharp
// Fetch manifest
var manifest = await httpClient.GetFromJsonAsync<Manifest>("manifest.json");

// Fetch chunk
var chunk = await httpClient.GetFromJsonAsync<QuestionChunk>($"chunks/{chunkId}.json");
```

## Caching Behavior

| Asset | Cache-Control | Strategy |
|-------|---------------|----------|
| `manifest.json` | `no-cache, must-revalidate` | Always check server |
| `chunks/*.json` | `immutable, max-age=31536000` | Never expires (content-addressed) |

### Why Content-Addressed Chunks?

Chunk filenames include a hash of their content:
- `csharp-basics-001-a1b2c3d4.json`

If content changes:
- New hash → new filename → new file
- Old file remains valid in cache forever
- Manifest points to new file

This enables aggressive caching while ensuring freshness.

## Error Handling

### Missing Manifest

```json
{
  "error": "Manifest not found",
  "status": 404
}
```

App should display: "Unable to load questions. Please try again."

### Missing Chunk

```json
{
  "error": "Chunk not found",
  "status": 404
}
```

App should skip chunk and try others, or show error if no chunks available.

### Invalid JSON

Handle `JsonException`:
- Log error details
- Display user-friendly message
- Suggest page refresh

## Versioning

### Schema Version

The `version` field in manifest enables future schema changes:

| Version | Changes |
|---------|---------|
| 1.0 | Initial schema |
| 1.1 | (Future) Add difficulty field |
| 2.0 | (Future) Breaking: Change options format |

### Backward Compatibility

App should handle older schema versions gracefully:

```csharp
if (manifest.Version.StartsWith("1."))
{
    // Handle v1.x schema
}
else
{
    throw new NotSupportedException($"Schema version {manifest.Version} not supported");
}
```
