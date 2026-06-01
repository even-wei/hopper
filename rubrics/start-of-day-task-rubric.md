# Start-of-Day Task Rubric

Score each item as pass/fail.

## Bootstrap & verify (Stage 0)

- Resolves time window correctly: reads most recent file in `workspace/daily/` for window-start, falls back to last 24h when none.
- Pulls from all in-scope sources (Linear: assigned + lead-projects; Slack: mentions + DMs + my threads; Notion: mentions + comments on my pages; GitHub: review-requested + my open PRs flagged by reviewDecision + my freshly-opened PRs without review activity). Skips a source gracefully if its MCP tool is unavailable.
- GitHub pull enumerates repos from `github_repos` config and runs three `gh pr list --repo <owner/repo>` queries per repo (gh pr list is repo-scoped; omitting --repo silently targets the current worktree's repo). All queries use `--limit 200 --json number,url,title,...` form (default --limit 30 silently truncates); query (b) filters to `reviewDecision = CHANGES_REQUESTED` (REVIEW_REQUIRED is not a routing trigger); query (c) requires `reviewRequests` empty AND `latestReviews` contains no human submitted reviews other than the PR author AND `comments` contains no human issue comments other than the PR author. Bot reviews/comments (`claude` / `claude-bot` / `claude[bot]` / `github-actions[bot]`) do not count as human engagement. Draft inline review-thread comments are NOT checked (gh pr list --json doesn't expose them); a redundant self-review on such a PR is acceptable noise.
- Reads `workspace/daily/.config.yml` for user-specific scope IDs; does not re-resolve IDs from MCP tools every run.
- Reads prior-evening "Tomorrow's Lead" if present, and verifies each carried item via MCP before re-proposing — marks resolved / still-open from evidence; only flags "not verified — confirm still owed?" for genuinely non-MCP / offline sources (especially after weekend gaps).
- Fetches lead projects and marks staleness (>5 days since last status update, or never, active state only).

## Autonomy classification (applies throughout)

- Every candidate action is classified Tier A (reversible + low-stakes), Tier B (irreversible / high-stakes / leaves the workspace toward others), or Tier C (notify-only) — by reversibility / stakes, not by stage.
- Tier-A work (drafting, MCP carry verification, own-issue comment/status, file write, memory stash) runs autonomously, not gated.
- Tier-B items (Slack posts, stakeholder messages, project status updates, others' issues, teammate dispatch) are queued in "Needs Your Call", never executed unattended.
- Status updates and Slack Connect are marked draft-only (queued but not executed even on accept).
- Standing-trusted classes from `.config.yml`, if present, auto-execute; absent config, all Tier-B is queued.

## Draft, advance & write (Stage 1)

- Drafts 1–2 themes (not 3+, not 0), each tied to specific digest items.
- Drafts 3–5 actions (not fewer, not more), each tagged to a theme, with a canonical source URL, an effort estimate, and a one-line rationale.
- Colleague pings drafted as "ping <name> in #<channel> with @mention", not DM (reserve DM for genuinely private content).
- PR-routed signals appear as `<skill> <full pr url>` dispatchable actions (not paraphrased verb-phrases, not bare `#<pr>` numbers), routed to `/recce-dev:claude-code-review` (review-requested + own freshly-opened PRs) or `/recce-dev:pr-review-response` (CHANGES_REQUESTED), and queued in "Needs Your Call" as executable-on-accept — not auto-run.
- Performs Tier-A actions and logs each in the autonomy tally.
- Writes `workspace/daily/<today>.md` (morning section + "Needs Your Call" block + GitHub supporting-signal section iff GitHub signals collected + "Stale lead projects" awareness bullet iff any) matching the template schema exactly. The file is written before the review (it is a working draft).

## One batched review (Stage 2)

- Exactly ONE input gate: the drafted Arc + Actions and the "Needs Your Call" queue are resolved together in a single pass — no per-stage gates.
- Queue items resolvable accept / edit / respond / skip. On accept of PR dispatches, parallel teammates are launched (one per accepted PR-routed action); the file records what was dispatched.
- `/recce-dev:claude-code-review` dispatch briefs contain the Comment-Only Override verbatim — no agent calls `gh pr review --approve` or `--request-changes` without explicit per-PR authorization in the review.
- After self-review NO-GO verdicts return, agent auto-proposes `/recce-dev:pr-review-response <pr url>` as a queued follow-up without waiting for the user to ask.
- Conflicting signals surfaced explicitly; agent does not pick silently.
- Non-interactive session: live review skipped; queue persists in the file; no Tier-B fires.

## Summarize (Stage 3)

- Prints a 6-line terminal summary: arcs; first action; file path; signal counts (Linear: N, Slack: M, Notion: K, GitHub: G); carry-over status; autonomy tally ("Did autonomously: N · Queued: M (K resolved)").

## Edge Cases

- No prior evening file → "Carried From Yesterday" omitted; window defaults to 24h.
- Quiet day → themes still drafted from carry-over and Linear backlog; file written either way.
- Conflicting signals → surfaced in the Stage 2 review; agent does not pick silently.
- Non-interactive / user steps away → file written with "Needs Your Call" block; nothing Tier-B fires; unresolved items re-surface next run (not a partial-write failure).
- Re-run on same day → existing morning section shown; agent asks "amend or replace?" before mutating.
- MCP tool unavailable → "Source X unavailable" noted in digest and `[source X unavailable]` line in the daily file.

Pass threshold: all items pass.
