---
name: create-agent
description: >
  Workflow for scaffolding new agent files. Guides the user through defining the agent's
  role, domain, and specialization, then generates a complete agent file following
  established conventions. Reads existing agents as templates to ensure consistency.
---

# Create Agent Workflow

Generates a new agent file for a target plugin by following the established patterns
in the repository. The workflow is conversational — it gathers information step by step,
confirms understanding, and only writes after explicit approval.

## Pre-Flight

Before anything else:

1. Verify the target plugin exists at `plugins/<plugin>/`
2. Verify `plugins/<plugin>/agents/` directory exists
3. Read at least ONE existing agent from that plugin as a reference template
4. If no plugin is specified, ask the user which plugin this agent belongs to

## Step 1 — Parse Input

Read the user's description and extract:

| Field | Example | Required |
|-------|---------|----------|
| **Plugin** | iris, code-reviewer | Yes |
| **Domain** | tech, health, or new domain | Yes (for multi-domain plugins) |
| **Role** | binary analyst, policy reviewer | Yes |
| **Specialization** | What makes this agent unique | Yes |
| **Name** | Auto-generated from domain + role | Auto |

### Name Generation

Agent names follow the pattern: `{domain}-{role-in-kebab-case}.md`

Examples:
- Domain: tech, Role: API security analyst → `tech-api-security-analyst.md`
- Domain: health, Role: vaccination researcher → `health-vaccination-researcher.md`

If the generated name already exists, STOP and inform the user. Offer alternatives.

## Step 2 — Confirm Understanding

Present to the user:

```
I understood the following:
- Plugin: <plugin>
- Domain: <domain>
- Role: <role description>
- Name: <generated-name>.md
- Location: plugins/<plugin>/agents/<generated-name>.md
- Specialization: <what makes it unique>
```

Use `ask_user` to confirm. Do NOT proceed without explicit approval.

## Step 3 — Read Reference Template

Read an existing agent from the SAME domain in the target plugin:

```
plugins/<plugin>/agents/<domain>-*.md
```

Select the first available agent. Extract the structure:

1. **Frontmatter**: YAML block with name, description, tools
2. **XML blocks in order**:
   - `<role>` — identity, uniqueness, ecosystem position, responsibilities
   - `<io_contract>` — input/output specification
   - `<philosophy>` — approach, methodology, depth expectations
   - `<investigation_techniques>` — specific methods and commands
   - `<output_format>` — markdown structure of the output
   - `<guardrails>` — mandatory safety rules
3. **Size**: Must stay under 30,000 characters

Note the structure, tone, depth, and patterns. The new agent MUST match this quality level.

## Step 4 — Generate Agent Content

Create the agent file with all required sections:

### 4.1 Frontmatter

```yaml
---
name: {domain}-{role-kebab}
description: >
  Specialized agent for {role description}. {What it investigates}.
  Produces structured research documents. Spawned by the orchestrate workflow.
tools: Read, Bash, Grep, Glob
---
```

### 4.2 Role Block

The `<role>` block must include:

- **Identity**: "You are a **{Role Title}** — a specialized research agent focused on..."
- **What Makes This Agent Unique**: 3-5 bullet points explaining why this agent exists
  and what distinguishes it from other agents in the ecosystem
- **Role Within the Agent Ecosystem**: How it relates to the orchestrator and other agents
- **Core Responsibilities**: Numbered list of 5-8 primary tasks

### 4.3 IO Contract

```xml
<io_contract>
- INPUT: A research topic/target and optional focus areas, provided by the orchestrate workflow
- OUTPUT: A single markdown file at the designated output path
- DEFAULT OUTPUT_PATH: .research/{RUN_ID}/agents/{agent-name}.md
- FORMAT: Structured markdown following the output_format specification below
- CONTEXT_BRIEF: If file `context-brief.md` exists in the run directory, read it and
  integrate previous findings. This is a continuation run — build on prior work,
  do not repeat it
</io_contract>
```

### 4.4 Philosophy Block

- Research methodology specific to this role
- Depth expectations (what "thorough" means for this agent)
- Quality standards
- Relationship with evidence and sources

### 4.5 Investigation Techniques

- Specific commands, tools, and approaches this agent uses
- Organized by technique category
- Include example commands where relevant
- For tech agents: bash commands, file analysis, etc.
- For health agents: literature review approaches, evidence grading, etc.

### 4.6 Output Format

Define the markdown structure the agent must produce:

```markdown
# {Agent Title}: {Topic}

## Executive Summary
...

## {Section 1}
...

## {Section N}
...

## Methodology
...

## Limitations
...
```

### 4.7 Guardrails

MANDATORY — every agent must include these. Adapt the specifics to the domain:

**For tech agents:**
```xml
<guardrails>
- ONLY write to the designated output path — never to system directories
- NEVER execute commands that modify system state (no install, no write outside .research/)
- NEVER use sudo or privilege escalation
- NEVER make network requests to external APIs on behalf of the target
- READ-ONLY analysis — observe, document, do not alter
- If a command could have side effects, skip it and document what you would have checked
- NEVER exceed 30,000 characters in output
- Pre-flight: verify output directory exists before writing
</guardrails>
```

**For health agents, ADD these:**
```xml
- Include medical disclaimer in every output
- Grade evidence quality (systematic review > RCT > cohort > case study > expert opinion)
- Flag sources older than 3 years
- NEVER provide personal health advice, diagnoses, or treatment recommendations
- NEVER include patient data (real or synthetic)
- State jurisdiction explicitly when discussing regulations
- Flag potential conflicts of interest in sources
```

## Step 5 — Write File

1. Verify the output path does NOT already exist
2. Create the file at `plugins/<plugin>/agents/<name>.md`
3. Report the file path and character count
4. If over 25,000 characters, warn that it's approaching the 30K limit

## Step 6 — Update Catalogs

After the agent file is created, offer to update:

1. **Orchestrate workflow** (`plugins/<plugin>/skills/<domain>/workflows/orchestrate.md`):
   - Add the new agent to the agent catalog/selection table
   - Add heuristics for when this agent should be selected

2. **AGENTS.md** (`plugins/<plugin>/AGENTS.md`):
   - Add the agent to the appropriate domain table
   - Update agent count

3. **plugin.json** (`plugins/<plugin>/.claude-plugin/plugin.json`):
   - Update agent count in description

Use `ask_user` before each catalog update:
- "Should I add this agent to the orchestrate workflow catalog?"
- "Should I update AGENTS.md with this new agent?"
- Choices: ["Yes", "No, I'll do it manually"]

## Step 7 — Summary

Present a summary of everything that was created/modified:

```
Agent created successfully:
- File: plugins/<plugin>/agents/<name>.md (<N> characters)
- Catalogs updated: <list>
- Next steps: <suggestions>
```

## Error Handling

| Error | Action |
|-------|--------|
| Plugin directory not found | Ask user for correct path |
| Agent name already exists | Show existing file, ask user to rename or skip |
| File write fails | Report error, do not retry automatically |
| Reference template not found | Use a generic template, warn user about quality |
| Generated file exceeds 30K | Trim, suggest splitting into agent + workflow extension |

## Guardrails

1. Never overwrite existing files without explicit confirmation
2. Never modify files outside the target plugin directory (during creation)
3. Always read a reference template before generating
4. Always confirm understanding before writing
5. Generated agents inherit the domain's guardrails automatically
