---
name: pr-explain
description: Use to generate a per-PR review brief HTML that helps reviewers focus on judgment (decision points) instead of mechanical correctness. Input is a GitHub PR number or URL. Output is a standalone HTML file at workspace/pr-views/. Triggers on "explain this PR", "review brief", "pr-explain", and explicit invocations like `/hopper:pr-explain <PR#>`.
---

# Per-PR Review Brief

Generates a standalone HTML page per PR that surfaces the decision points worth a reviewer's attention. Mechanical correctness (lint, tests, security, backwards compat) collapses to ticks in the hero sidebar; the page's centerpiece is Decision Points. Trivial PRs (no decisions, no reviews, no refs) skip artifact generation and emit a 3-line terminal notice.

The Codex-readable canonical source is `hopper/prompts/pr-explain-task.md`. The full prompt body is inlined below for Claude Code skill consumers, followed by the HTML output schema appendix so the skill is self-contained regardless of where it's invoked.

## Prompt

```text
Goal:
- Generate a per-PR review brief HTML at workspace/pr-views/<owner>-<repo>-<pr>.html that helps a reviewer focus on judgment (decision points) rather than mechanical correctness. Mechanical work — lint, tests, security checks, backwards compat — collapses to ticks in the hero sidebar. The page's centerpiece is Decision Points: places where a reviewer's judgment can change the implementation.

Inputs:
- A PR number or full GitHub PR URL (e.g. `1286` or `https://github.com/DataRecce/recce-cloud-infra/pull/1286`)
- workspace/scratch/superpowers-brainstorming.html (style vocabulary reference — Inter font, soft shadows, panel cards)
- workspace/pr-views/datarecce-recce-cloud-infra-1286.html (the prototype output — copy its HTML structure exactly)

Source Rules:
- PR data: `gh api repos/{owner}/{repo}/pulls/{N}` for metadata; `/commits`, `/reviews`, `/files`, `/comments` with `--paginate`; also `gh api repos/{owner}/{repo}/issues/{N}/comments --paginate` for the general PR comment thread (where self-reviews and author-posted verdicts live; the `/reviews` endpoint does NOT include comment-based self-reviews).
- CI status: `gh pr checks {N} --json name,state,bucket` (valid `gh pr checks --json` fields are: `bucket, completedAt, description, event, link, name, startedAt, state, workflow`; `conclusion` is NOT a valid field — using it errors out).
- Linear refs in PR body (DRC-XXXX or linear.app URLs): fetch via `mcp__linear-server__get_issue` for title + project. If MCP unavailable, omit the motivation chain panel; do not block.
- Paired-PR refs in body (`owner/repo#N`): fetch via `gh api`
- For the user's own repos, `gh search prs --author=@me` is available — but this skill uses an explicit PR identifier, not @me search.

Hard Rules (failure-mode guardrails):

1. NO DECISION THEATER. A decision earns a card only if the PR evidence shows a live alternative was explicitly considered, requested, rejected, deferred, or made impossible by a named constraint. You MUST cite that evidence (PR body section, review comment, commit message, TODO marker, etc.). Do not infer alternatives from ordinary implementation choices. If the reviewer-counterargument is merely "could have implemented differently," drop the card. Promote a card only if a reviewer's verification action differs from doing nothing — if the reviewer's response to the decision would be "glance and move on," it doesn't merit a card. If two cards merit the same verification action, merge them. Cap at 5 cards, but never fill to the cap. Zero or one card is valid and preferred for quiet PRs.

2. FAILED QUALITY CHECKS BREAK OUT. Any non-passing check (lint, tests, auth/security, backwards compat, migration, paired-PR contract, CI, self-review verdict) goes ABOVE the hero as a red/amber blocker panel. Never hide a failed tick inside the green sidebar cluster.

3. "NOT DONE" ENTRIES EARN THEIR SPACE. Each entry must pass: "a reasonable reviewer would ask why this is absent." Drop diff-negative trivia (e.g. "did not modify unrelated module" is not worth a line).

