# Convexent Claude Code Plugin

A config-only Claude Code plugin that gives AI agents access to Convexent financial modeling via the `convexent` CLI.

## What this does

This repo contains agent instructions (`CLAUDE.md`) and slash command skills that teach Claude Code how to use the `convexent` CLI to build, inspect, edit, and analyze financial models. No code — just configuration.

## Prerequisites

- [Claude Code](https://claude.com/claude-code) installed
- `convexent` CLI installed (`npm install -g convexent`)
- A Convexent API key or account

### Cowork / Cloud environments

Add this to your environment's **setup script** in the Cowork web UI:

```bash
npm install -g convexent
```

This runs before Claude Code launches and makes the CLI available in the session.

## Installation

Add this repo as a Claude Code project dependency. In your project's `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "convexent@convexent": true
  },
  "extraKnownMarketplaces": {
    "convexent": {
      "source": {
        "source": "git",
        "url": "https://github.com/convexent/convexent-claude-plugin.git"
      }
    }
  }
}
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
# Browser-based login (interactive)
convexent auth login

# Or set an API key directly
convexent auth set-token sm_your_api_key
convexent auth set-url https://api.convexent.com  # if not default
convexent auth status
```

For Cowork environments, you can also set `CONVEXENT_API_KEY` as an environment variable in the Cowork environment config.
