# hopper

> Operational skills toolkit for AI coding agents — Claude Code, Codex, and friends.

Named after [Grace Hopper](https://en.wikipedia.org/wiki/Grace_Hopper) — Navy Rear Admiral, compiler pioneer, and the person who literally pulled the first bug out of a computer. This repo carries the same spirit: the small reliable helpers that translate between you and the machine.

## What's inside

```
hopper/
├── .claude-plugin/marketplace.json    Claude Code marketplace manifest
├── plugins/hopper/                    Claude Code plugin
│   ├── .claude-plugin/plugin.json
│   └── skills/
│       ├── start-of-day/SKILL.md      hopper:start-of-day
│       ├── end-of-day/SKILL.md        hopper:end-of-day
│       ├── pr-explain/SKILL.md        hopper:pr-explain
│       └── skill-tune/SKILL.md        hopper:skill-tune
├── prompts/                           Codex / generic agent canonical source
│   ├── start-of-day-task.md
│   ├── end-of-day-task.md
│   ├── pr-explain-task.md
│   └── skill-tune-task.md
├── templates/                         Schemas referenced by prompts
│   └── daily-brief.md
├── AGENTS.md                          Entry point for Codex
└── CLAUDE.md                          Entry point for Claude Code
```

## Use with Claude Code

```bash
/plugin marketplace add even-wei/hopper
/plugin install hopper@hopper
```

To develop against a local checkout instead of the remote:

```bash
/plugin marketplace add /path/to/hopper
/plugin install hopper@hopper
```

After install, the following skills are callable:

- `hopper:start-of-day`, `hopper:end-of-day` — interactive daily brief that writes `workspace/daily/<today>.md`.
- `hopper:pr-explain` — generates a standalone per-PR review brief HTML at `workspace/pr-views/`.
- `hopper:skill-tune` — tunes a target SKILL.md against recent transcripts using SkillOpt-style bounded edits with a per-skill rejected-edit buffer ([arXiv:2605.23904](https://arxiv.org/abs/2605.23904)).

## Use with Codex

Codex reads `AGENTS.md` and treats `prompts/<name>-task.md` as the canonical task prompts. The `templates/` directory carries schemas referenced by the prompts.

## Configuration

Both daily-brief skills expect `workspace/daily/.config.yml` in the directory you invoke them from. Example:

```yaml
linear_user: <uuid>
slack_user: <slack-uid>
notion_user: <notion-uid>
github_user: <gh-login>
```

Lead projects are not configured here — both skills fetch them live from Linear
(projects where `lead == linear_user` and status is `started` or `planned`).

## Roadmap

- [x] `start-of-day`, `end-of-day` (v0.1.0)
- [x] `pr-explain` (v0.1.1)
- [x] `skill-tune` (v0.1.3) — SkillOpt-style human-in-the-loop skill tuner
- [ ] `plan`, `bugfix`, `feature`, `review`, `retro`, `handoff`, `batch-plan`, `perspective-review` (sourced from [agent-ops](https://github.com/InfuseAI/agent-ops) prompts)
- [ ] Additional secretarial skills as needed

## License

MIT
