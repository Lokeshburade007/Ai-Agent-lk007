# Docs — Personal AI Agent

Working notes for the 4-week build. The high-level plan is in [../readme.md](../readme.md); these files track **what's actually done, what's pending, and the decisions made along the way.**

## Index

**Start here:**
| File | What it covers |
|---|---|
| [quickstart.md](quickstart.md) | **5-minute "how do I use this thing" guide** — read this first |
| [aider-cheatsheet.md](aider-cheatsheet.md) | Every Aider slash-command + the two gotchas |

**The four-week build log:**
| File | What it covers |
|---|---|
| [week-1-foundation.md](week-1-foundation.md) | Env setup, Ollama, VS Code, model pulls, scaffolds |
| [week-2-prompts-and-brain.md](week-2-prompts-and-brain.md) | Extracting reusable intelligence from Claude Code |
| [week-3-autonomous-workflow.md](week-3-autonomous-workflow.md) | Continue.dev + Aider + lkai wrapper |
| [week-4-optimization.md](week-4-optimization.md) | RAG, project memory, handbook, benchmark template |

**Reference / future:**
| File | What it covers |
|---|---|
| [auto-file-edit-options.md](auto-file-edit-options.md) | Comparison of Aider / Cline / Continue.dev agent mode |
| [architecture-hybrid-mac-vps.md](architecture-hybrid-mac-vps.md) | Mac (brain) + VPS (body) hybrid — future remote-access path |

**Living docs (in [ai-workflows/](../ai-workflows/)):**
| File | What it covers |
|---|---|
| [../ai-workflows/handbook.md](../ai-workflows/handbook.md) | Personal AI engineering handbook — skeleton to fill in from real use |
| [../ai-workflows/benchmark.md](../ai-workflows/benchmark.md) | Fillable 5-task benchmark: Qwen vs DeepSeek vs Claude Code |
| [../ai-workflows/prompts/](../ai-workflows/prompts/) | Before / during / after phase prompts |
| [../ai-workflows/lkai](../ai-workflows/lkai) | The wrapper script — source of truth |
| [../ai-workflows/mkmem](../ai-workflows/mkmem) | Drop `.ai/` project-memory scaffold into any project |

## Convention

Each week file uses a checklist:

- [x] = done in a real session, with date
- [ ] = pending
- [~] = in progress / partial

Commands shown are the ones actually run, not idealized.
