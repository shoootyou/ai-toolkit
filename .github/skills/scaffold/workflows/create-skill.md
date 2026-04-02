---
name: create-skill
description: >
  Workflow for scaffolding new skill directories with SKILL.md and workflow files.
  Guides the user through defining the skill's purpose, workflows, and conventions,
  then generates all files following established patterns.
---

# Create Skill Workflow

Generates a new skill directory with SKILL.md and workflow files for a target plugin.
Follows the established patterns from existing skills in the repository.

## Pre-Flight

Before anything else:

1. Verify the target plugin exists at `plugins/<plugin>/`
2. Verify `plugins/<plugin>/skills/` directory exists
3. Read at least ONE existing SKILL.md from that plugin as a reference
4. If no plugin is specified, ask the user which plugin this skill belongs to

## Step 1 — Parse Input

Read the user's description and extract:

| Field | Example | Required |
|-------|---------|----------|
| **Plugin** | iris, code-reviewer | Yes |
| **Skill Name** | tech, health, research, deploy | Yes |
| **Purpose** | What this skill does | Yes |
| **Workflows** | List of workflows needed | Yes |
| **Domain** | If domain-specific or cross-domain | Optional |

### Skill Name Rules

- Must be kebab-case (e.g., `code-review`, `deploy`, `research`)
- Must be unique within the plugin
- Directory: `plugins/<plugin>/skills/<skill-name>/`

## Step 2 — Define Workflows

For each workflow the user needs, gather:

| Field | Description |
|-------|-------------|
| **Name** | Workflow filename (kebab-case, e.g., `investigate`, `orchestrate`) |
| **Purpose** | What this workflow does |
| **Trigger** | What user input activates it |
| **Internal?** | Is it user-facing or only called by other workflows? |

Present the workflow list and confirm with `ask_user`:

```
Skill: <name>
Plugin: <plugin>
Location: plugins/<plugin>/skills/<name>/

Workflows:
1. <workflow-1> — <purpose> (trigger: "<trigger words>")
2. <workflow-2> — <purpose> (internal)
...

Files to create:
- plugins/<plugin>/skills/<name>/SKILL.md
- plugins/<plugin>/skills/<name>/workflows/<workflow-1>.md
- plugins/<plugin>/skills/<name>/workflows/<workflow-2>.md
```

Do NOT proceed without explicit approval.

## Step 3 — Read Reference Template

Read an existing SKILL.md from the target plugin:

```
plugins/<plugin>/skills/*/SKILL.md
```

Extract the pattern:

1. **Frontmatter**: YAML with name, description (single line), tools
2. **Title and description**: What the skill does
3. **Workflow Routing Table**: Maps triggers to workflow files
4. **How It Works**: Step-by-step description of the skill's flow
5. **Principles**: Behavioral guidelines (conversational refinement, etc.)
6. **Context Files**: Links to workflow files
7. **Guardrails**: Safety rules

Also read at least one workflow file to understand the workflow pattern:
- Frontmatter with name and description
- Steps with clear instructions
- Error handling
- Guardrails

## Step 4 — Generate SKILL.md

Create the main skill file following this structure:

```markdown
---
name: <skill-name>
description: <Single line describing what this skill does, when to use it, and what it produces.>
tools: Read, Write, Edit, Bash, Grep, Glob, Task
---

# <Skill Title>

<Brief description of the skill's purpose and scope.>

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **<Workflow 1>** | "<trigger words>" | `workflows/<name>.md` |
| **<Workflow 2>** | (internal) | `workflows/<name>.md` |

## How It Works

1. **User invokes the skill** with a topic/request
2. **<Workflow 1> activates:**
   - <step-by-step description>
3. **<Workflow 2> takes over (if applicable):**
   - <step-by-step description>
4. **Results appear** at <output location>

## Principles

### Conversational Refinement
Never assume understanding. Always:
1. Restate what you understood
2. Ask for confirmation using `ask_user`
3. Only proceed after explicit user approval

### <Additional principles specific to this skill>

## Context Files
- See [workflows/<name>.md](workflows/<name>.md) for <description>

## Guardrails

<Safety rules appropriate for this skill's scope>
```

## Step 5 — Generate Workflow Files

For each workflow, create a file following this structure:

```markdown
---
name: <workflow-name>
description: >
  <What this workflow does and when it is invoked.>
---

# <Workflow Title>

<Brief description.>

## Pre-Flight

<Validation checks before starting.>

## Step 1 — <First Step>

<Detailed instructions for step 1.>

## Step 2 — <Second Step>

<Detailed instructions for step 2.>

## Step N — <Final Step>

<Detailed instructions for the final step.>

## Error Handling

| Error | Action |
|-------|--------|
| <error condition> | <recovery action> |

## Guardrails

<Safety rules specific to this workflow.>
```

### Workflow Quality Standards

Each workflow must:
- Have clear, numbered steps
- Include pre-flight validation
- Specify error handling for common failures
- Include guardrails appropriate to its scope
- Reference other workflows it depends on or delegates to
- Stay under 30,000 characters

## Step 6 — Write Files

Create files in order:

1. Create the skill directory: `plugins/<plugin>/skills/<name>/`
2. Create the workflows directory: `plugins/<plugin>/skills/<name>/workflows/`
3. Write `SKILL.md`
4. Write each workflow file, one at a time

After each file:
- Report the file path and character count
- If any file exceeds 25,000 characters, warn about the 30K limit

## Step 7 — Update Plugin Documentation

After all files are created, offer to update:

1. **Plugin AGENTS.md** — Add the new skill to the skill system section
2. **Plugin README.md** — Add usage examples for the new skill
3. **Root AGENTS.md** — Update plugin description if agent/skill counts changed

Use `ask_user` for each:
- "Should I update the plugin's AGENTS.md with this new skill?"
- Choices: ["Yes", "No, I'll do it manually"]

## Step 8 — Summary

Present a summary:

```
Skill created successfully:
- Directory: plugins/<plugin>/skills/<name>/
- SKILL.md: <N> characters
- Workflows:
  - <workflow-1>.md: <N> characters
  - <workflow-2>.md: <N> characters
- Documentation updated: <list or "none">
- Next steps: <suggestions>
```

## Error Handling

| Error | Action |
|-------|--------|
| Plugin directory not found | Ask user for correct path |
| Skill name already exists | Show existing skill, ask user to rename or skip |
| File write fails | Report error, do not retry automatically |
| Reference template not found | Use minimal template, warn about quality |
| Workflow count exceeds 10 | Warn user, suggest splitting into sub-skills |

## Guardrails

1. Never overwrite existing files without explicit confirmation
2. Never modify files outside the target plugin directory (during creation)
3. Always read a reference template before generating
4. Always confirm understanding before writing
5. All generated files must include appropriate guardrails sections
6. SKILL.md must always include a Guardrails section
7. Every workflow must include a Guardrails section
