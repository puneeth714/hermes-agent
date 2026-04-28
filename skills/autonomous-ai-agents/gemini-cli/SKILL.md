---
name: gemini-cli
description: "Delegate coding to Gemini CLI (features, PRs, research)."
version: 1.0.0
author: Gemini CLI + Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [Coding-Agent, Gemini, Google, Code-Review, Refactoring, Automation]
    related_skills: [claude-code, codex, hermes-agent]
---

# Gemini CLI — Hermes Orchestration Guide

Delegate coding and research tasks to [Gemini CLI](https://github.com/google/gemini-cli) via the Hermes terminal. Gemini CLI is an autonomous agent capable of reading/writing files, running shell commands, and managing complex software engineering workflows.

## Prerequisites

- **Install:** `npm install -g @google/gemini-cli`
- **Auth:** Run `gemini` once to log in via browser OAuth.
- **Check status:** `gemini --version`

## Orchestration Modes

Hermes interacts with Gemini CLI in two primary ways: Non-Interactive (Headless) and Interactive (via tmux).

### Mode 1: Non-Interactive (`-p`) — Headless (PREFERRED)

Headless mode runs a specific task and exits. This is ideal for one-shot automation where you want the result returned directly to Hermes.

```
terminal(command="gemini -p 'Refactor the auth module to use async/await' -y --skip-trust", workdir="/path/to/project", timeout=300)
```

**Key Flags for Headless Mode:**
- `-p, --prompt "task"`: The task to perform.
- `-y, --yolo`: Auto-accept all actions (critical for non-interactive runs).
- `--skip-trust`: Bypasses the workspace trust prompt.
- `-o json`: Returns structured JSON (useful for parsing results).

### Mode 2: Interactive Session via tmux

For complex, multi-turn tasks where you want to watch the progress or provide feedback, use a tmux session.

```
# Start a tmux session
terminal(command="tmux new-session -d -s gemini-session -x 140 -y 40")

# Launch Gemini
terminal(command="tmux send-keys -t gemini-session 'cd /path/to/project && gemini --skip-trust' Enter")

# Send the initial task
terminal(command="sleep 3 && tmux send-keys -t gemini-session 'Fix the memory leak in the data processor' Enter")

# Capture output to monitor progress
terminal(command="sleep 30 && tmux capture-pane -t gemini-session -p -S -100")
```

## Advanced Features

### Git Worktrees
Gemini can work in an isolated git worktree to avoid messing up your current branch.
```
terminal(command="gemini -p 'Fix issue #123' -w fix-123 -y", workdir="/path/to/project")
```

### Resuming Sessions
If a task was interrupted, you can resume it.
```
terminal(command="gemini -r latest -p 'Continue with the tests' -y", workdir="/path/to/project")
```

### Output Formats
For programmatic integration, use JSON output.
```
terminal(command="gemini -p 'List all exported functions' -o json", workdir="/path/to/project")
```

## CLI Flags Reference

| Flag | Description |
|------|-------------|
| `-p, --prompt` | Non-interactive mode with the given prompt. |
| `-y, --yolo` | Automatically accept all actions (no confirmation prompts). |
| `--skip-trust` | Trust the current workspace for this session. |
| `-w, --worktree` | Start Gemini in a new git worktree. |
| `-o, --output-format` | Output format: `text`, `json`, `stream-json`. |
| `-r, --resume` | Resume a previous session (`latest` or index). |
| `-m, --model` | Specify a specific model to use. |
| `--approval-mode` | `default`, `auto_edit`, `yolo`, `plan`. |

## Best Practices for Hermes Agents

1. **Always use `-y` and `--skip-trust`** when running in headless mode to prevent the agent from hanging on prompts.
2. **Set a generous timeout.** Autonomous tasks can take several minutes to complete.
3. **Use JSON output** if you need to programmatically verify the success of the task.
4. **Prefer Headless Mode** for well-defined tasks (bug fixes, refactoring) and **Interactive Mode** for exploratory work.
5. **Cleanup:** Always kill tmux sessions after interactive use to free up system resources.
