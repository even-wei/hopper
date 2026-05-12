# Start-of-Day Task Rubric

Score each item as pass/fail.

## Bootstrap (Stage 0)

- Resolves time window correctly: reads most recent file in `workspace/daily/` for window-start, falls back to last 24h when none.
- Pulls from all in-scope sources (Linear: assigned + lead-projects; Slack: mentions + DMs + my threads; Notion: mentions + comments on my pages; GitHub: review-requested + my open PRs flagged by reviewDecision + my freshly-opened PRs without review activity). Skips a source gracefully if its MCP tool is unavailable.
- GitHub pull enumerates repos from `github_repos` config and runs three `gh pr list --repo <owner/repo>` queries per repo (gh pr list is repo-scoped; omitting --repo silently targets the current worktree's repo). All queries use `--limit 200 --json number,url,title,...` form (default --limit 30 silently truncates); query (b) filters to `reviewDecision = CHANGES_REQUESTED` (REVIEW_REQUIRED is not a routing trigger); query (c) requires `reviewRequests` empty AND `latestReviews` contains no human submitted reviews other than the PR author AND `comments` contains no human issue comments other than the PR author. Bot reviews/comments (`claude` / `claude-bot` / `claude[bot]` / `github-actions[bot]`) do not count as human engagement. Draft inline review-thread comments are NOT checked (gh pr list --json doesn't expose them); a redundant self-review on such a PR is acceptable noise.
- Reads `workspace/daily/.config.yml` for user-specific scope IDs; does not re-resolve IDs from MCP tools every run.
- Reads prior-evening "Tomorrow's Lead" if present.

## Digest (Stage 1)

- Prints compact digest: counts per source + top-ranked items + prior-evening "Tomorrow's Lead" verbatim.
- Shows ranking labels (direct ask / decision-blocking / project shift / carry-over / FYI) so miscategorization is visible.
- Asks the Stage 1 input gate question about deprioritization / weighting before proceeding.

## Themes (Stage 2)

- Proposes 1–2 themes (not 3+, not 0).
- Each theme has rationale tying to specific digest items + Stage 1 user input.
- Asks the Stage 2 input gate to confirm / edit / replace before proceeding.

## Actions (Stage 3)

- Proposes 3–5 actions (not fewer, not more).
- Each action is tagged to a theme.
- Each action includes a source link, an effort estimate, and a one-line rationale.
- PR-routed signals appear as `<skill> <pr url>` dispatchable actions (not paraphrased verb-phrases, not bare `#<pr>` numbers).
- Stage 3 input gate is the single batch dispatch gate: confirm dispatch all / dispatch except {list} / skip dispatch; plus confirm/edit/drop/add for non-PR actions.
- When the user confirms dispatch, parallel teammates are launched (one per PR-routed action) before the file is written; the file describes what was dispatched.
- `/recce-dev:claude-code-review` dispatch brief contains the Comment-Only Override verbatim — no agent calls `gh pr review --approve` or `--request-changes` without explicit Stage 3 authorization scoped to the named PR.
- After self-review NO-GO verdicts return, agent auto-proposes `/recce-dev:pr-review-response <pr url>` as a follow-up without waiting for the user to ask.

## File Write (Stage 4)

- Writes `workspace/daily/<today>.md` matching the template schema exactly.
- Includes "Carried From Yesterday" section iff prior-evening file existed.
- Includes GitHub supporting-signal section iff GitHub signals were collected.
- Prints a 5-line terminal summary: arcs, first action, file path, signal counts (Linear: N, Slack: M, Notion: K, GitHub: G), carry-over status.
- Optional follow-up offered: "Want a Linear status-update draft for theme [X]?" when a theme is tied to a project.

## Edge Cases

- No prior evening file → Stage 1 says so explicitly; "Carried From Yesterday" omitted; window defaults to 24h.
- Quiet day → digest reports counts; themes still proposed from carry-over and Linear backlog.
- Conflicting signals → conflict surfaced explicitly in Stage 1; agent does not pick silently.
- User aborts mid-walk-through → no partial file written; agent prints what was confirmed.
- Re-run on same day → existing morning section shown; agent asks "amend or replace?" before mutating.
- MCP tool unavailable → "Source X unavailable" noted in digest and in `[source X unavailable]` line in the daily file.

Pass threshold: all items pass.
