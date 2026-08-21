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

### Issues and PRs
- `gh issue list` — works on `SarahsWorld`.
- `gh pr list` — works on `SarahsWorld`.
- `gh pr create` — works.
- `gh pr checkout <num>` — works.
- `gh pr diff <num>` — works.
- `gh pr view <num>` — works.
- `gh pr merge <num> --squash` — works.
- `gh pr review --approve` — **fails** for your own PR (`Review Can not approve your own pull request`).
- `gh pr checks <num>` — works only if status checks exist.

### Labels
- `gh label create <name>` — works.
- `gh label list` — works.

### Releases
- `gh release create <tag>` — works.
- `gh release delete <tag>` — deletes the release but **leaves the tag**.
- `git tag -d <tag> && git push origin --delete <tag>` — needed to fully remove the tag.

### Actions
- Workflows run on `ubuntu-latest` runners.
- `gh run list`, `gh run view`, `gh run watch` — work.
- Environment variables work at workflow, job, and step levels.
- `GITHUB_TOKEN` has limited permissions: Contents read, Metadata read, Packages read.

### API
- `gh api repos/gankey696/SarahsWorld/hooks` — works; currently no webhooks configured.

## What Does Not Work

- `gh secret list` / `gh secret set` — HTTP 403: "Resource not accessible by integration". Secrets cannot be managed from the sandbox.
- `gh repo create` / `gh repo fork` — fails with broker 404 / "not connected for this owner".
- `gh webhook` — not a `gh` command; webhooks must be managed via UI or API.
- Approving your own PR.
- Accessing private repos other than `SarahsWorld` under `gankey696`.

## Rate Limits

- The GitHub token broker can return 429 (`route_rps_rate_limit_exceeded`).
- Retry after a short delay if this happens.

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

## Open Questions

- How to trigger workflows from `gh` without a push?
- Can workflow-level env vars replace repository secrets for non-sensitive config?
- What is the best way to receive GitHub events in the sandbox without webhooks?