4. NO PANEL DECORATION. If a panel would only restate the PR description in prettier form, skip it entirely. Better to ship 4 panels that change reviewer behavior than 7 that look complete.

5. CONSTRAINTS ARE COMPACT ROWS, NOT CARDS. When only one path was viable due to a named technical constraint, render as a one-line "Constraint: X (no live alternative because Y)" between the Decision Points panel and the What-NOT-Done panel. Do not classify a constraint as a "decision" — that framing invites inflation. Omit the constraint row entirely if the constraint is uninteresting (e.g. "must use the framework we already use").

6. ONE ARTIFACT PER PR. Re-runs overwrite the same file. Never accumulate per-round versions. The artifact reflects the current state.

7. EVENT-TRIGGERED, NOT SCHEDULED. This skill is invoked on demand (by hand) or chained from another skill (e.g. after `gh pr create`). It is never invoked on a cron schedule. Stay quiet until asked.

Walk-Through (numbered stages = execution order; render order in the HTML follows the Output Schema, not stage numbers):

Stage 0 — Bootstrap (silent):
- Parse input → (owner, repo, number). Accept bare number only if `gh repo view` resolves the current repo; otherwise require a URL.
- TRIVIAL-PR SKIP CHECK (run before any other fetches beyond PR metadata): if all of the following hold, STOP and write no HTML file:
  - `additions + deletions < 50`
  - `commits ≤ 1`
  - `reviews = 0`
  - `issue_comments` excluding the PR author = 0
  - PR body contains no Linear refs, no paired-PR refs, no spawned follow-up refs
  Print exactly these 3 lines to terminal and exit:
  1. `PR #N is trivially small — nothing to brief. Read the diff.`
  2. `<owner>/<repo>#<N> — <title>`
  3. `Open with: gh pr view <N> --web`
- Otherwise fetch in parallel: PR metadata, commits (--paginate), reviews (--paginate), files (--paginate), issue comments (--paginate), review-line comments (--paginate). Also fetch CI status.
- Detect author-as-reviewer multi-round patterns: scan the PR-author-posted issue comments for verdict keywords (`NO-GO`, `CHANGES_REQUESTED`, `GO`, `APPROVED`, `BLOCKER`, `ISSUE`, claude[bot] verdict-styled headings). Treat each such comment as a self-review round. The timeline (Stage 5) uses BOTH the `/reviews` endpoint AND these author-self-comment rounds. The Self-review verdict quality check (Stage 1) reads from the latest self-review round.
- Scan PR body for: Linear refs (regex `DRC-\d+` or `linear.app/.*/issue/.*`), paired-PR refs (regex `[\w-]+/[\w-]+#\d+`), spawned follow-up Linear refs.
- For each Linear ref: fetch issue title + project (graceful skip if MCP unavailable).

