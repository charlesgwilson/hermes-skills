---
name: slack-cli
description: CLI to manage Slack via slackcli. Use for monitoring unread messages, reading channels and DMs, sending and drafting messages as the user, adding emoji reactions, and setting follow-up reminders. Requires browser session tokens for full feature access.
version: 1.3.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [Slack, Communication, Messaging, Monitoring, Notifications, CLI, Personal-Assistant]
    related_skills: [apple-reminders, himalaya]
prerequisites:
  commands: [slackcli]
---

# Slack CLI — Hermes Orchestration Guide

Personal Slack assistant powered by [slackcli](https://github.com/shaharia-lab/slackcli). Monitor unread messages, read channel and DM history, send or draft messages as the user, react to messages on request, and create follow-up reminders.

## Prerequisites

- **Install:** `brew install shaharia-lab/slackcli/slackcli`
- **Version check:** `slackcli --version`
- **Update:** `slackcli update`
- **Auth check:** `slackcli auth list`
- **Reminders (optional):** `which remindctl` — required for reminder workflow. If missing, inform the user and skip; do not install it.

### Recommended Auth: Browser Session Tokens

Browser tokens are the recommended path — they unlock `messages draft` and are the intended setup for this skill.

1. Open Slack in your browser → DevTools (F12) → Network tab
2. Trigger any Slack API request (send a message, switch channels)
3. Right-click any `api.slack.com` request → **Copy as cURL**
4. In terminal:

```bash
slackcli auth parse-curl --from-clipboard --login
```

Verify: `slackcli auth list` — workspace should appear with `(default)` marker.

**Token refresh:** Browser tokens expire with your Slack session. Re-run `auth parse-curl` if auth errors appear.

### Alternative: Standard App Token

```bash
slackcli auth login --token xoxb-your-token-here --workspace-name myteam
```

Works for all commands except `messages draft`.

## When to Use

- Reading Slack messages, channels, DMs, or threads
- Sending or drafting messages as the user
- Adding emoji reactions to messages (when explicitly requested)
- Setting follow-up reminders tied to specific Slack messages
- Searching the workspace for messages, people, or channels
- Monitoring unread messages across the workspace

## When NOT to Use

- Creating or archiving channels, managing workspace settings, inviting members — not supported by slackcli
- Sending messages without explicit user approval — always confirm first
- High-frequency polling — slackcli triggers parallel API fetches that hit rate limits quickly
- Real-time push notifications — this is a pull-based tool; it checks on demand, not continuously

## Self-Identification

`auth list` shows only the workspace ID — it does not expose the current user's identity. Self-identify before any task that requires knowing the user's Slack username or DM channel ID:

```
# Get current user ID and username — works for any account, no name required
terminal(command="slackcli search messages 'from:me' --limit 1 --json")
```

From the result: `matches[0].user` is the user ID, `matches[0].username` is the Slack username.

To find a DM channel ID without sending anything:

```
# Replace <username> with matches[0].username from above
terminal(command="slackcli search messages 'from:me in:@<username>' --limit 1 --json")
```

`matches[0].channel.id` is the DM channel ID (D-prefix). Works for self-DMs and DMs with others.

## Common Operations

### Monitor Unread Messages

`conversations unread` aggregates across all channels, DMs, and groups — use it as the monitoring entry point.

```
terminal(command="slackcli conversations unread --json")
```

Parse the `unread_channels` array. Prioritize: `is_im: true` (DMs) first, then `mention_count > 0` (@mentions), then general channel unreads. For each item of interest, fetch context:

```
terminal(command="slackcli conversations get <channel-id> <timestamp> --json")
terminal(command="slackcli conversations read <channel-id> --thread-ts <timestamp> --limit 20 --json")
```

Do not call `conversations unread` more than once per minute — it triggers parallel metadata fetches that can hit rate limits.

### Read Channel or DM History

`conversations list` outputs text only — it does not support `--json`. Use it for channel discovery; use `conversations read` for parseable output.

```
# Discover channels (text output only — no --json)
terminal(command="slackcli conversations list")
terminal(command="slackcli conversations list --types im")

# Read recent messages (requires channel ID, not user ID)
terminal(command="slackcli conversations read <channel-id> --limit 30 --json")
terminal(command="slackcli conversations read <channel-id> --thread-ts <timestamp> --json")
terminal(command="slackcli conversations read <channel-id> --oldest 1745400000 --latest 1745486400 --json")
terminal(command="slackcli conversations read <channel-id> --exclude-replies --limit 50 --json")

# Single message lookup (result includes users[] with real_name and email)
terminal(command="slackcli conversations get <channel-id> <timestamp> --json")
```

### Send or Reply

Always present the draft to the user and wait for explicit confirmation before calling `messages send`.

```
# Send to a channel
terminal(command="slackcli messages send --recipient-id C1234567890 --message 'Message text'")

# Send a DM — user ID (U-prefix) auto-converts to DM channel
terminal(command="slackcli messages send --recipient-id U1234567890 --message 'Message text'")

# Reply in a thread
terminal(command="slackcli messages send --recipient-id C1234567890 --thread-ts 1745400123.000100 --message 'Reply text'")
```

### Draft a Message

Drafts appear in Slack's compose box for the user to review and edit before sending. Use when the user wants final control. **Requires browser session tokens.**

```
terminal(command="slackcli messages draft --recipient-id C1234567890 --message 'Draft text'")
terminal(command="slackcli messages draft --recipient-id U1234567890 --message 'Draft DM text'")
terminal(command="slackcli messages draft --recipient-id C1234567890 --thread-ts 1745400123.000100 --message 'Draft reply'")
```

### Add an Emoji Reaction

Only add reactions when the user explicitly requests one. The single autonomous exception: after creating a user-requested reminder, add `i-have-set-a-reminder-for-this-and-will-get-back-to-you` to the source message.

`messages react` requires a channel ID (C- or D-prefix) — user IDs will fail. Use the Self-Identification pattern to get a DM channel ID first.

```
# Emoji name without colons
terminal(command="slackcli messages react --channel-id <channel-id> --timestamp <ts> --emoji thumbsup")
terminal(command="slackcli messages react --channel-id <channel-id> --timestamp <ts> --emoji white_check_mark")
terminal(command="slackcli messages react --channel-id <channel-id> --timestamp <ts> --emoji i-have-set-a-reminder-for-this-and-will-get-back-to-you")
```

### Set a Follow-Up Reminder

slackcli has no native reminder command. Slack slash commands (e.g. `/remind`) sent via `messages send` appear as literal text — they are not executed. Use `remindctl` (apple-reminders skill) for real reminders.

Check first:

```
terminal(command="which remindctl")
```

If not found, tell the user reminders are unavailable and stop. Do not attempt to install it.

```
terminal(command="remindctl add 'Follow up with @<username> in #<channel> — <topic> (slack: <channel-id>/<ts>)' --due '<ISO-datetime>'")
```

After creating the reminder, add `i-have-set-a-reminder-for-this-and-will-get-back-to-you` to the source message.

### Search Messages, People, and Channels

```
# Message search — supports Slack search operators
terminal(command="slackcli search messages 'deployment rollback' --json")
terminal(command="slackcli search messages 'deployment rollback' --in engineering --from alice --json")
terminal(command="slackcli search messages 'from:me' --limit 1 --json")
terminal(command="slackcli search messages 'from:me in:@<username>' --limit 1 --json")

# People and channel search
terminal(command="slackcli search people '<name or email>' --json")
terminal(command="slackcli search channels 'incident' --json")
```

### Saved Items

```
terminal(command="slackcli saved list --json")
terminal(command="slackcli saved list --state to_do --json")
terminal(command="slackcli saved list --state completed --json")
```

## CLI Reference

### auth

| Command | Key Flags | Notes |
|---------|-----------|-------|
| `auth login` | `--token`, `--workspace-name` | xoxb-* or xoxp-* |
| `auth login-browser` | `--xoxd`, `--xoxc`, `--workspace-url` | Manual browser token entry |
| `auth parse-curl` | `--login`, `--from-clipboard` | Recommended setup path |
| `auth list` | — | Shows workspace name/ID only — does not expose user ID |
| `auth set-default <id>` | — | Changes default workspace |
| `auth remove <id>` | — | Removes workspace credentials |
| `auth logout` | — | Clears all workspaces |

### conversations

| Command | Key Flags | Notes |
|---------|-----------|-------|
| `conversations list` | `--types`, `--limit`, `--exclude-archived`, `--cursor`, `--workspace` | No `--json` support — text only |
| `conversations read <channel-id>` | `--thread-ts`, `--exclude-replies`, `--limit`, `--oldest`, `--latest`, `--workspace`, `--json` | Channel ID required; `--oldest`/`--latest` are Unix timestamps |
| `conversations get <channel-id> <ts>` | `--workspace`, `--json` | Returns `users[]` with real_name and email |
| `conversations unread` | `--types`, `--workspace`, `--json` | Core monitoring primitive |

### messages

| Command | Key Flags | Notes |
|---------|-----------|-------|
| `messages send` | `--recipient-id`, `--message`, `--thread-ts`, `--workspace` | U-prefix user ID auto-converts to DM channel |
| `messages react` | `--channel-id`, `--timestamp`, `--emoji`, `--workspace` | Requires channel ID — user ID fails; emoji without colons |
| `messages draft` | `--recipient-id`, `--message`, `--thread-ts`, `--workspace` | Browser tokens only |

### search

| Command | Key Flags | Notes |
|---------|-----------|-------|
| `search messages <query>` | `--in`, `--from`, `--limit`, `--page`, `--sort`, `--sort-dir`, `--workspace`, `--json` | Supports Slack operators: `from:me`, `in:@<username>`, `has:reaction` |
| `search people <query>` | `--limit`, `--workspace`, `--json` | Name, username, or email |
| `search channels <query>` | `--limit`, `--workspace`, `--json` | |

### saved

| Command | Key Flags | Notes |
|---------|-----------|-------|
| `saved list` | `--limit`, `--state`, `--workspace`, `--json` | States: saved, to_do, completed |

## Pitfalls & Gotchas

1. **Apple Silicon crash (issue #43, unresolved as of v0.6.0)** — binary exits code 137 on M-series Macs. Fix: install Rosetta 2 (`softwareupdate --install-rosetta`) or build from source (`bun install && bun run build`).
2. **`messages draft` fails silently with bot tokens** — returns error with xoxb-*/xoxp-* tokens. Switch to browser auth via `auth parse-curl`.
3. **Browser tokens expire** — symptoms look like auth failure despite `auth list` showing a workspace. Re-run `auth parse-curl --from-clipboard --login`.
4. **`conversations list` has no `--json` support** — the flag errors. Use `search messages` with Slack operators for machine-readable channel discovery.
5. **`messages react` does not accept user IDs** — requires a real channel ID (C- or D-prefix). Use `search messages "from:me in:@<username>"` to get the DM channel ID without sending anything.
6. **Slack slash commands are not executed via API** — `/remind` sent through `messages send` appears as literal text. Use `remindctl` for real reminders.
7. **Rate limiting on `conversations unread`** — parallel internal fetches can hit Slack limits. Call at most once per minute.
8. **`--oldest`/`--latest` are Unix timestamps** — not ISO dates. Convert before passing.
9. **Multi-workspace confusion** — all commands use the default workspace unless `--workspace` is passed. Check `auth list` if results look wrong.
10. **Corrupted config is silent** — corrupted `~/.config/slackcli/workspaces.json` returns an empty workspace list with no error, identical to "not authenticated."

## Rules for Hermes Agents

1. **Self-identify with `search messages "from:me" --limit 1 --json`** before any task requiring user identity or DM channel IDs. Never guess from OS username or system context.
2. **Never use `--json` with `conversations list`** — it is unsupported and will error.
3. **Always use `--json` for all other read operations** — text output is not reliably parseable.
4. **Never send a message without explicit user confirmation** — present recipient, text, and thread context; wait for yes before calling `messages send`.
5. **Prefer `messages draft` over `messages send`** when the user wants to review before committing.
6. **Never react autonomously** — all emoji reactions require explicit user instruction, except `i-have-set-a-reminder-for-this-and-will-get-back-to-you` after a user-requested reminder.
7. **Check `which remindctl` before any reminder request** — if missing, tell the user and stop; do not install it.
8. **On Apple Silicon, run `slackcli --version` first** — exit 137 means the binary is crashing; halt and report with fix instructions.

## Verification

```bash
slackcli search messages "from:me" --limit 1 --json
```

Success: exit code 0, JSON output, `matches[0].user` and `matches[0].username` populated.

If exit 137: see Pitfall #1 (Apple Silicon).
If auth error: re-run `slackcli auth parse-curl --from-clipboard --login`.
