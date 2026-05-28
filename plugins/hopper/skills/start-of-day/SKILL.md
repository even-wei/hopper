---
name: start-of-day
description: Use at the start of the workday to surface decision-relevant signals from Linear, Slack, and Notion since the last brief, propose 1–2 themes with 3–5 concrete actions interactively, and write workspace/daily/<today>.md morning section. Triggers on "start of day", "morning brief", "daily kickoff", "what's on my plate today", "begin the day".
---

# Start-of-Day Daily Brief

Walks through the morning brief in stages and writes `workspace/daily/<today>.md`. Reads scope IDs from `workspace/daily/.config.yml`.

The Codex-readable canonical source for this prompt is `hopper/prompts/start-of-day-task.md`. The full prompt body is inlined below for Claude Code skill consumers, followed by the daily-brief schema appendix so the skill is self-contained regardless of where it's invoked.

## Prompt

```text
Goal:
- Help the user decide today's focus by surfacing decision-relevant signals from Slack, Linear, and Notion since the last daily brief, then walking through theme and action proposals interactively. Write the morning section of workspace/daily/<today>.md when confirmed.

Inputs:
- workspace/daily/.config.yml (linear_user, slack_user, notion_user)
- workspace/daily/ directory (most recent prior file is the "last brief" anchor)
- Live source pulls (per Source Rules below)

Source Rules:
- Time window: from most recent prior file's mtime in workspace/daily/ to now. If no prior file, fall back to last 24h.
- Linear: list issues where assignee = linear_user updated since window-start, plus issues in the user's lead projects (Linear projects where `lead.id == linear_user` and `status.type ∈ {started, planned}`; fetched live via `list_projects(member=linear_user)` and filtered) updated since window-start. For each, fetch comments since window-start.
- Slack: messages where the user is @-mentioned, DMs to the user, and new replies in threads the user previously posted in. All since window-start. Before treating any Slack signal as evidence of a Recce-product bug, read the thread parent message and fetch attached image/file metadata — keyword overlap ("banner," "rendering," "not showing," "not loading") is insufficient because those words also appear in Slack-UI, browser-UI, and OS-UI questions. If the parent message lives in #random or another non-product channel, default prior is "probably not a Recce bug." Phrase any attribution loosely ("Slack thread <link> in #random — Andy reply suggests X"), never certainly ("Andy surfaced a render bug").
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
- Verify each carried item is still open before re-proposing. For "Owed to others" / "Tomorrow's Lead" items inherited from a prior day, check for a closure signal since the carry: issue state change, new comment, thread reply, calendar event. If the source is non-MCP (offline conversation, F2F meeting, hallway), do NOT assume slippage — flag in Stage 1 as "carry from N days ago, not verified — confirm still owed?" rather than restating as fact. For carries that have slipped 3+ days, put the verification check above the action: "Confirm this is still open" before "Send the ping." Weekend gaps (no Sat/Sun daily file) especially: assume offline work may have closed loops, ask before treating Friday's open items as Monday's carries.
- Pull source data per Source Rules.
- Score and rank items per the Signal Heuristic.
- Fetch the user's lead projects from Linear: `list_projects(member=linear_user)` filtered to `lead.id == linear_user` and `status.type ∈ {started, planned}`. For each such project, fetch the latest project status update timestamp. Mark a project as **stale** if its last status update is >5 days ago, or if it has no status updates at all (and the project is in an active state — not Canceled / Completed-and-archived). Carry this list into Stage 1's digest and Stage 4's Supporting Signal — *awareness only*, not a draft offer (drafting lives in end-of-day's Stage 6).

Stage 1 — Digest review:
- Print compact digest: per-source counts + top-ranked items with their ranking label + prior-evening "Tomorrow's Lead" verbatim.
- If any user-authored PRs in CHANGES_REQUESTED were detected in Stage 0, include a "Your PRs awaiting fixes" sub-section. One line per PR: `<repo> #<number> — <title> — <url> — updatedAt <updatedAt> — proposed routing: /recce-dev:pr-review-response <url>`. Omit the sub-section entirely if none detected (do not print "0 items").
- If any lead projects are marked **stale** in Stage 0, include a "Stale lead projects (no status update in 5+ days)" sub-section. One line per project: `<project name> — <days-stale>d since last update — <project URL>`. Awareness only — do not auto-create theme/actions; user decides whether to fold into today's arc. Omit the sub-section entirely if none.
- Surface any conflicting signals explicitly (do not pick silently).
- Input gate: "Anything you already know is on your mind today that I should weight heavily? Anything in the digest you'd deprioritize?"

