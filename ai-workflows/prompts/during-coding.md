# During-Coding Prompt

Load with `--read` for the implementation phase, after the plan is agreed.

---

While implementing:

**Match before invent.** Before you write any new code, look at how similar code is structured elsewhere in this repo and match its conventions. New helpers, new abstractions, new patterns are red flags unless explicitly requested.

**One file at a time.** Don't sprawl a single task across many files. If the task forces ≥3 files, surface that to me before proceeding — it usually means I asked for the wrong thing.

**Imports are part of the code.** Every identifier you use must be explicitly imported. After writing a function, look at every name that isn't local — confirm each one has an import line. The single most common bug at this layer.

**Trace before you trust.** For every function you write, mentally execute it once with a real-looking input. Walk through what each line returns. If you can't predict the output, the code is too clever — simplify.

**No "in case we need it later."** If we don't need a parameter now, don't add it. If we don't need a config option now, don't add it. We can add it when we need it.

**No premature abstractions.** Three similar lines is not duplication worth solving. Three similar files is the bar.

**Errors are values.** Don't swallow them with empty `catch {}`. Either handle the specific case meaningfully (and say *why*) or let them propagate.

**Comments only when WHY is non-obvious.** Comments that restate the code are noise. Comments that explain a constraint, an invariant, a workaround for a known bug — keep those.

**No filler.** Don't add `// TODO: improve later`, `// production-ready`, `// for now`. Either write it right or surface that you can't, and pause.

When you finish a file, run through this checklist in your head:
- [ ] Every import resolved
- [ ] No unused variables, no dead code
- [ ] Function names describe what they do, not how
- [ ] Error paths considered
- [ ] No commented-out code left behind

If anything fails, fix before moving on.
