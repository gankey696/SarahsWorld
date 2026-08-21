# Sarah's GitHub Learning Notes

Repository: `gankey696/SarahsWorld`
This is my home repo for learning GitHub and tracking what works from the sandbox.

## Authentication

- The Letta Cloud sandbox uses a GitHub App (`letta-integration` bot account) for `git` and `gh`.
- Do **not** run `gh auth login` or store tokens manually.
- For `gh`, run commands inside a clone or pass `-R owner/repo`.
- For commands that don't accept `-R`, prefix with `GH_REPO=owner/repo`.
- The GitHub integration is connected to `gankey696` but only grants access to `SarahsWorld`. Other repos under the same owner are not accessible.

## What Works

### Repository operations
- `git clone`, `git fetch`, `git push` — automatic token via credential helper.
- `gh repo view` — works on `SarahsWorld`.
- `gh repo view -R owner/repo` — works if the repo is public; private repos under the same owner fail with "no GitHub App installation".
- Repo settings query: `gh api repos/.../ --jq '{has_issues, has_wiki, has_pages, has_discussions, default_branch, visibility, fork, archived, disabled}'` — works.
- Topics query: `gh api repos/.../topics` — works (empty by default).
- Languages query: `gh api repos/.../languages` — works (empty, no code files yet).

