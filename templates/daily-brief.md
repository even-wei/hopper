# Daily Brief Template

Generated artifact path: `workspace/daily/YYYY-MM-DD.md`. One file per day, two halves.

```markdown
# Daily Brief — YYYY-MM-DD

## Morning

### Today's Arc
1–2 themes for today, each one sentence + one line of "why this is the arc"
tied to source signals (Linear state change, Slack ask, Notion comment, etc.).

### Concrete Actions
3–5 actions, each tagged to a theme. Format:
- [ ] [theme tag] Action verb-phrase — `<source link or reference>` — est. effort
       why: one-line rationale tying back to signal

### Supporting Signal (collapsed by default)
- Linear: bullets of state changes / new comments since last brief, with links
- Slack: bullets of mentions, DMs, threads requiring response
- Notion: bullets of mentions and comments on pages I authored
- (Transcripts: deferred until Google Meet flow stabilizes)

### Carried From Yesterday
Pulled from prior evening's "Tomorrow's Lead" section, if present.
If no prior evening file exists, omit this section.

### ⏳ Needs Your Call
Tier-B actions the agent drafted but did NOT execute (irreversible / high-stakes /
leaves the workspace toward others). Omit this section if empty. One entry per item:
- [ ] (executable-on-accept | draft-only) Action verb-phrase — <#channel / issue / person> — why: one line
       draft: <the ready-to-send artifact, inline or linked>

---

## Evening

### Reflection
Per theme + per action, status with one of: ✅ done, 🟡 partial, ❌ skipped.
1–2 lines per item on what happened.

### Decisions Captured
"<decision> — <context> — <link>". Pulled from Slack/Linear/Notion activity
during the day. Stashed to the auto-memory system automatically (Tier-A).

### Open Loops
- Waiting on others: things you asked others for that haven't come back
- Owed to others: things others asked of you that you haven't answered

### Stakeholder Summary
2–3 lines, public-tone, paste-ready for Linear status / standup / Slack.

### Tomorrow's Lead
1–2 candidate themes/actions to start tomorrow with, based on what shifted today.
Tomorrow morning's run reads this and proposes it back as a starting point.

### ⏳ Needs Your Call
Tier-B actions drafted but not executed (status-update drafts, Slack messages, etc.).
Omit if empty. Resolve accept / edit / respond / skip. One entry per item:
- [ ] (executable-on-accept | draft-only) Action verb-phrase — <#channel / issue / person> — why: one line
       draft: <the ready-to-send artifact, inline or linked>
```

## Conventions

- File path: `workspace/daily/YYYY-MM-DD.md` (today's local date, ISO format).
- Status markers: emoji (`✅ 🟡 ❌`) for compactness.
- Source links: always inline; never reference an item without a clickable link.
- Carry-over: yesterday's "Tomorrow's Lead" → today's "Carried From Yesterday".
- Needs Your Call: queued Tier-B actions (irreversible / high-stakes / leaves the workspace). Resolve accept / edit / respond / skip in one batched pass; omit the section when empty.
