# Hopper — Claude Code Instructions

This repo is a Claude Code plugin marketplace + Codex prompt set. Two surfaces, one source of truth pattern.

## Layout

- `.claude-plugin/marketplace.json` registers the `hopper` plugin with Claude Code.
- `plugins/hopper/skills/<name>/SKILL.md` are the skills Claude Code auto-loads. The prompt body is inlined inside each SKILL.md so the skill is self-contained.
- `prompts/<name>-task.md` are the canonical Codex-readable prompt files. Keep them in sync with the inlined version inside each `SKILL.md` (manually for now — automate if drift becomes annoying).
- `templates/daily-brief.md` is the artifact schema referenced by the daily-brief prompts. Also inlined into each daily SKILL.md.

## Editing rules

- When updating a prompt, edit **both** `prompts/<name>-task.md` and `plugins/hopper/skills/<name>/SKILL.md`. They are intentionally duplicated to keep each surface self-contained.
- New skills go under `plugins/hopper/skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`) and the inlined prompt body.
- New Codex-only prompts that have no Claude Code skill counterpart can live under `prompts/` standalone.
- Keep skill `description` fields specific enough to disambiguate from neighboring skills (especially `handoff:*` from the third-party handoff plugin).

## Install

```bash
/plugin marketplace add even-wei/hopper
/plugin install hopper@hopper
```

## Local development

```bash
/plugin marketplace add /path/to/hopper
/plugin install hopper@hopper
```

Re-running the install picks up local edits.

## License

MIT.
