# code-monkey

Self-expanding automation hub for your personal GitHub. Drop ideas in
issues, label them `build`, and opencode opens a PR.

Built on top of the upstream
[`anomalyco/opencode/github`](https://github.com/anomalyco/opencode/tree/dev/github)
action. No paid LLM API required — runs on free Zen models by default.

## How it works

```
issue ── label "build" ──▶ issue-to-pr.yml ──▶ opencode (free model) ──▶ PR
                                                                      │
PR comment "/oc fix this" ──▶ opencode.yml ──▶ opencode ──▶ commit ───┘
PR opened ──▶ pr-review.yml ──▶ opencode (reviewer) ──▶ review comment
```

Everything is a `workflow_call` into `.github/workflows/reusable-opencode.yml`,
which handles a free-model fallback chain so a single 429/rate-limit doesn't
kill a run.

## One-time setup

### 1. Create a classic PAT

Fine-grained PATs can't create repos or pre-grant workflow scope to repos
that don't exist yet, so use a **classic PAT** with these scopes:

- `repo` (full)
- `workflow`

Save it as the secret **`OPENCODE_GH_PAT`** in this repo (Settings → Secrets
and variables → Actions).

### 2. Create a free Zen API key

1. Sign in at <https://opencode.ai/auth>.
2. Copy the API key.
3. Save it as the secret **`OPENCODE_API_KEY`** in this repo.

No credit card needed — the default model chain is fully free.

### 3. Onboard a new repo

Actions → **onboard-repo** → Run workflow.

Inputs:
- `repo_name` — lowercase, no spaces
- `description` — optional
- `private` — leave unchecked (public = free runner minutes)
- `template` — `minimal`, `node`, or `python`

The workflow creates the repo, scaffolds template files, copies both
secrets to the new repo, adds topics, and opens a welcome issue labeled
`build` so you can see the whole pipeline end-to-end.

## Day-to-day usage

In any repo that has been onboarded:

- **Issue → PR:** Open an issue, add the `build` label. opencode opens a
  draft PR with a plan; reply `/oc go` to implement.
- **PR review:** Every PR gets an automatic review from the `review` agent.
- **Ad-hoc commands:** On any issue or PR, comment `/oc <instruction>`.
  Examples: `/oc explain this issue`, `/oc add a /healthz endpoint`,
  `/oc review`.

## Layout

```
.
├── .github/workflows/
│   ├── reusable-opencode.yml      # workflow_call — the engine
│   ├── opencode.yml               # /oc comment trigger
│   ├── issue-to-pr.yml            # build/plan label trigger
│   ├── pr-review.yml              # auto-review
│   ├── onboard-repo.yml           # dispatch: create + scaffold new repo
│   └── scheduled-maintenance.yml  # weekly TODO/dep scan
├── .opencode/
│   ├── agent/                     # shared agent defs (build, review, plan)
│   └── skills/                    # shared skills (write-github-workflow, ...)
├── standards/                     # shared docs (core, architecture, patterns, testing)
├── templates/                     # scaffolded into new repos at boot
│   ├── minimal/
│   ├── node/
│   └── python/
├── opencode.json                  # model, MCP, permission defaults
└── AGENTS.md                      # house rules for opencode
```

## Copy-at-bootstrap vs runtime-shared

`reusable-opencode.yml` checks out `ianloubser/code-monkey` as a sidecar
(`.code-monkey/`) on every run and **merges** the shared assets into the
consumer repo's `.opencode/`. Repo-local files win (`cp -n`).

| Asset | Where it lives | How consumer gets it | Drift behavior |
|---|---|---|---|
| Repo-specific conventions | `templates/<name>/AGENTS.md` | Copied at scaffold | Frozen at bootstrap |
| CI workflows, manifests | `templates/<name>/` | Copied at scaffold | Frozen at bootstrap |
| Shared agents | `.opencode/agent/` (hub) | Merged every run, repo-local wins | **Live** — push to `main` |
| Shared skills | `.opencode/skills/` (hub) | Merged every run, repo-local wins | **Live** — push to `main` |
| Shared standards | `standards/` (hub) | Sidecar checkout, read on demand | **Live** — push to `main` |

To update something for every consumer repo at once, edit it in
code-monkey. To change a per-repo rule, edit the consumer repo's local
`AGENTS.md` or add a repo-local agent/skill with the same name.

## Adding a new shared skill

1. Create `.opencode/skills/<name>/SKILL.md` in this repo.
2. Add YAML frontmatter with `name` (1–64 chars, lowercase + hyphens)
   and `description` (1–1024 chars, specific).
3. Push to `main`. Next opencode run in any consumer repo picks it up.
4. Update this README if the skill changes the workflow contract.

## Configuration knobs

The reusable workflow accepts these inputs (per caller):

| Input | Default | Purpose |
|---|---|---|
| `model` | _empty_ | Single model override; bypasses the chain |
| `model_chain` | big-pickle → deepseek-v4-flash-free → nemotron-3-ultra-free → mimo-v2.5-free | Comma-separated fallback chain |
| `agent` | repo default | `build`, `review`, or `plan` |
| `prompt` | _empty_ | Override the default prompt |
| `allowed_actors` | `ian` | Comma-separated GitHub usernames |
| `timeout_minutes` | `20` | Per-run timeout |

Zen rotates free models periodically. To refresh, update the default
`model_chain` in `reusable-opencode.yml` (or use
`scheduled-maintenance.yml` to automate the detection — see TODO).

## Safety

- All work lands on a branch; never pushes to `main` directly.
- Public repos: actor allowlist gates every trigger.
- `share: false` — opencode sessions are not published.
- Fork PRs never receive secrets (GitHub default).
- `concurrency` group per issue/PR prevents overlapping runs.

## License

MIT (do whatever).