Stage 1 — Quality checks (run FIRST after bootstrap; if anything fails, the reviewer's attention shifts immediately):
- Categories — include ONLY relevant ones (don't show N/A):
  - Lint (ruff / eslint / etc.) — for code PRs
  - Tests (count + suite name; note new test files added) — for code PRs
  - Auth / security — if any auth surface touched
  - Backwards compat — only if schema/API surface changed
  - Migration — only if DB schema involved
  - Paired-PR contract — only if a paired PR is referenced
  - CI (gh pr checks aggregate)
  - Self-review verdict — read the latest author-posted self-review round (Stage 0). PASS if approved or no verdict posted. FAIL/BLOCKER if a verdict requesting changes (`NO-GO`, `CHANGES_REQUESTED`, `BLOCKER`, `ISSUE`) was posted AND no later commit addresses it. The "addresses" test: a later commit whose message references the verdict, OR a same-author comment posted after the verdict that closes it out.
- For docs / eval-scaffolding / style-only PRs where code categories don't apply: substitute the code categories with the most fitting alternates: Markdown lint (if a markdown-lint workflow exists), Link check (if a link-checker runs), Build script (if a verify script lives in the PR like `build_fixtures.sh`), Self-review verdict. Still include CI and Paired-PR if applicable. The "include ONLY relevant" rule remains — don't pad with N/A categories.
- For each: pass → green tick in the hero sidebar with one-line note ("Tests ✓ — 200 PR + 4584 full suite").
- For ANY failure or pending: emit a red blocker panel ABOVE the hero. Do not put it in the green cluster. The page must read top-to-bottom: blockers first, then hero, then decisions.

Stage 2 — Decision identification (the critical stage):
- Candidate sources, scan in order. For each candidate, you MUST be able to point at the line of PR evidence proving the alternative was considered:
  a. PR body sections matching "Decisions", "Trade-offs", "Approach", "Why this design"
  b. Commit messages tagged like `fix(*): address ... review`, `fix(*): address NO-GO`, `fix(*): address ... feedback` — these flag forced revisits
  c. Inline review comments where the reviewer raised an alternative (`Why not X?`, `Consider Y`)
  d. Author response comments that show alternative-considered (`I considered X, chose Y because`)
  e. Inline TODO markers with "deferred", "follow-up", "out of scope"
  f. Author self-review issue comments (Stage 0 self-review rounds) — when the author critiques their own decision and reverses, that round IS the citable evidence
- For each candidate, apply Hard Rule 1: cite the evidence AND apply the verification-action test ("would a reviewer's response differ from doing nothing?"). If you cannot answer YES with concrete verification, drop the card.
- Classify:
  - TRADE-OFF (card): a live alternative existed and was rejected with rationale
  - DEFERRED (card): explicit choice to defer to a follow-up, anchored in BOTH a TODO (code) AND a Linear issue
  - CONSTRAINT (row, NOT a card): only one path was viable due to a named technical constraint. Render as a compact row outside the Decision Points panel (see Hard Rule 5).
- For each kept TRADE-OFF or DEFERRED card, produce fields:
  - Question (one line, neutral phrasing)
  - Type (one of: Trade-off / Deferred)
  - Options: chosen path (prefix with ✓ visually), rejected path (strike-through visually) with one-line rejection reason
  - Why this one (rationale)
  - Risk if wrong (what breaks; especially what tests would NOT catch)
  - Review ask (what to verify in the diff — 1-2 bullets; this is the verification-action test made concrete)
  - Evidence (commit SHAs, file:line refs, review comment timestamps, self-review-round timestamps — these are the PR-evidence citations required by Hard Rule 1)
  - Origin badge: "caught by round-N self-review BLOCKER" (use this phrasing whether the round was an external `/reviews` submission OR an author-posted self-review issue comment — see Stage 0) or "caught by <reviewer> round-N ISSUE" or "scope-cap" — visible in card head alongside the type tag
- Cap: 5 cards. Never fill to the cap. Zero or one card is valid and preferred for quiet PRs. If 0 decisions pass the filter, emit the panel with: "This PR contains no decisions worth a review card — mechanical refactor or routine implementation. Follow the diff and the quality checks." Do not fabricate.

Stage 3 — "What was NOT done":
- Sources: PR body negative sections (`Not changed`, `Preserved`, `Out of scope`, `Backwards compatible`), commit messages mentioning "untouched"/"unchanged"/"preserve", inline TODOs deferred to follow-ups, asymmetries in scope (e.g. "list endpoints intentionally skip new fields").
- Apply Hard Rule 3 filter: would a reasonable reviewer ask why this is absent? If yes, include with a one-line reason. If no, drop.
- 0-8 entries typical. Better to drop than to pad.

Stage 4 — Motivation chain (compressed, secondary):
- Mermaid `flowchart LR` showing: Project → upstream slice (if any, marked ✓ if merged) → DRC issue → this PR ⟷ paired PR(s) → spawned follow-up issue(s)
- Highlight this PR with green fill (#147a63)
- Small panel, gray "·" num icon. This is reference, not action.
- Skip the panel entirely if no Linear ref AND no paired PR AND no spawned follow-up.

Stage 5 — Timeline:
- One column per calendar day spanned (UTC). Self-review rounds (Stage 0) appear on the timeline as review markers alongside external `/reviews` markers — both use the same dot shapes (NO-GO red, CHANGES_REQUESTED amber, APPROVED green-outlined). Tag self-review markers with `(self)` so the reader can distinguish.
- Events: commit (blue dot), fix-commit (green dot, annotated "↑ responds to <review time>"), NO-GO (red dot), CHANGES_REQUESTED (amber dot), APPROVED (green outlined dot), merge (black dot), doc-only commit (gray dot).
- A fix-commit is any commit between a NO-GO / CHANGES_REQUESTED review (external OR self) and the next review state on the same PR, OR any commit whose message explicitly says it addresses review feedback (e.g. `fix(*): address ... review`). No arbitrary time window.
- Compact: time (UTC), 9px dot, short label, SHA in inline code.

Stage 6 — Takeaways:
- 3 bullets distilling future-self / handoff knowledge — insights that apply NEXT TIME the codebase hits a similar shape.
- These are NOT a recap of what shipped. They are the "you should walk away knowing X" of an algorithm-teaching study guide.
- Each bullet should be quotable on its own.

Stage 7 — Compose & write:
- File path: `workspace/pr-views/<owner>-<repo>-<pr>.html` (owner and repo lowercased, joined by `-`; e.g. `datarecce-recce-cloud-infra-1286.html`).
- Single HTML file. No node_modules. Mermaid via CDN: `https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js`.
- Style vocabulary: copy the CSS variables and panel/decision/hero classes from `workspace/pr-views/datarecce-recce-cloud-infra-1286.html` (the prototype). Match it byte-for-byte on the structural CSS. If the prototype file is missing, STOP and print no artifact. Do not approximate the CSS — a missing prototype is a setup error worth surfacing, not a corner to cut.
- Print to terminal exactly these 4 lines after writing:
  1. `Wrote review brief: file:///absolute/path.html`
  2. `Decisions: N (X trade-offs, Y deferred)` — do NOT count constraints in this number; constraint rows are not decisions.
  3. `Quality: all green` OR `Quality: N failed/pending — first: <name>`
  4. `Open with: open "file:///absolute/path.html"` (macOS hint; do not auto-run open)
- Any required diagnostic (e.g. Linear MCP unavailable) appends to line 3 after the quality summary, separated by ` · `. Never print additional lines beyond these 4. If you cannot satisfy the 4-line contract for any reason, fail before writing the file rather than degrade the contract. (Note: the trivial-PR skip in Stage 0 has its OWN 3-line terminal contract and never writes an HTML file — this 4-line contract applies only when an artifact is produced.)

Verification (before declaring the artifact ready, perform these checks):
- File exists at the expected path.
- File contains exactly one `<header class="hero">` element and exactly one `<section class="panel panel-decisions">` element.
- Decision card count: 0–5 (`<article class="decision">` elements).
- Failed quality checks (if any): present in a `<section class="blockers">` element ABOVE the hero, not inside the sidebar.
- Motivation chain panel: present only if at least one of Linear / paired-PR / spawned-follow-up was discoverable; otherwise the panel is absent entirely (never empty).
- Terminal output: exactly 4 lines (or 3 lines if trivial-PR skip fired), matching the format in Stage 7 (or Stage 0).
- If any of these fails, do not declare success — re-run or report the gap explicitly.

Edge Cases:
- PR is draft: skip the full brief. Emit a 1-panel stub: title + draft badge + "No review brief until ready for review." Reason: review attention is wasted on drafts.
- PR is open (not merged) and unreviewed: timeline shows commits only; hero status badge is amber "OPEN"; brief is still useful for self-review.
- PR is closed without merge: status badge red "CLOSED"; brief reflects final state at close.
- PR is huge (>1500 LOC outside test files): emit a single-line callout in the hero: "PR size flag: <N> LOC outside tests. Consider whether this should have been split." Not a decision card — a meta-signal.
- Linear MCP unavailable: omit motivation chain panel; append ` · motivation-chain unavailable (Linear MCP unreachable)` to terminal line 3.
- 0 decisions pass the filter: Decision Points panel includes the explicit "mechanical refactor — follow the diff" message (Stage 2). Other panels still emit.
- Multi-round review history: only the final state of each decision is shown in cards. Round-by-round detail belongs in the timeline.
- Self-only-review PRs: `/reviews` will return `[]`; the rounds live in issue-comment edit history (Stage 0 detects them). Treat self-review rounds as first-class for both the Self-review verdict quality check (Stage 1) and the timeline (Stage 5).
- Idle PR with unaddressed self-review verdict: Stage 1 Self-review verdict fails → blocker panel above hero. This is the primary signal a reviewer needs ("why has this been open for 6 weeks?") and the brief surfaces it before anything else.

Output:
- File written: `workspace/pr-views/<owner>-<repo>-<pr>.html` (overwrite if exists), OR no file if trivial-PR skip fired in Stage 0.
- Terminal: 4-line summary (see Stage 7), OR 3-line trivial-PR notice (see Stage 0).
- No side effects: do not post comments to GitHub, do not modify Linear, do not auto-open the browser.

Quality Bar:
- Page renders in Safari/Chrome from a `file://` URL without errors (Mermaid loads from CDN, fonts fall back gracefully, no broken layout).
- Decision Points panel passes the evidence-citation filter AND the verification-action test for every card.
- Failed quality checks (if any) are surfaced ABOVE the hero, not inside the sidebar.
- "NOT done" entries each have a one-line reason and pass the reviewer-asks filter.
- Motivation chain panel is OMITTED when no Linear / paired / follow-up exists — never empty.
- Visual continuity with the prototype: Inter font, soft shadow cards, purple decision accents, gray secondary panels.
- Trivial PRs are skipped at Stage 0 with the 3-line terminal notice — no 19KB HTML to say "nothing here."
```

## Output Schema

Generated artifact path: `workspace/pr-views/<owner>-<repo>-<pr>.html`. One file per PR, overwritten on re-run. Trivial PRs (Stage 0 skip path) produce NO file — only a 3-line terminal notice. Render order top-to-bottom on the page:

```html
<!doctype html>
<html lang="en">
<head>
  <!-- title, meta, mermaid CDN, inline CSS (copy from prototype) -->
</head>
<body>
  <main>
    <!-- OPTIONAL: BLOCKERS panel — emitted ONLY if any quality check failed/pending (incl. Self-review verdict) -->
    <section class="blockers">
      <!-- red/amber surface above hero; the primary signal for idle PRs with unaddressed self-NO-GO -->
    </section>

    <!-- HERO: title + lede + meta sidebar (stats + quality ticks) -->
    <header class="hero">
      <div class="intro">
        <p class="eyebrow">Review Brief · Prototype</p>
        <h1>PR #N — <title></h1>
        <p class="lede"><one-line context></p>
        <p class="meta-line"><PR URL · author · approver · paired links></p>
        <p><strong>How to read this brief:</strong> spend your time on Decision Points; quality is ticks on the right.</p>
      </div>
      <aside class="meta">
        <span class="badge badge-merged|badge-open|badge-closed">● STATUS · timestamp</span>
        <div class="meta-group"><h3>Stats</h3>...</div>
        <div class="meta-group"><h3>Quality checks</h3>...green ticks only (incl. Self-review verdict)...</div>
      </aside>
    </header>

    <!-- CENTERPIECE: Decision Points panel (purple accent, largest) -->
    <section class="panel panel-decisions">
      <h2><span class="num">D</span> Decision Points — where your judgment is needed</h2>
      <p class="panel-lede">...</p>
      <div class="decisions">
        <article class="decision">
          <div class="decision-head">
            <h3><Question></h3>
            <div>
              <span class="decision-id">D1</span>
              <span class="type-tag type-tradeoff|type-deferred">Trade-off|Deferred</span>
              <span class="origin-tag"><origin (self-review or external)></span>
            </div>
          </div>
          <dl class="field">
            <dt>Options</dt><dd>chosen ✓ / rejected ✗ with reason</dd>
            <dt>Why this one</dt><dd>...</dd>
            <dt>Risk if wrong</dt><dd>...</dd>
          </dl>
          <dl class="field review-ask">
            <dt>Review ask</dt><dd><ul><li>...</li></ul></dd>
          </dl>
          <div class="status-row"><strong>Evidence:</strong> SHA · file:line · self-review-round timestamp</div>
        </article>
        <!-- up to 5 cards, but never fill to the cap -->
      </div>
    </section>

    <!-- OPTIONAL: CONSTRAINT row(s) between Decisions and NOT-Done -->
    <section class="constraint-row">Constraint: ... (no live alternative because ...)</section>

    <!-- What was NOT done — negative space -->
    <section class="nope">
      <h2><span class="num">∅</span> What was deliberately NOT done</h2>
      <ul><li><strong>Title.</strong> <span class="reason">reason</span></li>...</ul>
    </section>

    <!-- Motivation chain — small, gray; OMITTED entirely if no Linear/paired/follow-up exists -->
    <section class="panel panel-mot">
      <h2><span class="num">·</span> Where this PR sits</h2>
      <div class="mermaid">...flowchart LR...</div>
    </section>

    <!-- Timeline — small, gray; includes BOTH /reviews events AND self-review-round events (tagged `(self)`) -->
    <section class="panel panel-time">
      <h2><span class="num">·</span> What happened, in time order</h2>
      <div class="timeline">... day columns with event rows ...</div>
      <div class="legend">...color key...</div>
    </section>

    <!-- Takeaways — blue accent -->
    <section class="takeaway">
      <h2>Walk-away knowledge (for future-self / handoff)</h2>
      <ol><li>...</li><li>...</li><li>...</li></ol>
    </section>

    <footer>...</footer>
  </main>
  <script>mermaid.initialize({...});</script>
</body>
</html>
```

Conventions:
- Status badge color: green (MERGED) / amber (OPEN) / red (CLOSED).
- Decision type tag: purple (Trade-off) / gray (Deferred). Constraints render as their own row, not as decision cards.
- Quality ticks: green only. Any failure (incl. Self-review verdict) surfaces as a BLOCKERS panel above the hero.
- Decision IDs (D1, D2, ...) are renderer-assigned, monospace, gray; used only for reference.
- Mermaid theme: light, Inter font, `lineColor: #5f6f78`, `primaryBorderColor: #c8d4da`.
- Self-review timeline markers tagged `(self)` to distinguish from external `/reviews` events.

## Failure modes to recognize and refuse

- "Decision theater": padding the panel with fabricated trade-offs to look thorough. Hard Rule 1 has TWO bite-points now: cite PR evidence for the rejected alternative, AND pass the verification-action test (a reviewer's response must differ from doing nothing). A 0-card brief is honest; a 5-card brief of fabrications is harmful.
- "Decoration": a panel that restates the PR description in prettier form. Skip the panel.
- "Hidden failure": rendering a failed check as a green tick "with caveat." A failure surfaces ABOVE the hero or nowhere. This includes idle PRs with unaddressed self-review verdicts — Self-review verdict is a first-class quality check.
- "Padding NOT-done": listing everything the PR didn't touch. Filter to entries a reviewer would actually ask about.
- "Constraint inflation": classifying a forced choice as a "decision" to fill the card panel. Constraints are compact rows, not decision cards (Hard Rule 5).
- "Trivial brief": producing a 19KB HTML to convey "nothing here." Trivial PRs (Stage 0 skip path) emit only a 3-line terminal notice — no file.
