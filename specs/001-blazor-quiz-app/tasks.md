# Tasks: Blazor Quiz App

**Input**: Design documents from `/specs/001-blazor-quiz-app/`
**Prerequisites**: plan.md ✅, spec.md ✅, data-model.md ✅, contracts/ ✅, quickstart.md ✅

**Tests**: Tests included per Constitution Principle V (Test-First Development - 80% coverage minimum)

**Organization**: Tasks grouped by user story for independent implementation and testing.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: User story label (US1, US2, US3, US4)
- File paths are relative to repository root

---

## Phase 1: Setup

**Purpose**: Project initialization, solution structure, build configuration

- [ ] T001 Create solution file `GameTime.Quiz.sln` at repository root
- [ ] T002 [P] Create `global.json` with .NET 10.0.100 SDK version
- [ ] T003 [P] Create `Directory.Build.props` with central build properties (nullable, implicit usings, LangVersion)
- [ ] T004 [P] Create `Directory.Build.targets` with clean/publish targets
- [ ] T005 [P] Create `Directory.Packages.props` with all package versions (FluentUI 5.0.0, xUnit 2.9.0, etc.)
- [ ] T006 [P] Create `nuget.config` with NuGet source
- [ ] T007 [P] Create `.editorconfig` with C# code style rules
- [ ] T008 [P] Create `.gitignore` for .NET projects (bin/, obj/, publish/)
- [ ] T009 [P] Create `build.ps1` Windows build script (Clean, Restore, Build, Test, Lint, Publish, Run targets)
- [ ] T010 [P] Create `build.sh` Linux build script for Cloudflare Pages deployment

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared library and core infrastructure that ALL user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

### Shared Models

- [ ] T011 Create `src/Shared/Shared.csproj` class library project
- [ ] T012 [P] Create `src/Shared/Models/Question.cs` record with Id, Category, QuestionText, Options, CorrectIndex, Explanation, SourceUrl
- [ ] T013 [P] Create `src/Shared/Models/QuestionChunk.cs` record with Id, GeneratedAt, Category, SourceUrl, Questions
- [ ] T014 [P] Create `src/Shared/Models/Manifest.cs` record with Version, GeneratedAt, TotalQuestions, Categories, Chunks
- [ ] T015 [P] Create `src/Shared/Models/CategorySummary.cs` record with Name, Count
- [ ] T016 [P] Create `src/Shared/Models/ChunkReference.cs` record with Id, Category, Count

### Shared Tests

- [ ] T017 Create `tests/Shared.Tests/Shared.Tests.csproj` test project
- [ ] T018 [P] Create `tests/Shared.Tests/Models/QuestionTests.cs` with validation tests (2 options, valid correctIndex)
- [ ] T019 [P] Create `tests/Shared.Tests/Models/ManifestTests.cs` with deserialization tests

### QuizApp Project Setup

- [ ] T020 Create `src/QuizApp/QuizApp.csproj` Blazor WASM standalone project with FluentUI reference
- [ ] T021 [P] Create `src/QuizApp/wwwroot/index.html` with Blazor bootstrap and Fluent UI CSS
- [ ] T022 [P] Create `src/QuizApp/wwwroot/css/app.css` with custom styles
- [ ] T023 [P] Create `src/QuizApp/wwwroot/_headers` with Cloudflare caching rules
- [ ] T024 [P] Create `src/QuizApp/_Imports.razor` with common using statements
- [ ] T025 Create `src/QuizApp/Program.cs` with FluentUI services, HttpClient configuration
- [ ] T026 [P] Create `src/QuizApp/App.razor` with Router, FluentToastProvider, FluentDialogProvider
- [ ] T027 [P] Create `src/QuizApp/Layout/MainLayout.razor` with FluentLayout structure

### QuizApp Tests Setup

- [ ] T028 Create `tests/QuizApp.Tests/QuizApp.Tests.csproj` test project with bUnit

**Checkpoint**: Foundation ready - all projects build, tests pass, user story implementation can begin

---

## Phase 3: User Story 1 - Play a Quiz Game (Priority: P1) 🎯 MVP

**Goal**: User can start a quiz, answer 10 questions, and see final score

**Independent Test**: Load app → Click "Start Quiz" → Answer 10 questions → See "X/10 correct"

### Tests for User Story 1

- [ ] T029 [P] [US1] Create `tests/QuizApp.Tests/Services/QuizServiceTests.cs` with GetRandomQuestionsAsync tests
- [ ] T030 [P] [US1] Create `tests/QuizApp.Tests/Services/QuizStateTests.cs` with StartGame, SubmitAnswer, Score computation tests

