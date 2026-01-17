# Components Reference

This document describes the Blazor components and services in the QuizApp.

## Pages

### Home.razor

**Route**: `/`

Landing page with app introduction and start button.

```razor
@page "/"
@inject NavigationManager Navigation
@inject IQuizService QuizService

<PageTitle>GameTime .NET Quiz</PageTitle>

<FluentStack Orientation="Orientation.Vertical" VerticalGap="24">
    <FluentLabel Typo="Typography.H1">
        🎮 GameTime .NET Quiz
    </FluentLabel>
    
    <FluentLabel Typo="Typography.Body1">
        Test your .NET knowledge with 10 random questions!
    </FluentLabel>
    
    <FluentButton Appearance="Appearance.Accent" OnClick="StartQuiz">
        Start Quiz
    </FluentButton>
</FluentStack>

@code {
    private async Task StartQuiz()
    {
        // Pre-load questions before navigating
        await QuizService.GetRandomQuestionsAsync();
        Navigation.NavigateTo("/quiz");
    }
}
```

### Quiz.razor

**Route**: `/quiz`

Main quiz gameplay page showing one question at a time.

```razor
@page "/quiz"
@inject QuizState State
@inject NavigationManager Navigation
@implements IDisposable

<PageTitle>Question @(State.CurrentIndex + 1) - Quiz</PageTitle>

<FluentStack Orientation="Orientation.Vertical" VerticalGap="16">
    <ProgressIndicator Current="@(State.CurrentIndex + 1)" Total="10" />
    
    @if (State.CurrentQuestion is not null)
    {
        <QuestionCard 
            Question="@State.CurrentQuestion"
            OnAnswer="HandleAnswer"
            ShowFeedback="@_showFeedback"
            SelectedIndex="@_selectedIndex" />
    }
    
    @if (_showFeedback)
    {
        <FluentButton OnClick="NextQuestion">
            @(_isLastQuestion ? "See Results" : "Next Question")
        </FluentButton>
    }
</FluentStack>

@code {
    private bool _showFeedback;
    private int _selectedIndex = -1;
    private bool _isLastQuestion => State.CurrentIndex >= 9;

    protected override void OnInitialized()
    {
        State.OnStateChanged += StateHasChanged;
    }

    private void HandleAnswer(int index)
    {
        _selectedIndex = index;
        State.SubmitAnswer(index);
        _showFeedback = true;
    }

    private void NextQuestion()
    {
        _showFeedback = false;
        _selectedIndex = -1;
        
        if (State.IsComplete)
        {
            Navigation.NavigateTo("/results");
        }
    }

    public void Dispose()
    {
        State.OnStateChanged -= StateHasChanged;
    }
}
```

### Results.razor

**Route**: `/results`

Final score display with replay options.

```razor
@page "/results"
@inject QuizState State
@inject NavigationManager Navigation

<PageTitle>Results - Quiz</PageTitle>

<FluentCard>
    <FluentStack Orientation="Orientation.Vertical" VerticalGap="24" HorizontalAlignment="HorizontalAlignment.Center">
        <FluentLabel Typo="Typography.H2">
            Quiz Complete!
        </FluentLabel>
        
        <FluentLabel Typo="Typography.Display">
            @State.Score / 10
        </FluentLabel>
        
        <FluentLabel Typo="Typography.Body1">
            @GetScoreMessage()
        </FluentLabel>
        
        <FluentStack Orientation="Orientation.Horizontal" HorizontalGap="16">
            <FluentButton Appearance="Appearance.Accent" OnClick="PlayAgain">
                Play Again
            </FluentButton>
            <FluentButton OnClick="GoHome">
                Home
            </FluentButton>
        </FluentStack>
    </FluentStack>
</FluentCard>

@code {
    private string GetScoreMessage() => State.Score switch
    {
        10 => "Perfect! You're a .NET expert! 🏆",
        >= 8 => "Excellent work! 🌟",
        >= 6 => "Good job! Keep learning! 📚",
        >= 4 => "Not bad! Room for improvement. 💪",
        _ => "Keep practicing! You'll get better! 🎯"
    };

    private async Task PlayAgain()
    {
        State.Reset();
        Navigation.NavigateTo("/quiz");
    }

    private void GoHome()
    {
        State.Reset();
        Navigation.NavigateTo("/");
    }
}
```

