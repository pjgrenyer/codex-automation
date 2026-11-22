---
name: re-review-pr
description: Re-review a GitHub pull request after earlier review feedback has been addressed, verify the fixes and subsequent changes, and post only new actionable findings as inline PR comments. Use when the user asks for a second or follow-up review of a PR; do not use for a first review or a local/read-only review.
---

# Re-review a pull request

Treat the first GitHub pull-request URL in the request as the target. Confirm it identifies exactly one accessible PR before reviewing or posting anything.

## Reconstruct the prior review state

- Read the PR description, linked requirements, applicable repository instructions, complete current diff, review history, inline threads, replies, and commits.
- Identify the review cycle the user wants revisited. Prefer the authenticated account's latest substantive review before the follow-up commits unless the user names another reviewer or review. If the baseline remains ambiguous, inspect the full history and current diff rather than guessing that an earlier concern was addressed.
- Record the current head SHA and the commit reviewed previously when GitHub exposes it. Use that range to focus investigation, but assess the entire current PR for interactions and regressions.
- For every earlier actionable comment, classify its current state:
  - **Addressed:** the defect is absent at the current head and relevant verification passes.
  - **Still present:** the same defect remains; do not create a duplicate finding.
  - **Superseded or outdated:** the code moved or the implementation changed enough that the original thread no longer describes the current defect.
  - **Unverifiable:** available code, tests, or environment cannot establish the result.
- Do not resolve, dismiss, edit, or otherwise change existing review threads unless the user explicitly requests it.

## Verify the updated PR

- Re-test the affected scenarios behind prior comments; do not infer that a reply, resolved thread, or code change proves the issue is fixed.
- Inspect every commit and diff hunk added since the previous review, plus surrounding code needed to understand behavior.
- Review the resulting PR for correctness, regressions, security, reliability, compatibility, contract violations, and incomplete fixes.
- Run checks required by repository instructions and the changed area. When execution would distinguish a defect from suspicion, create and run a narrow reproduction. Unless the user also asked for code changes, remove only temporary artifacts created for that reproduction.
- Report only concrete, actionable defects caused by the PR. Exclude style preferences, speculative hardening, pre-existing defects, and findings already covered by an adequate existing thread.

## Prepare findings

Classify new findings as `P0` through `P3`, where `P0` is catastrophic and broadly blocking, `P1` is high impact in normal or important use, `P2` affects a narrower realistic scenario, and `P3` is a low-impact correctness defect worth fixing.

Each finding must have a concise title, severity, current changed file and line, affected scenario, causal explanation, and concrete evidence such as a reproduction, failing assertion, contract conflict, or traced execution path.

An incomplete fix may receive a new inline finding only when it identifies a materially new defect or the old comment can no longer anchor the problem to the current diff. Link the earlier thread and explain what remains. Otherwise, leave the existing thread as the single record and mention its still-open status only in the final chat summary.

## Publish inline

- Immediately before posting, re-fetch the PR head and diff. If the head changed, revalidate affected findings against the new head.
- Post each surviving new finding as an inline review comment on the smallest possible current diff range containing the causal line. Never attach a finding to an unchanged, approximate, or unrelated line.
- Prefer one pending review containing all new inline comments, then submit it with a neutral comment event. Do not approve or request changes unless the user explicitly asks for that state.
- Do not substitute a top-level review summary for inline findings. Do not post an empty review when no new finding survives verification.
- Use this comment form:

  ```text
  [P2] Concise diagnostic title

  Affected scenario: ...

  Explain what fails, why this current line causes it, and the evidence that verifies the failure.
  ```

- Check for duplicate or overlapping findings and verify comment paths, line positions, severities, and the final comment count before submission.

Finish in chat with the baseline and head reviewed, the status of earlier comments, checks and reproduction evidence, and links or identifiers for new inline comments. If nothing was posted, say so explicitly and summarize any remaining open or unverifiable prior comments.
