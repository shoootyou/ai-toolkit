---
name: tech
description: IRIS Tech Research skill. USE WHEN investigating software, libraries, CLIs, APIs, frameworks, DevOps tools, or any technology topic. Guides users through defining a tech research topic, configuring parameters, and launching a multi-agent investigation with 27 specialized technology research agents.
tools: Read, Write, Edit, Bash, Grep, Glob, Task
---

# IRIS Tech Research

Software and technology research skill. This is the domain-specific skill for investigating
technology topics: CLI tools, libraries, frameworks, APIs, infrastructure, security, and
any software-related subject. It guides users through topic definition, parameter
configuration, and investigation launch via conversational interaction.

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **Investigate** | "tech research", "investigate software", "analyze tool" | `workflows/investigate.md` |
| **Orchestrate** | (internal — invoked by investigate) | `workflows/orchestrate.md` |
| **Continue** | "continue", "deepen" | `../research/workflows/continue.md` (shared) |
| **Status** | "tech status", "check investigation" | `../research/workflows/status.md` (shared) |
| **Archive** | "archive tech research" | `../research/workflows/archive.md` (shared) |

## How It Works

1. **User invokes IRIS tech research** with a technology topic
2. **Investigate workflow** activates:
   - Checks for existing investigations (offers to continue or start new)
   - Gathers topic through conversational refinement
   - Confirms understanding with the user
   - Generates a request file in `.requests/` with GUID-based ID
   - Creates a UUID run folder in `.research/{uuid}/agents/`
   - Transitions to the `orchestrate` workflow
3. **Orchestrate workflow takes over:**
   - Spawns researcher agents in parallel
   - Monitors completion (mandatory wait gate — never skips agents)
   - Spawns the synthesizer agent
   - Status tracked in request file (`researching` → `synthesizing` → `completed`)
4. **Results appear** in `.research/{uuid}/SUMMARY.md`

## Cumulative Research

Investigations support multiple runs for iterative deepening:
- Each run creates a new UUID folder under `.research/`
- The request file tracks all runs in `research-runs`
- Use `/iris:research continue` to deepen an existing investigation
- The continue workflow reads previous SUMMARY.md and injects context into new agents

## Principles

### Conversational Refinement
Never assume you understand the topic. Always:
1. Restate what you understood
2. Ask for confirmation using `ask_user`
3. Only proceed after explicit user approval

### Progressive Disclosure
Start with the broad topic, then drill into specifics:
1. What do you want to investigate?
2. What specific aspects interest you most?
3. What questions should the investigation answer?
4. What language should the final report be in?

### Request File Contract
Every investigation produces a request file in `.requests/` with:
- Full frontmatter (name, topic, language, status, etc.)
- Detailed body describing the investigation scope
- Updated by the orchestrator as the investigation progresses

## Context Files
- See [workflows/investigate.md](workflows/investigate.md) for the main investigation workflow
- See [../research/workflows/continue.md](../research/workflows/continue.md) for deepening investigations
- See [../research/workflows/status.md](../research/workflows/status.md) for checking investigation status
- See [../research/workflows/archive.md](../research/workflows/archive.md) for archival

## Guardrails

This skill and all agents it triggers are **read-only research tools**.
- The ONLY writable directory is `.research/` and `.requests/`
- No code execution that modifies the system
- No privilege escalation
- No network requests to external APIs on behalf of targets
- No destructive filesystem operations
