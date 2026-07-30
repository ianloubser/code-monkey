---
name: dependency-update
description: Scan the repo for outdated dependencies and open a single grouped PR with safe bumps. Skips major version bumps unless asked.
license: MIT
compatibility: opencode
metadata:
  audience: opencode
  workflow: maintenance
---

## What I do

Audit and update outdated dependencies for the current repo, then open
a PR.

## When to use me

Use when the user says "bump deps", "update dependencies", "check for
outdated packages", or on a weekly maintenance run.

## Per-stack

### Node
```bash
npm outdated --json
# For safe updates (patch + minor):
npx npm-check-updates -i --target minor
# Or non-interactive, group patch+minor:
npx npm-check-updates -u --target minor
npm install
npm test
```

### Python
```bash
pip list --outdated --format=json
# Safe updates (pin-compatible):
pip-compile --upgrade
# Or ad-hoc:
pip install -U <package>
```

## Rules

1. **Group into one PR per ecosystem.** Don't open 12 separate PRs.
2. **Patch + minor only by default.** Major bumps get their own PR with
   a "breaking changes" section in the body.
3. **Run the full test suite after updates.** If tests fail, revert the
   offending bump and note it in the PR body.
4. **Don't bump transitive deps directly.** Let the resolver do it.
5. **Pin in the lockfile, not the manifest.** Bumping `package.json` to
   exact versions hides future updates.

## PR body

```
## Bumps
- <package>: <old> → <new> (patch|minor|major)
- ...

## Test plan
- `npm test`: PASS
- `npm run lint`: PASS

## Risk
Low. Patch + minor only. CI green.
```

## What NOT to do

- Don't `--force` a push.
- Don't add new deps while bumping.
- Don't bump framework majors (Next 14 → 15, React 18 → 19) without an
  explicit ask.
