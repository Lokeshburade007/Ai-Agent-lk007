# After-Coding Prompt

Load with `--read` once the implementation is complete. Forces the agent to **self-review before declaring done**.

---

After producing the edits, do all of the following before you declare the task complete:

**1. Re-read every modified file.** End to end. Not "diff to last edit" — the whole file. You're looking for what you wrote vs. what's actually there now.

**2. Run through the user's original ask one more time.** Does the code you wrote do what was asked? Underscore: not "approximately", "actually". If you compromised on any sub-requirement, name it now.

**3. List potential bugs you introduced.** This is not optional. Examples:
- "I changed the signature of `getUser`; callers in `routes/admin.ts` may break."
- "I added a `userId` field, but didn't migrate existing rows."
- "The new test relies on `Date.now()`; will flake in CI if not mocked."

If you think there are no bugs, say "I see no bugs introduced" — but make sure you actually looked, don't just assume.

**4. List the test cases that should exist.** Even if I didn't ask for tests. Just the list — golden path + 2–3 edge cases. I'll decide which to write.

**5. Suggest one improvement you deliberately did NOT make**, and explain why you deferred it. Examples:
- "I could extract `validateEmail` into a shared util, but it's only used here — premature."
- "The query could use a compound index — I'll wait for evidence of slowness."

This catches the cases where you over-built.

**6. Tell me how to verify.** A concrete command or click-path I can run. Not "run the tests" — *which* test file, *which* npm script.

Only after all six steps should you say "ready for review."

If at step 1–5 you discover a real problem, fix it and restart from step 1. **Don't ship work you yourself wouldn't merge.**