### Implementation for User Story 1

- [ ] T031 [US1] Create `src/QuizApp/Services/IQuizService.cs` interface with GetRandomQuestionsAsync method
- [ ] T032 [US1] Create `src/QuizApp/Services/QuizService.cs` implementing manifest fetch, chunk fetch, question randomization
- [ ] T033 [US1] Create `src/QuizApp/Services/QuizState.cs` scoped service with Questions, CurrentIndex, Answers, Score, events
- [ ] T034 [US1] Register QuizService and QuizState in `src/QuizApp/Program.cs`
- [ ] T035 [P] [US1] Create `src/QuizApp/Pages/Home.razor` with app title, description, "Start Quiz" button
- [ ] T036 [US1] Create `src/QuizApp/Pages/Quiz.razor` with question display, answer buttons, navigation logic
- [ ] T037 [US1] Create `src/QuizApp/Pages/Results.razor` with final score display (X/10 correct)
- [ ] T038 [P] [US1] Create `src/QuizApp/Components/ProgressIndicator.razor` showing "Question X of 10"

**Checkpoint**: User Story 1 complete - can play full quiz game with score display

---

## Phase 4: User Story 2 - View Question Feedback (Priority: P1)

**Goal**: User sees immediate correct/incorrect feedback with explanation after each answer

**Independent Test**: Answer any question → See success/error indicator + explanation text

### Tests for User Story 2

- [ ] T039 [P] [US2] Add feedback display tests to `tests/QuizApp.Tests/Components/QuestionCardTests.cs`

### Implementation for User Story 2

- [ ] T040 [US2] Create `src/QuizApp/Components/QuestionCard.razor` with answer buttons, feedback display, explanation
- [ ] T041 [US2] Update `src/QuizApp/Pages/Quiz.razor` to use QuestionCard component with feedback state
- [ ] T042 [US2] Add AnswerState enum (NotAnswered, Correct, Incorrect) to QuizState for tracking
- [ ] T043 [P] [US2] Add source URL link to QuestionCard for "Learn More" functionality

**Checkpoint**: User Story 2 complete - feedback and explanations shown after each answer

---

## Phase 5: User Story 3 - Start New Game After Completion (Priority: P2)

**Goal**: User can replay with fresh random questions after completing a quiz

**Independent Test**: Complete quiz → Click "Play Again" → Verify 10 new random questions

### Tests for User Story 3

- [ ] T044 [P] [US3] Add Reset tests to `tests/QuizApp.Tests/Services/QuizStateTests.cs`

### Implementation for User Story 3

- [ ] T045 [US3] Add Reset() method to QuizState that clears state and notifies listeners
- [ ] T046 [US3] Update `src/QuizApp/Pages/Results.razor` with "Play Again" and "Home" buttons
- [ ] T047 [US3] Wire "Play Again" to call QuizState.Reset() and navigate to Quiz page
- [ ] T048 [US3] Wire "Home" button to navigate to Home page

**Checkpoint**: User Story 3 complete - users can replay indefinitely

---

## Phase 6: User Story 4 - Load App Offline (Priority: P3)

**Goal**: App works offline after initial load with cached questions

**Independent Test**: Load app online → Disconnect → Reload → Verify app functions

### Tests for User Story 4

- [ ] T049 [P] [US4] Add service worker registration tests (manual verification documented)

### Implementation for User Story 4

- [ ] T050 [US4] Create `src/QuizApp/wwwroot/service-worker.js` with Workbox-style caching
- [ ] T051 [US4] Create `src/QuizApp/wwwroot/service-worker.published.js` for production build
- [ ] T052 [US4] Update `src/QuizApp/wwwroot/index.html` to register service worker
- [ ] T053 [US4] Configure service worker to cache _framework/ assets (immutable)
- [ ] T054 [US4] Configure service worker to cache question chunks on first fetch
- [ ] T055 [US4] Add manifest.json network-first strategy with fallback to cache

**Checkpoint**: User Story 4 complete - app fully functional offline

---

## Phase 7: QuestionGenerator (Data Repo Support)

**Purpose**: Console app for generating questions (used by data repo GitHub Actions)

