---
name: cursor-agent
description: Delegate coding tasks to Cursor Agent CLI. Use for building features, refactoring, PR reviews, and iterative coding. Requires the cursor CLI (agent binary) installed.
version: 1.1.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [Coding-Agent, Cursor, Code-Review, Refactoring, PTY, Automation, Cloud]
    related_skills: [claude-code, codex, opencode, hermes-agent]
---

# Cursor Agent — Hermes Orchestration Guide

Delegate coding tasks to [Cursor Agent CLI](https://cursor.com/docs/cli/overview) via the Hermes terminal. The `agent` binary can read files, write code, run shell commands, and manage git workflows — locally or handed off to Cursor's cloud.

## Prerequisites

- **Install:** `curl https://cursor.com/install -fsSL | bash`
- **Auth (API key):** set `CURSOR_API_KEY` env var, or pass `--api-key <key>`
- **Auth (OAuth):** `agent login` (browser flow)
- **Check status:** `agent status` or `agent whoami`
- **Version check:** `agent --version`
- **Update:** `agent update`

## Two Orchestration Modes

Hermes interacts with Cursor in two fundamentally different ways. Choose based on the task.

### Mode 1: Print Mode (`-p`) — Non-Interactive (PREFERRED for most tasks)

Print mode runs a one-shot task, returns the result, and exits. No PTY needed. No interactive prompts. This is the cleanest integration path.

```
terminal(command="agent -p 'Add error handling to all API calls in src/' --output-format json", workdir="/path/to/project", timeout=120)
```

**When to use print mode:**
- One-shot coding tasks (fix a bug, add a feature, refactor)
- CI/CD automation and scripting
- Structured data extraction with `--output-format json`
- Any task where you don't need multi-turn conversation

**Print mode skips ALL interactive dialogs** — no workspace trust prompt, no action confirmations. Pass `--trust` and `--force` explicitly for guaranteed non-interactive execution.

### Mode 2: Interactive PTY via tmux — Multi-Turn Sessions

Interactive mode gives you a full conversational TUI where you can send follow-up prompts and watch the agent work in real time. **Requires tmux orchestration.**

```
# Start a tmux session
terminal(command="tmux new-session -d -s cursor-work -x 140 -y 40")

# Launch Cursor Agent inside it
terminal(command="tmux send-keys -t cursor-work 'cd /path/to/project && agent' Enter")

# Wait for startup (~3-5 seconds), then send your task
terminal(command="sleep 5 && tmux send-keys -t cursor-work 'Refactor the auth module to use JWT tokens' Enter")

# Monitor progress
terminal(command="sleep 15 && tmux capture-pane -t cursor-work -p -S -50")

# Send follow-up
terminal(command="tmux send-keys -t cursor-work 'Now add unit tests for the new JWT code' Enter")

# Exit when done
terminal(command="tmux send-keys -t cursor-work '/exit' Enter")
```

**When to use interactive mode:**
- Multi-turn iterative work (refactor → review → fix → test cycle)
- Tasks requiring human-in-the-loop decisions
- When you need session slash commands (`/plan`, `/worktree`, `/best-of-n`)

## Dialog & Permission Handling

Cursor Agent presents two types of prompts that can block automation. **Print mode and interactive mode handle them differently.**

### Workspace Trust

**`--trust` is print mode only.** It has no effect in interactive mode.

```
# Print mode — --trust skips the workspace trust prompt
terminal(command="agent -p --trust --force 'Refactor the auth module'", workdir="/path/to/project")
```

**Interactive mode:** the trust prompt appears in the TUI on first visit to a directory. Accept it manually — it is cached after the first acceptance and will not appear again for that directory.

### Action Confirmations

`--force` / `--yolo` auto-approves all tool use (file writes, shell commands) and works in **both modes**:

```
# Print mode — pair --trust and --force for fully non-interactive execution
terminal(command="agent -p --trust --force 'Fix the null pointer in auth.py'", workdir="/project")

# Interactive mode — --force only (--trust is a no-op here)
terminal(command="tmux send-keys -t cursor-work 'agent --force \"Fix the null pointer in auth.py\"' Enter")
```

## Operational Modes

| Mode | CLI Flag | Session Command | Behavior |
|------|----------|-----------------|----------|
| **Agent** (default) | _(none)_ | _(none)_ | Full file read/write and shell access |
| **Plan** | `--plan` or `--mode plan` | `/plan` or `Shift+Tab` | Read-only; analyzes and proposes plans, no edits |
| **Ask** | `--mode ask` | `/ask` or `Shift+Tab` | Q&A style; explanations and questions only, no changes |

## Interactive Session Commands

| Command | Purpose |
|---------|---------|
| `/plan` | Switch to Plan mode (read-only design) |
| `/ask` | Switch to Ask mode (Q&A only) |
| `/worktree [name]` | Create isolated git worktree for current task |
| `/best-of-n` | Run task in parallel across models; compare results side-by-side |
| `/exit` | End session |
| `Shift+Tab` | Cycle through Agent → Plan → Ask modes |

## CLI Flags Reference

### Session & Execution
| Flag | Effect |
|------|--------|
| `-p, --print` | Non-interactive one-shot mode (exits when done) |
| `--continue` | Resume most recent session in current directory |
| `--resume [chatId]` | Resume specific session by ID; opens picker if no ID given |
| `--workspace <path>` | Specify project directory |
| `-w, --worktree [name]` | Run in isolated git worktree at `~/.cursor/worktrees/<repo>/<name>`; name auto-generated if omitted |
| `--worktree-base <branch>` | Branch/ref to base the new worktree on (default: current HEAD) |
| `--skip-worktree-setup` | Skip setup scripts from `.cursor/worktrees.json` |
| `-c, --cloud` | Open cloud composer picker on launch (hand off to Cursor cloud) |

### Mode & Model
| Flag | Effect |
|------|--------|
| `--plan` | Start in Plan mode (shorthand for `--mode plan`) |
| `--mode <mode>` | `plan` (read-only design) or `ask` (Q&A only) |
| `--model <name>` | Force specific model (e.g., `composer-2-fast`, `gpt-5.3-codex`) |
| `--list-models` | Print available models and exit |

### Permission & Safety
| Flag | Effect |
|------|--------|
| `--trust` | Skip workspace trust prompt — **print/headless mode only** |
| `--force` / `--yolo` | Auto-approve all actions (file writes, shell, etc.) — both modes |
| `--sandbox <mode>` | `enabled` or `disabled` — override sandbox config |
| `--approve-mcps` | Auto-approve MCP server connections |

### Output & Input
| Flag | Effect |
|------|--------|
| `--output-format <fmt>` | `text` (default), `json`, or `stream-json` |
| `--stream-partial-output` | Stream partial output as text deltas (requires `stream-json` format) |

### Auth & Network
| Flag | Effect |
|------|--------|
| `--api-key <key>` | API key for auth (alternative to `CURSOR_API_KEY` env var) |
| `-H, --header <header>` | Add custom header (`'Name: Value'`, repeatable) |

## Print Mode Deep Dive

### Structured JSON Output
```
terminal(command="agent -p --trust 'Analyze auth.py for security issues' --output-format json", workdir="/project", timeout=120)
```

Returns a JSON object with:
```json
{
  "type": "result",
  "subtype": "success",
  "is_error": false,
  "duration_ms": 10246,
  "result": "The analysis text...",
  "session_id": "568193ac-c8b2-4abc-...",
  "request_id": "32858943-4818-...",
  "usage": { "inputTokens": 2284, "outputTokens": 374, "cacheReadTokens": 46336, "cacheWriteTokens": 0 }
}
```

**Key fields:** `session_id` for resumption with `--resume`, `subtype` for success/error detection, `result` for the text response.

### Streaming Output
For real-time progress on long tasks, use `stream-json` with `--stream-partial-output`:
```
terminal(command="agent -p 'Refactor the data layer' --output-format stream-json --stream-partial-output", workdir="/project", timeout=180)
```

### Piped Input
Run inline (foreground only) — piped stdin is unreliable in background subshells:
```
# Pipe a file for analysis — foreground only
terminal(command="cat src/auth.py | agent -p 'Review this code for bugs'", timeout=60)

# Pipe git diff for review
terminal(command="git diff HEAD~3 | agent -p 'Summarize these changes'", timeout=60)
```

For backgrounded tasks, use `--workspace` and let the agent read files directly rather than piping.

### Session Continuation
```
# Resume most recent session in this directory
terminal(command="agent -p 'Continue and add connection pooling' --continue", workdir="/project", timeout=120)

# Resume a specific session by ID
terminal(command="agent -p 'Try a different approach' --resume '<chat-id>'", workdir="/project", timeout=120)
```

## Cloud Execution

The `-c`/`--cloud` flag opens the cloud composer picker on launch, handing work off to Cursor's cloud infrastructure. Useful for long autonomous runs that don't need local compute.

```
terminal(command="agent --cloud", workdir="/project", pty=true)
```

Once the picker opens, describe your task and Cursor runs it remotely. Track active cloud agents at [cursor.com/agents](https://cursor.com/agents).

**Privacy note:** Cloud execution transmits your code to Cursor's servers. Do not use on private or confidential repositories without explicit authorization.

## Worktree / Best-of-N Patterns

### Isolated Worktree via print mode
For print-mode tasks, use `git worktree add` to create an isolation directory and point `--workspace` at it:
```
terminal(command="git worktree add -b feature-auth /tmp/feature-auth main", workdir="/project")
terminal(command="agent -p --trust --force 'Implement OAuth refresh token flow'", workdir="/tmp/feature-auth", timeout=180)
terminal(command="git worktree remove /tmp/feature-auth", workdir="/project")
```

### Isolated Worktree via interactive mode
Use Cursor's native `-w` flag for an interactive worktree session via tmux:
```
terminal(command="tmux new-session -d -s feature-work -x 140 -y 40")
terminal(command="tmux send-keys -t feature-work 'cd /path/to/project && agent -w feature-auth --worktree-base main --force' Enter")
terminal(command="sleep 5 && tmux send-keys -t feature-work 'Implement OAuth refresh token flow' Enter")
terminal(command="sleep 30 && tmux capture-pane -t feature-work -p -S -50")
```
Creates an isolated git worktree at `~/.cursor/worktrees/<reponame>/feature-auth`.

### Best-of-N (parallel model comparison)
In an interactive session, `/best-of-n` runs the same task across multiple models in isolated worktrees and presents results side-by-side:
```
terminal(command="tmux send-keys -t cursor-work '/best-of-n Implement the caching layer' Enter")
```

### Parallel Worktrees for Batch Work
```
# Create worktrees
terminal(command="git worktree add -b fix/issue-78 /tmp/issue-78 main", workdir="/project")
terminal(command="git worktree add -b fix/issue-99 /tmp/issue-99 main", workdir="/project")

# Run Cursor in each (print mode — no PTY needed)
terminal(command="agent -p --trust --force 'Fix issue #78: null pointer in auth handler. Commit when done.'", workdir="/tmp/issue-78", background=true)
terminal(command="agent -p --trust --force 'Fix issue #99: race condition in cache invalidation. Commit when done.'", workdir="/tmp/issue-99", background=true)

# Monitor
process(action="list")

# After completion, push and create PRs
terminal(command="cd /tmp/issue-78 && git push -u origin fix/issue-78")
terminal(command="gh pr create --head fix/issue-78 --title 'fix: null pointer in auth handler'")

# Cleanup
terminal(command="git worktree remove /tmp/issue-78", workdir="/project")
terminal(command="git worktree remove /tmp/issue-99", workdir="/project")
```

## PR Review Pattern

### Quick Review (Print Mode)
```
terminal(command="git diff main...feature-branch | agent -p 'Review this diff for bugs, security issues, and style problems.'", workdir="/project", timeout=60)
```

### Deep Review (Interactive + Worktree)
```
terminal(command="tmux new-session -d -s review -x 140 -y 40")
terminal(command="tmux send-keys -t review 'cd /path/to/repo && agent -w pr-review --force' Enter")
terminal(command="sleep 5 && tmux send-keys -t review 'Review all changes vs main. Check for bugs, security issues, race conditions, and missing tests.' Enter")
terminal(command="sleep 30 && tmux capture-pane -t review -p -S -60")
```

### PR Review from Branch
```
terminal(command="gh pr checkout 42 && agent -p --trust 'Review this PR thoroughly for bugs, security, and test coverage.' --output-format json", workdir="/tmp/pr-review-clone", timeout=120)
```

## Parallel Cursor Instances

Run multiple independent Cursor tasks simultaneously:

```
# Task 1: Fix backend bug
terminal(command="tmux new-session -d -s task1 -x 140 -y 40 && tmux send-keys -t task1 'cd ~/project && agent -p --trust --force \"Fix the auth bug in src/auth.py\"' Enter")

# Task 2: Write tests
terminal(command="tmux new-session -d -s task2 -x 140 -y 40 && tmux send-keys -t task2 'cd ~/project && agent -p --trust --force \"Write integration tests for the API endpoints\"' Enter")

# Task 3: Update docs
terminal(command="tmux new-session -d -s task3 -x 140 -y 40 && tmux send-keys -t task3 'cd ~/project && agent -p --trust \"Update README with the new API endpoints\"' Enter")

# Monitor all
terminal(command="sleep 30 && for s in task1 task2 task3; do echo '=== '$s' ==='; tmux capture-pane -t $s -p -S -5 2>/dev/null; done")

# Cleanup
terminal(command="tmux kill-session -t task1 && tmux kill-session -t task2 && tmux kill-session -t task3")
```

## Session Management

```
agent ls                        # Open session picker to browse and resume sessions
agent resume                    # Resume the latest session (interactive)
agent --continue                # Resume latest (also works with -p)
agent --resume <chat-id>        # Resume specific session by ID
agent create-chat               # Start a new empty chat
```

Sessions are saved to `~/.cursor/` and persist across reboots.

## MCP Integration

### MCP Server Management
| Subcommand | Effect |
|------------|--------|
| `agent mcp list` | List configured MCP servers and their status |
| `agent mcp enable <id>` | Add an MCP server to the local approved list |
| `agent mcp disable <id>` | Disable an MCP server (will not be loaded) |
| `agent mcp login <id>` | Authenticate with an MCP server |
| `agent mcp list-tools <id>` | List available tools for a specific MCP server |

### Using MCP in Tasks
```
# Auto-approve all MCP connections (skips confirmation prompts)
terminal(command="agent --approve-mcps -p 'Query the database and summarize schema'", workdir="/project", timeout=60)

# Custom headers (e.g., for authenticated MCP endpoints)
terminal(command="agent -H 'X-Team-Token: abc123' -p 'Use the team MCP server to fetch issue list'", workdir="/project", timeout=60)
```

## Cost & Performance Tips

1. **Prefer `-p` for single tasks** — exits cleanly, no session overhead.
2. **Use `--model composer-2-fast`** for simple tasks (default, fastest). Use `--model gpt-5.3-codex` for complex multi-step work.
3. **Use `--plan` before editing** — design the approach first in read-only mode, then re-run in agent mode to apply. Saves tokens when direction needs clarification.
4. **Avoid piping to backgrounded processes** — use `--workspace` and let the agent read files directly instead.
5. **Use worktrees for parallel work** — prevents git conflicts across concurrent tasks.
6. **Use `--sandbox enabled`** for untrusted tasks; disable only when shell access is required.
7. **Clean up tmux sessions** after interactive tasks — `tmux kill-session -t <name>`.
8. **Use `--output-format stream-json`** for long tasks where you need progress feedback.

## Verification

Smoke test to confirm the cursor CLI is installed, authenticated, and functional:

```
terminal(command="agent -p --trust 'Respond with exactly: CURSOR_SMOKE_OK'")
```

Success criteria:
- Output contains `CURSOR_SMOKE_OK`
- Command exits without authentication or provider errors
- For code tasks: expected files changed and tests pass

**Note:** `--trust` is required — without it, the agent blocks on the workspace trust prompt even in print mode on first visit to a directory.

## Pitfalls & Gotchas

1. **`--trust` only works in print mode (`-p`)** — it has no effect in interactive mode. Accept the workspace trust prompt manually in the TUI on first visit (one-time per directory, then cached).
2. **PTY required for interactive `agent` TUI** — use tmux; raw PTY alone causes `\r`/`\n` issues with the TUI.
3. **`-p` print mode skips ALL dialogs** — preferred for automation; use it whenever you don't need multi-turn.
4. **Cloud (`--cloud`) transmits your code** — do not use on private or confidential repositories without authorization.
5. **`/best-of-n` is session-only** — cannot be invoked from print mode.
6. **`agent --continue` scopes to current directory** — run from the right workdir.
7. **`--sandbox enabled` may block shell commands** — check what your task needs before enabling.
8. **Background tmux sessions persist** — clean up with `tmux kill-session -t <name>` when done.
9. **The binary is `agent`, not `cursor`** — `cursor` opens the IDE; `agent` is the CLI coding agent.
10. **Piped stdin may be empty when backgrounded** — `cat file | agent -p ...` can lose stdin in background subshells. Use `--workspace` and let the agent read the file directly, or run the pipe inline (foreground).

## Rules for Hermes Agents

1. **Prefer print mode (`-p`) for single tasks** — cleaner, no dialog handling, structured output.
2. **In print mode, always pass `--trust --force`** — skips workspace trust and action confirmations together.
3. **In interactive mode, use `--force` only** — `--trust` has no effect outside print mode.
4. **Set `--workspace` explicitly** — keeps the agent focused on the right project.
5. **Use tmux for multi-turn interactive work** — the only reliable way to orchestrate the TUI.
6. **Use `--cloud` for long autonomous tasks** — opens the cloud composer picker to offload work from local machine.
7. **Monitor tmux sessions** with `tmux capture-pane -t <session> -p -S -50`.
8. **Clean up tmux sessions** when done — `tmux kill-session -t <name>`.
9. **Do not use cloud on sensitive code** without explicit user authorization.
10. **Report results to user** — after completion, summarize files changed and next steps.
