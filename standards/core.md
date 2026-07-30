# Core standards (injected into every run)

These rules are always in effect. They override the repo-level `AGENTS.md`
if a conflict exists. Detailed guidance lives in the sibling files
(`architecture.md`, `code-patterns.md`, `testing.md`).

## Non-negotiables

1. **Smallest diff.** Do exactly what was asked. No drive-by refactors.
2. **Never push to `main`.** Branch, PR, wait for the user.
3. **No secrets in code, logs, or commits.** Use env vars / repo secrets.
4. **One issue, one PR.** Don't bundle unrelated changes.
5. **Idempotent re-runs.** Never break when re-triggered on the same
   issue/PR. Detect existing work and pick up where you left off.

## PR contract

Every PR you open must include:

- `Fixes #N` or `Closes #N` (or `Part of #N` for multi-PR work)
- One-paragraph summary of *what* and *why*
- "Test plan" with the commands run and their output
- "Risk/rollback" if you touched infra, deps, or auth

If any of these are missing, the PR is incomplete — fix it before you
finish.

## Money & quota awareness

- You're probably running on a free Zen model. State the model you used
  in the PR body when you finish.
- If a task seems too complex for a free model (large refactor across
  many files, deep debugging, architecture decision), say so in the PR
  body and recommend the user re-run with a paid model.
- Don't loop retrying a failing model. The chain tries the next one.
