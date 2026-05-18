---
name: end-of-day
description: Use at the end of the workday to reflect against the morning's commitment, capture decisions, surface open loops, draft a stakeholder-ready summary, and seed tomorrow's lead. Appends evening section to workspace/daily/<today>.md. Triggers on "end of day", "evening brief", "wrap up the day", "daily debrief", "close the day".
---

# End-of-Day Daily Brief

Walks through the evening reflection in stages and appends to `workspace/daily/<today>.md`. Reads scope IDs from `workspace/daily/.config.yml`.

The Codex-readable canonical source for this prompt is `hopper/prompts/end-of-day-task.md`. The full prompt body is inlined below for Claude Code skill consumers, followed by the daily-brief schema appendix so the skill is self-contained regardless of where it's invoked.

## Prompt

```text
Goal:
- Help the user close the workday with a decision-shaped reflection: did the morning's commitment land, what decisions got made, what loops are open, what's the stakeholder-ready summary, and what's tomorrow's lead. Append the evening section to workspace/daily/<today>.md when confirmed.

Inputs:
- workspace/daily/<today>.md (must exist; if missing, see Edge Cases)
- workspace/daily/.config.yml (linear_user, linear_lead_projects, slack_user, notion_user)
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

Walk-Through (interactive — input gate at each stage):

Stage 0 — Bootstrap (silent):
- Read today's workspace/daily/<today>.md.
- If missing → halt and ask: "No morning brief found for today. Reconstruct one from sources before reflecting, or just write a free-form day-summary?" Default to free-form day-summary on user choice.
- Pull source data per Source Rules.
- For each morning theme/action, infer status (done | partial | skipped) and gather one line of evidence.
- For each project in `linear_lead_projects`, fetch the latest project status update timestamp. Mark a project as **stale** if its last status update is >5 days ago, or if it has no status updates at all (and the project is in an active state — not Canceled / Completed-and-archived). Carry this list into Stage 6's follow-ups menu.

Stage 1 — Reflection (single batched gate):
- Present every morning theme + every morning action together, each with inferred status and evidence line.
- Input gate (single round, all items at once): "Confirm, correct, or add nuance to any of these. Reply with item-by-item edits or a global 'all good.'"

Stage 2 — Decisions captured:
- Extract decisions per the Decision Heuristic from today's source activity. Each: "<decision> — <context> — <link>".
- Input gate: "Confirm each, edit wording, or add ones I missed."

Stage 3 — Open loops:
- Two lists:
  - Waiting on others: things you asked others for that haven't come back.
  - Owed to others: things others asked of you that you haven't answered.
- Cross-reference previous days' open loops in workspace/daily/ — carry forward any still-unresolved.
- Input gate: "Anything to add, drop, or mark resolved?"

Stage 4 — Stakeholder summary:
- Draft 2–3 lines, public-tone, paste-ready, based on confirmed reflection + decisions.
- Input gate: "Edit or accept."

Stage 5 — Tomorrow's lead:
- Propose 1–2 candidate themes/actions for tomorrow based on: open loops, partial actions, today's late signals that didn't fit, deferred items.
- Input gate: "Edit or accept. (Tomorrow morning will read this and propose it back to you.)"

Stage 6 — Write & optional follow-ups:
- Append the evening section to workspace/daily/<today>.md per the daily-brief template (schema appended below). Format the Decisions Captured section exactly as the template specifies: one line per decision, format `"<decision> — <context> — <link>"` with no bullet prefix.
- Print a summary to terminal.
- Offer the optional follow-ups menu (once, user picks any/all/none). Present all three options every run; mark options as "N/A" when nothing applicable:
  - "Draft Linear status update for project [X]?" — offer once per lead project matching either trigger:
    - **Progress trigger:** project had confirmed progress today (issue state change, PR merged into a project issue, status comment from the user, etc.).
    - **Staleness trigger:** project is marked **stale** in Stage 0 (last status update >5 days ago, or never received one, and project is in an active state).
    Mark "N/A — no lead-project needs an update" if no project matches either trigger.
  - "Draft Slack message to update [#channel]?"
  - "Stash decisions to auto-memory?" ("N/A — no decisions captured" if Stage 2 produced none; otherwise write a project or reference type memory file at ~/.claude/projects/-Users-evenwei-InfuseAI-workspace/memory/ and update MEMORY.md index)

Edge Cases:
- No morning file: halt and ask reconstruct vs free-form day-summary; default to free-form day-summary on user choice.
- Re-run on same day: read existing evening section first; ask "amend or replace?" — default to amend; never silently overwrite.
- Reflection contradicts evidence (user says "done" but agent saw no activity): flag once, accept user call.

Output:
- File written (appended): workspace/daily/<today>.md (evening section).
- Terminal summary.
- Optional follow-up artifacts (Linear status drafts, Slack drafts, memory writes) when accepted.

Quality Bar:
- Evening section matches the daily-brief schema below exactly.
- Reflection covers every morning theme and every morning action — not a subset.
- Stakeholder summary is 2–3 lines, paste-ready.
- Decisions sourced with links.
- Tomorrow's Lead is 1–2 items.
- Reflection input gate is batched (single round), not per-item.
- Optional follow-ups menu is offered exactly once.
- Lead-project staleness scan: every active lead project with no status update in the last 5 days is surfaced in Stage 6's Linear-update offer, even without same-day progress.
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
