---
name: end-of-day
description: Use at the end of the workday to reflect against the morning's commitment, capture decisions, surface open loops, draft a stakeholder-ready summary, and seed tomorrow's lead — advancing reversible work autonomously (incl. stashing decisions to memory) and queuing only genuine calls in a single batched review. Appends the evening section to workspace/daily/<today>.md. Triggers on "end of day", "evening brief", "wrap up the day", "daily debrief", "close the day".
---

# End-of-Day Daily Brief

Advances the evening reflection autonomously and appends to `workspace/daily/<today>.md`, surfacing queued decisions in a single batched review. Reads scope IDs from `workspace/daily/.config.yml`.

The Codex-readable canonical source for this prompt is `hopper/prompts/end-of-day-task.md`. The full prompt body is inlined below for Claude Code skill consumers, followed by the daily-brief schema appendix so the skill is self-contained regardless of where it's invoked.

## Prompt

```text
Goal:
- Help the user close the workday with a decision-shaped reflection: did the morning's commitment land, what decisions got made, what loops are open, what's the stakeholder-ready summary, and what's tomorrow's lead. Advance the reversible work autonomously and surface only genuine calls in a single batched review. Append the evening section to workspace/daily/<today>.md as you go.

Inputs:
- workspace/daily/<today>.md (must exist; if missing, see Edge Cases)
- workspace/daily/.config.yml (linear_user, slack_user, notion_user; optional standing_trusted list)
- Live source pulls since this morning's timestamp

Source Rules:
- Time window: from today's morning-section timestamp to now. If no morning section, see Edge Cases.
- For each morning theme/action, search sources for evidence the user advanced it: Linear state changes the user made, PRs the user merged, Slack messages the user sent, Notion comments the user posted, all since window-start.
- Linear, Slack, Notion source-pull rules match start-of-day-task.md (mine + lead projects; mentions + DMs + my threads; mentions + comments on my pages).
- Transcripts: deferred. Skip; do not block.
- If any source's MCP tool is unavailable, print "Source X unavailable, continuing without it" and continue. Note `[source X unavailable]` in the daily file.

Decision Heuristic:
- A "decision" is a concrete commitment, scope change, approval, or trade-off recorded in Slack / Linear / Notion. Examples: "CL approved snapshot-base default-on rollout", "deferred DRC-3361 to next week", "decided to land PR #1348 separate from #1349".
- Distinguish from FYI updates ("the deploy went out") and questions ("should we …?"). Decisions have a definite verb: approved, decided, deferred, scoped, agreed, rejected.

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
- Read today's workspace/daily/<today>.md. If missing -> halt and ask: "No morning brief found for today. Reconstruct one from sources, or write a free-form day-summary?" Default to free-form day-summary on user choice.
- Pull source data per Source Rules. For each morning theme/action, infer status (done | partial | skipped) with one line of evidence.
- Verify open-loop closure yourself: for each prior open loop whose source is MCP-checkable, check for a closure signal (issue state change, new comment, thread reply, PR merge, calendar event) and mark it resolved or still-open from evidence — do not punt MCP-answerable carries to the user. Flag "carry from N days ago, not verified — confirm still owed?" only for genuinely non-MCP / offline sources. After weekend gaps, verify via MCP first; ask only about the offline residue.
- Fetch the user's lead projects (`list_projects(member=linear_user)` filtered to `lead.id == linear_user` and `status.type ∈ {started, planned}`); mark each **stale** (no status update in >5 days, or none, active state only) and note which had progress today.

Stage 1 — Draft, advance & write (autonomous, no gate):
- Build the full evening section: Reflection (every morning theme + every morning action, with status + evidence line), Decisions Captured (per the Decision Heuristic, each "<decision> — <context> — <link>"), Open Loops (Waiting on others / Owed to others, verified per Stage 0), Stakeholder Summary (2–3 lines, public-tone, paste-ready), and Tomorrow's Lead (1–2 items; colleague pings drafted as channel @mention, not DM).
- Perform Tier-A actions now and log each: stash decisions to auto-memory (write a `project` or `reference` type memory file at ~/.claude/projects/-Users-evenwei-InfuseAI-workspace/memory/ and update the MEMORY.md index); move the status of any issue YOU own where the reflection makes the transition unambiguous.
- Pre-stage Tier-B items into the "Needs Your Call" queue, each fully drafted: a Linear status-update draft for every lead project that had progress today OR is stale (draft-only, per the draft-as-text policy); Slack update drafts for relevant channels (executable-on-accept for internal channels, draft-only for Slack Connect).
- Append the evening section + the `### ⏳ Needs Your Call` block to workspace/daily/<today>.md now. Format Decisions Captured exactly as the schema specifies: one line per decision, no bullet prefix. Appending is Tier-A.

