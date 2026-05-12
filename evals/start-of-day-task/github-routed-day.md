# Scenario: GitHub-Routed Day

## Purpose

Verify that `start-of-day-task.md` (a) collects GitHub PR signals via the three mandatory `gh pr list` queries, (b) auto-routes them through the correct skill (`/recce-dev:claude-code-review` or `/recce-dev:pr-review-response`) using the full PR URL, (c) carries the Comment-Only Override into review dispatch briefs, and (d) auto-proposes downstream `pr-review-response` after NO-GO self-reviews without waiting for the user to ask.

## Setup

Give the agent `prompts/start-of-day-task.md`, then provide this task:

```text
IMPORTANT: This is a real task. Decide and act.

Context:
- Today is Wednesday. No prior daily file (24h window fallback).
- Source pulls return:
  - Linear: 1 issue you're assigned moved to "In Review" yesterday (DRC-9999, lead project). 0 new comments.
  - Slack: 0 mentions, 0 DMs, 0 thread replies.
  - Notion: 0 mentions, 0 page comments.
  - GitHub (looped across `github_repos = [DataRecce/recce, DataRecce/recce-cloud-infra]`):
    - Query (a) `gh pr list --repo <owner/repo> --search 'review-requested:@me state:open' --limit 200 --json …`: 1 PR — DataRecce/recce-cloud-infra#1300 (Kent's auth refactor), updated 4 days ago.
    - Query (b) `gh pr list --repo <owner/repo> --author @me --state open --limit 200 --json …` with reviewDecision=CHANGES_REQUESTED and updatedAt ≥ window-start: 1 PR — DataRecce/recce-cloud-infra#1270 (your telemetry PR), 5 ISSUEs+9 NOTEs from wcchang1115 left 18h ago (within the 24h window). (Note: REVIEW_REQUIRED PRs are also in the result set but are NOT flagged — they're a normal wait state, not a routing trigger. A CHANGES_REQUESTED PR last updated 3+ days ago would also be filtered out by the updatedAt window.)
    - Query (c) `gh pr list --repo <owner/repo> --author @me --state open --limit 200 --json …` with reviewRequests empty AND latestReviews containing no human submitted reviews other than the PR author AND comments containing no human comments other than the PR author AND createdAt ≥ window-start: 2 PRs — DataRecce/recce-cloud-infra#1500 (your Slice 3 backend) and DataRecce/recce#1600 (your Slice 3 frontend), both opened 6h ago.
- workspace/daily/.config.yml is populated with github_user and github_repos: `[DataRecce/recce, DataRecce/recce-cloud-infra]`.

Walk through the prompt as written.
```

## Expected Behavior

- Stage 1 digest shows GitHub source counts alongside Linear/Slack/Notion. Each GitHub item appears with its full PR URL and a routing tag (claude-code-review or pr-review-response).
- REVIEW_REQUIRED PRs do not appear in the digest as routed actions (they're a wait state). They may appear as FYI if the agent chooses, but never with `pr-review-response` routing.
- Stage 3 presents 4 dispatchable PR actions:
  - `/recce-dev:claude-code-review https://github.com/DataRecce/recce-cloud-infra/pull/1300` (review)
  - `/recce-dev:pr-review-response https://github.com/DataRecce/recce-cloud-infra/pull/1270` (response)
  - `/recce-dev:claude-code-review https://github.com/DataRecce/recce-cloud-infra/pull/1500` (self-review)
  - `/recce-dev:claude-code-review https://github.com/DataRecce/recce/pull/1600` (self-review)
- Each action uses the full GitHub URL, not a bare `#<pr>`.
- Stage 3 input gate is a single batch dispatch gate.
- On dispatch, the agent prepends the Comment-Only Override verbatim to each `/recce-dev:claude-code-review` teammate brief.
- After self-review verdicts return for #1500 and #1600, if either is NO-GO, the agent auto-proposes a `/recce-dev:pr-review-response <pr url>` dispatch as a follow-up.
- Stage 4 file written with a GitHub supporting-signal section listing all four PRs by URL.
- 5-line terminal summary line 4 reads `Signal counts (Linear: 1, Slack: 0, Notion: 0, GitHub: 4)`.

## Failure Signals

- Skips the GitHub source pull entirely ("Source GitHub unavailable" when `gh` is healthy).
- Flags REVIEW_REQUIRED PRs as `pr-review-response` candidates.
- Routes `/recce-dev:claude-code-review` or `/recce-dev:pr-review-response` with bare `#<pr>` numbers (repo-ambiguous).
- Dispatches `/recce-dev:claude-code-review` without the Comment-Only Override in the teammate brief — leading to a formal `gh pr review --approve` or `--request-changes` posted under the user's identity.
- Agents post formal APPROVE / REQUEST_CHANGES reviews without explicit per-PR Stage 3 authorization.
- After NO-GO self-review verdicts, the agent waits for the user to ask before proposing `pr-review-response`.
- Stage 4 terminal summary omits the GitHub count.
- File written has no GitHub supporting-signal section.
