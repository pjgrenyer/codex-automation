# Decision 0002: Publish the automation with owner-only writes

- Status: accepted
- Date: 2026-11-22
- Supersedes: [Decision 0001](0001-keep-the-automation-repository-private.md)

## Context

Every calling repository stores and explicitly passes its own
`OPENAI_API_KEY`. The automation repository contains reusable workflow code and
review instructions, but no caller credentials. Keeping it private prevents
public repositories from calling the reusable workflow and does not add secret
isolation for the calling repositories.

## Decision

Make `pjgrenyer/codex-automation` public while retaining owner-only write
access. Reference its `main` branch from calling repositories.

Repository governance is:

- no collaborators with write access;
- `main` protected against force-pushes and deletion, including for the
  repository administrator;
- Issues, Discussions, Projects, and the wiki disabled; and
- pull requests permitted as proposals, with only the owner able to merge.

## Consequences

- Public and private repositories can call the reusable workflow.
- Anyone can read, clone, fork, and inspect the workflow and review skills.
- Forks and pull requests do not grant write access to the upstream repository.
- Secrets remain scoped to each caller and are not published with this
  repository.
- A change merged into `main` is consumed by every caller without a version
  update. Protecting the owner account and reviewing changes to `main` remain
  critical supply-chain controls.