Stage 2 — One batched review (the single gate):
- Present, in one pass: (a) Reflection + Decisions + Open Loops + Stakeholder Summary + Tomorrow's Lead — "confirm, correct, or add nuance; reply item-by-item or a global 'all good'"; (b) the "Needs Your Call" queue — resolve each accept / edit / respond / skip.
- Execute accepted executable-on-accept items; leave skipped and draft-only items in the block. Re-write the file with the resolutions.
- If the user's reflection contradicts the evidence (says "done" but no activity seen), flag once and accept the user's call.
- This is the only input gate — Reflection / Decisions / Loops / Summary / Lead are all confirmed in this one pass, no per-stage gates. If the session is non-interactive, skip the live review — the queue persists in the file (see Edge Cases).

Stage 3 — Summarize:
- Print a terminal summary: themes delivered / partial / skipped tally, decisions count, open-loop counts (waiting / owed), and an autonomy tally: "Did autonomously: N · Queued for you: M (K resolved this run)".

Edge Cases:
- No morning file: halt and ask reconstruct vs free-form day-summary; default to free-form on user choice.
- Re-run on same day: read existing evening section first; ask "amend or replace?" — default to amend; never silently overwrite.
- Reflection contradicts evidence: flag once in the Stage 2 review, accept the user's call.
- Non-interactive session / user steps away: the evening section + "Needs Your Call" block are written and nothing Tier-B fires; unresolved items re-surface next run.

Output:
- File written (appended): workspace/daily/<today>.md (evening section + "Needs Your Call" block).
- Tier-A actions executed and logged (including the auto-memory stash); Tier-B items queued (and executed on accept in the review).
- Terminal summary with autonomy tally.

Quality Bar:
- Evening section matches the daily-brief schema (below) exactly, including the "Needs Your Call" block.
- Reflection covers every morning theme and every morning action — not a subset.
- Stakeholder summary is 2–3 lines, paste-ready; decisions sourced with links; Tomorrow's Lead is 1–2 items.
- Exactly ONE input gate (the Stage 2 batched review) — Reflection / Decisions / Loops / Summary / Lead confirmed in one pass; no per-stage gates.
- Every candidate action classified Tier A / B / C; nothing Tier-B executed unattended; status updates and Slack Connect always draft-only.
- Decisions stashed to auto-memory autonomously (Tier-A) when present; the old follow-ups menu is replaced by the "Needs Your Call" queue.
- Open loops verified via MCP where checkable; lead-project staleness still surfaced (now as queued status-update drafts).
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

### ⏳ Needs Your Call
Tier-B actions the agent drafted but did NOT execute (irreversible / high-stakes /
leaves the workspace toward others). Omit this section if empty. One entry per item:
- [ ] (executable-on-accept | draft-only) Action verb-phrase — <#channel / issue / person> — why: one line
       draft: <the ready-to-send artifact, inline or linked>

---

## Evening

### Reflection
Per theme + per action, status with one of: ✅ done, 🟡 partial, ❌ skipped.
1–2 lines per item on what happened.

### Decisions Captured
"<decision> — <context> — <link>". Pulled from Slack/Linear/Notion activity
during the day. Stashed to the auto-memory system automatically (Tier-A).

### Open Loops
- Waiting on others: things you asked others for that haven't come back
- Owed to others: things others asked of you that you haven't answered

### Stakeholder Summary
2–3 lines, public-tone, paste-ready for Linear status / standup / Slack.

### Tomorrow's Lead
1–2 candidate themes/actions to start tomorrow with, based on what shifted today.
Tomorrow morning's run reads this and proposes it back as a starting point.

### ⏳ Needs Your Call
Tier-B actions drafted but not executed (status-update drafts, Slack messages, etc.).
Omit if empty. Resolve accept / edit / respond / skip. One entry per item:
- [ ] (executable-on-accept | draft-only) Action verb-phrase — <#channel / issue / person> — why: one line
       draft: <the ready-to-send artifact, inline or linked>
```

Conventions:
- File path: `workspace/daily/YYYY-MM-DD.md` (today's local date, ISO format).
- Status markers: emoji (`✅ 🟡 ❌`) for compactness.
- Source links: always inline; never reference an item without a clickable link.
- Carry-over: yesterday's "Tomorrow's Lead" → today's "Carried From Yesterday".
- Needs Your Call: queued Tier-B actions (irreversible / high-stakes / leaves the workspace). Resolve accept / edit / respond / skip in one batched pass; omit the section when empty.
