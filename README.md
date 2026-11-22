# Codex automation

Shared Codex skills and a reusable GitHub Actions workflow for pull-request
reviews across private repositories owned by `pjgrenyer`.

## Scope and deliberate limitations

This repository is intentionally private. That is a recorded project decision,
not a temporary setup state; see
[`docs/decisions/0001-keep-the-automation-repository-private.md`](docs/decisions/0001-keep-the-automation-repository-private.md).

Consequences:

- Only private repositories owned by `pjgrenyer` can call this reusable
  workflow.
- Public repositories cannot use it while this repository remains private.
- GitHub Actions access is enabled for private repositories owned by
  `pjgrenyer`.
- Callers reference `main` deliberately, so a change merged here becomes
  available to every caller without updating each caller workflow.

## What it provides

- `$review-pr` performs the first substantive review of a pull request and can
  publish verified P0, P1, P2, and P3 findings.
- `$re-review-pr` verifies earlier feedback, reviews subsequent commits, and
  publishes only new actionable findings.
- `install-skills/action.yml` installs both skills into the caller's temporary
  runner workspace. The caller does not need to copy or maintain the skills.
- `.github/workflows/reusable-pr-review.yml` checks out the pull request and
  invokes the appropriate skill through `openai/codex-action`.

The initial review runs for `opened`, `ready_for_review`, and `reopened`
events. A `synchronize` event, normally created by pushing another commit to
the pull-request branch, runs the follow-up review.

## Prerequisites for a calling repository

Before enabling the workflow, confirm that:

1. The repository is private and owned by `pjgrenyer`.
2. GitHub Actions is enabled in the repository.
3. The repository permits `actions/checkout`, `openai/codex-action`, and
   reusable workflows from private repositories owned by `pjgrenyer`.
4. OpenAI API billing is configured. ChatGPT and API Platform billing are
   separate.
5. The repository has an Actions secret named `OPENAI_API_KEY`, as described
   below.

## Create an OpenAI credential

For CI, OpenAI recommends a project service account and a service-account API
key rather than a developer's personal key. This gives the automation its own
identity and lifecycle.

