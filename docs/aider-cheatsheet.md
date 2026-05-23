# Aider Cheatsheet

The auto file-edit agent, pointed at local Qwen via Ollama. Config lives in [../.aider.conf.yml](../.aider.conf.yml).

## Start a session

```bash
cd ~/your-project
git init                                # aider requires git in the project
aider                                   # uses .aider.conf.yml if present
```

For one-shot tasks:
```bash
aider --message "add /healthz endpoint that returns build sha"
```

## Inside the aider prompt — the commands you'll actually use

| Command | What it does |
|---|---|
| `/add <file>` | Add a file to the editable context |
| `/read <file>` | Add a file as **read-only** reference (great for system prompts, rules) |
| `/drop <file>` | Remove from context |
| `/ls` | List files in context |
| `/diff` | Show changes since last commit |
| `/undo` | Undo aider's last commit |
| `/clear` | Clear chat history (keeps files) |
| `/tokens` | See context usage |
| `/help` | Full command list |
| `/exit` or Ctrl+D | Quit |

## Speed tip — load a system prompt + rules

```bash
aider \
  --read ai-prompts/prompts/system/master.md \
  --read ai-memory/rules/.react-rules.md \
  src/components/Login.tsx
```

Aider loads `Login.tsx` as editable, the master prompt + react rules as read-only context. The model behaves dramatically more like a senior engineer this way.

## Bigger context window for Qwen

By default Ollama clips context to ~2k tokens, way below Qwen's 32k capability. Fix:

```bash
# In your shell or .zshrc:
export OLLAMA_NUM_CTX=8192   # or 16384 if you have RAM headroom

# Or per-session:
OLLAMA_NUM_CTX=8192 aider
```

Don't go to 32k unless you really need it — uses more RAM.

## When edits fail

If aider says "no exact match" or similar:

1. The file changed between repo map and edit. Run `/diff` to see, then ask aider to retry.
2. The model produced a malformed SEARCH block. Switch to a stronger model: `aider --model ollama_chat/deepseek-coder-v2`.
3. The task is too big. Break into smaller asks. 7B models lose the plot after ~3 file edits per turn.

## Switch models per task

| Task type | Best model |
|---|---|
| Small edits, autocomplete-style | `qwen2.5-coder:7b` (fast, good enough) |
| Multi-file refactor | `deepseek-coder-v2` (slower, smarter) |
| Architectural reasoning | Claude Code (don't burn it on small stuff) |

Switch mid-session:
```
/model ollama_chat/deepseek-coder-v2
```

## Auto-commit workflow

Aider commits after every accepted change:
```bash
git log --oneline -5            # see what aider did
git diff HEAD~1                 # review the latest aider commit
git reset --hard HEAD~1         # undo just the last aider change
```

Commits are tagged with "aider:" prefix so you can grep them later.

## Disable auto-commit for risky sessions

```bash
aider --no-auto-commits
```

You'll stage/commit manually but still get the edit speed.