### Issues and PRs
- `gh issue list` — works (empty by default).
- `gh issue create` — works (tested, created and closed issue #4).
- `gh issue close <num>` — works.
- `gh pr list` — works (empty by default).
- `gh pr create` — works (tested, created PR #5 from test-branch → main).
- `gh pr view <num> --json` — works (returns mergeable state, mergeStateStatus).
- `gh pr merge <num> --squash --delete-branch` — works (tested, merged PR #5).
- `gh pr checkout <num>` — works.
- `gh pr diff <num>` — works.
- `gh pr review --approve` — **fails** for your own PR (`Review Can not approve your own pull request`).
- `gh pr checks <num>` — works only if status checks exist.

### Branches
- `git checkout -b <name>` + `git push origin <name>` — works (creates remote branch).
- `git push origin --delete <branch>` — works (deletes remote branch).
- `gh api repos/.../branches` — works (but rate-limited frequently).

### Labels
- `gh label create <name>` — works.
- `gh label list` — works.

### Releases
- `gh release create <tag>` — works (tested, created v0.1.0).
- `gh release list` — works.
- `gh release delete <tag>` — deletes the release but **leaves the tag**.
- `git push origin --delete <tag>` — needed to fully remove the tag from remote.

### Actions
- Workflows run on `ubuntu-24.04` runners (not `ubuntu-latest` as configured — runner image is Ubuntu 24.04.4 LTS).
- `gh run list`, `gh run view`, `gh run watch` — work.
- `gh run view <id> --log` — works (full step logs with timestamps).
- Environment variables work at workflow, job, and step levels.
- `GITHUB_TOKEN` has limited permissions: Contents read, Metadata read, Packages read.
- `GITHUB_TOKEN` is NOT exposed as an env var inside steps (it's injected differently).
- `GITHUB_ACTOR` = `letta-integration[bot]`.
- `GITHUB_EVENT_NAME` = `push` (on push), `workflow_dispatch` (on manual trigger), `repository_dispatch` (on custom event).
- `workflow_dispatch` trigger — works via API.
- `repository_dispatch` trigger — works via API with custom payload.

### API
- `gh api repos/gankey696/SarahsWorld/hooks` — works; currently no webhooks configured.
- `gh api repos/.../events` — works (returns recent repo events).
- `gh api repos/.../commits` — works (returns commit history).
- `gh api repos/.../actions/runs` — works (returns workflow runs).
- `gh api repos/.../actions/workflows` — works (returns workflow definitions).
- `gh api repos/.../actions/permissions` — works (returns allowed_actions, enabled, sha_pinning_required).
- `gh api repos/.../collaborators` — works (returns gankey696).
- `gh api repos/.../contributors` — rate-limited.
- `gh api repos/.../actions/cache_usage` — 404 (not available for this repo).
- `gh api repos/.../actions/artifacts` — rate-limited.

## What Does Not Work

- `gh secret list` / `gh secret set` — HTTP 403: "Resource not accessible by integration". Secrets cannot be managed from the sandbox.
- `gh repo create` / `gh repo fork` — fails with broker 404 / "not connected for this owner".
- `gh webhook` — not a `gh` command; webhooks must be managed via UI or API.
- `gh api repos/.../actions/cache_usage` — 404 (not available for this repo or integration).
- Approving your own PR.
- Accessing private repos other than `SarahsWorld` under `gankey696`.
- Creating webhooks from the sandbox — no public endpoint to receive deliveries.
- Real-time event push — no inbound webhook receiver in the sandbox.

## Repo Details

- Owner: `gankey696`
- Name: `SarahsWorld`
- Visibility: public
- Default branch: `main`
- Features enabled: issues, wiki
- Features disabled: pages, discussions
- Not archived, not disabled, not a fork
- Collaborator: `gankey696` (owner)
- Bot identity in commits: `Letta Integration` (GitHub App)
- Bot actor in Actions: `letta-integration[bot]`

## Rate Limits

- The GitHub token broker can return 429 (`route_rps_rate_limit_exceeded`).
- Happens frequently when making multiple API calls in quick succession.
- Retry after a short delay (5-10 seconds) if this happens.
- Space out API calls to avoid hitting the limit.
- The limit is per-endpoint, not global — different endpoints can be called simultaneously.

## Workflow Examples

### Environment variables
```yaml
name: Environment Variables Test
on:
  push:
    branches: [main]

env:
  WORKFLOW_VAR: "set in workflow"

jobs:
  test:
    runs-on: ubuntu-latest
    env:
      JOB_VAR: "set in job"
    steps:
      - uses: actions/checkout@v4
      - name: Print environment variables
        env:
          STEP_VAR: "set in step"
        run: |
          echo "WORKFLOW_VAR=$WORKFLOW_VAR"
          echo "JOB_VAR=$JOB_VAR"
          echo "STEP_VAR=$STEP_VAR"
```

## Answered Questions

### Q1: How to trigger workflows from `gh` without a push?

Two methods work from the sandbox:

**`workflow_dispatch`** — manual trigger via API:
```bash
gh api repos/gankey696/SarahsWorld/actions/workflows/{workflow_id}/dispatches \
  --method POST -f ref=main
```
- Workflow must include `on: workflow_dispatch:`
- Returns 204 (no content) on success
- Run appears within seconds, completes in ~10s on ubuntu-latest

**`repository_dispatch`** — custom event trigger via API:
```bash
gh api repos/gankey696/SarahsWorld/dispatches \
  --method POST \
  -f event_type=sandbox_test \
  -f 'client_payload[message]=hello_from_sandbox'
```
- Workflow must include `on: repository_dispatch: types: [sandbox_test]`
- Payload available in workflow as `${{ github.event.client_payload }}`
- Returns 204 on success
- Both tested and verified working from the sandbox on 2026-08-21

### Q2: Can workflow-level env vars replace repository secrets for non-sensitive config?

**Yes.** Verified that env vars at workflow, job, and step levels all work correctly:
- `env:` at workflow level → available in all jobs/steps
- `env:` at job level → available in all steps in that job
- `env:` at step level → available only in that step
- `GITHUB_TOKEN` is NOT exposed as an env var inside steps (it's injected differently)
- `GITHUB_ACTOR` = `letta-integration[bot]`
- `GITHUB_REF` = `refs/heads/main` (on push to main)

For non-sensitive config (API endpoints, feature flags, build config), workflow-level env vars work fine. For secrets, you need repo secrets (which we can't manage from the sandbox — see "What Does Not Work").

### Q3: What is the best way to receive GitHub events in the sandbox without webhooks?

**Polling the Events API:**
```bash
gh api repos/gankey696/SarahsWorld/events --jq '.[] | {type, created_at, actor: .actor.login}'
```
- Returns recent events (PushEvent, CreateEvent, DeleteEvent, etc.)
- Limited to last 90 days, 300 events
- No webhook delivery needed
- Can be polled on a schedule (e.g., via heartbeat cron)

**Polling the Commits API:**
```bash
gh api repos/gankey696/SarahsWorld/commits --jq '.[] | {sha: .sha[0:8], message: .commit.message, date: .commit.committer.date}'
```
- More structured for tracking code changes
- Can filter by branch with `?sha=branch-name`

**Polling the Runs API:**
```bash
gh api repos/gankey696/SarahsWorld/actions/runs --jq '.workflow_runs[] | {id, status, conclusion, event}'
```
- Track workflow run results without webhooks

**Limitation:** No real-time push. All polling. The Events API is the closest to webhooks without configuring one. Webhooks can't be created from the sandbox (no `gh webhook` command, and API webhook creation is untested — would need `gh api repos/.../hooks --method POST` with a valid payload URL, which the sandbox doesn't have a public endpoint for).

## Workflow Files

### Dispatch test (`.github/workflows/dispatch-test.yml`)
Tests `repository_dispatch` and `workflow_dispatch` triggers. Logs the event name and payload.

### Env vars test (`.github/workflows/env-test.yml`)
Tests env vars at workflow, job, and step levels. Logs all vars and GITHUB_* context vars.
