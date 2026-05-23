# Week 1 — Foundation + Environment Setup

**Status as of 2026-05-23:** ✅ **complete.** Both models loaded, VS Code wired, monorepo pushed to GitHub, Aider installed and verified end-to-end.

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

### Task 3 — GitHub repos (evolved to monorepo)
- [x] Initially scaffolded as 4 separate repos and pushed (ai-prompts, ai-memory, starter-templates, ai-workflows)
- [x] **Architecture changed mid-session to monorepo** at `github.com/Lokeshburade007/Ai-Agent-lk007`. The 4 sub-repos were absorbed as plain folders.
- [x] Git author/committer fixed to `Lokeshburade007` (noreply email). Initial `snanydevicess@gmail.com` mapping to a different GitHub account was caught and amended via force-push.
- [ ] **MANUAL:** delete the 4 obsolete sub-repos: `gh auth refresh -h github.com -s delete_repo` then `gh repo delete Lokeshburade007/{ai-prompts,ai-memory,starter-templates,ai-workflows} --yes`

### Task 4 — Local coding models
- [x] `ollama pull qwen2.5-coder:7b` → 4.7GB
- [x] `ollama pull deepseek-coder-v2` → 8.9GB

### Task 5 — Smoke test
- [x] Express auth prompt → Qwen generated correct code in **97s** on M2 (bcrypt + jwt + `/login` route).
- [x] Aider auto-file-edit session end-to-end: prompt → SEARCH/REPLACE → diff → commit (`hello.py` + `test_hello.py` in `~/aider-test`). The 7B model made the "function returns None vs string" mistake — that's exactly what the Week 2 master prompt addresses.

### Bonus this session
- [x] Aider 0.86.2 installed via pipx
- [x] `~/.aider.conf.yml` + project-local `.aider.conf.yml` configured to use Qwen via Ollama
- [x] First end-to-end auto-file-edit demo completed
- [x] `docs/aider-cheatsheet.md` written
- [ ] **MANUAL:** `pipx install pytest` to close the test loop
- [ ] **MANUAL:** clean up duplicate `def hello()` in `~/aider-test/hello.py` (one-liner)

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
