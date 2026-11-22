# Codex automation

Shared Codex skills and a reusable GitHub Actions workflow for pull-request
reviews across repositories owned by `pjgrenyer`.

## Included skills

- `$review-pr` performs the first substantive review of a pull request.
- `$re-review-pr` verifies earlier feedback and reviews subsequent updates.

Both skills publish only verified, actionable inline findings. The first-review
skill supports P0 through P3 findings; the follow-up skill avoids duplicating
adequate existing review threads.

## Add the workflow to another repository

Create `.github/workflows/codex-review.yml` in the target repository:

```yaml
name: Codex PR review

on:
  pull_request:
    types: [opened, ready_for_review, reopened, synchronize]

concurrency:
  group: codex-review-${{ github.event.pull_request.number }}
  cancel-in-progress: false

jobs:
  review:
    if: github.event.pull_request.draft == false
    permissions:
      contents: read
      issues: write
      pull-requests: write
    uses: pjgrenyer/codex-automation/.github/workflows/reusable-pr-review.yml@main
    with:
      pr-url: ${{ github.event.pull_request.html_url }}
      event-action: ${{ github.event.action }}
    secrets: inherit
```

The calling repository must contain an Actions secret named
`OPENAI_API_KEY`. Because `pjgrenyer` is a personal GitHub account, GitHub
does not provide an organization-level Actions secret that can be shared by
all repositories. Set the same secret separately on each participating
repository, or move the repositories into an organization and use an
organization Actions secret.

## Visibility and access

GitHub permits private repositories to consume a reusable workflow from a
private repository owned by the same user after Actions access is enabled on
this repository. Public repositories can call reusable workflows only from a
public repository. Therefore this repository must be public if the workflow
is intended to cover both public and private `pjgrenyer` repositories.

## Security model

- The workflow uses `workspace-write` with network access because the skills
  must run project checks and use authenticated GitHub tooling.
- `openai/codex-action` retains its default `drop-sudo` safety strategy.
- The action's default trusted-user restriction remains enabled. Do not widen
  `allow-users` or `allow-bots` without reviewing the prompt-injection and
  secret-exposure consequences.
- The pull-request checkout does not persist Git credentials.
- The automatic `GITHUB_TOKEN` is used for GitHub review comments; no personal
  access token is stored.

The shared workflow deliberately references `main`, so updates here take
effect in calling repositories without changing their workflow files.
Shared Codex skills and reusable GitHub Actions workflows
