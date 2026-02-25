---
title: "Workflows & Examples"
description: "Real-world usage patterns and recipes for Phantom."
weight: 2
---

This page covers common workflows and real-world examples for using Phantom effectively.

## Basic Workflows

### Single Agent Task

The simplest workflow is running a single agent on a task:

```bash
# Run Claude to implement a feature
phantom run ~/myproject \
  --agent "claude --print" \
  --task "Implement user authentication with JWT tokens" \
  --name auth-feature

# Review the changes
phantom diff auth-feature

# Apply to base if satisfied
phantom apply auth-feature --cleanup
```

### Parallel Feature Development

Run multiple agents to implement different features simultaneously:

```bash
# Create a config file for parallel agents
cat > agents.yaml << 'EOF'
agents:
  - name: auth
    command: claude --print
    task: "Implement OAuth2 login"
  - name: api
    command: aider
    task: "Create REST API endpoints for user management"
  - name: tests
    command: gemini
    task: "Add integration tests for the API"
EOF

# Run all agents in parallel
phantom run-all ~/myproject --config agents.yaml

# Check results
phantom list

# Compare approaches
phantom compare auth api

# Apply the best result
phantom apply auth --cleanup
```

### Sequential Pipeline

Chain agents where each builds on the previous work:

```bash
# First agent implements the feature
# Second agent adds tests
# Third agent reviews and improves

phantom run-chain ~/myproject \
  --steps "claude --print,aider,cursor" \
  --name feature-pipeline \
  --cleanup
```

## Real-World Examples

### Example 1: Bug Fix with Verification

```bash
# Run agent to fix a bug
phantom run ~/myproject \
  --agent "claude --print" \
  --task "Fix the null pointer exception in UserService.java line 142" \
  --name bugfix-npe

# Add a hook to run tests after the fix
phantom hook add bugfix-npe --type on-success --command "cd /path/to/overlay && mvn test"

# If tests pass, apply the fix
phantom apply bugfix-npe --cleanup
```

### Example 2: Code Refactoring

```bash
# Refactor legacy code
phantom run ~/legacy-project \
  --agent "aider --message 'Refactor UserService to use dependency injection'" \
  --task "Refactor UserService class to use constructor injection and interfaces" \
  --name refactor-di

# Check what changed
phantom diff refactor-di --stat

# Export the changes for review
phantom export refactor-di --output refactor-di-changes.tar.gz
```

### Example 3: A/B Testing Implementations

```bash
# Run two different agents to solve the same problem
phantom run-all ~/myproject \
  --agents "claude --print,aider" \
  --task "Implement a caching layer for the database queries" \
  --name cache-impl

# Compare the two approaches
phantom compare cache-impl-0 cache-impl-1

# Choose and apply the best one
phantom apply cache-impl-0 --cleanup
phantom prune --older-than 0s  # Clean up the other
```

### Example 4: Documentation Generation

```bash
# Generate documentation for a module
phantom run ~/myproject \
  --agent "claude --print" \
  --task "Generate comprehensive API documentation for the /api/v1 endpoints" \
  --name docs-api

# Preview the generated docs
phantom diff docs-api

# Apply to base
phantom apply docs-api --cleanup
```

### Example 5: Security Audit

```bash
# Run security-focused agent
phantom run ~/myproject \
  --agent "claude --print" \
  --task "Review the authentication code for security vulnerabilities and suggest fixes" \
  --name security-audit

# Keep this overlay for reference (don't cleanup)
phantom lock security-audit

# Create a separate overlay for fixes
phantom clone security-audit --name security-fixes

# Apply the fixes
phantom apply security-fixes --cleanup
```

## Advanced Patterns

### Continuous Integration Hook

```bash
# Create an overlay with CI hooks
phantom run ~/myproject \
  --agent "claude --print" \
  --task "Implement the feature" \
  --name ci-feature

# Add multiple hooks
phantom hook add ci-feature --type on-success --command "npm run lint"
phantom hook add ci-feature --type on-success --command "npm test"
phantom hook add ci-feature --type on-failure --command 'curl -X POST $WEBHOOK_URL -d "Tests failed"'
```

### Snapshot-Based Recovery

```bash
# Before risky changes, create a snapshot
phantom snapshot risky-changes --name before-refactor

# Make changes
phantom run ~/myproject \
  --agent "claude --print" \
  --task "Major refactoring of the database layer" \
  --name risky-changes

# If something goes wrong, restore from snapshot
phantom stop risky-changes
# Restore logic would go here

# If successful, continue
phantom apply risky-changes --cleanup
```

### Multi-Repository Changes

```bash
# Work on multiple repos simultaneously
phantom run ~/frontend --agent "claude --print" --task "Update API client" --name fe-api
phantom run ~/backend --agent "claude --print" --task "Add new endpoints" --name be-api

# Verify both changes work together
# (manual testing or integration tests)

# Apply both changes
phantom apply fe-api --cleanup
phantom apply be-api --cleanup
```

### Conflict Resolution

```bash
# Two agents modified the same files
phantom run-all ~/myproject --agents "claude,aider" --task "Add logging" --name logging

# Check for conflicts
phantom conflicts logging-0 logging-1

# If conflicts exist, manually resolve by creating a merged overlay
phantom clone logging-0 --name logging-merged
# Then manually apply changes from logging-1

# Apply the merged result
phantom apply logging-merged --cleanup
phantom prune --older-than 0s
```

## Team Workflows

### Shared Overlay Review

```bash
# Developer 1 creates an overlay
phantom run ~/myproject --agent "claude --print" --task "Feature X" --name feature-x

# Export for review
phantom export feature-x --output feature-x-review.tar.gz

# Share with team (via file share, PR attachment, etc.)

# Developer 2 imports and reviews
# (import functionality would restore from tarball)

# After review, apply
phantom apply feature-x --cleanup
```

### Handoff Workflow

```bash
# First developer starts work
phantom run ~/myproject --agent "claude --print" --task "Start feature" --name handoff-feature

# Create a snapshot for handoff
phantom snapshot handoff-feature --name handoff-point-1

# Second developer continues
phantom sync handoff-feature  # Get latest from base if needed
# Continue work...

# Finalize
phantom apply handoff-feature --cleanup
```

## Best Practices

### 1. Use Descriptive Names

```bash
# Good
phantom run . --agent "claude" --task "..." --name auth-oauth-jwt

# Avoid
phantom run . --agent "claude" --task "..." --name test1
```

### 2. Always Review Before Applying

```bash
# Review changes
phantom diff feature-name

# Check for unintended modifications
phantom diff feature-name --stat

# Then apply
phantom apply feature-name --cleanup
```

### 3. Use Cleanup Flag Wisely

```bash
# Cleanup for one-off tasks
phantom run . --agent "claude" --task "Fix typo" --name typo --cleanup

# Keep for important work
phantom run . --agent "claude" --task "Major feature" --name major-feature
phantom lock major-feature  # Prevent accidental cleanup
```

### 4. Leverage Hooks for Quality Gates

```bash
# Always run tests before applying
phantom hook add feature-name --type on-success --command "npm test"
phantom hook add feature-name --type on-failure --command "npm run lint"
```

### 5. Regular Cleanup

```bash
# Weekly cleanup of old overlays
phantom prune --older-than 7d

# More aggressive cleanup
phantom gc --aggressive
```