- [ ] T056 Create `src/QuestionGenerator/QuestionGenerator.csproj` console app with AngleSharp
- [ ] T057 [P] Create `src/QuestionGenerator/Services/IPageScraper.cs` interface
- [ ] T058 [P] Create `src/QuestionGenerator/Services/PageScraper.cs` with AngleSharp HTML scraping
- [ ] T059 [P] Create `src/QuestionGenerator/Services/IContentHasher.cs` interface
- [ ] T060 [P] Create `src/QuestionGenerator/Services/ContentHasher.cs` with SHA256 hashing
- [ ] T061 [P] Create `src/QuestionGenerator/Services/IGitHubModelsClient.cs` interface
- [ ] T062 [P] Create `src/QuestionGenerator/Services/GitHubModelsClient.cs` for LLM API calls
- [ ] T063 [P] Create `src/QuestionGenerator/Services/IQuestionManager.cs` interface
- [ ] T064 [P] Create `src/QuestionGenerator/Services/QuestionManager.cs` for chunk management
- [ ] T065 Create `src/QuestionGenerator/Program.cs` with CLI argument parsing and orchestration

### QuestionGenerator Tests

- [ ] T066 Create `tests/QuestionGenerator.Tests/QuestionGenerator.Tests.csproj` test project
- [ ] T067 [P] Create `tests/QuestionGenerator.Tests/Services/ContentHasherTests.cs`
- [ ] T068 [P] Create `tests/QuestionGenerator.Tests/Services/PageScraperTests.cs` with mock HTML
- [ ] T069 [P] Create `tests/QuestionGenerator.Tests/Services/QuestionManagerTests.cs`

**Checkpoint**: QuestionGenerator ready for use in data repo workflow

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: CI/CD, documentation, final validation

- [ ] T070 [P] Create `.github/workflows/validate-pr.yml` with build, test, lint jobs
- [ ] T071 [P] Add README badges for build status
- [ ] T072 [P] Create `LICENSE` file with MIT license
- [ ] T073 Run `dotnet format --verify-no-changes` and fix any issues
- [ ] T074 Run full test suite and verify 80%+ coverage
- [ ] T075 Validate against quickstart.md - ensure all steps work
- [ ] T076 Test Cloudflare Pages deployment with build.sh

---

## Post-Implementation Workflow

After completing implementation tasks, follow these steps to ensure quality:

### Step 1: Run Linting

```powershell
# Check code formatting (CI will run this)
dotnet format --verify-no-changes

# Auto-fix any formatting issues
dotnet format

# If using build script
.\build.ps1 -Target Lint
```

**What to fix:**
- Indentation (4 spaces)
- Trailing whitespace
- Using statement order
- Naming convention violations
- Unused imports

### Step 2: Run Tests

```powershell
# Run all tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific project tests
dotnet test tests/Shared.Tests
dotnet test tests/QuizApp.Tests

# Using build script
.\build.ps1 -Target Test

# Verify coverage meets 80% threshold
# Coverage reports: tests/*/TestResults/*/coverage.cobertura.xml
```

**Test Requirements (per Constitution):**
- Minimum 80% line coverage
- 100% coverage on critical paths (QuizService, QuizState)
- All tests must pass before committing

### Step 3: Update Documentation

After implementation changes, update relevant docs:

| Change Type | Docs to Update |
|-------------|----------------|
| New component | `docs/components.md` |
| New/changed API | `docs/data-contracts.md` |
| Build/setup changes | `docs/development.md` |
| Deployment changes | `docs/deployment.md` |
| Architecture changes | `docs/architecture.md` |

**XML Documentation:**
Add XML docs to all public APIs:
```csharp
/// <summary>
/// Fetches random questions for quiz gameplay.
/// </summary>
/// <param name="count">Number of questions (default: 10).</param>
/// <returns>Randomized question list.</returns>
public async Task<IReadOnlyList<Question>> GetRandomQuestionsAsync(int count = 10)
```

---

## Post-Implementation Tasks

Add these tasks after completing each phase:

### After Phase 3 (US1: Play Quiz)

- [ ] T077 Run `dotnet format` and fix all formatting issues in src/QuizApp/
- [ ] T078 Verify QuizService tests cover edge cases (empty manifest, network failure)
- [ ] T079 Verify QuizState tests cover all state transitions
- [ ] T080 Update `docs/components.md` with Home, Quiz, Results page docs

### After Phase 4 (US2: Feedback)

- [ ] T081 Run `dotnet format --verify-no-changes` for clean check
- [ ] T082 Add QuestionCard component tests for feedback states
- [ ] T083 Update `docs/components.md` with QuestionCard component docs

### After Phase 5 (US3: Replay)

- [ ] T084 Run lint check before committing
- [ ] T085 Add integration test: full game → replay → verify new questions
- [ ] T086 Update quickstart.md if replay behavior changes

### After Phase 6 (US4: Offline)

- [ ] T087 Manual test: load app → disconnect → reload → verify works
- [ ] T088 Document offline testing in `docs/development.md`

### After Phase 7 (QuestionGenerator)

