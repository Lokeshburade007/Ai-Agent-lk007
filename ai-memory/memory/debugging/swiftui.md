# SwiftUI Debugging Playbook

## View doesn't update when @State changes

- `@State` only triggers updates within the view that owns it. If you pass it down, use `@Binding`.
- The property is on a `class`, not a `struct`. SwiftUI tracks `@State` on value types. For reference types, use `@StateObject` / `@ObservedObject` and make the class `ObservableObject` with `@Published` fields.
- You mutated a property inside an object that isn't `@Published`.

## "ObservableObject was created on a background thread"

You created the view model off the main thread.

Fix: annotate the VM `@MainActor`, and create it on the main thread.

```swift
@MainActor
final class LoginViewModel: ObservableObject {
    @Published var state: ViewState = .idle
}
```

## "Modifying state during view update is not allowed"

You called `setState` (changed a `@Published` property) inside `body` or inside a synchronous computation that runs during render.

Fix: defer to next runloop or move into an event handler:
```swift
.onAppear {
    Task { await viewModel.load() }
}
```

## View body called too often — performance issue

- `body` runs frequently. Heavy work belongs elsewhere.
- Use `let` constants captured outside `body` for static data.
- Profile with Instruments → Time Profiler.
- Wrap expensive subviews in `EquatableView` if you can implement equality cheaply.

## "Cannot infer contextual base in reference to member" / compile times explode

Long view bodies cause the type-checker to time out. Break into smaller subviews:

```swift
struct HomeView: View {
    var body: some View {
        VStack {
            header
            content
            footer
        }
    }
    
    private var header: some View { ... }
    private var content: some View { ... }
    private var footer: some View { ... }
}
```

## NavigationStack pops unexpectedly

- `@State var path = NavigationPath()` was lost because the parent view recreated.
- The destination data type doesn't match `navigationDestination(for:)` expects.
- Sheet/cover dismiss can pop nav stack inside it — by design, sheets are separate stacks.

## TextField cursor jumps to end / value resets

- Two-way binding to a computed property that doesn't preserve identity.
- `.onChange(of:)` modifies the value, causing re-binding.
- Use `@FocusState` and explicit field identification for multi-field forms.

## List doesn't show new items added to @Published array

- The array element type doesn't conform to `Identifiable`, and SwiftUI is keying by index. Add an `id`:
```swift
List(items, id: \.uuid) { item in Text(item.name) }
```
- The array reference is the same (you mutated in place on an `NSMutableArray` or class-based collection). Replace with a new array: `items = items + [newItem]`.

## Sheet doesn't dismiss / shows wrong content

- Two `.sheet` modifiers on the same view — only one can be active. Combine into one with `enum`-based state.
- Sheet content captured stale state via closure. Pass current state through `@State` / `@Binding`.

## "Publishing changes from background threads is not allowed"

You updated a `@Published` property off the main thread. The runtime crashes in debug, undefined in release.

Fix: `await MainActor.run { self.foo = bar }`, or annotate the publisher's class `@MainActor`.

## Memory leak — view holds reference to closure capturing self

```swift
.onAppear {
    Task {
        await self.load()   // strong reference cycle if Task is long-lived
    }
}
```

Usually fine for short tasks, but for long-running subscriptions/streams use `[weak self]`:
```swift
.onAppear {
    Task { [weak self] in
        guard let self else { return }
        await self.load()
    }
}
```

## Codable decode fails with cryptic error

The error reports a key path. Common causes:
- Server sent `null` for a non-optional field. Make it `Optional`.
- Date format mismatch. Configure `JSONDecoder.dateDecodingStrategy = .iso8601` (or custom).
- Snake case vs camelCase. Set `keyDecodingStrategy = .convertFromSnakeCase`.

Print the raw JSON when decoding fails to see what you actually got:
```swift
print(String(data: data, encoding: .utf8) ?? "<not utf8>")
```

## Preview crashes / shows nothing

- Preview uses a different code path; production-only code (Firebase init, etc.) may misbehave.
- Wrap the preview view with required environment objects.
- Use `#Preview` (Xcode 15+) macro; falls back gracefully on errors.
- If the preview is slow, mark the view `previewLayout` size-fitting and remove network calls (inject a fake service).

## Image not loading from URL

- `AsyncImage` requires iOS 15+. Lower versions need a third-party loader (Kingfisher, SDWebImage).
- HTTP (not HTTPS) is blocked by App Transport Security on iOS. Use HTTPS or exception domains.
- Server doesn't send proper `Content-Type`. SwiftUI is strict.
