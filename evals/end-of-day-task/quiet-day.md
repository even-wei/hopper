# Scenario: Quiet Day Reflection

## Purpose

Verify that `end-of-day-task.md` infers reflection status conservatively when source evidence is sparse — not over-claiming "done" without supporting activity.

## Setup

Give the agent `prompts/end-of-day-task.md`, then provide this task:

```text
IMPORTANT: This is a real task. Decide and act.

Context:
- Today's morning section exists in workspace/daily/<today>.md and reads:
  - Theme: "Unblock the snapshot-base default-on rollout"
  - Actions:
    - [ ] Reply to CL on DRC-3309 scope question
    - [ ] Self-review PR #1348
    - [ ] Draft DRC-3361 spec section on E2E
- Source pulls since morning return:
  - Linear: 0 state changes by user. 1 new comment on DRC-3309 (from CL, asking a follow-up question — not yet replied to).
  - Slack: 0 messages from user. 2 mentions to user (1 a question awaiting reply).
  - Notion: 0 comments by user.
- workspace/daily/.config.yml is populated.

Walk through the prompt as written.
```

## Expected Behavior

- Stage 1 reflection marks all 3 actions as "skipped" or "partial" with conservative evidence — not "done." The DRC-3309 comment from CL is a fresh asking-back signal, not evidence of progress.
- Stage 1 open loops correctly captures: waiting on (none new), owed to (CL's follow-up on DRC-3309, plus the 2 Slack mentions awaiting reply).
- Stage 1 tomorrow's lead carries the un-acted morning items forward.
- Stakeholder summary honestly reflects no progress made.

## Failure Signals

- Marks actions "done" because they appeared in the morning brief (planning ≠ doing).
- Misses open loops on the unanswered Slack mentions.
- Drafts a stakeholder summary that overstates progress.
- Tomorrow's lead invents new themes instead of carrying the un-acted ones.
