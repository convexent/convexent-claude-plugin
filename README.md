# Convexent Claude Code Plugin

A config-only Claude Code plugin that gives AI agents access to Convexent financial modeling via the `convexent` CLI.

## What this does

This repo contains agent instructions (`CLAUDE.md`) and slash command skills that teach Claude Code how to use the `convexent` CLI to build, inspect, edit, and analyze financial models. No code — just configuration.

## Prerequisites

- [Claude Code](https://claude.com/claude-code) installed
- Node.js 18+ (for `convexent`)
- A Convexent API key or account

## Installation

Add this repo as a Claude Code project dependency. In your project's `.claude/settings.json`:

```json
{
  "projects": [
    {
      "path": "/path/to/convexent-claude-plugin"
    }
  ]
}
```

Or clone and add to your Claude Code config:

```bash
git clone git@github.com:convexent/convexent-claude-plugin.git
```

## What's included

### `CLAUDE.md`
Agent instructions with:
- CLI setup and auth guidance
- Full command reference card
- Output format tips (JSON parsing with jq)
- Important API patterns and gotchas
- Exit code reference

### Skills (slash commands)

| Skill | Command | Description |
|-------|---------|-------------|
| `/convexent-help` | `/convexent-help [topic]` | CLI reference — command tree, workflow guides |
| `/model-edit` | `/model-edit <id> <instruction>` | Full AI edit workflow: propose → apply → verify |
| `/model-analyze` | `/model-analyze <id> <question>` | AI scenario analysis with follow-up support |

## Auth setup

Before using the CLI, authenticate:

```bash
convexent auth set-token sm_your_api_key
convexent auth set-url https://api.convexent.com  # if not default
convexent auth status
```
