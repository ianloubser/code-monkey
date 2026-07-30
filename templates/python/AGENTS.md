# AGENTS.md

This repo is wired into the **code-monkey** automation hub. Shared rules
(PR contract, scope discipline, no-secrets, free-model awareness) are
injected at runtime from `ian/code-monkey/standards/core.md`.

## Stack

- Language: **Python 3.12**
- Lint: `ruff check .`
- Test: `pytest`
- Deps: pinned in `requirements.txt`

## Conventions

- Type hints on every public function signature.
- `from __future__ import annotations` at the top of every module.
- Prefer `pathlib.Path` over `os.path`.
- Prefer f-strings over `.format()` or `%`.
- Use `pydantic` for data models at boundaries; plain dataclasses inside.
- Tests colocated as `test_<module>.py` next to `<module>.py` (or in
  `tests/` if integration).

## Runtime assets

- Shared agents → `.opencode/agent/` (repo-local wins)
- Shared skills → `.opencode/skills/`
- Shared standards → `.code-monkey/standards/` (read on demand)

## CI

`ruff check .` and `pytest` must pass before opening a PR. The
`ci.yml` workflow runs them on every push to `main` and on PRs.
