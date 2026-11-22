---
name: review-pr
description: Review a GitHub pull request for correctness and regressions and post verified, actionable findings as inline PR comments. Use when the user supplies a GitHub PR URL and asks for this review-and-comment workflow; do not use for reviews that should remain local or read-only.
---

# Review a pull request

Treat the first GitHub pull-request URL in the request as the target. Confirm that it identifies exactly one accessible PR before reviewing or posting anything.

## Establish the contract

- Read the PR description and every issue it links as motivation, acceptance criteria, or prerequisite context.
- Read the applicable repository instructions, especially `AGENTS.md` files governing changed paths.
- Identify related behavioral contracts: public APIs, interfaces, schemas, protocol definitions, compatibility promises, tests, and user-facing documentation.
- Inspect the complete diff and enough surrounding code and history to understand the changed behavior. Account for uncommitted user work and do not alter it.

## Verify the change

- Review for correctness, regressions, security, reliability, compatibility, and violations of the identified contracts.
- Run the checks required by the repository instructions and the changed area. Record the exact commands and outcomes.
- When execution would materially distinguish a real defect from a suspicion, create a narrow reproduction test and run it. Keep a useful test only when the user also asked for code changes; otherwise remove only the temporary files or edits created for the reproduction.
- Trace each suspected defect to a concrete affected scenario and the causal changed line. Do not report style preferences, general hardening suggestions, pre-existing defects, or hypothetical concerns without evidence.
- Recheck the final diff immediately before posting so paths and line positions are current.

## Findings

Report only findings that are actionable and independently defensible. Classify them as:

- `P0`: catastrophic and broadly blocking, such as data loss or severe compromise.
- `P1`: high-impact defect likely to affect normal or important use.
- `P2`: concrete defect affecting a narrower but realistic scenario.
- `P3`: low-impact correctness defect that is still worth fixing.

Each finding must include a concise title, severity, exact changed file and line, affected scenario, explanation of the incorrect behavior, and concrete evidence such as a reproduction, failing assertion, contract conflict, or directly traced execution path.

## Publish the review

- Use authenticated GitHub tooling to submit verified findings as inline review comments on the target PR.
- Attach each comment to the smallest possible range on the causal changed line. Comment only on lines present in the PR diff; never substitute an unchanged or approximate line.
- Prefer one pending review containing all inline comments, then submit it with a neutral comment event. Do not approve the PR or request changes unless the user explicitly asks for that review state.
- Keep each comment self-contained and use this form:

  ```text
  [P2] Concise imperative or diagnostic title

  Affected scenario: ...

  Explain what fails, why this changed line causes it, and the evidence that verifies the failure.
  ```

- Avoid duplicate or overlapping findings. Before submission, verify the pending comment count, paths, lines, and severities.
- If no actionable findings survive verification, post nothing to GitHub. Do not manufacture a finding or submit an empty review.

Finish in chat with the checks run, reproduction evidence used, and links or identifiers for the review comments posted. If nothing was posted, state that explicitly and summarize any verification limits.
