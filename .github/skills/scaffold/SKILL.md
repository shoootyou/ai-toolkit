---
name: scaffold
description: Repo-level utility for creating new agents, skills, and validating plugin structure. Not a marketplace plugin — internal tool for maintaining the AI Toolkit repository. Parses natural language input to determine intent and delegates to the appropriate workflow.
tools: Read, Write, Edit, Bash, Grep, Glob, Task
---

# Scaffold — AI Toolkit Generator

Internal utility for the AI Toolkit repository. Creates new agents, new skills, and
validates existing plugins against marketplace conventions.

## Invocation

This is NOT a plugin skill. It is invoked as a repo-level command:

```
/scaffold agent <description>
/scaffold skill <description>
/scaffold validate <path>
```

The input after the action keyword is **free-form natural language**. The skill parses
the intent (agent / skill / validate) from the first word after `/scaffold`, then
treats everything else as the description of what to create or validate.

## Workflow Routing

| Intent | Trigger Words | Workflow |
|--------|---------------|----------|
| **Create Agent** | "agent", "agente", "new agent" | `workflows/create-agent.md` |
| **Create Skill** | "skill", "new skill" | `workflows/create-skill.md` |
| **Validate** | "validate", "audit", "check", "validar" | `workflows/validate.md` |

### Routing Logic

1. Read the user's input after `/scaffold`
2. Extract the first significant word to determine intent
3. If ambiguous, ask the user using `ask_user`:
   - "Do you want to create an agent, a skill, or validate a plugin?"
   - Choices: ["Create an agent", "Create a skill", "Validate a plugin"]
4. Pass the full description text to the selected workflow

## How It Works

### Agent Creation (`/scaffold agent ...`)
1. Parses the description to understand the desired agent
2. Asks clarifying questions: target plugin, domain, role, specialization
3. Reads existing agents as reference templates
4. Generates the agent file following established conventions
5. Places it in the correct `plugins/<plugin>/agents/` directory
6. Updates affected catalogs (orchestrate.md, AGENTS.md)

### Skill Creation (`/scaffold skill ...`)
1. Parses the description to understand the desired skill
2. Asks clarifying questions: target plugin, purpose, workflows needed
3. Reads existing skills as reference templates
4. Generates SKILL.md + workflow files
5. Places them in `plugins/<plugin>/skills/<name>/`

### Plugin Validation (`/scaffold validate ...`)
1. Reads the target plugin directory
2. Checks against conventions from CONTRIBUTING.md
3. Validates: required files, naming, size limits, guardrails, consistency
4. Reports issues with severity levels

## Principles

### Always Confirm Understanding
Before generating anything:
1. Restate what you understood from the user's input
2. Present the plan (what files will be created, where)
3. Ask for confirmation with `ask_user`
4. Only proceed after explicit approval

### Never Overwrite
- If a file already exists at the target path, STOP and ask the user
- Offer: rename, skip, or replace (with explicit confirmation)

### Template-Driven
- Always read at least one existing agent/skill as a reference template
- Match the established patterns: frontmatter, XML blocks, section order
- Adapt the template to the new agent/skill's specific role

### Incremental Output
- Create files one at a time, confirming each before moving to the next
- After all files are created, offer to update catalogs

## Context Files

- See [workflows/create-agent.md](workflows/create-agent.md) for agent generation
- See [workflows/create-skill.md](workflows/create-skill.md) for skill generation
- See [workflows/validate.md](workflows/validate.md) for plugin validation
- See [../../CONTRIBUTING.md](../../CONTRIBUTING.md) for marketplace conventions

## Guardrails

This skill creates files in the repository structure. Safety rules:

1. **Scope**: Only writes to `plugins/` directories and `.github/` — never to system paths
2. **No overwrites**: Never replaces existing files without explicit user confirmation
3. **No execution**: Generated agents and skills are documentation — this skill never runs them
4. **No system modification**: No `sudo`, no package installs, no network requests
5. **Size limits**: All generated files must be under 30,000 characters
6. **Validation first**: Before writing, validate that the target plugin directory exists and follows conventions
7. **Dry run option**: User can request a preview of what will be generated before writing
