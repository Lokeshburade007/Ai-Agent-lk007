# Go Rules

Load this when working on any Go backend project.

## Architecture

**Clean architecture** with strict dependency direction:

```
cmd/
  api/main.go             # entry point
internal/
  domain/                 # entities + repository interfaces (pure Go)
  usecase/                # business logic, depends on domain
  infrastructure/         # repository impls (DB, HTTP clients)
  delivery/
    http/                 # handlers, routers, DTOs
  config/                 # env loading
pkg/                      # exportable, reusable libs (rare)
```

`internal/domain` has **only stdlib imports**. Compiles without any external dep.

## Always

- **`context.Context`** is the **first parameter** of every function that does I/O, talks to a DB, or makes a network call. Propagate it everywhere.
- **Errors wrapped with `fmt.Errorf("...: %w", err)`** — preserves the chain for `errors.Is` / `errors.As`.
- **Sentinel errors** at package level for known cases: `var ErrNotFound = errors.New("not found")`.
- **Repository interfaces in `domain`**, implementations in `infrastructure`. Use cases depend on the interface, not the impl.
- **Small interfaces** — accept interfaces, return structs. Interfaces live in the package that consumes them, not the one that implements them.
- **`slog`** (stdlib structured logging) over `log` or third-party loggers. Use `slog.Default()` configured at startup.
- **`net/http` stdlib + a thin router** (chi, gorilla/mux). Avoid heavyweight web frameworks (Gin, Echo are fine but unnecessary).
- **`pgx` or `sqlx`** for Postgres. Avoid raw `database/sql` for non-trivial queries — too easy to leak rows.
- **`golangci-lint`** in CI with a strict config. Treat warnings as errors.

## Never

- ❌ `context.Background()` inside an HTTP handler. Use `r.Context()`.
- ❌ Returning a naked `error` from a public function without wrapping context.
- ❌ Discarding errors silently (`_ = something()`). If you mean to ignore, comment why.
- ❌ Mutex on a single field — use atomic or refactor.
- ❌ `panic` for control flow. Reserve panic for genuinely unrecoverable states at startup.
- ❌ `init()` functions with side effects (DB connections, HTTP clients). Construct explicitly in `main()`.
- ❌ Singleton globals for stateful things. Constructor injection.
- ❌ Goroutines without an owner. Every `go func()` needs an obvious lifecycle: bounded by context, joined via WaitGroup, or supervised.

## Error handling pattern

```go
// domain/errors.go
var (
    ErrNotFound      = errors.New("not found")
    ErrAlreadyExists = errors.New("already exists")
)

// usecase
user, err := uc.repo.FindByID(ctx, id)
if err != nil {
    if errors.Is(err, domain.ErrNotFound) {
        return nil, fmt.Errorf("get user: %w", err) // propagate sentinel
    }
    return nil, fmt.Errorf("get user: %w", err) // wrap unknown
}

// http handler
if errors.Is(err, domain.ErrNotFound) {
    http.Error(w, "not found", http.StatusNotFound)
    return
}
```

## HTTP handler shape

```go
func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    id := chi.URLParam(r, "id")
    
    user, err := h.usecase.GetUser(ctx, id)
    if err != nil {
        h.writeError(w, err)
        return
    }
    
    writeJSON(w, http.StatusOK, toUserDTO(user))
}
```

## Concurrency

- Goroutines need **explicit lifecycle**:
  - Bounded by `ctx.Done()` for long-running work
  - Joined by `sync.WaitGroup` for fan-out
  - Supervised by `errgroup.Group` when any failure should cancel the rest
- **Channels** for signaling and pipelines. **Mutexes** for shared state.
- Don't share memory between goroutines without synchronization. Run with `-race` in CI.

## Testing

- **Table-driven tests** as the default style.
- **`testify/assert`** for readability. **`testify/mock`** for mocks (or generate via mockery).
- **Integration tests** in `_test.go` files tagged `//go:build integration`. Run separately in CI.
- **Testcontainers** for real DB / Redis in integration tests.
- **No mocking** of stdlib types (no mocked `http.Client`). Use httptest.

## Config

- One `Config` struct loaded from env at startup. Validated with `caarlos0/env` or similar.
- Crash on missing required env. Don't ship with silent defaults for security-relevant config (DB password, JWT secret).
- Pass `Config` (or a slice of it) to constructors. No package-level globals.
