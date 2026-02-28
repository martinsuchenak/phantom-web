---
title: "Commands Reference"
description: "Complete reference for all Phantom CLI commands, flags, and options."
weight: 1
---

This page provides a comprehensive reference for all Phantom commands.

## Overview

| Command | Description |
| --- | --- |
| `start` / `stop` / `restart` | Create, unmount, remount overlays |
| `list` / `status` / `inspect` | View overlay state |
| `run` / `run-all` / `run-chain` | Run agents (single, parallel, sequential) |
| `diff` / `compare` / `conflicts` | View and compare changes |
| `commit` / `apply` / `merge` / `sync` | Commit, apply to base, merge, sync with base |
| `watch` / `logs` / `replay` | Monitor and re-run agents |
| `hook` | Post-run automation (lint, test, notify) |
| `snapshot` / `export` / `clone` | Save, export, duplicate overlays |
| `lock` / `unlock` / `pin` / `unpin` | Protect overlays from cleanup and base drift |
| `prune` / `gc` / `health` / `rename` | Maintenance, diagnostics, and renaming |
| `config` / `init` / `template` / `project` | Configuration, tracking, and scaffolding |
| `completion` | Shell completion (bash, zsh, fish, powershell) |

## Overlay Management

### `phantom start`

Create and mount a new overlay. `<base-path>` can be a directory path or a registered [project](#phantom-project).

```bash
phantom start <base-path> --name <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--name`, `-n` | (required) | Name for the overlay |
| `--branch`, `-b` | `phantom/<name>` | Git branch name |
| `--base` | | Path to base directory |
| `--workdir`, `-w` | | Working directory inside overlay |

**Examples:**

```bash
# Create an overlay named "feature-auth"
phantom start ~/myproject --name feature-auth

# Create with custom branch name
phantom start ~/myproject --name feature-auth --branch auth-implementation
```

### `phantom stop`

Unmount an overlay without deleting it.

```bash
phantom stop <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--force`, `-f` | `false` | Force unmount |

### `phantom restart`

Unmount and remount an overlay.

```bash
phantom restart <overlay-name>
```

### `phantom list`

List all overlays.

```bash
phantom list [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--format`, `-f` | `table` | Output format (table, json, yaml) |
| `--all`, `-a` | `false` | Show all overlays including stopped |

### `phantom status`

Show detailed status of an overlay.

```bash
phantom status <overlay-name>
```

### `phantom inspect`

Show detailed information about an overlay in JSON format.

```bash
phantom inspect <overlay-name>
```

## Running Agents

### `phantom run`

Create an overlay and run a single agent. `<base-path>` can be a directory path or a registered [project](#phantom-project).

```bash
phantom run <base-path> --agent <command> --task <task> --name <name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--agent`, `-a` | (required) | Agent command to run |
| `--task`, `-t` | | Task description for the agent |
| `--name`, `-n` | (required) | Name for the overlay |
| `--timeout` | `30m` | Timeout for agent execution |
| `--cleanup` | `false` | Remove overlay after completion |
| `--no-commit` | `false` | Skip auto-commit after agent finishes |

**Examples:**

```bash
# Run Claude on a feature
phantom run ~/myproject --agent "claude --print" --task "implement auth" --name auth-feature

# Run with timeout and cleanup
phantom run ~/myproject --agent "aider" --task "fix tests" --name fix-tests --timeout 15m --cleanup
```

### `phantom run-all`

Run multiple agents in parallel on the same base. `<base-path>` can be a directory path or a registered [project](#phantom-project).

```bash
phantom run-all <base-path> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--config`, `-c` | | Path to agents config file (YAML) |
| `--agents` | | Comma-separated list of agent commands |
| `--timeout` | `30m` | Timeout per agent |
| `--cleanup` | `false` | Remove all overlays after completion |
| `--parallel` | `3` | Maximum parallel agents |

**Config File Format (agents.yaml):**

```yaml
agents:
  - name: claude
    command: claude --print
    task: "implement authentication"
  - name: aider
    command: aider
    task: "add unit tests"
  - name: gemini
    command: gemini
    task: "improve error handling"
```

**Examples:**

```bash
# Run from config file
phantom run-all ~/myproject --config agents.yaml --cleanup

# Run inline agents
phantom run-all ~/myproject --agents "claude --print,aider,gemini" --timeout 30
```

### `phantom run-chain`

Run agents sequentially, each building on the previous one's work. `<base-path>` can be a directory path or a registered [project](#phantom-project).

```bash
phantom run-chain <base-path> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--steps` | | Comma-separated list of agent commands |
| `--config`, `-c` | | Path to chain config file |
| `--name`, `-n` | `chain-<timestamp>` | Base name for overlays |
| `--cleanup` | `false` | Remove overlays after completion |

**Examples:**

```bash
# Chain two agents
phantom run-chain ~/myproject --steps "claude --print,aider" --name pipeline

# With cleanup
phantom run-chain ~/myproject --config chain.yaml --cleanup
```

## Viewing Changes

### `phantom diff`

Show changes in an overlay compared to the base.

```bash
phantom diff <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--stat` | `false` | Show diffstat only |
| `--color` | `auto` | Color output (auto, always, never) |

### `phantom compare`

Compare two overlays.

```bash
phantom compare <overlay-a> <overlay-b> [flags]
```

### `phantom conflicts`

Check for potential conflicts between overlays and calculate a **Merge Confidence Score**.

```bash
phantom conflicts <overlay-a> <overlay-b> ...
```

This command identifies files modified by more than one overlay. For Git-tracked repositories, it simulates a 3-way merge to distinguish between **Clean Merges** (different locations in the same file) and **Hard Conflicts** (overlapping changes).

The final **Merge Confidence Score** (0-100%) indicates how safely these overlays can be merged. In non-Git environments, any file overlap is considered an unresolvable hard conflict.

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--format` | `table` | Output format: `table`, `json` |

## Applying Changes

### `phantom commit`

Commit changes in an overlay to its git branch.

```bash
phantom commit <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--message`, `-m` | | Commit message |
| `--all`, `-a` | `false` | Stage all changes |

### `phantom apply`

Apply changes from an overlay back to the base repository.

> [!IMPORTANT]
> **Path Protection (Blast Radius Control)**: If a `.phantomignore` file exists in the repository root, `phantom apply` will validate that no protected files or directories have been modified in the overlay. If any violation is found, the apply operation is rejected.

```bash
phantom apply <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--cleanup` | `false` | Remove overlay after applying |
| `--merge` | `false` | Merge the overlay branch into base |
| `--push` | `false` | Push changes to remote |

### `phantom revert`

Revert a file or directory in an overlay to its base state. This removes any modifications, additions, or deletions made in the overlay for that specific path.

```bash
# Revert a modified config file in an overlay
phantom revert <overlay-name> config/database.yml

# Revert an entire directory
phantom revert <overlay-name> src/components/
```

### `phantom merge`

Merge an overlay branch into the base branch.

```bash
phantom merge <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--squash` | `false` | Squash commits |
| `--no-ff` | `false` | Create a merge commit |

### `phantom sync`

Sync an overlay with the latest base changes.

```bash
phantom sync <overlay-name>
```

## Monitoring

### `phantom watch`

Watch an overlay for changes and trigger actions.

```bash
phantom watch <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--command` | | Command to run on change |
| `--debounce` | `1s` | Debounce interval |

### `phantom logs`

View logs for an overlay or agent run.

```bash
phantom logs <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--follow`, `-f` | `false` | Follow log output |
| `--tail` | `100` | Number of lines to show |

### `phantom replay`

Replay an agent run.

```bash
phantom replay <overlay-name>
```

## Hooks

### `phantom hook`

Manage post-run hooks.

```bash
phantom hook <overlay-name> [flags]
```

**Subcommands:**

- `phantom hook add` - Add a hook
- `phantom hook list` - List hooks
- `phantom hook remove` - Remove a hook
- `phantom hook run` - Run hooks manually

**Hook Types:**

- `on-success` - Run when agent succeeds
- `on-failure` - Run when agent fails
- `on-complete` - Run regardless of outcome

**Examples:**

```bash
# Add a post-run test hook
phantom hook add auth-feature --type on-success --command "npm test"

# Add a notification hook
phantom hook add auth-feature --type on-complete --command 'notify-send "Phantom" "Task complete"'
```

## Snapshots & Export

### `phantom snapshot`

Create a snapshot of an overlay's current state.

```bash
phantom snapshot <overlay-name> [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--name` | `snapshot-<timestamp>` | Snapshot name |

### `phantom export`

Export an overlay to a tarball.

```bash
phantom export <overlay-name> --output <path>
```

### `phantom clone`

Clone an overlay to a new overlay.

```bash
phantom clone <source-overlay> --name <new-name>
```

## Protection

### `phantom lock`

Lock an overlay to prevent accidental cleanup.

```bash
phantom lock <overlay-name>
```

### `phantom unlock`

Unlock an overlay.

```bash
phantom unlock <overlay-name>
```

### `phantom pin`

Pin an overlay to prevent base drift (won't sync with base).

```bash
phantom pin <overlay-name>
```

### `phantom unpin`

Unpin an overlay.

```bash
phantom unpin <overlay-name>
```

## Maintenance

### `phantom prune`

Remove stopped overlays older than a threshold.

```bash
phantom prune [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--older-than` | `7d` | Age threshold |
| `--dry-run` | `false` | Show what would be deleted |

### `phantom gc`

Run garbage collection.

```bash
phantom gc [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--aggressive` | `false` | Aggressive GC |

### `phantom health`

Check system health and dependencies.

```bash
phantom health
```

### `phantom rename`

Rename an overlay.

```bash
phantom rename <old-name> <new-name>
```

## Configuration

### `phantom config`

View or edit configuration.

```bash
phantom config [flags]
```

**Subcommands:**

- `phantom config get <key>` - Get a config value
- `phantom config set <key> <value>` - Set a config value
- `phantom config edit` - Open config in editor
- `phantom config path` - Show config file path

### `phantom init`

Initialize Phantom configuration.

```bash
phantom init [flags]
```

**Flags:**

| Flag | Default | Description |
|------|---------|-------------|
| `--force` | `false` | Overwrite existing config |

### `phantom project`

Manage registered projects. This allows using a short project name instead of absolute directory paths for any command that accepts a base directory.

```bash
phantom project add myapp /path/to/my/app
phantom project list
phantom project remove myapp

# Use the short name 'myapp' instead of a full path
phantom start myapp -n my-feature
phantom run myapp -a aider -t "fix bug"
```

**Subcommands:**

- `phantom project add <name> <path>` - Register a new project and map its base directory path
- `phantom project list` - List all registered projects and their paths
- `phantom project remove <name>` - Delete a project mapping

### `phantom template`

Generate template files for configuration.

```bash
phantom template <type> [flags]
```

**Template Types:**

- `agents` - Agents config file
- `chain` - Chain config file
- `hooks` - Hooks config file

## Shell Completion

### `phantom completion`

Generate shell completion script.

```bash
phantom completion <shell>
```

**Supported Shells:**

- `bash`
- `zsh`
- `fish`
- `powershell`

## Global Flags

These flags are available for all commands:

| Flag | Default | Description |
|------|---------|-------------|
| `--config`, `-c` | `~/.phantom/config.yaml` | Config file path |
| `--verbose`, `-v` | `false` | Verbose output |
| `--quiet`, `-q` | `false` | Suppress non-essential output |
| `--no-color` | `false` | Disable color output |
| `--help`, `-h` | | Show help |
