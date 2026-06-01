# Start-of-Day Task Prompt

Use this at the start of your workday to surface signals across Slack, Linear, and Notion since your last brief, advance the reversible work autonomously, and commit to 1–2 themes with 3–5 concrete actions for the day. Writes `workspace/daily/<today>.md` as it goes, then surfaces a single batched review of queued decisions.

```text
Goal:
- Help the user decide today's focus by surfacing decision-relevant signals from Slack, Linear, and Notion since the last daily brief, advancing the mechanical work autonomously, and surfacing only genuine decisions in a single batched review. Write the morning section of workspace/daily/<today>.md as you go — the daily file is a working draft you can revise, not a commitment that needs pre-approval.

Inputs:
- workspace/daily/.config.yml (linear_user, slack_user, notion_user; optional standing_trusted list)
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

Autonomy Model (how this brief moves work forward):
- This brief is solicited — the user invoked it — so advance their work rather than pausing for permission at each step. Do the reversible, mechanical work; report it as an auditable trail ("moved X forward"), never "I decided for you." Reserve the user's attention for genuine, hard-to-reverse calls.
- Classify every candidate action by reversibility and stakes, NOT by which stage you are in:
  - Tier A — do it now, then log it (reversible AND low-stakes): read/triage signals; draft themes, actions, summaries, status-update text, tomorrow's lead; verify carry-over loops via MCP; write/append the daily file and stash decisions to auto-memory; add a comment to — or change the status of — an issue YOU own.
  - Tier B — queue it for review (irreversible OR high-stakes OR it leaves the workspace toward others): posting to Slack; messaging stakeholders; posting a project status update; commenting on / transitioning / closing someone ELSE's issue; sending email; dispatching a teammate that mutates external state (pushes commits, posts a PR review). Fully draft the artifact, then put it in the "Needs Your Call" queue — do not execute unattended.
  - Tier C — notify only (awareness, nothing to execute): stale-project flags, conflicting signals, FYI items. Surface; never auto-act.
- Escalation rule (apply literally; do NOT improvise on "this feels like it needs judgment"): if reversible AND low-stakes -> do it and log it; if irreversible OR high-stakes OR it leaves the workspace toward others -> queue it.
- Draft-only exceptions (queue but do NOT execute even on "accept" — the user posts): project status updates (draft-as-text policy) and Slack Connect channels (cannot post). Mark these "draft-only" in the queue; all other Tier-B items are "executable-on-accept."
- Standing trust (optional): if workspace/daily/.config.yml has a `standing_trusted:` list (e.g. `linear-own-comment`, `slack-internal-post`), promote those classes to auto-execute. This is a user-granted, user-revocable standing permission — NOT a time-based auto-accept timer. Absent the config, default every Tier-B item to the queue.

Review Surface — "Needs Your Call" (one batched pass, never per-stage):
- Always write a `### ⏳ Needs Your Call` block into the daily file. One entry per queued Tier-B item: the drafted artifact (text or link), a one-line why, the target (channel / issue / person), and a tag — `executable-on-accept` or `draft-only`. Omit the block only if the queue is empty.
- When the session is interactive, also present the whole queue as a SINGLE consolidated review at the end of the run. Resolve each item with one of: accept (execute now if executable-on-accept), edit (revise, then accept), respond (free-form redirect), skip (leave it — it stays in the block for later). Resolve the batch in one pass; do not reintroduce stage-by-stage gates.
- When the session is non-interactive, or the user steps away, the `Needs Your Call` block persists in the file and nothing Tier-B fires until the user resolves it; the next run re-surfaces unresolved items.

Walk-Through (autonomous-first; a single batched review at the end):

Stage 0 — Bootstrap & verify (silent, autonomous):
- Resolve time window per Source Rules. Read prior-evening "Tomorrow's Lead" from yesterday's file if present.
- Pull source data per Source Rules; score and rank items per the Signal Heuristic.
- Verify carry-overs yourself — do not punt MCP-answerable questions to the user. For each carried "Tomorrow's Lead" / "Owed to others" item whose source is MCP-checkable (Linear issue, Slack thread, PR, calendar), check the current state and mark it resolved or still-open from the evidence. Only flag "carry from N days ago, not verified — confirm still owed?" when the source is genuinely non-MCP (offline conversation, F2F, hallway). After weekend gaps, still verify via MCP first; ask only about the offline residue.
- Fetch the user's lead projects: `list_projects(member=linear_user)` filtered to `lead.id == linear_user` and `status.type ∈ {started, planned}`. Mark a project **stale** if its last status update is >5 days ago, or it has none (active state only — not Canceled / Completed-and-archived).