- [ ] T089 Run lint on src/QuestionGenerator/
- [ ] T090 Verify 80%+ coverage on QuestionGenerator
- [ ] T091 Create `docs/question-generator.md` with usage docs

### Final Validation (Before PR)

- [ ] T092 Run full lint check: `dotnet format --verify-no-changes`
- [ ] T093 Run full test suite: `dotnet test`
- [ ] T094 Generate coverage report and verify 80%+ threshold
- [ ] T095 Review all docs for accuracy
- [ ] T096 Run `.\build.ps1` full build with all targets

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)
    │
    ▼
Phase 2 (Foundational) ─── BLOCKS ALL USER STORIES
    │
    ├─────────────┬─────────────┬─────────────┐
    ▼             ▼             ▼             ▼
Phase 3       Phase 4       Phase 5       Phase 6
(US1: Play)   (US2: Feed)   (US3: Replay) (US4: Offline)
    │             │             │             │
    └─────────────┴─────────────┴─────────────┘
                        │
                        ▼
                    Phase 7
              (QuestionGenerator)
                        │
                        ▼
                    Phase 8
                    (Polish)
```

### User Story Dependencies

| Story | Depends On | Can Parallel With |
|-------|------------|-------------------|
| US1 (Play) | Phase 2 only | US2, US3, US4 (after Phase 2) |
| US2 (Feedback) | Phase 2, partially US1 (Quiz.razor) | US3, US4 |
| US3 (Replay) | Phase 2, US1 (Results.razor) | US4 |
| US4 (Offline) | Phase 2 | US1, US2, US3 |

### Parallel Opportunities per Phase

**Phase 1**: T002-T010 all parallel (different files)
**Phase 2**: T012-T016 parallel, T018-T019 parallel, T021-T024 parallel
**Phase 3**: T029-T030 parallel, T035+T038 parallel with T036
**Phase 4**: T039+T043 parallel with T040-T042
**Phase 7**: T057-T064 all parallel (different files)
**Phase 8**: T070-T072 parallel

---

## Parallel Example: Phase 2 Foundational

```bash
# All model files can be created in parallel:
T012: src/Shared/Models/Question.cs
T013: src/Shared/Models/QuestionChunk.cs
T014: src/Shared/Models/Manifest.cs
T015: src/Shared/Models/CategorySummary.cs
T016: src/Shared/Models/ChunkReference.cs

# After models complete, tests can run in parallel:
T018: tests/Shared.Tests/Models/QuestionTests.cs
T019: tests/Shared.Tests/Models/ManifestTests.cs
```

---

## Implementation Strategy

### MVP First (User Stories 1 + 2)

1. Complete Phase 1: Setup (T001-T010)
2. Complete Phase 2: Foundational (T011-T028)
3. Complete Phase 3: User Story 1 - Play Quiz (T029-T038)
4. Complete Phase 4: User Story 2 - Feedback (T039-T043)
5. **STOP and VALIDATE**: Full quiz with feedback works
6. Deploy to Cloudflare Pages for MVP demo

### Full Feature Set

7. Complete Phase 5: User Story 3 - Replay (T044-T048)
8. Complete Phase 6: User Story 4 - Offline (T049-T055)
9. Complete Phase 7: QuestionGenerator (T056-T069)
10. Complete Phase 8: Polish (T070-T076)

---

## Task Summary

| Phase | Tasks | Description |
|-------|-------|-------------|
| 1. Setup | T001-T010 (10) | Project structure, build config |
| 2. Foundational | T011-T028 (18) | Shared models, QuizApp scaffolding |
| 3. US1: Play | T029-T038 (10) | Core quiz gameplay |
| 4. US2: Feedback | T039-T043 (5) | Answer feedback with explanations |
| 5. US3: Replay | T044-T048 (5) | Play again functionality |
| 6. US4: Offline | T049-T055 (7) | Service worker caching |
| 7. QuestionGen | T056-T069 (14) | Data generation console app |
| 8. Polish | T070-T076 (7) | CI/CD, docs, validation |
| Post-Impl | T077-T096 (20) | Lint, tests, docs after each phase |
| **Total** | **96 tasks** | |

---

## Notes

- Tests are included per Constitution Principle V (80% coverage minimum)
- Each user story is independently testable after completion
- QuestionGenerator (Phase 7) is for the data repo workflow, not required for MVP
- Phases 3-6 (user stories) can be parallelized with multiple developers
- Commit after each task or logical group
- Run tests frequently to catch regressions early
- **Post-implementation tasks (T077-T096)**: Run lint, verify tests, update docs after each phase
- Always run `dotnet format` before creating a PR
