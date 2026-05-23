# ai-prompts

Reusable prompt library for the local AI coding agent.

## Layout

- `prompts/system/master.md` — senior-level master system prompt (MERN, Flutter, SwiftUI, Go, scalable architecture). Generated in Week 2 Task 1 using Claude Code.
- `prompts/specialized/` — task-specific prompts: debugging, architecture review, code optimization, security review, API design, database schema, UI generation.

## How to use

Point Continue.dev / Aider at the relevant prompt as a system message. The local model (Qwen2.5-Coder 7B / DeepSeek-Coder V2) leans heavily on these to behave like a senior engineer instead of a generic 7B assistant.
