# Start-of-Day Task Prompt

Use this at the start of your workday to surface signals across Slack, Linear, and Notion since your last brief, and to commit to 1–2 themes with 3–5 concrete actions for the day. Walks through stages interactively; writes `workspace/daily/<today>.md` at the end.

```text
Goal:
- Help the user decide today's focus by surfacing decision-relevant signals from Slack, Linear, and Notion since the last daily brief, then walking through theme and action proposals interactively. Write the morning section of workspace/daily/<today>.md when confirmed.

Inputs:
- workspace/daily/.config.yml (linear_user, linear_lead_projects, slack_user, notion_user)
- workspace/daily/ directory (most recent prior file is the "last brief" anchor)
- Live source pulls (per Source Rules below)

Source Rules:
- Time window: from most recent prior file's mtime in workspace/daily/ to now. If no prior file, fall back to last 24h.
- Linear: list issues where assignee = linear_user updated since window-start, plus issues in projects ∈ linear_lead_projects updated since window-start. For each, fetch comments since window-start.
- Slack: messages where the user is @-mentioned, DMs to the user, and new replies in threads the user previously posted in. All since window-start.
- Notion: comments mentioning notion_user and new comments on pages the user authored. All since window-start.
- GitHub: PRs the user authored that are open and in CHANGES_REQUESTED state (reviewer waiting on the user's fix). Fetch via `gh search prs --author=@me --state=open --review=changes-requested --json url,repository,title,updatedAt,number`. No time-window filter on this query — staleness is itself a signal worth surfacing.
- Transcripts: deferred. Skip; do not block on this source.
- If any source's MCP tool is unavailable or auth-expired, print "Source X unavailable, continuing without it" and continue. Note `[source X unavailable]` in the daily file.

Signal Heuristic (signal-first; rank items globally, regardless of source):
1. Direct ask of you — DM, @mention with question, Linear issue assigned + state change demanding action.
2. Decision-blocking — Linear issue blocked, PR awaiting your review, Slack thread waiting on your reply, PR you authored in CHANGES_REQUESTED state (reviewer waiting on your fix; default routing: dispatch `/recce-dev:pr-review-response <url>`).
3. Project-level shifts — status update, scope change, stakeholder comment on a lead project.
4. Carry-over — items from yesterday's "Tomorrow's Lead." Surfaced prominently regardless of fresh signal.
5. FYI / informational — channel announcements, status updates not requesting action. Compress to one-liner.

Show ranking labels in the digest so the user can spot miscategorization.

Walk-Through (interactive — input gate at each stage):

Stage 0 — Bootstrap (silent):
- Resolve time window per Source Rules.
- Read prior-evening "Tomorrow's Lead" from yesterday's file if present.
- Pull source data per Source Rules.
- Score and rank items per the Signal Heuristic.

Stage 1 — Digest review:
- Print compact digest: per-source counts + top-ranked items with their ranking label + prior-evening "Tomorrow's Lead" verbatim.
- If any user-authored PRs in CHANGES_REQUESTED were detected in Stage 0, include a "Your PRs awaiting fixes" sub-section. One line per PR: `<repo> #<number> — <title> — <url> — updatedAt <updatedAt> — proposed routing: /recce-dev:pr-review-response <url>`. Omit the sub-section entirely if none detected (do not print "0 items").
- Surface any conflicting signals explicitly (do not pick silently).
- Input gate: "Anything you already know is on your mind today that I should weight heavily? Anything in the digest you'd deprioritize?"

Stage 2 — Theme proposal:
- Propose 1–2 themes, each with rationale tied to digest + Stage 1 user input.
- Input gate: "Do these themes match how you're framing today, or should I rework them? You can edit, replace, or add."

Stage 3 — Action proposal:
- For confirmed themes, propose 3–5 actions tagged to themes.
- Each action: verb-phrase + source link + effort estimate + one-line rationale.
- Auto-fill for user-authored PRs in CHANGES_REQUESTED (from Stage 0): for each such PR, emit an action with this exact shape — no manual rephrasing:
  ```
  - [ ] [Theme: Close review loops] Dispatch teammate via `/recce-dev:pr-review-response` for PR #NNN (<title>) — <url> — est. 30–45 min teammate run
         why: <repo> #NNN is CHANGES_REQUESTED; reviewer waiting since <updatedAt>
  ```
  If the user has no other themes today, "Close review loops" is the single theme. If the user already confirmed other themes in Stage 2, attach "Close review loops" as an additional theme alongside them and list these actions under it.
- The Stage 3 gate still applies: the user can drop or edit any auto-filled action. This is pre-filled wording, not auto-execution.
- Input gate: "Confirm, edit, drop, or add actions. I'll write the file once you're set."

Stage 4 — Write & summarize:
- Write workspace/daily/<today>.md morning section per the daily-brief template. Copy the prior-evening "Tomorrow's Lead" verbatim into "Carried From Yesterday" (do not summarize or reword).
- Resolve every action's source link to a canonical URL (Linear issue URL, Slack permalink, Notion page URL, GitHub PR URL). A bare ID like "DRC-3309" is not sufficient — produce the full URL.
- Print exactly 5 lines to terminal, one per line in this order:
  1. Today's arcs (numbered, comma-separated).
  2. First action (verb-phrase + source URL + estimate).
  3. File path (workspace/daily/<today>.md).
  4. Signal counts (Linear: N, Slack: M, Notion: K).
  5. Carry-over status (theme + action carried, or "no prior-evening carry-over").
- Optional follow-up: for each theme tied to a Linear project, offer once: "Want a Linear status-update draft for theme [X]?"

Edge Cases:
- No prior evening file: Stage 1 says so explicitly; omit "Carried From Yesterday" in the file; window defaults to 24h.
- Quiet day (low signal): Stage 1 reports counts; Stage 2 still proposes themes from carry-over and Linear backlog.
- Conflicting signals: surface in Stage 1 explicitly; do not pick silently.
- User aborts mid-walk-through: do NOT write a partial file. Print what was confirmed so far.
- Re-run on the same day: read the existing morning section first; ask "amend or replace?" before mutating.

Output:
- File written: workspace/daily/<today>.md (morning section).
- Terminal summary: 5 lines.
- Optional Linear status-update drafts (when accepted).

Quality Bar:
- File matches templates/daily-brief.md schema exactly.
- Themes count is 1–2; actions count is 3–5.
- Every action has a source link and effort estimate.
- Ranking labels visible in the digest.
- All four input gates present.
- "Carried From Yesterday" included iff prior-evening file existed.
```
