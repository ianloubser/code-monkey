# Architectural standards

These are defaults. Override per-repo in `AGENTS.md` when there's a reason.

## Repo layout

```
.
├── .github/workflows/   # CI + automation
├── .opencode/           # agent + skill + command config
│   ├── agent/           # role agents (build, review, plan, ...)
│   └── skills/          # reusable SKILL.md definitions
├── standards/           # symlink or copy of code-monkey/standards/
├── src/                 # or app/, lib/, services/
├── tests/               # or colocated *.test.* next to source
├── package.json / pyproject.toml / Cargo.toml
├── README.md
└── AGENTS.md
```

Keep top-level directories flat. If a directory needs a README to explain
itself, the structure is wrong.

## Public APIs

- **Backwards-compatible by default.** Don't rename or remove exported
  symbols without a deprecation cycle (mark `@deprecated`, ship alongside
  the new one for one release).
- **Explicit > clever.** No magic globals, no reflection-based DI.
- **Errors are values.** Return result types or throw at boundaries; never
  both in the same function.

## Boundaries

- **State at the edges.** Pure logic in the middle. I/O (DB, HTTP, FS)
  isolated to leaf modules.
- **No cross-layer imports.** `domain/` never imports `infra/`. Tests can
  import anything.
- **One reason to change per module.** If a file needs to be edited for
  two unrelated features, split it.

## Dependencies

- **Add a dep only if you need it now.** "We'll use it later" is a lie.
- **Prefer stdlib over deps.** A 20-line implementation beats a 2MB dep.
- **Pin in lockfile, not in manifest.** Let the resolver pick; commit
  the resolution.
- **One major bump at a time.** Mass upgrades hide regressions.
