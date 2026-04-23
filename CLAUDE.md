# hermes-skills

Personal collection of Hermes Agent skills — CLI tools, personal assistants, cloud integrations, and more. Each skill is a `SKILL.md` in `skills/<category>/<name>/` with YAML frontmatter and an orchestration guide.

## References

Mandatory reading before writing or reviewing a skill:

- `references/skill_conventions.md` — structural patterns, Good/Bad examples, size targets, voice/tone (load first)
- `references/skill_rubric.md` — weighted scoring rubric; run before committing any skill
- `skills/_template/SKILL.md` — annotated template; start every new skill from this

## Structure

```
skills/
└── <category>/
    └── <name>/
        ├── SKILL.md            # required — frontmatter + orchestration guide
        └── references/         # optional — lazy-loaded domain knowledge
            └── <topic>.md
```

Skills that require a CLI binary must declare `prerequisites.commands`. Skills that take actions on behalf of the user (send messages, post content, modify data) must include `When to Use` and `When NOT to Use` sections.

## Required Frontmatter

```yaml
name: <kebab-case>
description: <one-line tool summary — not routing instructions>
version: 1.0.0
author: <Hermes Agent | community>
license: MIT
metadata:
  hermes:
    tags: [Tag1, Tag2]
    related_skills: [other-skill]   # or homepage: https://... for community skills
```

Add when applicable:

```yaml
prerequisites:
  commands: [toolname]      # CLI binaries Hermes checks at load time
  env_vars: [API_KEY]       # required environment variables
platforms: [macos]          # omit if cross-platform
```

## Required Sections

| Section | Rule |
|---------|------|
| Prerequisites | Install, auth, version check, update commands |
| When to Use | Required if skill takes actions on behalf of the user |
| When NOT to Use | Required if skill takes actions on behalf of the user |
| Common Operations | Natural task-name headers; no "Mode N:" for general tasks |
| CLI Reference | Flags tables — every entry verified against `<tool> --help` or source |
| Pitfalls & Gotchas | Confirmed by testing; 8–12 items |
| Rules for Hermes Agents | Behavioral directives; 4–10 numbered items |
| Verification | Smoke test command + explicit success criteria |

## Quality Gate

All must pass before committing. Score with `references/skill_rubric.md` — minimum 80/100.

- [ ] Rubric score ≥ 80/100
- [ ] All 6 required frontmatter fields present
- [ ] `prerequisites.commands` declared if tool requires a CLI binary
- [ ] `When to Use` / `When NOT to Use` present if skill takes user-facing actions
- [ ] Every CLI flag verified against `<tool> --help` or source — no invented flags
- [ ] Every pitfall confirmed by testing — no assumed gotchas
- [ ] Smoke test runs and produces the stated success marker
- [ ] Skill line count within target for its type (see `references/skill_conventions.md`)
- [ ] Skills table row added to `README.md`

## Never Do

- **Never** invent a flag, subcommand, or behavior — verify against help or source first `(honor-system)`
- **Never** embed routing instructions in the description field — write a tool summary
- **Never** use "Mode N:" section naming for general task categorization — reserve it for tools with genuinely distinct execution paths (e.g., print mode vs PTY)
- **Never** include a pitfall that was not confirmed by testing `(honor-system)`
- **Never** write more than 10 Rules — if you have more, consolidate or move to Pitfalls

## Workflow

1. Copy `skills/_template/SKILL.md` → `skills/<category>/<name>/SKILL.md`
2. Read `references/skill_conventions.md` — voice, structure, Good/Bad examples
3. Read an existing built-in Hermes skill for tone reference: `~/.hermes/skills/email/himalaya/SKILL.md` (communication), `~/.hermes/skills/apple/imessage/SKILL.md` (personal assistant), `~/.hermes/skills/autonomous-ai-agents/claude-code/SKILL.md` (complex multi-mode)
4. Write the skill — verify every CLI flag against `<tool> --help`
5. Run the smoke test — confirm it passes with the stated success marker
6. Score with `references/skill_rubric.md` — fix anything below 80/100
7. Add row to `README.md` skills table
8. Commit

## Lazy Loading Pattern

When a skill has substantial domain knowledge that applies only to specific sub-tasks, split it into `references/`. The main SKILL.md stays lean; agents load references when the task requires depth.

**Split into references/ when:**
- Auth setup has multiple paths with significant detail (>20 lines)
- A search or query operator syntax is extensive
- Message composition syntax is complex (e.g., himalaya's MML)
- Advanced patterns used in <20% of interactions

**Keep in SKILL.md:**
- The most common operations (80%+ of use cases)
- The flags table
- All pitfalls and rules

List references at the bottom of SKILL.md:

```markdown
## References

- `references/auth_setup.md` — detailed token extraction and multi-workspace setup
- `references/search_operators.md` — full Slack search operator syntax
```
