# Week 3 — Build Autonomous Coding Workflow

**Status:** not started. Depends on Week 1 (models running) and benefits from Week 2 (prompts/rules in place).

## Goal

Stop chatting with the local AI. Make it *act*: read files, edit them, run commands, commit changes — like Claude Code does.

## Task 1 — Configure Continue.dev

Project-level config at `.continue/config.json` (or `~/.continue/config.json` for global):

```json
{
  "models": [
    {
      "title": "Qwen Coder",
      "provider": "ollama",
      "model": "qwen2.5-coder:7b",
      "apiBase": "http://localhost:11434"
    },
    {
      "title": "DeepSeek Coder",
      "provider": "ollama",
      "model": "deepseek-coder-v2",
      "apiBase": "http://localhost:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen Coder",
    "provider": "ollama",
    "model": "qwen2.5-coder:7b"
  },
  "systemMessage": "Senior engineer. Match existing file style. Be concise."
}
```

Verify: open VS Code, hit `Cmd+L`, ask a question, confirm response streams from local.

## Task 2 — Per-project context rules

In every project root, drop a `.ai-rules.md`:

```md
You are a senior engineer on this project.

Stack: <fill in>
Architecture: <fill in>

Requirements:
- Match the existing file's style first
- Reusable, composable code
- Comments only when the WHY is non-obvious
- Don't break public APIs
```

Continue.dev's `@codebase` and Aider both pick this up.

## Task 3 — Install Aider

```bash
brew install python  # already done in week 1
pip3 install --user aider-chat
# or use pipx for cleaner isolation:
brew install pipx && pipx install aider-chat
```

First-run config at `~/.aider.conf.yml`:

```yaml
model: ollama/qwen2.5-coder:7b
weak-model: ollama/qwen2.5-coder:7b
edit-format: diff
auto-commits: true
gitignore: true
read:
  - ai-prompts/prompts/system/master.md
  - ai-memory/rules/.react-rules.md  # add per project
```

Run inside any project:

```bash
aider                                  # auto-discovers files via repo map
aider src/auth/*.ts                    # explicit files
aider --message "add password reset"   # one-shot
```

## Task 4 — Git-aware workflow

Aider auto-commits every accepted change. This is a feature, not noise — you get free undo:

```bash
git log --oneline                       # see what AI did
git reset --hard HEAD~1                 # undo last AI commit
git diff HEAD~1                         # review before reset
```

Keep commits small. The 7B model performs much better when each task has a clean diff to reference.

## Task 5 — Workflow prompts

Save to `ai-workflows/prompts/`:

### `before-coding.md`
```
Before you write any code:
1. List the files you'll need to read or modify.
2. Identify the data flow and key abstractions involved.
3. Flag any architectural risks or assumptions.
4. Ask me to confirm before proceeding.
```

### `during-coding.md`
```
While coding:
- One concern per function.
- Reuse existing utilities before creating new ones.
- No premature abstractions.
- Match the project's existing style.
```

### `after-coding.md`
```
After coding:
1. Re-read each modified file.
2. Identify any new bugs or regressions.
3. Suggest one improvement you didn't make and explain why you deferred it.
4. List what tests should be added.
```

Invoke with `aider --message-file ai-workflows/prompts/before-coding.md`.

## Verify Week 3 is good

You should be able to:

```bash
cd ~/some-project
aider --message "add a /healthz endpoint that returns build sha"
```

…and have the local agent:
1. Discover the right file via repo map
2. Edit it
3. Commit
4. Tell you what it did

If that loop works end-to-end with **no manual file selection**, Week 3 is real.

## Reference

- [auto-file-edit-options.md](auto-file-edit-options.md) — comparison of Aider vs Cline vs Continue agent mode
