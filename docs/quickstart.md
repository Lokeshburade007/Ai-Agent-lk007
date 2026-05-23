# Quickstart — How to use your local AI agent

The 5-minute "how do I use this thing" guide. For the why and the deep dive, see [README.md](README.md) and the weekly docs.

---

## 1. The 5-second version

```bash
cd ~/my-project       # inside any git repo
lkai                  # opens an interactive Aider session with senior-engineer prompt
> add a /healthz endpoint that returns build sha
```

Aider edits files, shows you the diff, auto-commits to git. Type `/exit` or Ctrl+D to leave.

---

## 2. The three ways to invoke the agent

| Tool | When to use | How to invoke |
|---|---|---|
| **`lkai`** (Aider wrapper, terminal) | Autonomous multi-file edits, refactors, features | `lkai [flags] [files] --message "..."` |
| **Continue.dev** (VS Code) | In-editor chat, tab autocomplete, `Cmd+I` to refactor highlighted code | `Cmd+L` to open sidebar |
| **`ollama run`** (raw chat) | One-off Q&A with no file context | `ollama run qwen2.5-coder:7b` |

Default to **`lkai` for tasks** and **Continue for inline help while typing.**

---

## 3. The most common commands

```bash
# Interactive session — best for anything multi-step
lkai
> [your request]

# One-shot edit (small, well-defined tasks)
lkai --message "rename getCwd to getCurrentWorkingDirectory everywhere"

# New file — pass the path as a positional argument
lkai src/components/Button.tsx --message "create a primary Button component with variant prop"

# Stack-aware (loads the right rule file automatically)
lkai --stack react src/components/LoginForm.tsx
lkai --stack node --message "add JWT refresh token rotation to /api/auth"
lkai --stack flutter lib/features/login/login_screen.dart

# Investigation / bug hunt
lkai --stack node --debug --concern debugging --message "investigate 502 on /api/orders"

# Senior-engineer review of an existing area
lkai --concern security-review src/auth/
lkai --concern architecture-review src/

# Three-phase deliberate workflow (for high-stakes work)
lkai --phase before --stack react --message "add settings page"
# (Qwen plans; you confirm; switch phase)
lkai --phase during --stack react src/pages/settings/
lkai --phase after src/pages/settings/
```

`lkai --help` lists every flag and example.

---

## 4. The three knobs that change behavior

| Flag | Loads | When to use |
|---|---|---|
| `--stack <name>` | Stack-specific rules (`node`, `react`, `flutter`, `swiftui`, `golang`) | Any stack-specific task |
| `--concern <name>` | Specialized prompt (`debugging`, `architecture-review`, `code-optimization`, `security-review`, `api-design`, `database-schema`, `ui-generation`) | Focused task type |
| `--phase <name>` | Phase prompt (`before`, `during`, `after`) | Multi-step features where you want planning + self-review |
| `--debug` | Stack-specific debugging playbook (requires `--stack`) | Hunting a bug |

Stack them. Example for a security-critical Node refactor:
```bash
lkai --stack node --concern security-review --phase before --message "add API-key auth alongside JWT"
```

---

## 5. Inside VS Code (Continue.dev)

After reloading VS Code (`Cmd+Shift+P → Developer: Reload Window`):

- **`Cmd+L`** — open chat sidebar. Pick "Qwen Coder 7B" from the dropdown. Ask anything about open files; reference files with `@filename`, the whole codebase with `@codebase`.
- **Tab autocomplete** — start typing in any file, gray suggestions appear after ~500ms. Press Tab to accept.
- **`Cmd+I`** — highlight code, hit Cmd+I, type "make this typed" / "extract this into a hook" / "add error handling" — Continue applies the edit inline.

For multi-file work or autonomous edits, **switch to `lkai` in the terminal**. Continue is better at single-file chat; Aider is better at coordinated edits across files.

---

## 6. The two gotchas

**Gotcha 1 — Creating new files**
```bash
# ❌ wrong: aider loops because the file doesn't exist
lkai --message "create src/components/Login.tsx: ..."

# ✅ right: pass the new path so aider creates it first
lkai src/components/Login.tsx --message "create a Login component"
```

**Gotcha 2 — Interactive prompts**
When `lkai` shows `>`, that's its cursor — type your request **after** it. Don't type `>` yourself (zsh will think it's a redirect).

---

## 7. Switching models mid-session

Inside an interactive session:
```
> /model ollama_chat/deepseek-coder-v2     # smarter, slower
> /model ollama_chat/qwen2.5-coder:7b      # faster, default
```

Rule of thumb: Qwen for daily edits and small features. DeepSeek when Qwen gets stuck on multi-file refactors or tricky logic.

---

## 8. Undoing what the agent did

Aider auto-commits every accepted change. Free undo:
```bash
git log --oneline -5            # see what the agent did
git diff HEAD~1                 # review the latest commit
git reset --hard HEAD~1         # undo just the last AI commit
```

For larger surgery: `git reflog` shows every step including aborted edits.

---

## 9. When the output is bad

Most common cause: too little context. Fixes in order of effort:

1. **Add more files to the chat**: `> /add path/to/file.ts`
2. **Layer the right rule file**: `--stack <name>`
3. **Use the planning phase first**: `--phase before` — forces Qwen to think before typing
4. **Switch to DeepSeek**: `> /model ollama_chat/deepseek-coder-v2`
5. **Ask Claude Code instead** — for tricky design or novel problems, your remaining Claude subscription is the right tool. Local Qwen is 80% of daily coding; Claude is the hard 20%.

---

## 10. Where to look for more

- [readme.md](../readme.md) — the original 4-week plan
- [aider-cheatsheet.md](aider-cheatsheet.md) — every Aider slash command
- [week-1-foundation.md](week-1-foundation.md) → [week-4-optimization.md](week-4-optimization.md) — what was built and why
- [auto-file-edit-options.md](auto-file-edit-options.md) — why Aider over Cline/Continue agent
- [architecture-hybrid-mac-vps.md](architecture-hybrid-mac-vps.md) — future remote-access architecture
- `lkai --help` — flags + examples

Built it once. Use it forever.
