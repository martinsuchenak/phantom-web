---
title: "Installation"
description: "Install Phantom on your system. Available for macOS and Linux via Homebrew, binary download, or build from source."
weight: 2
---

## Prerequisites

Before installing Phantom, ensure you have the required dependencies for your operating system.

### Linux

Phantom supports two backends on Linux:

- **Native overlayfs** (requires root/sudo) - No additional installation needed
- **fuse-overlayfs** (rootless) - Install with: `apt install fuse-overlayfs` or `yum install fuse-overlayfs`

The appropriate backend is auto-detected at runtime.

### macOS

Phantom requires a FUSE implementation on macOS:

- **macFUSE** - Download from [osxfuse.github.io](https://osxfuse.github.io/)
- **FUSE-T** - Install with: `brew install macos-fuse-t/homebrew-fuse-t/fuse-t`

You'll also need `unionfs-fuse`:
```bash
brew install unionfs-fuse
```

## Install via Homebrew (Recommended)

The easiest way to install Phantom on macOS and Linux is via Homebrew:

```bash
brew tap martinsuchenak/tap
brew install phantom
```

To upgrade to the latest version:
```bash
brew upgrade phantom
```

## Download Binary

Download the latest binary for your platform from [GitHub Releases](https://github.com/martinsuchenak/phantom/releases/latest).

### Linux (amd64)

```bash
curl -LO https://github.com/martinsuchenak/phantom/releases/latest/download/phantom_Linux_amd64.tar.gz
tar xzf phantom_Linux_amd64.tar.gz
sudo mv phantom /usr/local/bin/
```

### Linux (arm64)

```bash
curl -LO https://github.com/martinsuchenak/phantom/releases/latest/download/phantom_Linux_arm64.tar.gz
tar xzf phantom_Linux_arm64.tar.gz
sudo mv phantom /usr/local/bin/
```

### macOS (Intel)

```bash
curl -LO https://github.com/martinsuchenak/phantom/releases/latest/download/phantom_Darwin_amd64.tar.gz
tar xzf phantom_Darwin_amd64.tar.gz
sudo mv phantom /usr/local/bin/
```

### macOS (Apple Silicon)

```bash
curl -LO https://github.com/martinsuchenak/phantom/releases/latest/download/phantom_Darwin_arm64.tar.gz
tar xzf phantom_Darwin_arm64.tar.gz
sudo mv phantom /usr/local/bin/
```

## Build from Source

To build Phantom from source, you'll need [Go 1.21+](https://golang.org/dl/) installed.

```bash
# Clone the repository
git clone https://github.com/martinsuchenak/phantom.git
cd phantom

# Build for current platform
make build

# Install to /usr/local/bin
make install
```

### Cross-Compile for All Platforms

```bash
make build-all
```

Binaries will be output to the `dist/` directory.

## Verify Installation

After installation, verify that Phantom is working correctly:

```bash
phantom version
```

You should see output similar to:
```
phantom version 0.1.0
```

## Shell Completion

Phantom supports shell completion for bash, zsh, fish, and PowerShell.

### Bash

```bash
phantom completion bash > /etc/bash_completion.d/phantom
```

### Zsh

```bash
phantom completion zsh > "${fpath[1]}/_phantom"
```

### Fish

```bash
phantom completion fish > ~/.config/fish/completions/phantom.fish
```

### PowerShell

```powershell
phantom completion powershell > phantompowershellcompletion.ps1
. ./phantompowershellcompletion.ps1
```

## Next Steps

- [Getting Started](/docs/) - Learn the basics of Phantom
- [Commands Reference](/docs/commands/) - Explore all available commands
- [Configuration](/docs/configuration/) - Customize Phantom behavior
