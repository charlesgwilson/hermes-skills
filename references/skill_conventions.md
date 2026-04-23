# Hermes Skill Conventions

Patterns derived from built-in Hermes skills: himalaya, apple-reminders, imessage, notion, linear, github-pr-workflow, and the claude-code reference (v2.2.0).

---

## Frontmatter

### Required fields

```yaml
name: kebab-case-name
description: <one-line tool summary>
version: 1.0.0
author: Hermes Agent        # or "community" for third-party skills
license: MIT
metadata:
  hermes:
    tags: [Tag1, Tag2]
    related_skills: [other-skill]   # or homepage: https://... for community skills
```

### Optional but important

```yaml
prerequisites:
  commands: [slackcli, gh]          # Hermes checks these at load time
  env_vars: [LINEAR_API_KEY]        # required env vars
platforms: [macos]                  # omit if cross-platform
```

Add `prerequisites.commands` whenever the skill depends on a CLI binary. Hermes uses this to verify dependencies before loading. Himalaya and apple-reminders both declare it; omitting it means the agent discovers the missing tool mid-task.

### Description field

The description is a **tool summary**, not routing instructions. Write what the tool does and what you use it for. Hermes routes by matching description + tags to the user's request — do not embed "Load this skill immediately when..." in the description.

| | Example |
|---|---|
| **Good** | `CLI to manage emails via IMAP/SMTP. Use himalaya to list, read, write, reply, forward, search, and organize emails from the terminal.` |
| **Bad** | `Use this skill for ALL email tasks. Load this skill immediately when the user mentions email or inbox.` |

---

## Section Structure

Sections appear in this order. Not every skill needs every section — use judgment based on tool complexity.

```
# <Tool Name> — Hermes Orchestration Guide   ← H1, always present
<one-sentence description of what the skill does>

## Prerequisites           ← always
## When to Use             ← required for action-taking skills
## When NOT to Use         ← required for action-taking skills
## <Core Operations>       ← named for the tool (see below)
## CLI Reference           ← flags tables
## Pitfalls & Gotchas      ← always
## Rules for Hermes Agents ← always
## Verification            ← always
## References              ← only if reference files exist
```

### "When to Use" / "When NOT to Use"

Required for any skill that **takes actions on behalf of the user**: sends messages, posts content, modifies data, or otherwise acts in the user's name. iMessage and apple-reminders both have these sections; himalaya does not (read-heavy email skill).

"When NOT to Use" is where you establish safety boundaries — what Hermes should refuse or escalate even if the user asks.

| | Example |
|---|---|
| **When to Use** | `Sending or drafting Slack messages as the user` |
| **When NOT to Use** | `Sending messages without explicit user approval — always confirm first` |

---

## Core Operations Naming

Name the operations section after the tool's primary domain — not a generic header. Sub-sections use **natural task names**.

| | Example |
|---|---|
| **Good** | `## Common Operations` with sub-sections: `### Monitor Unread Messages`, `### Send or Reply` |
| **Good** | `## Common Queries` / `## Common Mutations` (Linear pattern for API-heavy tools) |
| **Bad** | `## Mode 1: Monitor`, `## Mode 2: Read`, `## Mode 3: Send` |

**"Mode N:" naming is reserved** for tools with genuinely distinct execution paths — specifically where the orchestration pattern, tooling, or permissions differ fundamentally between modes. The claude-code skill uses "Mode 1: Print Mode" and "Mode 2: Interactive PTY" because these paths require different tooling (PTY + tmux vs. direct invocation). Do not use "Mode N:" as a task categorizer.

---

## Code Examples

All examples must be:
- **Runnable** — exact flags, real syntax, no pseudo-code
- **`terminal()` style** for Hermes terminal calls, not bare shell blocks
- **Verified** against `<tool> --help` or source before writing

```
# Good — real, runnable, terminal() style
terminal(command="slackcli conversations unread --json")

# Bad — pseudo-code, made-up flags
terminal(command="slackcli watch --channel <id> --notify")
```

