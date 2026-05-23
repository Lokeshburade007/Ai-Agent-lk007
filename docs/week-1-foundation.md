# Week 1 — Foundation + Environment Setup

**Status as of 2026-05-23:** mostly done. DeepSeek pull running in background; user has 3 manual follow-ups.

## Goal

Get a local LLM running, wired into VS Code, with scaffolded repos to host the rest of the build.

## Hardware reality check

- MacBook Air M2, 16GB RAM
- Free disk: **63GB** (readme said 35GB — outdated, you have headroom)
- Ollama models will live in `~/.ollama/models/`

## Checklist

### Task 1 — Storage audit
- [x] `du -sh ~/*` run; biggest folder is `~/Library` (39GB) which is normal macOS caches. No cleanup needed.

### Task 2 — Core tools
- [x] Homebrew — already installed (v5.1.11)
- [x] git — already installed (v2.51.0)
- [x] Ollama — `brew install ollama` → v0.24.0, running as `brew services start ollama` on `localhost:11434`
- [x] VS Code — already installed (v1.121.0)
- [x] VS Code extensions installed via CLI:
  - Continue.continue
  - eamodio.gitlens
  - usernamehw.errorlens
  - esbenp.prettier-vscode
  - bradlc.vscode-tailwindcss
- [x] `code` CLI symlinked at `~/bin/code`
- [ ] **MANUAL:** add `export PATH="$HOME/bin:$PATH"` to `~/.zshrc`, then `source ~/.zshrc`

### Task 3 — GitHub repos
Four repos scaffolded **locally only** (git init + initial commit on `main`, no remote):
- [x] `ai-prompts/` with `prompts/system/master.md` placeholder + `prompts/specialized/`
- [x] `ai-memory/` with `memory/debugging/`, `memory/projects/`, `rules/`
- [x] `starter-templates/`
- [x] `ai-workflows/`
- [ ] **MANUAL:** create the four repos on github.com and push:
  ```bash
  cd ai-prompts && git remote add origin git@github.com:<you>/ai-prompts.git && git push -u origin main
  # repeat for ai-memory, starter-templates, ai-workflows
  ```

### Task 4 — Local coding models
- [x] `ollama pull qwen2.5-coder:7b` → 4.7GB, ~3 min download
- [~] `ollama pull deepseek-coder-v2` → 8.9GB, downloading in background

### Task 5 — Smoke test
- [x] Express auth prompt → Qwen generated correct Express + bcrypt + jwt code in **97 seconds** on M2. Quality note: it inserted a fake-looking bcrypt hash for the demo user, typical of 7B models — replace with real data in real code.
- [ ] React login (Tailwind) prompt
- [ ] Flutter clean architecture prompt

## Key Week 1 decisions made

| Decision | Choice | Why |
|---|---|---|
| Brew service vs manual `ollama serve` | brew service | Auto-restarts at login; matches "always-on personal agent" goal |
| Pull deepseek now or later | Sequential after qwen | User chose to avoid bandwidth saturation |
| GH repo creation method | User does it manually | Avoid auto-creating repos on the user's GH account |
| Edit `~/.zshrc` automatically | No | Auto-mode classifier (correctly) blocked dotfile edits; user does this themselves |

## Verify Week 1 is good

```bash
brew services list | grep ollama       # should show "started"
curl -s localhost:11434/api/tags        # should return JSON with both models
ollama list                             # should show qwen2.5-coder:7b and deepseek-coder-v2
"/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code" --list-extensions | grep -E "continue|gitlens|errorlens|prettier|tailwind"
```

When all four return clean, Week 1 is done.