Stage 2 — Theme proposal:
- Propose 1–2 themes, each with rationale tied to digest + Stage 1 user input.
- Input gate: "Do these themes match how you're framing today, or should I rework them? You can edit, replace, or add."

Stage 3 — Action proposal:
- For confirmed themes, propose 3–5 actions tagged to themes.
- Each action: verb-phrase + source link + effort estimate + one-line rationale.
- For actions that ping a colleague (status nudge, reschedule, light ask): default to "ping <name> in #<channel> with @mention" rather than "DM <name>." DMs hide cross-loop signal from collaborators who may also have context. Reserve DM phrasing for genuinely private content (1:1 agenda drafts before posting, personal scheduling conflicts).
- Auto-fill for user-authored PRs in CHANGES_REQUESTED (from Stage 0): for each such PR, emit an action with this exact shape — no manual rephrasing:
  ```
  - [ ] [Theme: Close review loops] Dispatch teammate via `/recce-dev:pr-review-response` for PR #NNN (<title>) — <url> — est. 30–45 min teammate run
         why: <repo> #NNN is CHANGES_REQUESTED; reviewer waiting since <updatedAt>
  ```
  If the user has no other themes today, "Close review loops" is the single theme. If the user already confirmed other themes in Stage 2, attach "Close review loops" as an additional theme alongside them and list these actions under it.
- The Stage 3 gate still applies: the user can drop or edit any auto-filled action. This is pre-filled wording, not auto-execution.
- Input gate: "Confirm, edit, drop, or add actions. I'll write the file once you're set."

Stage 4 — Write & summarize:
- Write workspace/daily/<today>.md morning section per the daily-brief template (schema appended below). Copy the prior-evening "Tomorrow's Lead" verbatim into "Carried From Yesterday" (do not summarize or reword).
- If any lead projects are marked **stale** in Stage 0, add a "Stale lead projects" bullet under Supporting Signal listing each project (name + days-stale + URL). Plain awareness line, no action attached.
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
- File matches the daily-brief schema below exactly.
- Themes count is 1–2; actions count is 3–5.
- Every action has a source link and effort estimate.
- Ranking labels visible in the digest.
- All four input gates present.
- "Carried From Yesterday" included iff prior-evening file existed.
- Stale lead projects (no status update in 5+ days, active state) are surfaced in Stage 1's digest and the daily file's Supporting Signal — awareness only, never auto-elevated into themes/actions.
```

## Daily Brief Schema

Generated artifact path: `workspace/daily/YYYY-MM-DD.md`. One file per day, two halves.

```markdown
# Daily Brief — YYYY-MM-DD

## Morning

### Today's Arc
1–2 themes for today, each one sentence + one line of "why this is the arc"
tied to source signals (Linear state change, Slack ask, Notion comment, etc.).

### Concrete Actions
3–5 actions, each tagged to a theme. Format:
- [ ] [theme tag] Action verb-phrase — `<source link or reference>` — est. effort
       why: one-line rationale tying back to signal

### Supporting Signal (collapsed by default)
- Linear: bullets of state changes / new comments since last brief, with links
- Slack: bullets of mentions, DMs, threads requiring response
- Notion: bullets of mentions and comments on pages I authored
- (Transcripts: deferred until Google Meet flow stabilizes)

### Carried From Yesterday
Pulled from prior evening's "Tomorrow's Lead" section, if present.
If no prior evening file exists, omit this section.

---

## Evening

### Reflection
Per theme + per action, status with one of: ✅ done, 🟡 partial, ❌ skipped.
1–2 lines per item on what happened.

### Decisions Captured
"<decision> — <context> — <link>". Pulled from Slack/Linear/Notion activity
during the day. Feeds the auto-memory system.

### Open Loops
- Waiting on others: things you asked others for that haven't come back
- Owed to others: things others asked of you that you haven't answered

### Stakeholder Summary
2–3 lines, public-tone, paste-ready for Linear status / standup / Slack.

### Tomorrow's Lead
1–2 candidate themes/actions to start tomorrow with, based on what shifted today.
Tomorrow morning's run reads this and proposes it back as a starting point.
```

Conventions:
- File path: `workspace/daily/YYYY-MM-DD.md` (today's local date, ISO format).
- Status markers: emoji (`✅ 🟡 ❌`) for compactness.
- Source links: always inline; never reference an item without a clickable link.
- Carry-over: yesterday's "Tomorrow's Lead" → today's "Carried From Yesterday".
