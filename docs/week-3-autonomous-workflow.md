# Week 3 — Build Autonomous Coding Workflow

**Status as of 2026-05-23:** ✅ **complete.** Aider installed and tested end-to-end (Week 1), Continue.dev wired to local Ollama (this session), workflow prompts written, and an `agent` wrapper script makes everything callable from anywhere with one command.

## Goal

Stop chatting with the local AI. Make it *act*: read files, edit them, run commands, commit changes — like Claude Code does.

## Task 1 — Configure Continue.dev ✅

`~/.continue/config.yaml` is now wired to local Ollama with **three models**: Qwen for chat/edit, DeepSeek for chat/edit, plus a dedicated Qwen autocomplete config (for lower-latency tab completion).

The config also includes **inline rules** that mirror the senior-engineer master prompt (Continue's YAML doesn't yet support `--read` of external files the way Aider does — so the rules are inlined).

Verify:

1. Reload VS Code (`Cmd+Shift+P → Developer: Reload Window`)
2. `Cmd+L` to open the Continue chat sidebar
3. Pick "Qwen Coder 7B" from the model dropdown
4. Ask: "what does this file do?" inside any open file
5. Response should stream from local Ollama (no API key prompt)

For tab completion: just start typing in any file — gray-text suggestions should appear after ~500ms.

## Task 2 — Per-project context rules ✅ (mostly redundant after Week 2)

The original plan called for a `.ai-rules.md` in every project. After Week 2, this is largely covered by:

- **Stack rules** in `ai-memory/rules/<stack>-rules.md` (loaded via `agent --stack <name>` or per-project `.aider.conf.yml`)
- **Master prompt** auto-loaded into every Aider session via `~/.aider.conf.yml`
- **Continue's inline rules** in `~/.continue/config.yaml`

The only thing a per-project `.ai-rules.md` adds is **project-specific** rules (e.g., "this app uses Redis, not Memcached" or "ignore the legacy `/v1` namespace"). Drop one in only when a project has constraints the global rules don't capture.

Template for when you do:
```md
# Project rules — <project name>

Stack-specific overrides on top of the global master prompt:

- Storage: <Postgres / Mongo / Redis>
- State management on frontend: <Riverpod / Zustand / Redux>
- Notable historical decisions: <e.g. "all timestamps in UTC", "user IDs are UUIDs not Mongo ObjectIDs">

Things to avoid:
- <e.g. "don't suggest GraphQL — we're committed to REST">
- <e.g. "don't import lodash — use ramda">
```

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

## Task 5 — Workflow prompts ✅

Three phase prompts written. Load with `agent --phase before|during|after` or `aider --read ai-workflows/prompts/<name>-coding.md`:

- [before-coding.md](../ai-workflows/prompts/before-coding.md) — forces the agent to **plan in 5 steps and stop for confirmation** before producing edits. Catches misunderstandings before code is written.
- [during-coding.md](../ai-workflows/prompts/during-coding.md) — implementation rules: match before invent, one file at a time, every import resolved, trace before trust, no filler / no premature abstractions.
- [after-coding.md](../ai-workflows/prompts/after-coding.md) — self-review checklist in 6 steps: re-read whole files, restate ask, list bugs introduced, list missing tests, surface one deferred improvement, give a concrete verify command.

These are layered with stack rules + specialized prompts. Example for a high-stakes refactor:

```bash
agent --phase before --stack node --concern architecture-review \
      --message "refactor the auth middleware to support API keys alongside JWT"
# agent stops after the plan; user confirms; switch to during phase:

agent --phase during --stack node \
      src/middlewares/auth.ts src/routes/api/*.ts

# when satisfied, run self-review:
agent --phase after src/middlewares/auth.ts src/routes/api/*.ts
```

## Verify Week 3 is good

You already passed the headline test on day 1 — `aider --message "create hello.py and a pytest test"` produced working SEARCH/REPLACE blocks, auto-committed, and ran in ~30s on M2. From here, the daily workflow is:

```bash
cd ~/some-project

# casual edit, no special context:
agent --message "rename getCwd to getCurrentWorkingDirectory everywhere"

# new feature, deliberate process:
agent --phase before --stack react --message "add a settings page with theme + language"
# review the plan, agree, then:
agent --phase during --stack react src/pages/settings/
# when implementation looks done:
agent --phase after src/pages/settings/
```

Plus inline in VS Code: `Cmd+L` for chat, tab autocomplete from Qwen, `Cmd+I` to refactor highlighted code — all hitting your local Ollama.

## Reference

- [auto-file-edit-options.md](auto-file-edit-options.md) — comparison of Aider vs Cline vs Continue agent mode
- [aider-cheatsheet.md](aider-cheatsheet.md) — every Aider command you'll use
