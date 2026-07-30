# AGENTS.md

This repo is wired into the **code-monkey** automation hub. Shared rules
(PR contract, scope discipline, no-secrets, free-model awareness) are
injected at runtime from `ian/code-monkey/standards/core.md`.

## Stack

- Language: **TypeScript** on Node 20
- Lint: `npm run lint`
- Test: `npm test` (vitest by default — override in `package.json`)

## Conventions

- ESM, `"type": "module"` in `package.json`.
- Prefer `import type` for type-only imports.
- No `any` in committed code. Use `unknown` + narrowing.
- One concern per file. If a file is over ~200 lines, consider splitting.
- All exported functions have explicit return types.

## Runtime assets

- Shared agents → `.opencode/agent/` (repo-local wins)
- Shared skills → `.opencode/skills/`
- Shared standards → `.code-monkey/standards/` (read on demand)

## CI

`npm run lint` and `npm test` must pass before opening a PR. The
`ci.yml` workflow runs them on every push to `main` and on PRs.
