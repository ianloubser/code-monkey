---
name: scaffold-repo
description: Bootstrap a new personal repo with code-monkey defaults: workflows, AGENTS.md, opencode.json, and required secrets.
license: MIT
compatibility: opencode
metadata:
  audience: opencode
  workflow: bootstrap
---

## What I do

Create a new repo that is fully wired into the code-monkey automation
hub. After running this, the new repo will:

- Have the standard workflows (issue-to-pr, pr-review, /oc comment trigger)
- Have `AGENTS.md` and `opencode.json` from the matching template
- Have the `OPENCODE_API_KEY` and `OPENCODE_GH_PAT` secrets set
- Be on the `main` branch with a green initial commit

## When to use me

Use when the user says "create a new repo for X", "bootstrap a project
called Y", or "scaffold a repo".

## How

**Always prefer the workflow, not raw git.** Trigger the
`onboard-repo` workflow in `ianloubser/code-monkey` via the GitHub API:

```bash
gh workflow run onboard-repo \
  -R ianloubser/code-monkey \
  -f repo_name=<name> \
  -f description="<desc>" \
  -f private=false \
  -f template=minimal|node|python
```

Templates:

- `minimal` — just the automation wiring, no language stack
- `node` — adds `package.json` and a Node CI workflow
- `python` — adds `requirements.txt` and a Python CI workflow

## Rules

1. **Public by default.** Private repos don't get free GitHub Actions
   minutes. Only suggest private if the user explicitly asks or the
   content must be private.
2. **Validate the name.** Lowercase, no spaces, alphanumeric + `-` + `.` + `_`.
3. **Confirm with the user before triggering** the workflow — they may
   want a different template or description.
4. **Don't create the repo manually** (no `gh repo create` followed by
   `git push`). The workflow handles secrets, topics, and the welcome
   issue in one go.
5. **After triggering**, link the user to the workflow run URL so they
   can watch it.

## What NOT to do

- Don't add collaborators — the user owns these repos solo.
- Don't push to a fresh repo from a local checkout. The workflow is
  the single source of truth for the initial state.
- Don't include the PAT or API key in any local file. They're secret
  inputs to the workflow, period.
