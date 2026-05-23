# ai-memory

Persistent engineering memory for the local AI agent. Plain markdown + git history is the "memory" layer.

## Layout

- `memory/debugging/` — debugging playbooks per stack (Flutter, Node.js, React, MongoDB, SwiftUI). Week 2 Task 5.
- `memory/projects/` — per-project architecture notes, API docs, design decisions, reusable patterns. Week 4 Task 2.
- `rules/` — stack-specific rule files (`.flutter-rules.md`, `.react-rules.md`, etc.). Week 2 Task 3.

## Why this matters

7B local models have weak general knowledge compared to Claude Opus / GPT-5. They get strong by reading *your* memory at inference time. The denser and more specific this directory is, the closer local output gets to Claude-Code quality.
