# AI Toolkit

A curated collection of AI-powered plugins for AI coding agents that support the plugin marketplace protocol.

## Compatibility

This marketplace works with any agent that supports the `/plugin marketplace` protocol:

- [GitHub Copilot](https://github.com/features/copilot/cli)
- [Claude Code](https://code.claude.com)

## Installation

Add the marketplace:

```bash
/plugin marketplace add shoootyou/ai-toolkit
```

Browse available plugins:

```bash
/plugin marketplace browse ai-toolkit
```

Install a plugin:

```bash
/plugin install iris@ai-toolkit
```

## Plugins

### IRIS — Integrated Research Intelligence System

> Multi-agent platform with **58 specialized agents** for deep research and adaptive learning.

IRIS operates in two independent modes:

#### Research Mode — Investigate any topic

A router analyzes your topic and delegates to the right domain. Each domain spawns parallel agents, then a synthesizer consolidates findings into a comprehensive report.

| Domain | Agents | Focus |
|--------|--------|-------|
| **Tech** | 27 | Software architecture, binary analysis, security, APIs, dependencies, infrastructure, performance, and more |
| **Healthcare** | 25 | Clinical evidence, epidemiology, pharmaceuticals, bioethics, policy, medical devices, and more |

**Skills:** `/iris:research` (auto-routing) · `/iris:tech` · `/iris:health`

#### Learn Mode — Learn about any topic

Six pedagogical agents work together to create educational material adapted to your level, learning style, and language.

| Agent | Role |
|-------|------|
| Researcher | Gathers authoritative sources on the topic |
| Curriculum Designer | Builds a structured learning path |
| Content Creator | Writes explanations, analogies, and examples |
| Assessor | Creates exercises, quizzes, and Socratic questions |
| Fact-Checker | Validates sources and flags bias |
| Synthesizer | Produces the final learning document |

**Skill:** `/iris:teach`

```bash
/plugin install iris@ai-toolkit
```

[View full documentation →](./plugins/iris/)

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on adding new plugins.

## Author

**Rodolfo Castelo** — rodolfo@shoootyou.dev

## License

[MIT](./LICENSE)
