# Data Model: Blazor Quiz App

**Feature**: 001-blazor-quiz-app  
**Date**: 2026-01-16  
**Status**: Complete

## Entity Overview

```
┌─────────────┐     ┌─────────────────┐     ┌─────────────┐
│  Manifest   │────▶│  ChunkReference │────▶│   Chunk     │
└─────────────┘     └─────────────────┘     └─────────────┘
                                                   │
                                                   ▼
                                            ┌─────────────┐
                                            │  Question   │
                                            └─────────────┘

┌─────────────┐     ┌─────────────┐
│  QuizState  │────▶│  Question   │ (runtime only)
└─────────────┘     └─────────────┘
```

## Entities

### Question

The core quiz question entity.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| Id | string | ✅ | Unique identifier (UUID format) |
| Category | string | ✅ | Topic category (e.g., "csharp-basics") |
| QuestionText | string | ✅ | The question being asked |
| Options | string[] | ✅ | Exactly 2 answer options |
| CorrectIndex | int | ✅ | Index of correct option (0 or 1) |
| Explanation | string | ✅ | Explanation of the correct answer |
| SourceUrl | string | ✅ | URL to source documentation |

**Validation Rules**:
- `Id` must be non-empty, valid UUID format
- `Options` must have exactly 2 elements
- `CorrectIndex` must be 0 or 1
- `QuestionText`, `Explanation`, `SourceUrl` must be non-empty

### QuestionChunk

Container for a batch of questions.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| Id | string | ✅ | Chunk identifier (matches filename without .json) |
| GeneratedAt | DateTimeOffset | ✅ | When this chunk was generated |
| Category | string | ✅ | Category of questions in this chunk |
| SourceUrl | string | ✅ | Source page this chunk was derived from |
| Questions | Question[] | ✅ | Array of questions in this chunk |

**Filename Convention**: `{category}-{sequence}-{hash8}.json`
- Example: `csharp-basics-001-a1b2c3d4.json`

### Manifest

Index of all available question chunks.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| Version | string | ✅ | Schema version (e.g., "1.0") |
| GeneratedAt | DateTimeOffset | ✅ | When manifest was last updated |
| TotalQuestions | int | ✅ | Total questions across all chunks |
| Categories | CategorySummary[] | ✅ | Summary per category |
| Chunks | ChunkReference[] | ✅ | References to all chunks |

### CategorySummary

Summary of questions per category.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| Name | string | ✅ | Category identifier |
| Count | int | ✅ | Number of questions in category |

### ChunkReference

Reference to a chunk file.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| Id | string | ✅ | Chunk identifier (filename without .json) |
| Category | string | ✅ | Category of the chunk |
| Count | int | ✅ | Number of questions in chunk |

### QuizState (Runtime Only)

In-memory game state, not persisted.

| Field | Type | Description |
|-------|------|-------------|
| Questions | IReadOnlyList<Question> | The 10 questions for current game |
| CurrentIndex | int | Current question index (0-9) |
| Answers | IReadOnlyList<int> | User's answers (indices) |
| IsComplete | bool | Computed: CurrentIndex >= Questions.Count |
| Score | int | Computed: Count of correct answers |
| CurrentQuestion | Question? | Computed: Questions[CurrentIndex] or null |

## Relationships

1. **Manifest → ChunkReference**: One-to-many. Manifest contains references to all chunks.
2. **ChunkReference → Chunk**: One-to-one. Each reference points to exactly one chunk file.
3. **Chunk → Question**: One-to-many. Each chunk contains multiple questions.
4. **QuizState → Question**: Runtime composition. QuizState holds a subset of loaded questions.

## State Transitions

### QuizState Lifecycle

```
[Not Started] ──StartGame(questions)──▶ [In Progress]
                                              │
                                        SubmitAnswer(index)
                                              │
                                              ▼
                                 ┌────────────────────────┐
                                 │ CurrentIndex < Count?  │
                                 └────────────────────────┘
                                        │           │
                                       Yes          No
                                        │           │
                                        ▼           ▼
                               [In Progress]   [Complete]
                                        │
                                        └──SubmitAnswer()──▶ ...
```

## Data Flow

```
┌──────────────────┐
│  Cloudflare CDN  │
│  (data repo)     │
└────────┬─────────┘
         │ HTTPS GET
         ▼
┌──────────────────┐     1. Fetch manifest.json
│   QuizService    │     2. Select random chunks
└────────┬─────────┘     3. Fetch chunk files
         │               4. Flatten to question list
         ▼
┌──────────────────┐     5. Shuffle and take 10
│   QuizState      │     6. Track answers
└────────┬─────────┘     7. Compute score
         │
         ▼
┌──────────────────┐
│   UI Components  │
└──────────────────┘
```

## Caching Behavior

| Asset | Location | TTL | Strategy |
|-------|----------|-----|----------|
| manifest.json | CDN + Service Worker | 0 | Network-first |
| chunks/*.json | CDN + Service Worker | 1 year | Cache-first (immutable) |
| QuizState | Memory only | Session | No persistence |
