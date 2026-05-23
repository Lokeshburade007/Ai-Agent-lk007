# Benchmark — Local Qwen vs DeepSeek vs Claude Code

Fillable comparison. Run the same 5 tasks three ways. Track time, iterations, final quality. The point is to know **which tasks belong on which tool** — not to declare a winner.

Last run: <date>

## How to time

```bash
# For Aider / lkai:
time lkai --stack <X> --message "<task>"

# For Claude Code: run in a parallel terminal; clock-time the user-perceived completion.
```

## Quality scale (1–5)

| Score | Meaning |
|---|---|
| 5 | Production-ready, no edits needed |
| 4 | One small fix (typo, import, naming) |
| 3 | A few real issues, but the structure is right |
| 2 | Right idea, wrong execution — partial rewrite needed |
| 1 | Wrong approach entirely |

## The 5 benchmark tasks

Pick tasks that exercise different parts of the system. Use the same prompt verbatim across the three runs for each row.

### Task 1 — Flutter login screen

**Prompt:** "Create a Flutter login screen with email + password fields, validation (email format, password min 8), submit button calling a fake `AuthRepository.signIn(email, password) → Future<Either<Failure, User>>`. Use Riverpod for state. Show loading + error states."

| Tool | Wall time | Iterations | Quality (1–5) | Final RAM peak | Notes |
|---|---|---|---|---|---|
| `lkai --stack flutter` (Qwen 7B) | | | | | |
| `lkai --stack flutter --model deepseek-coder-v2` | | | | | |
| Claude Code | | | n/a | | |

### Task 2 — MERN auth endpoint

**Prompt:** "Add a `POST /auth/refresh` endpoint to an existing Express app. Validates a refresh-token cookie, rotates it (hashes new one in DB, returns new access+refresh pair). Wrap with rate limit (5 req/min/IP)."

| Tool | Wall time | Iterations | Quality (1–5) | Final RAM peak | Notes |
|---|---|---|---|---|---|
| `lkai --stack node` (Qwen 7B) | | | | | |
| `lkai --stack node --model deepseek-coder-v2` | | | | | |
| Claude Code | | | n/a | | |

### Task 3 — SwiftUI list with search

**Prompt:** "Create a SwiftUI screen showing a searchable list of `Item` (id, name, tags: [String]). Search filters by name OR any tag, case-insensitive, debounced 300ms. Use `@StateObject` view model with async fetch from a fake `ItemsRepository`."

| Tool | Wall time | Iterations | Quality (1–5) | Final RAM peak | Notes |
|---|---|---|---|---|---|
| `lkai --stack swiftui` (Qwen 7B) | | | | | |
| `lkai --stack swiftui --model deepseek-coder-v2` | | | | | |
| Claude Code | | | n/a | | |

### Task 4 — Refactor a 200-line file

**Prompt:** "Refactor `<file>` to extract <X> into a reusable hook/helper. Keep behavior identical. Update all call sites."

> Pick a real file in a real project for this one. Don't synthesize.

| Tool | Wall time | Iterations | Quality (1–5) | Final RAM peak | Notes |
|---|---|---|---|---|---|
| `lkai` (Qwen 7B) | | | | | |
| `lkai --model deepseek-coder-v2` | | | | | |
| Claude Code | | | n/a | | |

### Task 5 — Track down a bug

**Prompt:** Describe a real bug you have, including the symptom + repro steps. Let each tool diagnose + fix.

> Choose a bug that took you >15 min to find originally. Don't tell the tool the answer.

| Tool | Wall time to fix | Iterations | Quality of fix (1–5) | Final RAM peak | Notes |
|---|---|---|---|---|---|
| `lkai --debug --concern debugging` (Qwen 7B) | | | | | |
| `lkai --debug --concern debugging --model deepseek-coder-v2` | | | | | |
| Claude Code | | | n/a | | |

## Summary heuristic to fill in

After running the 5 tasks, write down which tool wins on which axis.

| Axis | Winner | By how much | Why |
|---|---|---|---|
| Speed (wall time) | | | |
| Quality on easy tasks | | | |
| Quality on novel / long-context tasks | | | |
| Cost ($) | Local — always | infinite | |
| Privacy | Local — always | infinite | |
| First-try success rate | | | |
| Recovery when first try is wrong | | | |

## Conclusions worth keeping

After benchmarking, add the practical rules of thumb here. Examples (to be replaced with your findings):

- > Use Qwen 7B for: small edits, single-file features, debug investigations on familiar stacks.
- > Use DeepSeek for: multi-file refactors when Qwen drifts; logic-heavy work.
- > Use Claude Code for: novel architecture, complex multi-file changes, anything where the model has to *reason* not just transform.

## RAM watch during runs

For each run, in another terminal:
```bash
ps -o rss= -p $(pgrep -f "ollama runner") | awk '{print $1/1024 " MB"}'
```

Track peak. If Ollama gets close to 14GB (your 16GB Mac will start swapping), record it — that's the upper-bound for how long sessions can run before you need to `ollama stop`.