## Components

### QuestionCard.razor

Displays a question with answer options and feedback.

```razor
<FluentCard>
    <FluentStack Orientation="Orientation.Vertical" VerticalGap="16">
        <FluentLabel Typo="Typography.H4">
            @Question.QuestionText
        </FluentLabel>
        
        @foreach (var (option, index) in Question.Options.Select((o, i) => (o, i)))
        {
            <FluentButton 
                Style="width: 100%"
                Appearance="@GetAppearance(index)"
                Disabled="@ShowFeedback"
                OnClick="@(() => OnAnswer.InvokeAsync(index))">
                @option
            </FluentButton>
        }
        
        @if (ShowFeedback)
        {
            <FluentCard Style="@GetFeedbackStyle()">
                <FluentStack Orientation="Orientation.Vertical" VerticalGap="8">
                    <FluentLabel Typo="Typography.Body1Strong">
                        @(IsCorrect ? "✓ Correct!" : "✗ Incorrect")
                    </FluentLabel>
                    <FluentLabel Typo="Typography.Body2">
                        @Question.Explanation
                    </FluentLabel>
                    <FluentAnchor Href="@Question.SourceUrl" Target="_blank">
                        Learn More →
                    </FluentAnchor>
                </FluentStack>
            </FluentCard>
        }
    </FluentStack>
</FluentCard>

@code {
    [Parameter, EditorRequired]
    public Question Question { get; set; } = null!;
    
    [Parameter]
    public EventCallback<int> OnAnswer { get; set; }
    
    [Parameter]
    public bool ShowFeedback { get; set; }
    
    [Parameter]
    public int SelectedIndex { get; set; } = -1;

    private bool IsCorrect => SelectedIndex == Question.CorrectIndex;

    private Appearance GetAppearance(int index)
    {
        if (!ShowFeedback)
            return Appearance.Neutral;
            
        if (index == Question.CorrectIndex)
            return Appearance.Accent; // Green/success
            
        if (index == SelectedIndex)
            return Appearance.Neutral; // Red/error styling via CSS
            
        return Appearance.Neutral;
    }

    private string GetFeedbackStyle() => IsCorrect
        ? "background-color: var(--success-background);"
        : "background-color: var(--error-background);";
}
```

### ProgressIndicator.razor

Shows current question progress.

```razor
<FluentStack Orientation="Orientation.Horizontal" HorizontalGap="8" VerticalAlignment="VerticalAlignment.Center">
    <FluentLabel Typo="Typography.Body2">
        Question @Current of @Total
    </FluentLabel>
    <FluentProgress Value="@Current" Max="@Total" />
</FluentStack>

@code {
    [Parameter]
    public int Current { get; set; }
    
    [Parameter]
    public int Total { get; set; } = 10;
}
```

## Services

### IQuizService

Interface for question fetching.

```csharp
public interface IQuizService
{
    Task<IReadOnlyList<Question>> GetRandomQuestionsAsync(int count = 10);
}
```

### QuizService

Fetches and caches questions from the data endpoint.

```csharp
public sealed class QuizService : IQuizService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<QuizService> _logger;
    private List<Question>? _cachedQuestions;

    public QuizService(IHttpClientFactory factory, ILogger<QuizService> logger)
    {
        _httpClient = factory.CreateClient("DataApi");
        _logger = logger;
    }

    public async Task<IReadOnlyList<Question>> GetRandomQuestionsAsync(int count = 10)
    {
        var allQuestions = await GetAllQuestionsAsync();
        return Shuffle(allQuestions).Take(count).ToList();
    }

    private async Task<IReadOnlyList<Question>> GetAllQuestionsAsync()
    {
        if (_cachedQuestions is not null)
            return _cachedQuestions;

        var manifest = await _httpClient.GetFromJsonAsync<Manifest>("manifest.json")
            ?? throw new InvalidOperationException("Failed to load manifest");

        var tasks = manifest.Chunks.Select(c => 
            _httpClient.GetFromJsonAsync<QuestionChunk>($"chunks/{c.Id}.json"));
        
        var chunks = await Task.WhenAll(tasks);
        
        _cachedQuestions = chunks
            .Where(c => c is not null)
            .SelectMany(c => c!.Questions)
            .ToList();

        return _cachedQuestions;
    }

    private static IReadOnlyList<T> Shuffle<T>(IEnumerable<T> source)
    {
        var list = source.ToList();
        var rng = Random.Shared;
        for (int i = list.Count - 1; i > 0; i--)
        {
            int j = rng.Next(i + 1);
            (list[i], list[j]) = (list[j], list[i]);
        }
        return list;
    }
}
```

