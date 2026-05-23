# Week 4 — Optimization + Advanced Features

**Status as of 2026-05-23:** ✅ **complete for the work that can be done pre-emptively.** The remaining items (actual model optimization decisions, real benchmark numbers, lived-in handbook content) are *deliberately deferred* — they need real usage to be meaningful. Templates and infrastructure are in place; fill them as you go.

## Goal

Move from "I made it work" to "I actually use this every day."

## Task 1 — Optimize models ✅

Current state:
```
qwen2.5-coder:7b           4.7 GB   primary chat/edit/autocomplete
deepseek-coder-v2:latest   8.9 GB   heavy reasoning fallback
nomic-embed-text:latest    274 MB   embeddings for @codebase RAG (Task 3)
```

Total disk: ~14GB out of 60+GB free. Nothing to prune yet.

**RAM management** during long Aider sessions:
```bash
ollama ps                                   # see what's loaded in memory
ollama stop qwen2.5-coder:7b                # unload a SPECIFIC model
# unload everything currently loaded:
for m in $(ollama ps | tail -n +2 | awk '{print $1}'); do ollama stop "$m"; done
brew services stop ollama                   # nuclear: kill the whole service
brew services start ollama                  # turn it back on
brew services restart ollama                # if Ollama process gets stuck
```

**Default behavior** (Ollama 0.24+): models auto-unload **5 min** after the last request. So in most cases you don't have to do anything — close VS Code, walk away, RAM is back automatically.

When to manually intervene:
- Need RAM RIGHT NOW (Chrome is choking) → `brew services stop ollama`
- Going AFK for hours → `brew services stop ollama` (start it again when back)
- If Activity Monitor shows 14GB+ used by ollama → you're at the 16GB ceiling, swap kicks in. Unload models or `brew services stop ollama`.

The CLI command `ollama stop` (no model name) does NOT work — it requires a specific model name in v0.24. Use the loop one-liner above for "stop all."

**When to revisit Task 1 for real:**
- After 1–2 weeks: which model do you actually run? Keep that one; consider dropping the other.
- If disk pressure ever hits: drop deepseek (8.9GB) first — Qwen alone covers ~80% of tasks.

## Task 2 — Project memory ✅ (template + helper script)

A `.ai/` scaffold template lives at [../ai-memory/memory/projects/_template/](../ai-memory/memory/projects/_template/). Drop it into any project with:

```bash
cd ~/path/to/project
mkmem            # copies the template into ./.ai/
```

The scaffold:
```
.ai/
  architecture.md   — project shape, stack, boundaries
  patterns.md       — project-specific conventions
  api.md            — endpoint / function inventory
  data-model.md     — schema overview
  decisions/        — ADR template (one file per decision)
  glossary.md       — domain terms
```

Wire into the project's `.aider.conf.yml`:
```yaml
read:
  - .ai/architecture.md
  - .ai/patterns.md
```

Now every Aider session in that project gets the project's memory in context — alongside the global master prompt + stack rules.

## Task 3 — Codebase indexing (RAG) ✅

`nomic-embed-text` (274MB, 768-dim) pulled and wired into Continue.dev as the `embed` role in `~/.continue/config.yaml`. This powers `@codebase` semantic search in the Continue chat sidebar.

**To use it in VS Code:**

1. Reload window: `Cmd+Shift+P → Developer: Reload Window`
2. Open the Continue sidebar (`Cmd+L`) inside a project
3. Continue indexes the codebase on first open — takes 30s–2min depending on project size
4. Reference the whole codebase semantically in chat:
   ```
   @codebase how is authentication handled in this project?
   @codebase show me all places that touch the orders schema
   ```

**When RAG actually helps:**
- Projects >100 files where the repo map gets noisy
- Cross-cutting questions ("which code paths call the payment service?")
- Onboarding to an unfamiliar codebase

**When RAG doesn't help (and the repo map is better):**
- Small projects (<50 files)
- "Edit this specific file" tasks — direct file references are more reliable

Aider does **not** use RAG; it relies on its repo map + your `--read` selections. That's a feature: explicit context beats fuzzy retrieval for autonomous edits.

## Task 4 — Personal engineering handbook ✅ (skeleton)

[../ai-workflows/handbook.md](../ai-workflows/handbook.md) — 7-section skeleton with prompts in each section. **Empty sections aren't gaps — they're invitations.** Fill them as patterns emerge from real use.

Sections:
1. Daily workflow
2. Coding standards (cross-stack)
3. Architecture patterns I default to
4. Debugging workflow
5. Deployment steps
6. Prompt patterns that consistently work
7. Operating notes for the agent itself (Qwen quirks, DeepSeek tradeoffs, Continue.dev caveats)

Review weekly to catch drift. Date each section when you change it. Prune what no longer matches.

## Task 5 — Benchmark ✅ (template)

[../ai-workflows/benchmark.md](../ai-workflows/benchmark.md) — fillable benchmark with the 5 representative tasks, a 1–5 quality scale, RAM-watch command, and a summary heuristic.

Run it **once you have 1–2 weeks of real use** — that gives you the right context to choose which tasks should genuinely be benchmarked vs. which feel obviously local-friendly.

The output isn't a winner declaration — it's a **routing table**: which tool to reach for first per task type.

## Verify Week 4 is good

The infrastructure is in place. The lived-in answers come later. After 2 weeks of real use, you should be able to answer these without thinking:

- Which model do I open for which task type? *(Fill in handbook §7)*
- Where do I store decisions so future-me and the AI both find them? *(In `.ai/decisions/` per project — drop via `mkmem`)*
- What's my measured speed/quality ratio vs Claude Code? *(Fill in benchmark.md after running 5 tasks)*

When all three have real answers — not skeleton placeholders — Week 4 is done for real.
