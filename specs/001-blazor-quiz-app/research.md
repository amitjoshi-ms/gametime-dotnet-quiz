# Research: Blazor Quiz App

**Feature**: 001-blazor-quiz-app  
**Date**: 2026-01-16  
**Status**: Complete (minimal research - tech stack locked in constitution)

## Research Summary

Most technology decisions were made during project initialization and are locked in the constitution. This research documents those decisions and adds implementation-specific details.

## Decision Log

### 1. Frontend Framework

**Decision**: Blazor WebAssembly (standalone)  
**Rationale**: 
- Constitution locks Blazor WASM as the framework
- .NET learning value for a .NET quiz app
- No server-side rendering needed for static hosting
**Alternatives Considered**: 
- Vanilla JS (rejected: no .NET learning value)
- React/Vue (rejected: not .NET ecosystem)

### 2. UI Component Library

**Decision**: Microsoft.FluentUI.AspNetCore.Components v5.0.0  
**Rationale**:
- Constitution locks Fluent UI
- Microsoft design system consistency
- Accessibility built-in (WCAG 2.1 AA)
- Modern, professional appearance
**Alternatives Considered**:
- MudBlazor (rejected: not Microsoft ecosystem)
- Radzen (rejected: commercial focus)

### 3. Data Fetching Strategy

**Decision**: HttpClient with System.Text.Json  
**Rationale**:
- Native to .NET, no additional dependencies
- Efficient JSON deserialization
- Works well with Blazor WASM
**Implementation Notes**:
- Use `IHttpClientFactory` for proper lifecycle management
- Configure base address to data repo Cloudflare Pages URL
- Handle CORS (data repo has permissive headers)

### 4. State Management

**Decision**: Scoped service (`QuizState`) with events  
**Rationale**:
- Simple for single-page quiz flow
- No need for complex state libraries (Fluxor, etc.)
- State resets naturally on page reload
**Implementation Notes**:
- `QuizState` holds: Questions, CurrentIndex, Answers, computed Score
- Raises `OnStateChanged` event for component updates
- Registered as scoped service in DI

### 5. Offline Support

**Decision**: Service Worker with Workbox-style caching  
**Rationale**:
- Constitution requires offline capability
- Blazor WASM has built-in PWA template support
- Questions are immutable, perfect for caching
**Implementation Notes**:
- Cache `_framework/*` files (immutable)
- Cache question chunks on first fetch
- Manifest uses network-first strategy

### 6. Randomization Strategy

**Decision**: Client-side Fisher-Yates shuffle  
**Rationale**:
- No server needed
- Truly random per game session
- Simple implementation
**Implementation Notes**:
```csharp
public static IReadOnlyList<T> Shuffle<T>(IEnumerable<T> source)
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
```

### 7. Error Handling

**Decision**: Try-catch with toast notifications  
**Rationale**:
- Fluent UI provides toast service
- User-friendly error messages
- Graceful degradation
**Implementation Notes**:
- Catch `HttpRequestException` for network errors
- Catch `JsonException` for malformed data
- Display actionable messages ("Try again" button)

## Open Questions

None - all decisions finalized through constitution and prior discussion.

## References

- [Fluent UI Blazor Documentation](https://www.fluentui-blazor.net/)
- [Blazor WASM PWA Template](https://learn.microsoft.com/en-us/aspnet/core/blazor/progressive-web-app)
- [Cloudflare Pages Headers](https://developers.cloudflare.com/pages/configuration/headers/)
