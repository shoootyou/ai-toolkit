# Contributing to AI Toolkit

Thank you for your interest in contributing plugins to AI Toolkit. This guide covers the structure, standards, and process for adding new plugins.

## Plugin Structure

Each plugin lives in `plugins/<plugin-name>/` and must include:

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest (required)
├── agents/                  # Agent definitions (optional)
│   └── <agent-name>.md
├── skills/                  # Skills with workflows (optional)
│   └── <skill-name>/
│       ├── SKILL.md
│       └── workflows/
├── CLAUDE.md                # Agent instructions (recommended)
├── AGENTS.md                # Open-standard agent guide (recommended)
└── README.md                # Human-facing documentation (required)
```

## Plugin Manifest

Every plugin must have a `.claude-plugin/plugin.json`:

```json
{
  "name": "your-plugin-name",
  "description": "Brief description of what the plugin does",
  "version": "1.0.0",
  "author": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "license": "MIT"
}
```

## Naming Conventions

- Plugin names: `kebab-case` (e.g., `code-reviewer`, `doc-generator`)
- Agent files: `<prefix>-<role>.md` (e.g., `review-security-analyst.md`)
- Skill folders: match the skill name (e.g., `skills/review/`)

## Agent Standards

If your plugin includes agents, each agent file must:

1. Use YAML frontmatter with `name`, `description`, and `tools`
2. Include XML blocks in order: `<role>`, `<io_contract>`, `<philosophy>`, `<investigation_techniques>`, `<output_format>`, `<guardrails>`
3. Stay under 30,000 characters
4. Include mandatory guardrails (no destructive commands, no sudo, no network requests unless explicitly required)

## Guardrails

All plugins must respect these rules:

- No destructive filesystem operations outside designated output directories
- No `sudo`, `rm -rf`, or system-level modifications
- No package installations without explicit user consent
- No network requests unless the plugin's purpose requires it
- Pre-flight checklist in every agent that writes files

## Adding Your Plugin to the Marketplace

1. Fork this repository
2. Create your plugin under `plugins/<your-plugin>/`
3. Add an entry to `.claude-plugin/marketplace.json`
4. Submit a pull request

### Marketplace Entry

Add your plugin to the `plugins` array in `.claude-plugin/marketplace.json`:

```json
{
  "name": "your-plugin",
  "source": "./plugins/your-plugin",
  "description": "What your plugin does",
  "version": "1.0.0",
  "author": { "name": "Your Name" },
  "license": "MIT",
  "keywords": ["relevant", "tags"],
  "category": "category-name"
}
```

## Pull Request Checklist

Before submitting:

- [ ] Plugin has a valid `plugin.json` manifest
- [ ] README.md documents usage, installation, and examples
- [ ] All agents include guardrails
- [ ] No files write outside their designated output directory
- [ ] Agent files are under 30K characters each
- [ ] Tested locally with Claude Code (`claude --plugin-dir ./plugins/your-plugin`)
- [ ] Marketplace entry added to `marketplace.json`

## Testing Locally

```bash
# Test your plugin in isolation
claude --plugin-dir ./plugins/your-plugin

# Test via the marketplace
claude --plugin-dir .
```

## Questions

Open an issue using the appropriate template or reach out to the maintainers.
