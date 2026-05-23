# Before-Coding Prompt

Load with `--read` before any non-trivial task. Forces the agent to **plan before typing**.

---

Before writing any code, do the following — in this order — and **stop after each numbered item to let me confirm**:

**1. Repeat the goal in one sentence**, in your own words. If your sentence reveals a misunderstanding, I'll correct it now — much cheaper than after you've written 100 lines.

**2. List the files you will read or modify.** Don't say "the relevant files." Name them. If the repo map doesn't make this obvious, ask me which files to look at.

**3. Describe the data flow** in 3–5 lines. What goes in, what comes out, which functions/components touch it in between. If it crosses a layer boundary (UI ↔ state ↔ network ↔ DB), name the boundary.

**4. List risks or assumptions** you're making. Examples:
- "I'll assume the API returns ISO timestamps — is that right?"
- "If two users do this concurrently, we have a race; do we care?"
- "This will break the existing `/v1/foo` endpoint shape — is that acceptable?"

**5. Propose the smallest change that solves the goal.** Not the "right" architecture for some future. The smallest. Then ask me whether to proceed, or whether I want a different scope.

**Hard rule:** do not produce SEARCH/REPLACE blocks in this phase. Only produce the plan. We agree on the plan first, then you implement.

If the task is genuinely trivial (typo fix, rename, one-line bug), say "this is trivial — proceeding" and just do it. Use judgment.