1. Sign in to the [OpenAI API Platform](https://platform.openai.com/).
2. Create or select a project dedicated to this review automation.
3. Ensure that the API project has billing or prepaid credits available. A
   ChatGPT subscription does not automatically fund API usage.
4. In the project settings, create a service account for the review workflow.
5. Create an API key for that service account with the minimum permissions
   needed to use the Responses API.
6. Copy the key immediately and store it in a password manager or other secret
   manager. OpenAI displays the full secret value only when it is created.

References:

- [OpenAI API quickstart](https://developers.openai.com/api/docs/quickstart)
- [API keys and service accounts](https://developers.openai.com/api/docs/guides/api-keys)
- [API billing](https://help.openai.com/en/articles/9039756-managing-billing-for-chatgpt-and-the-api-platform)

Do not commit the key, put it in a workflow file, paste it into a pull request,
or use a personal GitHub token in its place.

## Add the secret to a calling repository

Secrets belong to the calling repository. A secret stored only in
`codex-automation` is not automatically available to a reusable workflow
called by another repository.

### GitHub web interface

For each private repository that should use automated review:

1. Open the repository on GitHub.
2. Select **Settings**.
3. Select **Secrets and variables**, then **Actions**.
4. Select **New repository secret**.
5. Enter `OPENAI_API_KEY` as the name.
6. Paste the service-account API key as the value and save it.

### GitHub CLI

To set the secret without placing it in shell history:

```bash
read -rsp "OpenAI API key: " OPENAI_REVIEW_KEY
printf '\n'
printf '%s' "$OPENAI_REVIEW_KEY" |
  gh secret set OPENAI_API_KEY --repo pjgrenyer/REPOSITORY_NAME
unset OPENAI_REVIEW_KEY
```

Replace `REPOSITORY_NAME` with the private repository name. Verify that the
secret exists without revealing its value:

```bash
gh secret list --repo pjgrenyer/REPOSITORY_NAME
```

For several repositories, read the value once and send it to each repository:

```bash
read -rsp "OpenAI API key: " OPENAI_REVIEW_KEY
printf '\n'
for repository in repo-one repo-two repo-three; do
  printf '%s' "$OPENAI_REVIEW_KEY" |
    gh secret set OPENAI_API_KEY --repo "pjgrenyer/$repository"
done
unset OPENAI_REVIEW_KEY
```

Use an explicit list. Do not automatically install a paid automation secret
across every repository without reviewing the intended scope.

## Install the caller workflow

Create `.github/workflows/codex-review.yml` in the private target repository:

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
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

The same file is available as
[`examples/caller-workflow.yml`](examples/caller-workflow.yml).

The secret is passed by name rather than with `secrets: inherit`. GitHub
documents cross-repository secret inheritance for organizations and
enterprises; `pjgrenyer` is a personal account, so the caller passes this one
secret explicitly.

Commit this workflow to the target repository's default branch. GitHub reads
the trigger definition from the default branch when future pull-request events
occur.

## Verify the installation

1. Create a small branch in the target repository.
2. Open a non-draft pull request. Alternatively, open a draft and then mark it
   ready for review.
3. Open the target repository's **Actions** tab and find **Codex PR review**.
4. Confirm that the initial run completes successfully.
5. Push another commit to the pull-request branch.
6. Confirm that the next run invokes the follow-up review.

A successful run may post no GitHub comment. Both skills intentionally post
nothing when no verified actionable finding survives review.

## Runtime permissions and security

- The workflow uses `workspace-write` with network access because the skills
  run repository checks and use authenticated GitHub tooling.
- `openai/codex-action` retains its default `drop-sudo` safety strategy.
- The action's default trusted-user restriction remains enabled. Do not widen
  `allow-users` or `allow-bots` without reviewing prompt-injection and
  secret-exposure risks.
- The pull-request checkout uses `persist-credentials: false`.
- GitHub supplies a short-lived `GITHUB_TOKEN` for review comments. No personal
  GitHub access token is stored as a secret.
- GitHub does not expose repository secrets to workflows triggered from
  untrusted forks. This automation is intended for trusted contributors with
  write access, consistent with the Codex action's default trigger policy.

## Troubleshooting

### The reusable workflow cannot be found

- Confirm that the caller is a private repository owned by `pjgrenyer`.
- Confirm that it references
  `pjgrenyer/codex-automation/.github/workflows/reusable-pr-review.yml@main`.
- Public repositories cannot call this private workflow by design.

### `OPENAI_API_KEY` is missing

- Open the calling repository's Actions secrets and confirm the exact name is
  `OPENAI_API_KEY`.
- A secret created in another repository is not sufficient.
- Confirm that the caller maps the secret explicitly under `jobs.review.secrets`.

### The OpenAI request returns 401 or 429

- For 401 errors, replace an invalid or revoked key in the calling repository.
- For 429 billing or credit errors, inspect the API project's billing, credits,
  usage limits, and rate limits.
- API Platform usage is billed separately from ChatGPT usage.

### The workflow runs but posts no review comments

Open the run summary. If it completed successfully and reports no findings,
this is expected: the skills do not submit empty reviews.

### A contributor's pull request does not start Codex

The Codex action permits trusted users with write access by default. Fork pull
requests also do not receive repository secrets. Do not weaken these controls
without a separate security design for untrusted code and prompts.

## Rotate or revoke the OpenAI key

1. Create a replacement key for the service account.
2. Update `OPENAI_API_KEY` in every calling repository.
3. Verify a review run in at least one representative repository.
4. Revoke the old key in the OpenAI API Platform.

Never delete or revoke the old key before all intended callers have been
updated, unless the key is suspected to be compromised.
