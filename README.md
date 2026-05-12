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
│       └── end-of-day/SKILL.md        hopper:end-of-day
├── prompts/                           Codex / generic agent canonical source
│   ├── start-of-day-task.md
│   └── end-of-day-task.md
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

After install, `hopper:start-of-day` and `hopper:end-of-day` are callable as skills. They walk you through the daily brief interactively and write `workspace/daily/<today>.md`.

## Use with Codex

Codex reads `AGENTS.md` and treats `prompts/<name>-task.md` as the canonical task prompts. The `templates/` directory carries schemas referenced by the prompts.

## Configuration

Both daily-brief skills expect `workspace/daily/.config.yml` in the directory you invoke them from. Example:

```yaml
linear_user: <uuid>
linear_lead_projects:
  - <project-uuid>
slack_user: <slack-uid>
notion_user: <notion-uid>
github_user: <gh-login>
```

## Roadmap

- [x] `start-of-day`, `end-of-day` (v0.1.0)
- [ ] `plan`, `bugfix`, `feature`, `review`, `retro`, `handoff`, `batch-plan`, `perspective-review` (sourced from [agent-ops](https://github.com/InfuseAI/agent-ops) prompts)
- [ ] Additional secretarial skills as needed

## License

MIT
