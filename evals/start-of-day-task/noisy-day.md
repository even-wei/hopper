# Scenario: Noisy Day

## Purpose

Verify that `start-of-day-task.md` ranks correctly under high-volume input and does not over-include — sticking to 1–2 themes and 3–5 actions even when ten things competed for top-of-list.

## Setup

Give the agent `prompts/start-of-day-task.md`, then provide this task:

```text
IMPORTANT: This is a real task. Decide and act.

Context:
- Today is mid-sprint Tuesday. Prior evening file exists.
- Prior evening's "Tomorrow's Lead":
  - Theme: "Land DRC-3349 column-dedupe fix"
  - Action: "Self-review PR #1348 and merge"
- Source pulls return:
  - Linear: 12 new comments across 7 issues (3 on lead projects, 4 on assigned issues). 2 PRs awaiting your review (one on a lead project). 1 issue you're assigned was just blocked.
  - Slack: 8 @-mentions (3 are direct asks, 2 are decision-blocking threads, 3 are FYI). 4 DMs (one is "can you review this by EOD?"). 5 active threads with new replies.
  - Notion: 2 mentions on a design spec you authored, 1 comment on a lead project's status page.
- workspace/daily/.config.yml is populated.

Walk through the prompt as written.
```

## Expected Behavior

- Stage 1 digest shows per-source counts (12/8/2 etc.) and surfaces top-ranked items with ranking labels.
- Conflicting signals (e.g., the blocked Linear issue vs. the carried-over PR review) are surfaced explicitly, not picked silently.
- Stage 2 proposes 1–2 themes — not 4. Themes synthesize across the noise rather than naming each item.
- Stage 3 proposes 3–5 actions — not 12. Actions are the highest-leverage items per theme.
- Lower-tier signal (FYI mentions, ambient Notion edits) are compressed into one-liners or omitted.

## Failure Signals

- Proposes a theme per source.
- Lists every Linear comment as an action.
- Skips the digest and jumps straight to themes.
- Picks the "blocked Linear issue" over the carried-over PR without flagging the conflict.
- Hides ranking labels.
