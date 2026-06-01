# Scenario: Quiet Day

## Purpose

Verify that `start-of-day-task.md` produces a coherent brief even when source activity is minimal — relying on carry-over and Linear backlog rather than reactive signals.

## Setup

Give the agent `prompts/start-of-day-task.md`, then provide this task:

```text
IMPORTANT: This is a real task. Decide and act.

Context:
- Today is Monday after a 3-day weekend.
- Prior evening's "Tomorrow's Lead" exists and reads:
  - Theme: "Unblock the snapshot-base default-on rollout"
  - Action: "Reply to CL on DRC-3309 scope question"
- Source pulls return:
  - Linear: 0 new comments, 0 state changes since prior brief.
  - Slack: 1 mention (a "FYI: deploy went out Friday" message in #recce-eng).
  - Notion: 0 comments, 0 mentions.
- workspace/daily/.config.yml is populated.

Walk through the prompt as written.
```

## Expected Behavior

- Stage 0 verifies and surfaces the prior-evening "Tomorrow's Lead" prominently; the Stage 3 summary reports per-source counts honestly (mostly zeros).
- Stage 1 drafts themes built around the carry-over (the snapshot-base rollout), not invented from thin air (reviewed in Stage 2).
- Stage 1 drafts 3–5 actions even though fresh signal is sparse — drawing from the carried theme and Linear backlog.
- Stage 1 writes the file with "Carried From Yesterday" populated.

## Failure Signals

- Forces 5 actions purely from the FYI Slack message and overstates its importance.
- Drops "Carried From Yesterday" because today's signals are quiet.
- Skips drafting any theme/action with a "nothing to do today" output instead of drawing from carry-over and backlog.
- Proposes 3+ themes to fill the file.
