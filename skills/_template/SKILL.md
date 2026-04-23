---
# REQUIRED — all six fields must be present
name: kebab-case-name
description: CLI to <action> via <tool>. Use for <task-1>, <task-2>, and <task-3>. # tool summary only — no routing instructions
version: 1.0.0
author: Hermes Agent        # or "community" for third-party skills
license: MIT
metadata:
  hermes:
    tags: [Tag1, Tag2]      # PascalCase; at least 2
    related_skills: []      # or homepage: https://... for community skills

# OPTIONAL — add when applicable
# prerequisites:
#   commands: [toolname]    # CLI binaries required; Hermes checks at load time
#   env_vars: [API_KEY]     # required environment variables
# platforms: [macos]        # omit if cross-platform
---

# <Tool Name> — Hermes Orchestration Guide

<One sentence: what this skill does and what tool it uses.>

## Prerequisites

- **Install:** `<install command>`
- **Version check:** `<tool> --version`
- **Auth check:** `<auth check command>` *(omit if no auth required)*
- **Update:** `<update command>`

### Setup

<Auth setup steps. Use numbered list for multi-step flows. Show exact commands.>

```bash
<exact setup command>
```

## When to Use

*(Required for skills that take actions on behalf of the user. Remove for read-only tools.)*

- <task the agent should perform with this skill>
- <another task>

## When NOT to Use

*(Required for skills that take actions on behalf of the user. Remove for read-only tools.)*

- <boundary — what the agent should refuse or escalate>
- <safety rule — e.g., "Sending messages without explicit user approval">

## Common Operations

*(Rename this section to match the tool's domain: "Common Queries", "Quick Reference", etc.)*

### <Task Name>

<One sentence of context if non-obvious. Otherwise go straight to the example.>

```
terminal(command="<exact command> --flag value")

# With option
terminal(command="<exact command> --other-flag")
```

### <Another Task>

```
terminal(command="<exact command>")
```

### <Send / Post / Modify — action-taking tasks>

*(Always require explicit user confirmation before executing. Show the approval step in the Rules section.)*

```
terminal(command="<exact send/post/modify command>")
```

## CLI Reference

### <command-group>

| Command | Key Flags | Notes |
|---------|-----------|-------|
| `<subcommand>` | `--flag`, `--other` | Brief note on behavior or gotcha |
| `<subcommand>` | `--flag` | |

*(Only document flags verified against `<tool> --help` or source. Never invent.)*

## Pitfalls & Gotchas

*(Each item must be confirmed by testing — not assumed. Use "Confirmed:" for clarity on tested items.)*

1. **<Specific failure mode>** — <what breaks and what to do instead>.
2. **<Another confirmed gotcha>** — <exact symptom and fix>.

*(Target: 8–12 items. More than 12 → extract to a reference file.)*

## Rules for Hermes Agents

*(Behavioral directives only — things that change how the agent acts. Remove rules that restate what examples already show.)*
*(Target: 4–10 rules.)*

1. **<Action directive>** — <one sentence. What to do or not do and why it matters.>
2. **<Safety directive>** — <e.g., always confirm before sending; never guess user identity.>
3. **<Tool-specific constraint>** — <e.g., which flag is unsupported, which auth path is required.>

## Verification

```bash
<smoke test command>
```

Success: <exact success criteria — what output or exit code proves it worked>.
If <failure mode>: <fix>.

## References

*(Only include this section if reference files exist in references/.)*

- `references/<file>.md` — <what it covers and when to load it>
