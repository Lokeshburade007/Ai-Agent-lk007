# Personal AI Engineering Handbook

The accumulated playbook for how I work with this local agent. **A skeleton — fill in as patterns emerge from real use.** Empty sections aren't gaps; they're invitations.

Last reviewed: 2026-05-23 (skeleton)

---

## 1. Daily workflow

The shape of a normal coding session.

### Starting a task
> Fill in once you have a routine. Example:
> 1. `cd ~/project && lkai --stack <X>` (interactive)
> 2. Tell it the goal in 1–2 sentences
> 3. Let it propose the change, review the diff
> 4. Accept or redirect

### When to switch from `lkai` to Continue
> Tab autocomplete vs. terminal agent — when each wins.

### When to drop to plain `aider` / `ollama run`
> Edge cases where the wrappers are wrong.

### When to escalate to Claude Code
> The 20% local model can't handle. Track what kinds of tasks trip Qwen, not as a list — as a *pattern*.

---

## 2. Coding standards (cross-stack)

Personal style choices that apply everywhere. Stack-specific rules live in [ai-memory/rules/](../ai-memory/rules/).

### Naming
> Variables, functions, files. The personal preferences that aren't industry-standard.

### Function shape
> Max lines per function. When to break out. When to inline.

### Comments
> What you write WHY-comments about. What you never comment.

### Tests
> Default test framework per stack. Coverage philosophy. Property-based / snapshot / unit ratios.

### Logging
> Format, levels, what gets logged, what never gets logged.

---

## 3. Architecture patterns I default to

### Per-stack defaults that have actually worked
> Cite real projects where these held up.

- **Node**: <decision>
- **React**: <decision>
- **Flutter**: <decision>
- **SwiftUI**: <decision>
- **Go**: <decision>

### Patterns I've tried and abandoned
> Just as valuable. Why each one didn't survive contact with reality.

---

## 4. Debugging workflow

My personal step-by-step before I ask the agent for help.

### Step 1 — Read the error literally
> Don't skim. The fix is usually in the message.

### Step 2 — Check recent diffs
> `git log --oneline -5` + `git diff HEAD~1`.

### Step 3 — Hypothesize, then test
> One hypothesis at a time. Cheapest test that disproves it.

### Step 4 — Ask the agent
> Use `lkai --stack <X> --concern debugging --debug` once I've done steps 1–3.

### Step 5 — Escalate
> When local + Claude both fail, post in <community / forum>.

### Stack-specific debugging
> See [ai-memory/memory/debugging/](../ai-memory/memory/debugging/) for per-stack playbooks.

---

## 5. Deployment steps

The recipe per project type. Should be runnable from memory.

### MERN (Node + React)
> Fill in: where the backend deploys, where the frontend deploys, env management, CI config.

### Flutter
> iOS: TestFlight + App Store steps. Android: Play Console internal track.

### SwiftUI
> Same as Flutter iOS side, plus signing.

### Go service
> Container build, registry, target host.

---

## 6. Prompt patterns that consistently work

The shapes of prompts that produce good output. Update when one earns its keep.

### Pattern: "Plan, then implement"
```
agent --phase before --stack <X> --message "<task>"
```
Why it works: forces explicit assumptions before code is written.

### Pattern: "Investigate, then fix"
```
agent --stack <X> --debug --concern debugging --message "<symptom>"
```
Why it works: stack debugging playbook + diagnostic prompt prevents jumping to a fix.

### Pattern: <to be discovered>
> Add when you find one.

### Anti-patterns I keep falling into
> Things I say to the agent that produce bad output.

- "Make it better" — too vague, agent invents.
- "Add proper error handling" — agent goes overboard, wraps everything in try/catch. Say *what* errors to handle.
- <to be discovered>

---

## 7. Operating notes for the agent itself

### Tradeoffs I keep re-deciding
> Things I find myself re-evaluating per project. Capture the decision shape so it's faster next time.

### Things I've learned about Qwen specifically
> Quirks of the local 7B model that affect prompt design.

### Things I've learned about DeepSeek
> When DeepSeek is worth the slower inference.

### Things I've learned about Continue.dev
> Use cases where it beats Aider; cases where it loses.

---

## How to use this document

- **Read it weekly** to catch drift.
- **Update from real lessons**, not aspirational ones.
- **Date each section** when you change it — `<!-- updated 2026-MM-DD -->`.
- **Prune** what no longer matches reality. Stale rules are worse than no rules.

If this document grows past ~10 pages, split sections into their own files. Long handbooks don't get read.
