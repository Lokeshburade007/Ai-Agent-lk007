# Auto File-Edit Agents — Options Compared

**Question:** "Can I have an agent that edits my files automatically when I give it a prompt, like Claude Code?"

**Short answer:** Yes — and you don't need to build it. The architecture is mature in several open-source tools.

## How auto file-edit actually works

| Step | What happens |
|---|---|
| 1. Repo map | Agent scans the project, builds a compressed index of symbols/files. |
| 2. File selection | You `/add file.py` (Aider) or the agent auto-picks files based on the prompt. |
| 3. Context window | Selected file contents + your prompt + system rules → sent to the LLM. |
| 4. Structured edits | LLM responds with `SEARCH/REPLACE` blocks (Aider), unified diffs, or tool-calls. |
| 5. Apply + commit | Agent parses, applies to disk, commits to git (revertable). |
| 6. Lint/test loop | Optionally runs tests, feeds errors back for self-correction. |

## Options (all work with Ollama + your M2)

| Tool | Style | Fit for local 7B | Why |
|---|---|---|---|
| **Aider** | CLI, terminal-first | ⭐⭐⭐⭐⭐ | Designed with small models in mind. Mature SEARCH/REPLACE edit format, auto-commits to git, repo map. The reference choice. |
| **Cline** (VS Code) | In-editor sidebar | ⭐⭐⭐⭐ | More visual, supports MCP servers and tool use, heavier on context. |
| **Roo Code** | Cline fork with modes | ⭐⭐⭐⭐ | Adds Architect/Code/Ask modes. Same backend strengths as Cline. |
| **Continue.dev agent mode** | Native VS Code | ⭐⭐⭐ | Improving fast but Aider still more robust on local models. Great for inline autocomplete + chat though. |
| **OpenHands** (ex-OpenDevin) | Fully autonomous, container | ⭐⭐ | Overkill. Struggles on 7B. Best with frontier models. |

## Recommended setup

**Use both Aider and Continue.dev together. They complement, not compete:**

- **Continue.dev (in VS Code):**
  - Inline tab-completion as you type
  - Chat sidebar for "explain this", "what does this do"
  - `Cmd+I` for highlight → "refactor this"
- **Aider (in terminal):**
  - Multi-file autonomous edits: "add password reset flow"
  - Auto-commits give you free undo via `git reset`
  - Best for "do this whole feature" tasks

That combo is functionally a local Claude Code.

## Why not build your own from scratch?

You can, but the hard parts are subtle:

| Problem | Why it's hard |
|---|---|
| Edit format | 7B models hallucinate diff line numbers; SEARCH/REPLACE works because it's exact-match. Aider iterated on this for years. |
| Fuzzy matching | When whitespace differs slightly, you need tolerant matching. Edge cases everywhere. |
| Context window | 7B models have 4–32k context. You need a repo map + retrieval, not "load everything." |
| Error recovery | Model produces invalid edit → retry with the error → don't loop forever. |
| Tool calling on small models | Function-calling reliability degrades fast below 13B. Aider avoids it; Cline works around it. |

**Verdict:** Building your own is a 3–6 month side-project that won't beat Aider. Use Aider, customize via prompts/rules from [week-2-prompts-and-brain.md](week-2-prompts-and-brain.md). That's where your effort gets the most leverage.

## Quick start (after Week 1 done)

```bash
pipx install aider-chat
cd ~/your-project
git init                                # aider requires git
aider --model ollama/qwen2.5-coder:7b
```

In the aider prompt:
```
> /add src/auth/login.ts
> add a "forgot password" link that POSTs to /api/auth/forgot
```

Aider will edit the file, show you the diff, and commit on accept.
