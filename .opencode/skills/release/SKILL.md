---
name: release
description: Cut a release for this repo: bump version, draft notes from merged PRs, create a GitHub release, optionally publish.
license: MIT
compatibility: opencode
metadata:
  audience: maintainers
  workflow: release
---

## What I do

Prepare and (optionally) cut a release for the current repo:

1. Detect the version-bump scope (major / minor / patch) from the
   merged PRs since the last tag.
2. Bump the version in the manifest (`package.json`, `pyproject.toml`,
   `Cargo.toml`, etc.).
3. Generate release notes from PR titles + labels.
4. Create a git tag and a GitHub Release.

## When to use me

Use when the user says "cut a release", "ship v1.2.0", "publish a new
version", or asks for a changelog.

## Process

1. **Read recent merged PRs.** Use `gh pr list --state merged --limit 50
   --json number,title,labels,author,mergedAt`.
2. **Group by label:**
   - `breaking` / `major` → MAJOR bump
   - `feat` / `enhancement` → MINOR bump
   - everything else (fix, perf, chore, docs) → PATCH bump
   - Highest wins.
3. **Confirm with the user.** Always state the proposed bump and let
   them confirm before writing tags or releases. Free models are cheap
   to ask; wrong releases are expensive.
4. **Generate notes:**
   ```
   ## What's changed
   - <grouped bullets, newest first>

   ## Full changelog
   https://github.com/<owner>/<repo>/compare/<prev-tag>...<new-tag>
   ```
5. **Create the tag + release:** `gh release create <tag> --title
   "<tag>" --notes-file notes.md --generate-notes` (omit
   `--generate-notes` if you already wrote detailed notes).
6. **Update the manifest version** *before* tagging if the tag is meant
   to point at the release commit.

## Conventions

- **Tag format:** `v<MAJOR>.<MINOR>.<PATCH>` (e.g. `v1.4.2`).
- **Pre-release:** `v1.4.2-rc.1`. Bump suffix on each RC.
- **Never force-push tags.** If you need to fix a release, cut a new
  one and mark the old as pre-release / deprecated.
- **Draft first, publish second.** Default to `--draft` so the user
  can review notes before going public.

## What NOT to do

- Don't auto-publish. Always draft.
- Don't bump major without a confirmation.
- Don't include `dependabot` PRs in user-facing notes — bucket them
  under "Maintenance" at the bottom.
