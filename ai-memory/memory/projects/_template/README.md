# `.ai/` — project memory

Drop this folder into any project so the agent (Aider + Continue) has stable context every time you open the project.

## Files

| File | Purpose | Update when |
|---|---|---|
| `architecture.md` | What this project is, key boundaries, why it exists. The "if I forgot everything, this is what I'd want to know" doc. | New module added, major refactor, scope changes |
| `api.md` | Inventory of endpoints / public functions. One line each, link to the file. | New endpoint shipped |
| `data-model.md` | Schema overview — tables/collections, key relationships, why the schema looks like it does. | Schema migrations |
| `decisions/NNNN-<slug>.md` | Architecture Decision Records — one per non-trivial design call. | When you (or the agent) make a non-obvious choice |
| `patterns.md` | Reusable patterns in this codebase the agent should follow. | When a new pattern emerges |
| `glossary.md` | Domain terms with definitions. | When a term gets adopted |

## How to wire it into Aider sessions

Add to the project's `.aider.conf.yml`:

```yaml
read:
  - .ai/architecture.md
  - .ai/patterns.md
  # Decisions and api are too large to always load. Reference them by name in prompts:
  # "see .ai/decisions/0004-why-postgres.md"
```

Or per session:
```bash
lkai --stack node \
     --read .ai/architecture.md \
     --read .ai/patterns.md \
     src/something.ts
```

## Why this exists

7B local models forget everything between sessions. Without project memory, every prompt starts from zero and the agent re-asks the same questions ("which state library do we use?"). With these files, the agent walks in knowing the project's shape and the patterns you've already settled on.

The doc files **also help future-you.** Treat them as a living README that lives next to the code, not in someone's head.

## Drop it into a new project

From your agent monorepo:
```bash
mkmem ~/path/to/new-project   # see ai-workflows/mkmem
```

Or manually:
```bash
cp -R ~/Documents/LK/Important/PersonalAIAgent/ai-memory/memory/projects/_template/.ai ~/path/to/new-project/
```
