# Week 2 — Prompt Engineering + AI Brain Building

**Status as of 2026-05-23:** ✅ **complete.** Master prompt + 7 specialized prompts + 5 stack rule files + 5 debugging playbooks all written and committed. Starter templates intentionally documented as "use community scaffolders + apply rules" instead of maintaining frozen boilerplates.

## Why this week matters more than the others

Week 2 is the **strategic window** while Claude Code subscription is still active.

A 7B local model isn't smarter than Claude Opus — it's a *different shape*. It has fast inference, infinite usage, full privacy, but weak general reasoning. The way you close the gap is by feeding it **distilled judgment** at inference time: senior-engineer system prompts, stack-specific rules, debugging playbooks, architecture templates.

The intelligence transfer happens here. After Week 2, your local model behaves much more like a senior engineer because it inherits the patterns you extracted from Claude.

## Task 1 — Master system prompt ✅

[ai-prompts/prompts/system/master.md](../ai-prompts/prompts/system/master.md) — senior-engineer behavior tuned for local 7B models. Specifically addresses the failure modes observed in Week 1 Aider sessions:
- Inventing files not asked for (`main.py` appearing out of nowhere)
- Mental execution of tested code (no more `assert hello() == "hello"` when `hello()` returns None)
- Exact SEARCH/REPLACE format rules (no more reflection loops on empty files)
- Per-stack defaults (MERN, Flutter, SwiftUI, Go)

**Auto-loaded** via `~/.aider.conf.yml` `read:` directive — applies to every Aider session machine-wide.

## Task 2 — Specialized prompts ✅

All 7 written. Use with `aider --read ai-prompts/prompts/specialized/<name>.md`:

- [x] [debugging.md](../ai-prompts/prompts/specialized/debugging.md) — diagnose before fix; hypothesis-driven
- [x] [architecture-review.md](../ai-prompts/prompts/specialized/architecture-review.md) — staff-engineer review; surface unseen risks
- [x] [code-optimization.md](../ai-prompts/prompts/specialized/code-optimization.md) — measure first; algorithmic > micro
- [x] [security-review.md](../ai-prompts/prompts/specialized/security-review.md) — exhaustive OWASP-style checklist
- [x] [api-design.md](../ai-prompts/prompts/specialized/api-design.md) — REST defaults + idempotency
- [x] [database-schema.md](../ai-prompts/prompts/specialized/database-schema.md) — Postgres + Mongo conventions
- [x] [ui-generation.md](../ai-prompts/prompts/specialized/ui-generation.md) — a11y-first; React/Flutter/SwiftUI

## Task 3 — Stack-specific rules library ✅

All 5 written. Add to a project's `.aider.conf.yml` `read:` block per project:

- [x] [node-rules.md](../ai-memory/rules/node-rules.md) — Express + Mongoose + zod + Pino
- [x] [react-rules.md](../ai-memory/rules/react-rules.md) — hooks + TanStack Query + react-hook-form + Tailwind
- [x] [flutter-rules.md](../ai-memory/rules/flutter-rules.md) — clean arch + Riverpod + freezed + go_router
- [x] [swiftui-rules.md](../ai-memory/rules/swiftui-rules.md) — MVVM + async/await + NavigationStack + SwiftData
- [x] [golang-rules.md](../ai-memory/rules/golang-rules.md) — clean arch + context propagation + sentinel errors

(File names dropped the leading dot from the original readme.md plan — leading-dot makes them hidden on macOS, which is unhelpful.)

## Task 4 — Starter templates ✅ (strategy documented, not boilerplates committed)

[starter-templates/README.md](../starter-templates/README.md) explains the deliberate choice: **use community scaffolders + apply our rules** instead of committing frozen boilerplates that rot.

Rationale:
- A React/Flutter/Go starter committed today is outdated in 3 months.
- Maintaining 7 starters across dep upgrades is unsustainable for a personal project.
- `create-react-app`, `vite create`, `flutter create`, `go mod init` + our `ai-memory/rules/*` files do the same job, kept current by their respective communities.

Concrete one-line scaffold commands per stack are in the README.

## Task 5 — Debugging playbooks ✅

All 5 written, one per stack:

- [x] [node.md](../ai-memory/memory/debugging/node.md) — async errors, EADDRINUSE, CORS, JWT, slow endpoints
- [x] [react.md](../ai-memory/memory/debugging/react.md) — infinite rerenders, stale closures, hydration, query dedup
- [x] [mongo.md](../ai-memory/memory/debugging/mongo.md) — ObjectId mismatches, N+1, ESR indexes, write conflicts
- [x] [flutter.md](../ai-memory/memory/debugging/flutter.md) — overflow, hot-reload limits, BuildContext-after-async, release-only crashes
- [x] [swiftui.md](../ai-memory/memory/debugging/swiftui.md) — state ownership, body update cycles, MainActor publishing, Codable errors

## Week 2 output — how to actually use this stuff

**Master prompt auto-loads** from `~/.aider.conf.yml` for every Aider session. You don't have to pass it explicitly.

For a specific stack + concern, layer extra context with `--read`:

```bash
# React UI work
aider --read ai-memory/rules/react-rules.md \
      --read ai-prompts/prompts/specialized/ui-generation.md \
      src/components/LoginForm.tsx

# Debugging a Mongo query
aider --read ai-memory/rules/node-rules.md \
      --read ai-memory/memory/debugging/mongo.md \
      --read ai-prompts/prompts/specialized/debugging.md \
      src/repositories/userRepo.ts

# Designing a new API endpoint
aider --read ai-memory/rules/node-rules.md \
      --read ai-prompts/prompts/specialized/api-design.md \
      --read ai-prompts/prompts/specialized/security-review.md \
      src/routes/orders.ts
```

The senior-engineer behaviors from the master prompt apply by default; stack rules + specialized prompts stack on top per task. **This is the actual "brain" of the local agent.**

A future enhancement (Week 4): a small `agent` shell function that auto-picks `--read` files based on the current project's stack. For now, copy-paste from above.