For multi-step patterns, show each step as a separate `terminal()` call with a comment explaining the flow.

---

## JSON Shapes

Include an inline JSON shape **only when field names are non-obvious** and an agent would plausibly extract the wrong field. Do not document the output shape of every command.

| Justify inline JSON | Skip inline JSON |
|---------------------|-----------------|
| Self-DM detection: `channel.user == channel.name` is the marker, not obvious | `conversations list` returns a list of channels — shape is guessable |
| `conversations get` returns `users[]` with `real_name` — agent would not guess this | `search people` returns `people[0].id` — obvious from field name |

---

## Pitfalls & Gotchas

Every pitfall must be **confirmed by testing**, not assumed. The phrase "may fail" or "could error" is only acceptable if you tested and observed the intermittent failure.

| | Example |
|---|---|
| **Good** | `` `conversations list --types im --json` errors with "unknown option '--json'" — confirmed: `--json` is not supported on `list`. `` |
| **Bad** | `Using --json with --types might cause issues in some versions.` |

Target: 8–12 pitfalls. More than 12 suggests the skill needs reference files for depth.

---

## Rules for Hermes Agents

Rules are **behavioral directives** — they change how the agent acts, not what it knows. Each rule should pass this test: *"If I remove this rule, will the agent behave differently in a way that matters?"*

Target: 4–10 rules. More than 10 indicates overlap with Pitfalls or duplication of what examples already show.

| | Example |
|---|---|
| **Good** | `Never send a message without explicit user confirmation — present recipient, text, and thread context first.` |
| **Good** | `Self-identify with \`search messages "from:me" --limit 1 --json\` before any task requiring user identity. Never guess from OS context.` |
| **Bad** | `Always use correct syntax when calling commands.` (restated default — remove) |
| **Bad** | `Check that the tool is installed before using it.` (already in Verification — remove) |

---

## Verification

Every skill needs a smoke test: a single command that confirms the tool is installed, authenticated, and functional. Include explicit success criteria.

```
## Verification

\`\`\`bash
slackcli search messages "from:me" --limit 1 --json
\`\`\`

Success: exit code 0, JSON output, `matches[0].user` populated.
If exit 137: see Pitfall #1.
If auth error: re-run setup.
```

---

## Reference Files

When a skill has substantial domain knowledge that only applies to specific sub-tasks, split it into `references/`. The main SKILL.md stays lean; agents load references on demand.

**Good candidates for references:**
- Detailed auth setup with multiple paths and edge cases
- Complete search operator syntax (e.g., Slack operators, GraphQL filter syntax)
- Message composition syntax (himalaya's MML reference)
- Advanced patterns used only occasionally

**Keep in the main SKILL.md:**
- The most common operations (80% of use cases)
- The flags table (quick lookup)
- Rules and pitfalls

List references at the bottom of SKILL.md with a short description of when to load each:

```markdown
## References

- `references/auth_setup.md` — detailed browser token extraction and multi-workspace setup
- `references/search_operators.md` — full Slack search operator syntax
```

---

## Size Targets

Derived from built-in skill line counts, not from Claude Code agent targets.

| Skill type | Target | Ceiling |
|------------|--------|---------|
| Single-purpose / read-only tool | ~100 lines | 150 lines |
| Standard personal assistant skill | ~150–250 lines | 300 lines |
| Complex multi-mode orchestration tool | ~300–400 lines | 500 lines |
| Reference file | ~100–200 lines | 250 lines |

Skills over their ceiling should extract content to `references/`. The claude-code Hermes skill (745 lines) is the exception — it documents an extremely complex tool with two fundamentally different orchestration modes.

---

## Voice and Tone

Imperative, task-focused, terse. Match the voice of himalaya, iMessage, and apple-reminders.

| | Example |
|---|---|
| **Good** | `Read recent messages from a channel or DM (requires channel ID, not user ID)` |
| **Bad** | `In order to read messages, you will need to first obtain the channel ID, because slackcli does not accept user IDs for this command.` |

No narrative prose. No preamble. Section headers name the task, not the concept. Examples show the action, not the background.
