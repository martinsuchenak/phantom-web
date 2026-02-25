---
title: "Documentation"
description: "Learn how to use Phantom to manage overlay filesystems and run multiple AI agents in parallel."
weight: 3
---

## Getting Started

Phantom is a CLI tool for managing overlay filesystems to enable multiple AI agents to work on the same codebase in parallel without conflicts. Each agent gets its own isolated overlay with independent writes and its own git branch.

### Quick Start

If you haven't installed Phantom yet, follow the [Installation guide](/installation/).

Once installed, initialize your configuration:

```bash
phantom init
```

This creates a default configuration file at `~/.phantom/config.yaml`.

### Basic Workflow

1. **Create an overlay and run an agent**
   ```bash
   phantom run ~/myproject --agent "claude --print" --task "implement auth" --name auth-feature
   ```

2. **Check what changed**
   ```bash
   phantom diff auth-feature
   ```

3. **Apply changes back to the base repository**
   ```bash
   phantom apply auth-feature --cleanup
   ```

## Core Concepts

### Overlays

An overlay is a writable filesystem layer on top of your base project directory. The base stays untouched — all writes go to an isolated upper layer. Each overlay gets:

- A unique mount point (e.g., `~/.phantom/mnt/feature-auth/`)
- An isolated git branch (e.g., `phantom/feature-auth`)
- Its own state and configuration

### Agents

An agent is any CLI command that can modify files in a project. Common examples include:

- `claude --print` - Claude Code CLI
- `aider` - Aider AI pair programmer
- `gemini` - Gemini CLI
- `copilot` - GitHub Copilot CLI

### Workflows

Phantom supports three main workflow patterns:

- **Single Agent** (`run`) - Run one agent on a task
- **Parallel Agents** (`run-all`) - Run multiple agents simultaneously on the same base
- **Sequential Chain** (`run-chain`) - Run agents in sequence, each building on the previous

## Documentation Sections

Explore the documentation to learn more about Phantom's capabilities:

- **[Commands Reference](/docs/commands/)** - Every command, flag, and option
- **[TUI Dashboard](/docs/tui/)** - Interactive terminal UI reference
- **[Workflows & Examples](/docs/workflows/)** - Real-world usage patterns
- **[Configuration](/docs/configuration/)** - Config file, hooks, templates, environment variables

## How It Works

```
Base Directory (read-only lower layer)
         ↓
   ┌─────────────┐
   │  Overlay A   │  → phantom/feature-auth  → ~/.phantom/mnt/feature-auth/
   │  Overlay B   │  → phantom/feature-api   → ~/.phantom/mnt/feature-api/
   │  Overlay C   │  → phantom/feature-tests → ~/.phantom/mnt/feature-tests/
   └─────────────┘
         ↑
  Isolated upper layers (~/.phantom/overlays/<name>/upper/)
```

**Linux:** Native kernel overlayfs (root) or fuse-overlayfs (rootless, auto-detected).

**macOS:** unionfs-fuse via macFUSE or FUSE-T.

## Need Help?

- [Report an issue](https://github.com/martinsuchenak/phantom/issues) on GitHub
- Check the [FAQ](/docs/faq/) for common questions
