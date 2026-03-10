---
name: guild-cli-workflow
description: Guild CLI workflow for agent development. Use when creating agents, saving/publishing agents, cloning agents, or running guild agent commands. Handles proper CLI workflow and prevents direct git operations.
---

# Guild CLI Agent Development Workflow

For local agent development using the Guild CLI. This workflow manages agent code via the Guild git server.

## CRITICAL: Always Use the Guild CLI

**NEVER manually create agent files or use raw git commands.**

```bash
# Create a new agent
guild agent init --name my-agent --template LLM

# Clone an existing agent
guild agent clone guildai/slack-assistant

# Save changes (commits and syncs to Guild server)
guild agent save --message "Description of changes"

# Save and publish
guild agent save --message "Description" --wait --publish
```

## What the CLI Handles

- Proper `.gitignore` (includes `guild.json`)
- Correct file structure
- Git remote configuration to Guild server
- Version management and validation
- Publishing workflow

## NEVER Do These Things

- ❌ Manually create `package.json`, `tsconfig.json`, or `guild.json`
- ❌ Run `git push` directly (use `guild agent save`)
- ❌ Run `git pull` directly (use `guild agent pull`)
- ❌ Run `git commit` directly (use `guild agent save`)
- ❌ Edit `guild.json` (it's generated and gitignored)
- ❌ Push to GitHub (agents sync to Guild's git server)

## Common Commands

### Project Setup

```bash
# Install Guild CLI skills for coding assistants (Claude Code, etc.)
guild setup

# Install skills and create a CLAUDE.md template
guild setup --claude-md
```

### Creating Agents

```bash
# Interactive creation
guild agent init --name my-agent

# With specific template
guild agent init --name my-agent --template LLM
guild agent init --name my-agent --template COMPILED
guild agent init --name my-agent --template BLANK

# Fork an existing agent
guild agent init --fork guildai/slack-assistant
```

### Working with Existing Agents

```bash
# Clone to work on an agent
guild agent clone guildai/slack-assistant
cd slack-assistant

# Pull remote changes (from collaborators or web edits)
guild agent pull

# Check current version status
guild agent versions --limit 1

# Get latest code
guild agent code

# Search across all agent code files
guild agent grep "pattern"
guild agent grep "pattern" --published
```

### Saving Changes

```bash
# Save without publishing (creates draft)
guild agent save --message "WIP: still testing"

# Save and wait for validation
guild agent save --message "Fix bug" --wait

# Save, validate, and publish
guild agent save --message "Release v1.0" --wait --publish
```

### Publishing

```bash
# Publish latest validated version
guild agent publish

# Check publication status
guild agent versions --limit 1
```

### Testing

```bash
# Interactive test session
guild agent test

# Test with specific input
guild agent chat "Hello, can you help me?"
```

## File Structure

After `guild agent init`, you get:

```
my-agent/
├── .git/              # Git repo (remote is Guild server)
├── .gitignore         # Includes guild.json
├── agent.ts           # Your agent code
├── package.json       # Dependencies
├── tsconfig.json      # TypeScript config
└── guild.json         # Agent ID (gitignored, local only)
```

## Version Lifecycle

1. **Draft** - After `guild agent save` (no `--publish`)
2. **Validating** - After `--publish`, running validation
3. **Published** - Validation passed, available for use
4. **Failed** - Validation failed, check errors

## Troubleshooting

### "No changes to commit"

You already committed. Just push:

```bash
git push origin main
```

### "guild.json not found"

You're not in an agent directory. Either:

- `cd` into the agent directory
- Run `guild agent init` to create one

### Validation Failed

Check the error in `guild agent versions --limit 1`. Common issues:

- TypeScript compilation errors
- Missing dependencies
- Invalid schema

## See Also

- [Agent Development Guide](https://github.com/agents-for-dev/guildai__agent-builder__019a8e0d-5280-726e-0000-b896bbbc2320/blob/main/docs/AGENT_DEVELOPMENT.md) - Comprehensive patterns and SDK usage
- SDK: `@guildai/agents-sdk`
