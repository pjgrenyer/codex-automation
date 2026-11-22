# Decision 0001: Keep the automation repository private

- Status: superseded by [Decision 0002](0002-publish-read-only-automation.md)
- Date: 2026-11-22

## Context

`codex-automation` contains shared review instructions and the reusable GitHub
Actions workflow that runs them. GitHub can share actions and reusable
workflows from a private repository with other private repositories owned by
the same user when Actions access is enabled.

GitHub does not allow a public repository to call a reusable workflow stored
in a private repository. Making this repository public would allow both public
and private callers, but would also publish the review workflow and skill
instructions.

## Decision

Keep `pjgrenyer/codex-automation` private. Reference its `main` branch from
calling repositories.

## Consequences

- Only private repositories owned by `pjgrenyer` are supported callers.
- Public repositories owned by `pjgrenyer` are intentionally excluded.
- The repository's Actions access setting must remain **Accessible from
  repositories owned by `pjgrenyer`**.
- Updates merged into `main` are consumed by callers without a version change.
- If public-repository support is wanted later, revisit this decision before
  changing visibility or introducing a separate public distribution repository.