Stage 1 — Draft, advance & write (autonomous, no gate):
- Draft the full morning section: 1–2 themes (each tied to signal), 3–5 actions (each tagged to a theme, with a canonical source URL, an effort estimate, and a one-line why), Supporting Signal, and Carried From Yesterday (the prior-evening "Tomorrow's Lead" verbatim, annotated with the Stage 0 verification result).
- For colleague pings, draft as "ping <name> in #<channel> with @mention", not "DM <name>" — DMs hide cross-loop signal from collaborators who may have context. Reserve DMs for genuinely private content (1:1 agenda drafts, personal scheduling).
- Auto-fill for user-authored PRs in CHANGES_REQUESTED (from Stage 0): emit, per PR, exactly —
    - [ ] [Theme: Close review loops] Dispatch teammate via `/recce-dev:pr-review-response` for PR #NNN (<title>) — <url> — est. 30–45 min teammate run
           why: <repo> #NNN is CHANGES_REQUESTED; reviewer waiting since <updatedAt>
  If the user has no other themes today, "Close review loops" is the single theme; otherwise attach it alongside. The dispatch is a Tier-B teammate action (it pushes commits / replies to reviewers) — queue it in "Needs Your Call" as executable-on-accept; do NOT auto-run it.
- Perform Tier-A actions now and log each in the run tally (e.g. "verified 3 carries — DRC-XXXX already closed, dropped"; "moved own issue DRC-YYYY -> In Review").
- Pre-stage Tier-B items into the "Needs Your Call" queue, each fully drafted: status-update drafts for active/stale lead projects (draft-only), colleague pings, the PR-review-response dispatch. Tag each executable-on-accept or draft-only.
- Write workspace/daily/<today>.md morning section now — including the `### ⏳ Needs Your Call` block and, if any lead projects are stale, a "Stale lead projects" awareness bullet under Supporting Signal. Resolve every action's source link to a canonical URL (a bare ID like "DRC-3309" is not sufficient). Writing the file is Tier-A; it is a working draft.

Stage 2 — One batched review (the single gate):
- Present, in one pass: (a) the drafted Arc + Actions — "edit, replace, add, or 'all good'"; (b) the "Needs Your Call" queue — resolve each accept / edit / respond / skip. Surface any conflicting signals here explicitly; never pick silently.
- Execute accepted executable-on-accept items immediately (e.g. launch parallel `/recce-dev:pr-review-response` teammates, one per accepted PR dispatch); leave skipped and draft-only items in the block. Re-write the file with the resolutions and a short "Resolved this run" note.
- This is the only input gate. If the session is non-interactive, skip the live review — the queue persists in the file (see Edge Cases).

Stage 3 — Summarize:
- Print exactly 6 lines, in order:
  1. Today's arcs (numbered, comma-separated).
  2. First action (verb-phrase + source URL + estimate).
  3. File path (workspace/daily/<today>.md).
  4. Signal counts (Linear: N, Slack: M, Notion: K).
  5. Carry-over status (theme + action carried, or "no prior-evening carry-over").
  6. Autonomy tally: "Did autonomously: N · Queued for you: M (K resolved this run)".

Edge Cases:
- No prior evening file: say so in the Stage 2 review; omit "Carried From Yesterday" in the file; window defaults to 24h.
- Quiet day (low signal): still draft themes from carry-over and Linear backlog; the file is written either way.
- Conflicting signals: surface in the Stage 2 review explicitly; do not pick silently.
- Non-interactive session / user steps away: the file is written with the "Needs Your Call" block and nothing Tier-B fires; unresolved items re-surface next run. (The file is a working draft — writing it without a live review is expected, not a partial-write failure.)
- User redirects mid-review: apply the edit / respond and re-write the file; never leave it mid-mutation.
- Re-run on the same day: read the existing morning section first; ask "amend or replace?" before mutating.

Output:
- File written: workspace/daily/<today>.md (morning section + "Needs Your Call" block).
- Tier-A actions executed and logged; Tier-B items queued (and executed on accept in the review).
- Terminal summary: 6 lines.

Quality Bar:
- File matches the daily-brief schema (templates/daily-brief.md) exactly, including the "Needs Your Call" block.
- Themes count is 1–2; actions count is 3–5; every action has a canonical source URL and an effort estimate.
- Ranking labels visible in the digest; conflicting signals surfaced, not silently resolved.
- Exactly ONE input gate (the Stage 2 batched review) — no per-stage gates.
- Every candidate action classified Tier A / B / C and handled per the escalation rule; nothing Tier-B executed unattended; status updates and Slack Connect are always draft-only.
- Tier-A work is done before the review and reported in the autonomy tally.
- "Carried From Yesterday" included iff a prior-evening file existed; carries verified via MCP where checkable.
- Stale lead projects surfaced as awareness only (never auto-elevated into themes/actions).
```
