# hermes-skills

A personal collection of [Hermes Agent](https://github.com/NousResearch/hermes-agent) skills.

## Skills

| Skill | Description |
|-------|-------------|
| [cursor-agent](skills/autonomous-ai-agents/cursor-agent/SKILL.md) | Delegate coding tasks to Cursor Agent CLI — build features, refactor, review PRs, run parallel agents |

## Install

```bash
hermes skills install charlesgwilson/hermes-skills/autonomous-ai-agents/cursor-agent --category autonomous-ai-agents --force --yes
```

> **Note:** `--force` is required. Hermes flags the `curl | bash` pattern in the Prerequisites section as a supply-chain risk. This is the official Cursor install method from cursor.com — review the skill before overriding if needed.

## Skills in this collection

### cursor-agent

Orchestrates the [Cursor Agent CLI](https://cursor.com/docs/cli/overview) (`agent` binary) from within Hermes. Supports one-shot print mode for automation and interactive PTY sessions via tmux for multi-turn work. Covers worktrees, parallel instances, PR review patterns, MCP integration, and cloud handoff.

**Requires:** `agent` binary installed (`curl https://cursor.com/install -fsSL | bash`) and authenticated (`agent login` or `CURSOR_API_KEY`).

## Adding a skill

Each skill lives in `skills/<category>/<name>/SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: One-line tool summary for Hermes skill discovery.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [Tag1, Tag2]
    related_skills: [other-skill]
prerequisites:
  commands: [toolname]
---
```

See `CLAUDE.md` for the full workflow, quality gate, and required sections. Score every skill with `references/skill_rubric.md` before committing.

## License

MIT
