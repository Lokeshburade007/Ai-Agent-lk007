# Week 4 — Optimization + Advanced Features

**Status:** not started. Polish week — make the system livable for daily use.

## Goal

Move from "I made it work" to "I actually use this every day."

## Task 1 — Optimize models

Keep what you use, delete what you don't:

```bash
ollama list                             # see what's there
ollama rm <unused-model>                # delete
du -sh ~/.ollama/models/                # check disk
```

If RAM gets tight during heavy use:

```bash
# Free up memory between heavy sessions:
ollama stop                              # unloads loaded models
brew services restart ollama
```

For long Aider sessions, the 7B model lives in RAM. On 16GB Macs this is fine as long as you don't have 30 Chrome tabs open.

## Task 2 — Project memory

Inside each real project, create `.ai/memory/`:

```
.ai/memory/
  architecture.md           # what this project is, key decisions
  api.md                    # endpoint inventory
  data-model.md             # schema overview
  decisions/                # ADRs — one per design call
    001-why-postgres.md
    002-no-redis-yet.md
  patterns.md               # reusable patterns in this codebase
```

Point Aider / Continue at these via `--read` or `@files`.

Also keep a project-wide memory at `ai-memory/memory/projects/<project-name>.md` for cross-project lessons.

## Task 3 — Codebase indexing (RAG, optional)

For larger projects (>200 files) the repo map alone isn't enough. Add semantic search:

```bash
# Option A: Continue.dev built-in @codebase
# (works out of the box if you point it at a repo)

# Option B: Open WebUI + ChromaDB
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main

# Option C: Tiny local RAG
pip install chromadb sentence-transformers
ollama pull nomic-embed-text             # 137MB embedding model
```

Index code into Chroma, retrieve top-K chunks at query time, pass into the model. See [architecture-hybrid-mac-vps.md](architecture-hybrid-mac-vps.md) for hosting Chroma on the VPS.

## Task 4 — Personal AI engineering handbook

Save to `ai-workflows/handbook.md`. Sections to cover:

- [ ] **Coding standards** — naming, file structure, error handling, log format
- [ ] **Architecture patterns** — when MVVM vs MVC, when monolith vs services, when SSR vs SPA
- [ ] **Debugging workflow** — your personal step-by-step (start with logs, then bisect, then ask AI)
- [ ] **Deployment steps** — per stack: how you take a feature to prod
- [ ] **Prompt patterns** — your favorite prompt shapes that consistently work

This is the document that lets a future-you (or future agent) pick up where you left off.

## Task 5 — Benchmark

Pick 5 representative tasks and time them on both your stack and Claude Code:

| Task | Local (Qwen) | Local (DeepSeek) | Claude Code |
|---|---|---|---|
| Flutter login screen from scratch | | | |
| MERN auth endpoint | | | |
| SwiftUI list with search | | | |
| Refactor a 200-line file | | | |
| Track down a null pointer | | | |

Track: wall-clock time, # of iterations, final quality (1-5).

This benchmark tells you **which tasks to keep on Claude Code** and which to migrate to local.

## Verify Week 4 is good

You can answer these without thinking:

- Which model do I open for which task type?
- Where do I store decisions so future-me and the AI both find them?
- What's my measured speed/quality ratio vs Claude Code?

If yes — you have a permanent system, not a demo.
