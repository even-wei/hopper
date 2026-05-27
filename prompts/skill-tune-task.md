# Skill-Tune Task Prompt

Use this to tune a Claude Code SKILL.md (or any agent skill markdown file) against recent failure/success transcripts using SkillOpt-style bounded edits. Ports the SkillOpt paper ([arXiv:2605.23904v2](https://arxiv.org/abs/2605.23904)) into a single-session, human-in-the-loop workflow where you are the validation gate. Each pass proposes at most `L` atomic edits (`append`, `insert_after`, `replace`, `delete`), shows them as diffs, applies the accepted ones, and persists the rejected ones to a per-skill buffer that informs future passes.

```text
Goal:
- Improve a target SKILL.md (or any agent skill markdown file) by applying at most L bounded edits derived from recent transcripts of the skill being used. The edits are proposed as structured add/insert_after/replace/delete operations, ranked by systematic impact, and accepted only after per-edit human review. Rejected edits are persisted as negative feedback for future passes. An optional slow-update pass writes longitudinal guidance into a protected region of the skill.

Inputs:
- A target skill file path (e.g. `plugins/hopper/skills/start-of-day/SKILL.md`, or any markdown file used as an agent skill).
- Zero or more transcript sources, in this order of preference:
  1. Explicit paths to Claude Code session transcripts (`~/.claude/projects/<project>/<session-id>.jsonl`).
  2. Bare session IDs (resolved against `~/.claude/projects/*/<session-id>.jsonl`).
  3. The current Claude Code session (default fallback when no transcripts are given).
  4. Pasted friction-moment snippets in the user's invocation message.
- Optional `L` — the textual learning rate (max edits applied this pass). Default 4 (paper's default for medium-effort).
- Optional `--slow-update` flag — opt into Stage 7's epoch-boundary slow update.

Hard Rules (failure-mode guardrails):

1. PROTECTED SLOW-UPDATE REGION. If the target skill contains `<!-- SLOW_UPDATE_START -->`…`<!-- SLOW_UPDATE_END -->` markers, never propose edits that target text inside that region. Only Stage 7 may rewrite it. This is the textual analogue of the paper's separation between step-level edits and epoch-wise slow updates — without it, fast local changes silently erase durable longitudinal lessons.

2. BOUNDED EDITS ONLY. Each edit is exactly one of four atomic ops: `append`, `insert_after`, `replace`, `delete`. No edit may rewrite the whole document; no two accepted edits may target overlapping regions of the same text. The total accepted-edit count must be ≤ L. The bounded budget is the textual learning rate — without it, the optimizer can silently overwrite useful rules under cover of "rewriting for clarity."

3. EVIDENCE-DRIVEN, NOT SPECULATIVE. Every proposed edit must cite at least one concrete moment from the transcripts (line / quote / paraphrase). Proposals not anchored to evidence are dropped before ranking, even if they sound reasonable. This is what makes accepted edits procedural rather than instance-specific.

4. NO DUPLICATE OF EXISTING SKILL CONTENT. Edits that restate something already in the skill are dropped during merge. A success-driven edit only earns inclusion if the pattern is not yet covered.

5. NO REPEAT OF REJECTED EDITS. Before proposing, read `<skill-dir>/_rejected_edits.md` if it exists. Skip proposals that semantically match a recently-rejected entry. If a proposal is similar but materially different (e.g. wider scope, sharper wording), surface the prior rejection inline so the human can re-evaluate.

6. HUMAN IS THE GATE. No edit is applied without explicit user approval. The paper's "strictly greater than current validation score" rule maps to "the human said yes." Ties (uncertainty, "maybe later") are rejected, not silently accepted. The user may also edit the wording inline before accepting.

7. ONE PASS, ONE SKILL FILE. Re-runs against the same skill overwrite the same SKILL.md in place and append (never overwrite) the rejected-edit buffer. Never create per-pass copies of SKILL.md.

8. SLOW-UPDATE IS OPT-IN AND SEPARATE. Stage 7 runs only when `--slow-update` is passed. It uses a different prompt (longitudinal comparison across recent passes), writes only inside the protected region, and is itself subject to the human gate.

Walk-Through (numbered stages = execution order):

Stage 0 — Bootstrap (silent):
- Resolve target skill file → absolute path. Read full content.
- Detect protected region: scan for `<!-- SLOW_UPDATE_START -->` / `<!-- SLOW_UPDATE_END -->`. If present, record offsets and isolate the region's content. Edits in stages 2–6 must not target inside this region.
- Resolve transcripts in the order under "Inputs" above. If falling back to the current session, surface "Using current session as evidence — confirm or pass explicit transcripts." and proceed only after confirmation.
- Read rejected-edit buffer at `<skill-dir>/_rejected_edits.md` if it exists. Parse into a list of `{timestamp, op, target, content, reason}`.
- Resolve L: default 4. If user passed `L=N`, use N. Clamp to [1, 16].

Stage 1 — Evidence classification (interactive):
- For each transcript / friction moment, classify as one of:
  - **failure** — user pushed back, repeated an instruction, manually fixed Claude's output, said "no" / "stop" / "not what I asked", or the skill produced a downstream artifact that needed rework.
  - **success** — user accepted output without correction, explicitly confirmed, or the skill produced a clean downstream artifact.
  - **neutral** — skill was invoked but neither clearly helped nor hurt; do not use as evidence.
- If a transcript contains multiple distinguishable moments, split it: one classification per moment.
- Print the classification table with citing quotes. Input gate: "Confirm classifications or override (`failure → success`, etc.)."
- If after confirmation there are zero failure AND zero success moments, STOP — print "No actionable evidence; nothing to tune." Do not write the buffer.

Stage 2 — Reflection (parallel):
- **Failure analysis**: read all failure moments together. Identify the most prevalent, systematic patterns (not edge cases). Propose at most L atomic edits. Each proposal must include `{op, target_if_applicable, content, source_type: "failure", support_count, cited_moments, reasoning}`. Generalizable phrasing only; no instance-specific names, paths, or entities. Skip proposals that match a rejected-edit-buffer entry; if close-but-different, flag the prior rejection inline.
- **Success analysis**: read all success moments together. Identify patterns common across multiple moments and NOT already covered in the skill. Propose at most L atomic edits with `source_type: "success"`. Be conservative — prefer reinforcing existing sections over adding new top-level ones.

Stage 3 — Merge (sequential):
- **merge_failure**: dedupe similar wording (keep best-worded), resolve conflicts (choose stronger justification or synthesize), preserve unique insights, drop proposals supported by only one moment if task-specific, ensure no two edits target the same text region. Record `support_count` per merged edit.
- **merge_success**: dedupe conservatively. Only include edits for patterns not yet in the skill.
- **merge_final**: failure edits take priority. If a failure and success edit cover the same point, keep the failure version. Preserve success edits that fill orthogonal gaps. Carry forward `support_count` and `source_type`.

Stage 4 — Rank + clip:
- Score merged edits in this priority order: systematic impact (recurring failure addressed > edge case) > complementarity (fills a gap > duplicates) > generality (principle > instance-specific) > actionability (concrete > vague).
- Sort and clip to top-L. If fewer than L survive, that is fine — the cap is a ceiling, not a target. Zero accepted edits is a valid outcome (and is itself informative — the existing skill is already strong enough on the observed evidence).

Stage 5 — Validation gate (per-edit human review):
- For each of the up-to-L ranked edits, render as a unified diff against the current SKILL.md:
  - Show `op`, target anchor (if applicable), full replacement content, `source_type`, `support_count`, and the citing moment(s).
  - Show the rendered before/after where the edit lands.
- For each, input gate: "**accept** / **reject** / **edit-then-accept** / **skip-rest**".
- "edit-then-accept": user pastes amended wording, then it counts as accepted (and the user's chosen op overrides the proposed op without re-ranking).
- "skip-rest": stop the review; remaining un-reviewed edits are not applied and not buffered (they are simply dropped).
- Reject reasons are captured (free-text or shortcut: `too-vague` / `duplicate` / `instance-specific` / `wrong-target` / `would-break-existing`).

Stage 6 — Apply + persist:
- Apply accepted edits in dependency-safe order: deletes first, then replaces, then insert_afters, then appends. This prevents earlier anchors from shifting under later edits.
- Re-read the file after each edit to refresh offsets. If an anchor is no longer unique, abort that one edit and surface "Anchor lost after prior edit — re-run skill-tune to retry this edit alone." Other accepted edits still apply.
- Write the modified SKILL.md in place.
- Append rejected edits to `<skill-dir>/_rejected_edits.md`, one block per edit, schema in the appendix of the SKILL.md. Create the file with a header if it does not exist.
- Print exactly 6 lines to terminal, one per line in this order:
  1. Skill path.
  2. Accepted: N/L edits (with op-counts: e.g. "1 append, 2 insert_after, 1 replace").
  3. Rejected: M edits (buffered to `<skill-dir>/_rejected_edits.md`).
  4. Evidence: F failure moments, S success moments.
  5. Slow-update region: present / absent (and whether `--slow-update` is suggested next).
  6. Suggested next: run the skill once more on a real task, then re-tune with the new transcript.

Stage 7 — Slow-update (opt-in, only when `--slow-update` is passed):
- Read all prior `<skill-dir>/_rejected_edits.md` entries plus the current SKILL.md and (if available) the most recent prior version of SKILL.md in git history (`git -C <skill-dir> log -p --follow SKILL.md`).
- Identify persistent failure patterns the rejected buffer keeps surfacing AND stable success patterns the step-level edits keep reinforcing.
- Compose one concise longitudinal guidance block addressed directly to the agent ("When you encounter X, always do Y"). Keep it ≤200 tokens, complementary to (not duplicating) the main skill body.
- Wrap it between `<!-- SLOW_UPDATE_START -->` and `<!-- SLOW_UPDATE_END -->` markers and place it as a top-level section before the first `##` heading of the main body (or update in place if already present).
- Human gate: show the candidate block + before/after diff of the protected region; require explicit accept. Tie = reject.
- This is the only stage that may modify content inside `<!-- SLOW_UPDATE_START -->`…`<!-- SLOW_UPDATE_END -->`.

Edge Cases:
- Target file has no markdown structure (e.g. plain text): only `append` and `replace` ops are usable. Surface this and proceed.
- Target file lacks the protected region: all 7 stages still run; the slow-update step inserts the region for the first time when invoked.
- Rejected buffer is full of stale entries: if buffer exceeds 50 entries, suggest archiving older entries to `_rejected_edits.archive.md` before this pass.
- User passes a session ID for a session that is still active (in-progress): warn that evidence may be partial; proceed only if user confirms.
- If accepting all proposed edits would push the skill past ~2000 tokens (the paper's empirical ceiling for compact, transferable skills), warn the user and ask whether to drop the lowest-ranked edits or proceed.
- Skill file is tracked by git and dirty (uncommitted edits): warn and ask whether to stash or proceed; do not silently overwrite uncommitted work.

Output:
- Updated SKILL.md at the input path.
- Updated/created `<skill-dir>/_rejected_edits.md`.
- Optional updated protected region (only when `--slow-update`).
- Terminal summary: 6 lines (per Stage 6), plus an additional line if Stage 7 ran.

Quality Bar:
- ≤ L accepted edits this pass (textual learning rate respected).
- Zero edits targeting the protected region (unless Stage 7 ran).
- Every accepted edit traces to ≥1 cited transcript moment.
- No accepted edit duplicates existing skill content.
- No accepted edit semantically matches a rejected-buffer entry.
- Rejected-edit buffer appended-to, never overwritten.
- All input gates present (Stage 1 confirmation, per-edit Stage 5 reviews, optional Stage 7 gate).
```
