# Week 2 — Prompt Engineering + AI Brain Building

**Status:** not started.

## Why this week matters more than the others

Week 2 is the **strategic window** while Claude Code subscription is still active.

A 7B local model isn't smarter than Claude Opus — it's a *different shape*. It has fast inference, infinite usage, full privacy, but weak general reasoning. The way you close the gap is by feeding it **distilled judgment** at inference time: senior-engineer system prompts, stack-specific rules, debugging playbooks, architecture templates.

The intelligence transfer happens here. After Week 2, your local model behaves much more like a senior engineer because it inherits the patterns you extracted from Claude.

## Task 1 — Master system prompt

Run this prompt in **Claude Code** (not local model — quality matters):

```
Create a senior-level AI coding assistant system prompt optimized for:
- MERN stack (Express, MongoDB, React, Node)
- Flutter (clean architecture, Riverpod, MVVM)
- SwiftUI (MVVM, Combine)
- Golang backend (clean architecture, repository pattern)
- Scalable architecture, modular design, security best practices

The prompt should make the assistant:
- Prefer reusable components and clean abstractions
- Flag risks (security, performance, scalability) proactively
- Refuse to generate code without first understanding context
- Match the conventions of the file/project it's editing
- Be concise — no over-explanation
```

Save the output to: `ai-prompts/prompts/system/master.md` (replacing the placeholder).

## Task 2 — Specialized prompts

Generate one per concern, save to `ai-prompts/prompts/specialized/`:

- [ ] `debugging.md` — systematic bug isolation, hypothesis testing
- [ ] `architecture-review.md` — design critique, scalability flags
- [ ] `code-optimization.md` — perf/memory hot-paths
- [ ] `security-review.md` — OWASP top 10, secrets handling
- [ ] `api-design.md` — REST/GraphQL conventions, versioning
- [ ] `database-schema.md` — normalization, indexes, migrations
- [ ] `ui-generation.md` — accessibility, responsive, design tokens

## Task 3 — Stack-specific rules library

Drop these in `ai-memory/rules/` (Continue.dev and Aider both read these):

- [ ] `.flutter-rules.md` — clean architecture, Riverpod, MVVM, repository pattern
- [ ] `.react-rules.md` — hooks, server components, Tailwind, accessibility
- [ ] `.node-rules.md` — Express, async/await, error middleware, env validation
- [ ] `.swiftui-rules.md` — MVVM, Combine, navigation, async/await
- [ ] `.golang-rules.md` — clean architecture, error wrapping, context propagation

Format each one as:
```md
Always use:
- <pattern>
- <pattern>

Never use:
- <antipattern>
- <antipattern>

When asked to generate code:
- Match the file's existing style first
- Prefer composition over inheritance
- Keep functions under ~30 lines
```

These files are **the single biggest quality boost** for a local 7B model. Don't skip.

## Task 4 — Starter templates

Use Claude to generate clean boilerplates into `starter-templates/`:

- [ ] `mern-starter/` — Express + Mongo + React + Tailwind + auth
- [ ] `flutter-starter/` — Clean architecture + Riverpod + GoRouter
- [ ] `swiftui-mvvm-starter/`
- [ ] `go-backend-starter/` — Echo or Gin + Postgres + clean arch
- [ ] `auth-boilerplate/` — JWT + refresh tokens + bcrypt
- [ ] `api-template/` — REST + OpenAPI + rate limit + validation
- [ ] `docker-template/` — multi-stage builds + compose for dev

Each should have its own README + working `make dev` / `npm run dev`.

## Task 5 — Debugging playbook

Save to `ai-memory/memory/debugging/`:

- [ ] `flutter.md` — common build/runtime issues, hot-reload gotchas
- [ ] `node.md` — async error propagation, EADDRINUSE, module resolution
- [ ] `react.md` — hydration mismatches, stale closures, re-render loops
- [ ] `mongo.md` — index misses, ObjectId pitfalls, connection pool exhaustion
- [ ] `swiftui.md` — view update cycles, NavigationStack state, Combine retain cycles

## Week 2 output check

When done, you should be able to:

```bash
# Aider auto-loads these
aider --model ollama/qwen2.5-coder:7b \
  --read ai-prompts/prompts/system/master.md \
  --read ai-memory/rules/.react-rules.md
```

and feel the local model behaves like a senior engineer for React work — not like a generic 7B chatbot.
