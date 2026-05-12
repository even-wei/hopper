# End-of-Day Task Rubric

Score each item as pass/fail.

## Bootstrap (Stage 0)

- Reads today's `workspace/daily/<today>.md` if present.
- If today's file is missing, halts and asks: reconstruct or free-form day-summary? Default fallback is free-form day-summary on user choice.
- Pulls source data since the morning section's timestamp.
- For each morning theme/action, searches sources for evidence of progress (Linear state changes by user, PRs merged, Slack messages user sent, Notion comments user posted).

## Reflection (Stage 1)

- Presents every morning theme + every morning action with inferred status (`done | partial | skipped`) and an evidence line per item.
- Uses a single batched input gate (one round, all items at once) — not per-item.

## Decisions (Stage 2)

- Extracts decisions from today's source activity, sourced with links.
- Asks the input gate to confirm / edit / add.

## Open Loops (Stage 3)

- Two lists: waiting-on-others and owed-to-others.
- Cross-references previous days' open loops still unresolved (carries them forward).
- Asks the input gate to add / drop / mark resolved.

## Stakeholder Summary (Stage 4)

- Drafts 2–3 lines based on confirmed reflection + decisions, public-tone, paste-ready.
- Asks the input gate to edit / accept.

## Tomorrow's Lead (Stage 5)

- Proposes 1–2 candidate themes/actions for tomorrow based on open loops + partial actions + late signals + deferred items.
- Asks the input gate to edit / accept.

## File Write & Follow-ups (Stage 6)

- Appends evening section to `workspace/daily/<today>.md` matching the template schema exactly.
- Existing evening section (re-run case) is amended by default; agent shows current content and asks confirmation before mutating.
- Optional follow-ups menu offered exactly once with three options: Linear status update / Slack message / stash decisions to auto-memory.
- Auto-memory stash, when accepted, writes a `project` or `reference` type memory file at `~/.claude/projects/-Users-evenwei-InfuseAI-workspace/memory/` and updates `MEMORY.md` index.

## Edge Cases

- No morning file → free-form day-summary fallback (per user choice).
- Re-run on same day → amend by default; never silently overwrite.
- Reflection contradicts evidence → flag once, accept user call.
- MCP tool unavailable → "Source X unavailable" noted; run continues; `[source X unavailable]` recorded in file.

Pass threshold: all items pass.
