# AGENTS.md

## Overview

This repository is **AI Toolkit**, a plugin marketplace for AI coding agents (GitHub Copilot, Claude Code, and others supporting the `/plugin marketplace` protocol). It hosts multiple self-contained plugins under `plugins/`.

## Repository Structure

```
ai-toolkit/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace catalog
├── plugins/
│   └── <plugin-name>/          # Each plugin is self-contained
│       ├── .claude-plugin/
│       │   └── plugin.json     # Plugin manifest
│       ├── agents/             # Agent definitions (if any)
│       ├── skills/             # Skill definitions (if any)
│       ├── AGENTS.md           # Plugin-specific agent instructions
│       └── README.md           # Plugin documentation
├── AGENTS.md                   # This file (marketplace-level)
├── CLAUDE.md                   # Points here
├── CONTRIBUTING.md             # How to add plugins
├── LICENSE
└── README.md
```

## Available Plugins

| Plugin | Path | Description |
|--------|------|-------------|
| IRIS | `plugins/iris/` | Integrated Research Intelligence System — 58 agents: research (tech 27 + healthcare 25) and learning (education 6) as independent modes |

**For plugin-specific instructions, read the AGENTS.md inside the plugin directory.**

## Plugin Conventions

### Naming

- Plugin directories: `kebab-case` (e.g., `plugins/iris/`, `plugins/code-reviewer/`)
- Agent files: `<prefix>-<role>.md` (e.g., `tech-binary-analyst.md`)
- Skill folders: match the skill name (e.g., `skills/tech/`)

### Required Files

Every plugin must have:

- `.claude-plugin/plugin.json` — manifest with name, version, description, author
- `README.md` — human-facing documentation
- `AGENTS.md` — instructions for AI agents working with this plugin

### Guardrails (Global)

All plugins in this marketplace must:

1. Never execute destructive filesystem operations outside their designated output directories
2. Never use `sudo`, `rm -rf`, or system-level modifications
3. Never install packages without explicit user consent
4. Never make network requests unless the plugin's core purpose requires it
5. Include pre-flight safety checks in every agent that writes files
6. Clearly document which directories the plugin writes to

### Adding a Plugin

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full process. In short:

1. Create your plugin under `plugins/<your-plugin>/`
2. Include `plugin.json`, `README.md`, and `AGENTS.md`
3. Add an entry to `.claude-plugin/marketplace.json`
4. Submit a pull request

## Working with This Repository

If you are an AI agent working on this repo:

- **Marketplace-level tasks** (adding plugins, updating marketplace.json, repo-wide docs): work from the root
- **Plugin-specific tasks**: navigate to the plugin directory and read its `AGENTS.md` first
- **IRIS tasks**: read `plugins/iris/AGENTS.md` for full agent catalog, io_contract, skill system, and research workflow