### QuizState

Manages game state within a session.

```csharp
public sealed class QuizState
{
    public IReadOnlyList<Question> Questions { get; private set; } = [];
    public int CurrentIndex { get; private set; }
    public IReadOnlyList<int> Answers { get; private set; } = [];

    public Question? CurrentQuestion => 
        CurrentIndex < Questions.Count ? Questions[CurrentIndex] : null;

    public bool IsComplete => CurrentIndex >= Questions.Count;

    public int Score => Questions
        .Take(Answers.Count)
        .Zip(Answers)
        .Count(x => x.First.CorrectIndex == x.Second);

    public event Action? OnStateChanged;

    public void StartGame(IReadOnlyList<Question> questions)
    {
        Questions = questions;
        CurrentIndex = 0;
        Answers = [];
        OnStateChanged?.Invoke();
    }

    public void SubmitAnswer(int answerIndex)
    {
        if (IsComplete) return;
        
        Answers = [..Answers, answerIndex];
        CurrentIndex++;
        OnStateChanged?.Invoke();
    }

    public void Reset()
    {
        Questions = [];
        CurrentIndex = 0;
        Answers = [];
        OnStateChanged?.Invoke();
    }
}
```

## Layout

### MainLayout.razor

App shell with Fluent UI structure.

```razor
@inherits LayoutComponentBase

<FluentLayout>
    <FluentHeader>
        <FluentAnchor Href="/" Appearance="Appearance.Stealth">
            🎮 GameTime .NET Quiz
        </FluentAnchor>
    </FluentHeader>
    
    <FluentBodyContent>
        <FluentStack Orientation="Orientation.Vertical" 
                     Style="padding: 24px; max-width: 600px; margin: 0 auto;">
            @Body
        </FluentStack>
    </FluentBodyContent>
    
    <FluentFooter>
        <FluentLabel Typo="Typography.Caption">
            Built with Blazor & Fluent UI | 
            <FluentAnchor Href="https://github.com/amitjoshi-ms/gametime-dotnet-quiz" Target="_blank">
                GitHub
            </FluentAnchor>
        </FluentLabel>
    </FluentFooter>
</FluentLayout>
```

## App Configuration

### Program.cs

```csharp
using GameTime.Quiz.Services;
using Microsoft.FluentUI.AspNetCore.Components;

var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

// Fluent UI
builder.Services.AddFluentUIComponents();

// HTTP Client for data endpoint
builder.Services.AddHttpClient("DataApi", client =>
{
    client.BaseAddress = new Uri("https://gametime-dotnet-quiz-data.pages.dev/");
});

// Services
builder.Services.AddScoped<IQuizService, QuizService>();
builder.Services.AddScoped<QuizState>();

await builder.Build().RunAsync();
```

### App.razor

```razor
<FluentToastProvider />
<FluentDialogProvider />
<FluentTooltipProvider />

<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
        <FocusOnNavigate RouteData="@routeData" Selector="h1" />
    </Found>
    <NotFound>
        <PageTitle>Not found</PageTitle>
        <LayoutView Layout="@typeof(MainLayout)">
            <FluentLabel Typo="Typography.H2">Page not found</FluentLabel>
            <FluentAnchor Href="/">Go Home</FluentAnchor>
        </LayoutView>
    </NotFound>
</Router>
```

## Fluent UI Components Used

| Component | Usage |
|-----------|-------|
| `FluentLayout` | App shell structure |
| `FluentHeader` | Top navigation bar |
| `FluentFooter` | Bottom footer |
| `FluentStack` | Flex layout with gaps |
| `FluentCard` | Container with elevation |
| `FluentButton` | Primary actions |
| `FluentLabel` | Text with typography |
| `FluentProgress` | Progress indicator |
| `FluentAnchor` | Navigation links |
| `FluentToastProvider` | Toast notifications |
| `FluentDialogProvider` | Modal dialogs |

See [Fluent UI Blazor documentation](https://www.fluentui-blazor.net/) for full component reference.
