# SwiftUI Rules

Load this when working on any iOS/macOS app written in SwiftUI.

## Architecture

**MVVM**, with use cases for non-trivial flows:

```
App/
  Features/
    Login/
      LoginView.swift
      LoginViewModel.swift
      LoginUseCase.swift
  Models/             # Codable structs, domain entities
  Services/           # repositories, networking, persistence
  Core/               # theme, navigation, utilities
  Resources/          # assets, strings
```

## State ownership

- `@StateObject` — view model **created** by this view. Lives as long as the view.
- `@ObservedObject` — view model **injected** from parent. Don't create here.
- `@EnvironmentObject` — app-wide state (current user, theme, settings).
- `@State` — local view-only state (toggle open/closed, animation values).
- `@Binding` — two-way binding to parent state.

## Always

- **`async/await`** for new asynchronous code. Combine only when interacting with code that already exposes publishers.
- **`NavigationStack` + `NavigationPath`** (iOS 16+) for navigation. Programmatic, deep-linkable, supports `popToRoot`.
- **`Codable` structs** for models. No `NSObject` subclasses.
- **`@MainActor`** annotation on ViewModels that update UI.
- **SF Symbols** (`Image(systemName: "star.fill")`) for icons. No PNG/SVG for system iconography.
- **Localized strings**: `String(localized: "key", defaultValue: "...")` everywhere. No hardcoded user-facing text.
- **`Form` + `Section`** for settings/profile screens. Not custom layouts.
- **Color/Font/Spacing tokens** in an enum or struct. No hex literals or magic numbers in views.

## Never

- ❌ Force-unwrap (`!`) outside test code.
- ❌ Force-cast (`as!`) — use `as?` with handling.
- ❌ `GeometryReader` unless you need measured dimensions. Use frame/layout modifiers first.
- ❌ Heavy work in view `body` — `body` runs many times per second.
- ❌ Singletons that own mutable state. Use environment objects.
- ❌ `DispatchQueue.main.async` in new code. Use `await MainActor.run` or `@MainActor`.
- ❌ `print` for logs — use `os_log` / `Logger` with categories.

## ViewModels

```swift
@MainActor
final class LoginViewModel: ObservableObject {
    @Published private(set) var state: ViewState = .idle
    
    private let useCase: LoginUseCase
    init(useCase: LoginUseCase) { self.useCase = useCase }
    
    func submit(email: String, password: String) async {
        state = .loading
        do {
            try await useCase.execute(email: email, password: password)
            state = .success
        } catch {
            state = .failure(error)
        }
    }
}

enum ViewState {
    case idle, loading, success
    case failure(Error)
}
```

## View structure

- Keep `body` short. Extract subviews as `private struct` types in the same file.
- Computed view properties (`var header: some View { ... }`) over functions returning views — better for compiler.
- Don't pass entire view models to subviews. Pass the minimum data they need.

## Networking

- `URLSession.shared.data(for:)` via async/await. No completion handlers.
- Define `APIClient` protocol; concrete `URLSessionAPIClient` implementation.
- All requests return `Decodable` types or throw.

## Persistence

- **SwiftData** for new projects on iOS 17+.
- **Core Data** only if you must support older OS versions.
- **`@AppStorage`** for trivial preferences (selected theme, last opened tab).
- **Keychain** for secrets (tokens, passwords) — never UserDefaults.

## Testing

- **XCTest** for unit tests on ViewModels and use cases.
- **`@MainActor` test methods** when testing ViewModels.
- **Snapshot tests** for views (via `swift-snapshot-testing` package).
- Mock services via protocol injection — no swizzling.
