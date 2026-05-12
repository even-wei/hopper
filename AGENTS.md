# Hopper — Agent Instructions

This repo is operational tooling for AI coding agents. It is designed to work with **Codex** and **Claude Code** (and any other agent that can read Markdown prompts).

## How Codex should use this repo

- Treat `prompts/<name>-task.md` as the canonical task prompt set.
- Treat `templates/<name>.md` as schema referenced by those prompts.
- When invoked with a prompt name like `start-of-day-task`, read `prompts/start-of-day-task.md` and execute its instructions.
- The `plugins/` directory is Claude-Code-specific packaging; Codex can ignore it.

## How Claude Code should use this repo

- Install as a plugin: `/plugin marketplace add /path/to/hopper` then `/plugin install hopper@hopper`.
- After install, skills are available as `hopper:start-of-day` and `hopper:end-of-day`.
- The skill files (`plugins/hopper/skills/<name>/SKILL.md`) inline the prompt body — same content as `prompts/<name>-task.md`, no Read needed at invocation time.

## Editing rules

- Prompts and skills are intentionally duplicated to keep each surface self-contained. When you change a prompt, change both the `prompts/<name>-task.md` file and the matching `plugins/hopper/skills/<name>/SKILL.md` body.
- Keep prompt templates concise and operational.
- Prefer Markdown with structured sections.
- Use concrete success evidence: tests, logs, screenshots, metrics, DB rows, API responses, or UI states.
- Challenge vague wording, missing baselines, universal claims, and hidden assumptions.
- Do not add dated journal entries.

## Adding a new skill

1. Add `prompts/<name>-task.md` with the prompt body (Codex source).
2. Add `plugins/hopper/skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`) and the inlined prompt body. Description must be specific enough for Claude Code auto-trigger to disambiguate from other plugins.
3. Bump version in `.claude-plugin/marketplace.json` and `plugins/hopper/.claude-plugin/plugin.json`.

## License

MIT.
