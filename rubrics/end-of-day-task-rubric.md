# End-of-Day Task Rubric

Score each item as pass/fail.

## Bootstrap & verify (Stage 0)

- Reads today's `workspace/daily/<today>.md` if present.
- If today's file is missing, halts and asks: reconstruct or free-form day-summary? Default fallback is free-form day-summary on user choice.
- Pulls source data since the morning section's timestamp.
- For each morning theme/action, searches sources for evidence of progress (Linear state changes by user, PRs merged, Slack messages user sent, Notion comments user posted) and infers status (done | partial | skipped) with an evidence line.
- Verifies open-loop closure via MCP for every prior loop with an MCP-checkable source — marks resolved / still-open from evidence; only flags "not verified — confirm still owed?" for genuinely non-MCP / offline sources (especially after weekend gaps).
- Fetches lead projects; marks staleness and notes which had progress today.

## Autonomy classification (applies throughout)

- Every candidate action is classified Tier A / Tier B / Tier C by reversibility / stakes, not by stage.
- Tier-A work (drafting reflection/decisions/loops/summary/lead, MCP loop verification, auto-memory stash, own-issue status) runs autonomously, not gated.
- Tier-B items (Linear status-update posts, Slack messages, others' issues) are queued in "Needs Your Call", never executed unattended; status updates and Slack Connect are draft-only.
- Standing-trusted classes from `.config.yml`, if present, auto-execute; absent config, all Tier-B is queued.

## Draft, advance & write (Stage 1)

- Drafts Reflection covering every morning theme + every morning action (not a subset), each with status + evidence line.
- Extracts decisions from today's source activity, each sourced with a link, formatted one line per decision with no bullet prefix.
- Drafts Open Loops (waiting-on-others + owed-to-others), verified per Stage 0.
- Drafts Stakeholder Summary (2–3 lines, public-tone, paste-ready) and Tomorrow's Lead (1–2 items; colleague pings as channel @mention, not DM).
- Stashes captured decisions to auto-memory autonomously (Tier-A): writes a `project` or `reference` type memory file at `~/.claude/projects/-Users-evenwei-InfuseAI-workspace/memory/` and updates the `MEMORY.md` index. No menu prompt — done automatically whenever decisions exist.
- Pre-stages Tier-B items into "Needs Your Call": Linear status-update drafts for lead projects with progress today OR stale (draft-only); Slack update drafts (executable-on-accept internal, draft-only Connect).
- Appends the evening section + "Needs Your Call" block to `workspace/daily/<today>.md` before the review, matching the template schema exactly.

## One batched review (Stage 2)

- Exactly ONE input gate: Reflection + Decisions + Open Loops + Stakeholder Summary + Tomorrow's Lead confirmed together in a single pass, plus the "Needs Your Call" queue resolved accept / edit / respond / skip — not per-stage gates, not a separate follow-ups menu.
- Accepted executable-on-accept items executed; skipped and draft-only items left in the block; file re-written with resolutions.
- Reflection contradicting evidence (user says "done", no activity seen) flagged once; user call accepted.
- Non-interactive session: live review skipped; queue persists in the file; no Tier-B fires.

## Summarize (Stage 3)

- Prints a terminal summary: theme delivered/partial/skipped tally, decisions count, open-loop counts (waiting / owed), and an autonomy tally ("Did autonomously: N · Queued: M (K resolved)").

## Edge Cases

- No morning file → free-form day-summary fallback (per user choice).
- Re-run on same day → amend by default; never silently overwrite.
- Reflection contradicts evidence → flag once, accept user call.
- Non-interactive / user steps away → evening section + "Needs Your Call" block written; nothing Tier-B fires; unresolved items re-surface next run.
- MCP tool unavailable → "Source X unavailable" noted; run continues; `[source X unavailable]` recorded in file.

Pass threshold: all items pass.
