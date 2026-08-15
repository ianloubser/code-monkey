# AGENTS.md

This repo is wired into the **code-monkey** automation hub. Shared rules
(PR contract, scope discipline, no-secrets, free-model awareness) are
injected at runtime from `ianloubser/code-monkey/standards/core.md` — you do
**not** need to repeat them here.

This file is for **repo-specific** rules only.

## Repo-specific rules

_(none yet — add stack conventions, naming, or local overrides below)_

<!-- Example:
## Stack
- Language: TypeScript (Node 20)
- Test: vitest
- Lint: eslint + prettier
- DB: postgres via prisma

## Naming
- Components: PascalCase, in `src/components/`
- Routes: kebab-case, in `src/routes/`
-->

## Runtime assets

The following are merged in at run time from the code-monkey sidecar
(`.code-monkey/`):

- **Shared agents** → `.opencode/agent/` (repo-local wins on conflict)
- **Shared skills** → `.opencode/skills/`
- **Shared standards** → `.code-monkey/standards/` (read on demand)

To override a shared agent or skill, create a file with the same name
in the corresponding repo-local directory; `cp -n` keeps the local
copy.

## File conventions

- Reusable workflows from this hub are referenced as
  `uses: ianloubser/code-monkey/.github/workflows/<file>@main`.
- Secrets live in repo Settings → Secrets. Never in code.
