# Flutter Rules

Load this when working on any Flutter project.

## Architecture

**Clean architecture** with three layers. Dependencies point inward only:

```
lib/
  presentation/         # widgets, view models, navigation
  domain/               # entities, use cases, repository interfaces (PURE Dart)
  data/                 # repository impls, data sources (API, DB)
  core/                 # shared utilities, theme, errors, constants
```

`domain/` has **no Flutter or http imports**. It's pure Dart that could compile on a backend.

## State management

- **Riverpod** (latest, code-generation flavor with `@riverpod`).
- One `Notifier` or `AsyncNotifier` per screen / per feature.
- **No `setState`** in production widgets. Use `ConsumerWidget` or `ConsumerStatefulWidget` with `ref.watch`.
- Provider names match what they expose: `userProfileProvider`, not `userProvider`.

## MVVM pattern

- **View** (Widget): pure UI, reads ViewModel state via `ref.watch`, sends events via `ref.read(...).method()`.
- **ViewModel** (Notifier): holds state, exposes methods, calls use cases.
- **Use case** (in `domain/`): one verb, no state. e.g. `LoginUseCase`, `FetchUserUseCase`.
- **Repository** (interface in `domain/`, impl in `data/`): abstracts data sources.

## Always

- **Immutable models** with `freezed`. No mutable classes for state/DTOs.
- **`const` constructors** everywhere they fit. Free performance.
- **`Either<Failure, T>` or sealed result types** as repository return values. Never throw across layer boundaries.
- **`go_router`** for navigation. Declarative routes, deep-linkable.
- **`dio`** for HTTP. Configure interceptors for auth, logging, error mapping in one place.
- **`flutter_form_builder` + validators** for forms.
- **Theme tokens** via `Theme.of(context)`. Never hardcoded colors / font sizes in widgets.
- **Localization with `flutter_localizations` + ARB files** from day one, even if shipping one language.

## Never

- ❌ `Container` with only padding — use `Padding` widget directly.
- ❌ Deeply nested ternaries in `build` methods. Extract into private widget classes.
- ❌ `print` for logs — use `logger` package with levels.
- ❌ `Navigator.push` directly — use `context.go()` / `context.push()` via go_router.
- ❌ Mix Provider + Riverpod + GetX. Pick one (Riverpod). Stay there.
- ❌ Async work in `initState` without `Future.microtask`. Move it to a ViewModel.
- ❌ `BuildContext` across async gaps without `mounted` check.

## Widget structure

Long `build` methods → extract `_Header`, `_Body`, `_Footer` as private widget classes in the same file. Don't use functions returning widgets (they break const-ness and rebuild semantics).

```dart
class HomeScreen extends ConsumerWidget {
  const HomeScreen({super.key});
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Scaffold(
      appBar: const _Header(),
      body: _Body(),
    );
  }
}

class _Header extends StatelessWidget implements PreferredSizeWidget {
  const _Header();
  @override Size get preferredSize => const Size.fromHeight(kToolbarHeight);
  @override Widget build(BuildContext context) => AppBar(title: const Text('Home'));
}
```

## Error handling

- Repositories return `Either<Failure, T>`. `Failure` is a sealed class hierarchy.
- ViewModels expose `AsyncValue<T>` to UI. Widgets render `data/loading/error` branches explicitly via `when()`.
- Global error catcher: `FlutterError.onError` + `PlatformDispatcher.instance.onError` → send to Sentry.

## Testing

- **Unit tests** for use cases and ViewModels — mock repositories.
- **Widget tests** for screens — use `ProviderScope(overrides: ...)` to inject fake state.
- **Integration tests** for end-to-end flows (login → home → action).
- `flutter_test` + `mocktail` for mocking.
