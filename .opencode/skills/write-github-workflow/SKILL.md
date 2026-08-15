---
name: write-github-workflow
description: Create or edit GitHub Actions workflow files in this repo. Load when adding/modifying anything under .github/workflows/.
license: MIT
compatibility: opencode
metadata:
  audience: opencode
  workflow: github-actions
---

## What I do

Create or modify workflow files under `.github/workflows/`. Apply code-monkey
conventions so a new workflow works with the existing automation hub.

## When to use me

Use this skill when the user asks to "add a workflow", "create a GitHub
Action", "automate X on push/PR/schedule", or asks to modify an existing
workflow file.

## Conventions to apply

1. **Name the workflow after its job, not its trigger.** `name: deploy`,
   not `name: on-push-main`.
2. **Pin actions to a major version.** `actions/checkout@v4`,
   `actions/setup-node@v4`. Never `@latest` in committed workflows.
3. **For composite secrets, use `secrets: inherit`** when calling
   reusable workflows from the same org/user.
4. **For `uses: ianloubser/code-monkey/...`** reference the file with `@main`
   suffix: `uses: ianloubser/code-monkey/.github/workflows/reusable-opencode.yml@main`.
5. **Permissions: least privilege.** Start with `contents: read` and add
   `write` scopes only as needed. `id-token: write` is required for
   opencode's OIDC flow.
6. **Add `timeout-minutes`** to every job. Default to 10, bump to 30+
   for runs that open PRs.
7. **Add `concurrency`** for any workflow that reacts to PRs or issues
   to prevent overlapping runs on the same resource.
8. **For label-triggered automations**, use `on: issues: types: [labeled]`
   and check the label name in the job's `if:` — never auto-run on
   `opened` (causes accidental $$ burn on public repos).
9. **For the opencode action**, always set:
   - `share: "false"`
   - `use_github_token: true` (so commits come from the PAT, not the app)
   - explicit `model` (or rely on the caller to provide it)

## Skeleton

```yaml
name: <verb-noun>
on:
  <event>: { types: [<...>] }

jobs:
  <job>:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: write
    concurrency:
      group: <workflow>-${{ github.event.<id> }}
      cancel-in-progress: false
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 1
          persist-credentials: false
      - run: <command>
```

## Validation

After writing, run `yq eval '. | type' .github/workflows/<file>.yml` to
confirm valid YAML. If the workflow calls a reusable one, also verify
the `uses:` path matches the file name exactly.
