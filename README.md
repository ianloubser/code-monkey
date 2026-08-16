# code-monkey

Self-expanding automation hub for your personal GitHub. Drop ideas in
issues, label them `build`, and opencode opens a PR.

Built on top of the upstream
[`anomalyco/opencode`](https://github.com/anomalyco/opencode) CLI. The
**build path** (issue → PR) runs `opencode run` directly behind code-monkey's
own dispatcher (it owns branch + draft-PR + commits); `/oc` comments and PR
review still use the upstream `opencode github run` action. No paid LLM API
required — runs on free Zen models by default.

## How it works

```
issue ── label "build" ──▶ issue-to-pr.yml ──▶ reusable-opencode.yml (dispatcher)
  └─ dispatcher: create branch agent/<n>-<kebab> + push + open draft PR
  └─ opencode run --agent build (free-model fallback chain)
       └─ agent: implement in milestones, commit+push after each (timeout-safe)
       └─ agent: run lint/tests, finalize PR body, `gh pr ready` when green
PR comment "/oc fix this" ──▶ opencode.yml ──▶ opencode github run ──▶ commit ──▶ push
PR ready ──▶ pr-review.yml ──▶ opencode github run (reviewer) ──▶ review comment
```

Everything is a `workflow_call` into `.github/workflows/reusable-opencode.yml`,
which handles a free-model fallback chain so a single 429/rate-limit doesn't
kill a run. The build flow (prompt + branch/PR/commit contract) lives in the
reusable workflow and `build.md`, shipped from `code-monkey@main` — so updates
land in every consumer repo automatically, no per-repo edits.

## One-time setup

### 1. (Only for onboarding new repos) Create a classic PAT

Fine-grained PATs can't create repos or pre-grant workflow scope to repos
that don't exist yet, so use a **classic PAT** with these scopes:

- `repo` (full)
- `workflow`

Save it as the secret **`OPENCODE_GH_PAT`** in this repo (Settings → Secrets
and variables → Actions).

Everyday maintenance — issue-to-pr, pr-review, `/oc` comments, and the
weekly scheduled-maintenance — runs on the repo's own **`GITHUB_TOKEN`**;
no PAT is needed for those. The PAT is only required by **onboard-repo**,
which creates new repos and mirrors secrets into them.

### 2. Create a free Zen API key

1. Sign in at <https://opencode.ai/auth>.
2. Copy the API key.
3. Save it as the secret **`OPENCODE_API_KEY`** in this repo.

No credit card needed — the default model chain is fully free.

### 3. Onboard a new repo

Actions → **onboard-repo** → Run workflow.

> **Required repo setting (every onboarded repo, including this one):**
> Settings → Actions → General → Workflow permissions → enable
> **"Allow GitHub Actions to create and approve pull requests"**. The build
> path creates draft PRs and marks them ready using the runner's
> `GITHUB_TOKEN`; without this toggle, `gh pr create` / `gh pr ready` return
> 403 even though the workflow declares `pull-requests: write`.

Inputs:
- `repo_name` — lowercase, no spaces
- `description` — optional
- `private` — leave unchecked (public = free runner minutes)
- `template` — `minimal`, `node`, or `python`

The workflow creates the repo, scaffolds template files, copies the model
API key to the new repo, adds topics, and opens a welcome issue labeled
`build` so you can see the whole pipeline end-to-end.

## Day-to-day usage

In any repo that has been onboarded:

- **Issue → PR:** Open an issue, add the `build` label. The dispatcher
  pushes branch `agent/<n>-<kebab>` and opens a **draft PR** immediately;
  the builder then implements in milestones, committing and pushing after
  each one — a timed-out run keeps everything pushed so far. When the work
  is done and tests pass, the builder marks the PR ready for review.
- **PR review:** Every PR gets an automatic review from the `review` agent —
  but only once it is **not a draft** anymore. Drafts are skipped until
  marked ready.
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

> **Note:** `issue-to-pr.yml` is now a thin caller — the build prompt and
> branch/PR/commit contract live in `reusable-opencode.yml` + `build.md`,
> both shipped from `code-monkey@main`. So evolving the build flow no longer
> requires touching already-onboarded repos. (`pr-review.yml` and
> `opencode.yml` still inline their trigger config and remain frozen at
> bootstrap until the review/comment paths are migrated too.)

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
| `trigger` | _empty_ | `issue-build` = use code-monkey's dispatcher (`opencode run`, owns branch/PR/commits). Empty = legacy `opencode github run` path. |
| `prompt` | _empty_ | Override the default prompt (legacy path) |
| `allowed_actors` | `ian` | Comma-separated GitHub usernames |
| `timeout_minutes` | `20` | Per-run timeout |

Zen rotates free models periodically. To refresh, update the default
`model_chain` in `reusable-opencode.yml` (or use
`scheduled-maintenance.yml` to automate the detection — see TODO).

## Safety

- All work lands on a branch; never pushes to `main` directly.
- Draft PRs are never auto-reviewed — reviews fire only when a PR is marked
  ready.
- Public repos: actor allowlist gates every trigger.
- `share: false` — opencode sessions are not published.
- Fork PRs never receive secrets (GitHub default).
- `concurrency` group per issue/PR prevents overlapping runs.

## License

MIT (do whatever).
