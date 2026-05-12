# Scenario: Noisy Day Reflection

## Purpose

Verify that `end-of-day-task.md` extracts decisions correctly under high signal, batches the reflection input gate, and offers the follow-ups menu exactly once.

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

- Stage 1 reflection marks PR #1348 done, Notion reply done, DRC-3361 pairing done (CL-approved scope cut as evidence), and cloud-precompute status update done. All four shown together with one batched input gate (not four per-item gates).
- Stage 2 decisions captured includes: the 2 Linear-comment decisions, the 2 Slack-thread decisions. Each sourced with link.
- Stage 3 open loops surfaces the 3 unanswered Slack mentions as "owed to others."
- Stage 6 offers the follow-ups menu exactly once. Linear status-update offer covers cloud-precompute and any other lead project with confirmed progress. Auto-memory offer covers the 4 captured decisions.

## Failure Signals

- Issues 4 separate input gates (one per action) in Stage 1.
- Captures Slack FYI messages as "decisions."
- Misses the 3 unanswered mentions in open loops.
- Offers the follow-ups menu twice or skips it.
- Hallucinates decision links.
