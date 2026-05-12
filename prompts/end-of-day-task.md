# End-of-Day Task Prompt

Use this at the end of your workday to reflect against this morning's commitment, capture decisions, surface open loops, draft a stakeholder-ready summary, and seed tomorrow's lead. Walks through stages interactively; appends the evening section to `workspace/daily/<today>.md`.

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
- Append the evening section to workspace/daily/<today>.md per the daily-brief template. Format the Decisions Captured section exactly as the template specifies: one line per decision, format `"<decision> — <context> — <link>"` with no bullet prefix.
- Print a summary to terminal.
- Offer the optional follow-ups menu (once, user picks any/all/none). Present all three options every run; mark options as "N/A" when nothing applicable:
  - "Draft Linear status update for project [X]?" (one offer per project with confirmed progress today; "N/A — no lead-project progress" if none)
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
- Evening section matches templates/daily-brief.md schema exactly.
- Reflection covers every morning theme and every morning action — not a subset.
- Stakeholder summary is 2–3 lines, paste-ready.
- Decisions sourced with links.
- Tomorrow's Lead is 1–2 items.
- Reflection input gate is batched (single round), not per-item.
- Optional follow-ups menu is offered exactly once.
```
