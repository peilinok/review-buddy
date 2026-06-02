# Example: Reviewing a mixed auth/session PR

This example shows how Review Buddy should behave on a realistic pull request:

- medium-sized diff
- multiple related modules
- behavior change + tests
- code written by an AI coding agent or by a human in an unfamiliar subsystem

## Scenario

Imagine a PR with this rough shape:

- refresh-token rotation logic was rewritten
- session middleware was simplified
- tests were updated for the new flow
- the PR description says: "fix token refresh race and clean up session handling"

## Files in the hypothetical PR

- `auth/refresh.ts`
- `auth/token_store.ts`
- `server/middleware/session.ts`
- `server/app.ts`
- `tests/auth/refresh.test.ts`
- `tests/server/session-middleware.test.ts`

## Files in this folder

- `pr-description.md` — the author's PR description
- `review-brief.md` — phase 0 output
- `context-cards.md` — phase 1 output
- `chunk-plan.md` — phase 2 output
- `guided-review.md` — chunk-by-chunk review output
- `final-summary.md` — consolidated review verdict

## Why this example exists

The point is not to provide a single "correct" review. The point is to demonstrate the pacing and structure:

- first orient the reviewer
- then load context
- then decompose the change
- then review one chunk at a time
- then consolidate findings
