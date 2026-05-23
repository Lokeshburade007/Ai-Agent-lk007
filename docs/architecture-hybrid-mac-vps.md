# Hybrid Architecture — Mac (brain) + VPS (body)

**Question:** "I have a 2 CPU / 4GB RAM / 80GB VPS available. Is it useful for the personal AI agent?"

**Short answer:** Not as the LLM host. Very useful as the always-on orchestration layer.

## Hardware reality

| Model | RAM needed | Fits on 4GB VPS? |
|---|---|---|
| Qwen2.5-Coder 7B (Q4) | ~5–6GB | ❌ No |
| DeepSeek-Coder V2 (16B) | ~10GB | ❌ No |
| Qwen2.5-Coder 3B (Q4) | ~3GB | ⚠️ Barely, ~2–4 tok/s on 2 CPU — too slow for real use |
| Qwen2.5-Coder 1.5B (Q4) | ~1.5GB | ✅ Fits, ~5–10 tok/s, weak quality — autocomplete-tier only |

The killer is **2 CPU cores with no GPU**: even if RAM fit, you'd wait 30+ seconds per response. The VPS can't be your coding model host.

## What the VPS IS great for

| Use | RAM cost | Why it fits |
|---|---|---|
| **LiteLLM proxy** | ~100MB | Single API endpoint routing to Ollama-on-Mac, Claude API, etc. |
| **Tailscale exit node** | minimal | Stable URL → reaches your Mac's Ollama from anywhere |
| **Open WebUI** | ~500MB | Web frontend, use AI from phone/laptop on the road |
| **ChromaDB / Qdrant** (RAG) | 1–2GB | Vector index of your code + docs |
| **`nomic-embed-text`** (embeddings) | ~140MB | CPU-friendly embedding generation |
| **Scheduler / cron-driven agents** | minimal | "Summarize today's commits at 7pm", calls Mac's Ollama |
| **Gitea + webhooks** | ~300MB | Self-hosted git + agent-trigger events |
| **n8n / Activepieces** | ~500MB | Glue: Slack/email/calendar → agent → action |

## Recommended architecture

```
┌─────────────────────────────────┐         ┌──────────────────────────┐
│  Mac M2 (16GB) — when awake     │         │  VPS (2 CPU / 4GB)        │
│  • Ollama (Qwen, DeepSeek)      │◄────────│  • LiteLLM proxy          │
│  • Aider, Continue.dev          │         │  • Open WebUI (remote UI) │
│  • Heavy inference              │         │  • ChromaDB (RAG)         │
└─────────────────────────────────┘         │  • Cron-scheduled agents  │
              ▲                              │  • Webhooks / Tailscale   │
              │                              └──────────────────────────┘
              └── on when you're working      always on, always reachable
```

## Cheapest immediate win — 30 min setup

1. Install **Tailscale** on Mac + VPS (free tier).
2. On Mac, expose Ollama: `OLLAMA_HOST=0.0.0.0:11434 ollama serve` (or set in brew services).
3. On VPS, `docker run` Open WebUI pointed at `http://<mac-tailscale-ip>:11434`.
4. Bookmark `https://<vps>.<your-domain>` on your phone.

You now have your local LLM accessible from anywhere, no API costs.

## Fallback when Mac is asleep

For scheduled agents that need to run while your Mac is off, point the VPS scheduler at a cloud API (free Claude tier, Gemini, OpenRouter) or a tiny VPS-resident 1.5B model. Code path:

```python
def llm_call(prompt):
    if mac_reachable():
        return ollama_remote(prompt, base="http://<mac-ts-ip>:11434")
    else:
        return cloud_fallback(prompt)
```

## Why this beats Claude Code on one axis

Claude Code is interactive-only. This setup gives you:
- **Scheduled background jobs** ("daily standup summary at 9am")
- **Webhook-triggered work** ("when a PR is opened, run my review prompt")
- **Persistent always-on chat UI** from any device
- **A vector index of your entire codebase** the model can retrieve from

That's a category Claude Code doesn't even compete in.

## What NOT to do with the VPS

- ❌ Don't try to run 7B+ models there. The math doesn't work.
- ❌ Don't replicate everything from your Mac. The Mac is the brain; the VPS is the integration layer.
- ❌ Don't put secrets on the VPS without encryption-at-rest. It's exposed.
