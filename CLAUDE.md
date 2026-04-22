# hermes-skills

Personal collection of Hermes Agent skills (coding agents, developer tools, cloud CLIs, and more). Each skill is a `SKILL.md` in `skills/<name>/` with YAML frontmatter and an orchestration guide.

## Structure

```
skills/
└── <category>/
    └── <name>/
        └── SKILL.md     # required — frontmatter + orchestration guide
```

## Required Frontmatter

All seven fields are required:

```yaml
name: <kebab-case>
description: <one-line routing description>
version: 1.0.0
author: <name>
license: MIT
metadata:
  hermes:
    tags: [Tag1, Tag2]
    related_skills: [other-skill]
```

## Required SKILL.md Sections

| Section | Rule |
|---------|------|
| Prerequisites | Install, auth, version check, and update commands |
| Orchestration | Concrete `terminal()` examples — at least one per supported mode |
| CLI Reference | Flags table; every entry verified against `<tool> --help` or source |
| Pitfalls & Gotchas | Real gotchas confirmed by testing, not assumed |
| Rules for Hermes Agents | Numbered list of behavioral directives |
| Verification | Smoke test command + explicit success criteria |

## Quality Gate

All must pass before committing:

- [ ] All 7 frontmatter fields present
- [ ] Every CLI flag and subcommand appears in `<tool> --help` or tool source
- [ ] Pitfalls section has at least one gotcha confirmed by testing
- [ ] Smoke test runs and outputs the expected success marker
- [ ] Skills table row added to `README.md`

## Never Do

- **Never** invent a flag, subcommand, or behavior — verify against help or source first (honor-system)
- **Never** omit the Pitfalls section — skills without confirmed gotchas mislead agents

## Workflow

1. Read an existing skill (e.g., `autonomous-ai-agents/cursor-agent/SKILL.md`) — voice, structure, and pattern reference
2. Create `skills/<category>/<name>/SKILL.md` with all frontmatter and required sections
3. Run the smoke test; confirm it passes
4. Add row to the Skills table in `README.md`
5. Commit

## Reference

Upstream pattern precedent: `~/.hermes/skills/autonomous-ai-agents/claude-code/SKILL.md` (v2.2.0)
