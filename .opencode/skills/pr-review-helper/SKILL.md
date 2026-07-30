---
name: pr-review-helper
description: Write a code-monkey-style PR review with structured sections, file:line references, and a clear verdict. Used by the review agent.
license: MIT
compatibility: opencode
metadata:
  audience: reviewers
  workflow: pr-review
---

## What I do

Produce a PR review comment matching the code-monkey template.

## Output format

```markdown
## Review

### Summary
<1-2 sentences: what this PR does>

### Correctness
- `path/to/file.ts:42` — <bug, race, missing error handling, ...>
- ...

### Style
- `path/to/file.ts:13` — <naming, dead code, pattern mismatch, ...>
- ...

### Tests
- <new behavior covered?> <edge cases?> <integration vs unit?>
- ...

### Risk
- <deploy impact, auth/data exposure, performance, rollback story>

### Verdict
LGTM | Request changes | Needs discussion
```

## Rules

1. **Be concrete.** Every concern references a `file:line` (or a
   specific commit if the line is gone).
2. **Be terse.** Bullets, not paragraphs. One sentence per concern.
3. **Don't restate the diff.** The reviewer can read the PR. State
   what you found, not what was changed.
4. **Severity per concern.** Prefix with `[blocker]`, `[nit]`, or
   `[question]` when ambiguous.
5. **Verdict rules:**
   - `LGTM` — would merge as-is.
   - `Request changes` — at least one `[blocker]`.
   - `Needs discussion` — open questions only, no blockers.
6. **Don't approve your own work.** If you opened the PR, stay out of
   the review (or recuse yourself in the first line).
7. **No drive-by nits on style** if the repo's `AGENTS.md` doesn't
   define the style. Read it first.
