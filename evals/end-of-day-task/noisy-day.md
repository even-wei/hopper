# Scenario: Noisy Day Reflection

## Purpose

Verify that `end-of-day-task.md` extracts decisions correctly under high signal, confirms the reflection in a single batched review, stashes decisions to auto-memory autonomously, and queues follow-ups in "Needs Your Call" (resolved once).

## Setup

Give the agent `prompts/end-of-day-task.md`, then provide this task:

```text
IMPORTANT: This is a real task. Decide and act.

Context:
- Today's morning section exists with:
  - Theme: "Land DRC-3349 column-dedupe fix"
  - Actions:
    - [ ] Self-review PR #1348 and merge
    - [ ] Reply to Notion comment on snapshot-base spec
    - [ ] Pair with CL on DRC-3361 scope (15:00)
    - [ ] Update Linear status on cloud-precompute project
- Source pulls since morning return:
  - Linear: user merged PR #1348 (DRC-3349). User updated cloud-precompute project status. 4 new comments on user's lead-project issues, 2 of which are decisions ("CL approved scope cut on DRC-3361"; "deferred DRC-3360 docs to next week").
  - Slack: user posted in 6 threads, 2 of which include explicit decisions ("agreed to ship DRC-3349 fix in next release"; "rejected the alternative dedupe approach"). 3 mentions still awaiting reply.
  - Notion: user posted 2 comments — one was the awaited reply on snapshot-base spec.
- workspace/daily/.config.yml is populated.

Walk through the prompt as written.
```

## Expected Behavior

- Stage 1 drafts the reflection marking PR #1348 done, Notion reply done, DRC-3361 pairing done (CL-approved scope cut as evidence), and cloud-precompute status update done. All four confirmed together in the single Stage 2 batched review (not four per-item gates).
- Stage 1 decisions captured includes: the 2 Linear-comment decisions, the 2 Slack-thread decisions. Each sourced with link, and stashed to auto-memory autonomously (Tier-A).
- Stage 1 open loops surfaces the 3 unanswered Slack mentions as "owed to others."
- Stage 2's "Needs Your Call" queue holds the Linear status-update drafts (cloud-precompute + any lead project with confirmed progress) as draft-only, resolved in one pass; the 4 captured decisions are stashed to auto-memory autonomously in Stage 1 (no menu prompt).

## Failure Signals

- Issues separate input gates per action instead of the single Stage 2 batched review.
- Captures Slack FYI messages as "decisions."
- Misses the 3 unanswered mentions in open loops.
- Re-introduces a separate follow-ups menu instead of the "Needs Your Call" queue, or stashes decisions only on prompt instead of autonomously.
- Hallucinates decision links.
