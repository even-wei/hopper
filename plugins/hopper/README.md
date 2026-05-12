# hopper plugin

Daily-brief skills for Claude Code. Part of the [hopper](../../README.md) toolkit.

## Skills

- **`hopper:start-of-day`** — surface decision-relevant signals from Linear, Slack, Notion since the last brief; propose 1–2 themes with 3–5 actions interactively; write the morning section of `workspace/daily/<today>.md`.
- **`hopper:end-of-day`** — reflect against the morning's commitment, capture decisions, surface open loops, draft a stakeholder summary, seed tomorrow's lead; append to `workspace/daily/<today>.md`.

## Configuration

The skills read `workspace/daily/.config.yml` for scope IDs (`linear_user`, `linear_lead_projects`, `slack_user`, `notion_user`, `github_user`).

## Source of truth

The prompt bodies are inlined inside each `SKILL.md`. The same prompts also live at `../../prompts/<name>-task.md` for Codex usage.
