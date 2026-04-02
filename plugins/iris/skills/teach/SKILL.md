---
name: teach
description: IRIS Teach skill. USE WHEN the user wants to learn about a topic or generate educational material. Guides users through defining a topic, configuring learning parameters (level, audience, style), and launching 6 specialized pedagogical agents to create adaptive educational content. Independent from the research router.
tools: Read, Write, Edit, Bash, Grep, Glob, Task
---

# IRIS Teach

Educational content generation skill. This is the domain-specific skill for creating
learning materials on ANY topic: history, geography, science, mathematics, philosophy,
arts, languages, technology, health, and any other subject. It guides users through
topic definition, learning parameter configuration, and launches an orchestrated
investigation with 6 specialized pedagogical agents.

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **Investigate** | "teach", "learn", "aprender", "educate" | `workflows/investigate.md` |
| **Orchestrate** | (internal — invoked by investigate) | `workflows/orchestrate.md` |
| **Continue** | "continue", "deepen" | `../research/workflows/continue.md` (shared) |
| **Status** | "teach status", "check learning" | `../research/workflows/status.md` (shared) |
| **Archive** | "archive teach material" | `../research/workflows/archive.md` (shared) |

## User Communication Guide

The teach domain targets non-technical users (students, educators, general public).
All user-facing messages MUST use humanized language. Internal processing, variables,
file paths, and metadata continue to use technical terms.

**Why:** A student asking "teach me about the French Revolution" should never see words
like "agent", "orchestrate", "spawn", or "workflow". These are implementation details
that would confuse or alienate non-technical users.

**Rule:** Use humanized terms ONLY in `ask_user` messages and direct responses to the
user. Variables, YAML keys, file paths, code, and internal logic ALWAYS use technical
terms (agent, spawn, orchestrate, workflow).

| Internal Term | User-Facing Term | Example |
|---------------|-----------------|---------|
| agent | specialist | "6 specialists will work on your topic" |
| spawn agents | put specialists to work | "I'll put the team to work now" |
| agents completed | specialists finished | "4 specialists have finished their work" |
| agents still running | specialists still working | "2 specialists are still working" |
| agent failed | specialist couldn't complete | "One specialist couldn't finish their task" |
| orchestrate | coordinate | Never shown to user directly |
| workflow | (invisible) | Never mentioned to user |
| synthesizer | (part of "the team") | Never named individually to user |
| wait gate | waiting for the team | "Waiting for all specialists to finish" |
| request file | learning session | "I found a previous learning session" |
| investigation | learning session | "Starting your learning session" |
| research run | session run | Internal only |

## How It Works

1. **User requests to learn about a topic**
2. **Investigate workflow** activates:
   - Checks for existing investigations (offers to continue or start new)
   - Gathers topic, education level, target audience, learning style
   - Confirms understanding with the user
   - Generates a request file in `.requests/` with GUID-based ID
   - Creates a UUID run folder in `.research/{uuid}/agents/`
   - Transitions to the orchestration phase
3. **Orchestration phase:**
   - Puts 6 specialists to work in parallel
   - Monitors completion (mandatory wait — never skips anyone)
   - Triggers the final synthesis
   - Status tracked in request file (`researching` → `synthesizing` → `completed`)
4. **Results appear** in `.research/{uuid}/SUMMARY.md` as a complete educational document

## Specialist Team (6)

| Agent | Role | Specialization |
|-------|------|---------------|
| `teacher-researcher` | Investigator | Gathers information on any topic |
| `teacher-curriculum-designer` | Curriculum architect | Structures learning path with prerequisites and progression |
| `teacher-content-creator` | Content producer | Creates explanations, analogies, examples, visual aids |
| `teacher-assessor` | Evaluator | Designs exercises, quizzes, Socratic questions, rubrics |
| `teacher-fact-checker` | Credibility guardian | Validates sources, experts, detects biases, flags misinformation |
| `teacher-synthesizer` | Synthesizer | Combines all outputs into the final educational document |

## Educational Parameters

| Parameter | Values | Description |
|-----------|--------|-------------|
| `education-level` | basic, intermediate, advanced | Complexity depth |
| `target-audience` | children, teenagers, adults, professionals | Tone and vocabulary adaptation |
| `learning-style` | visual, textual, practical, mixed | Content presentation style |
| `include-exercises` | yes, no | Whether to include assessment materials |
| `language` | any language | Language for the final document |

## Cumulative Research

Educational investigations support multiple runs for iterative deepening:
- Each run creates a new UUID folder under `.research/`
- The request file tracks all runs in `learning-runs`
- Use `/iris:research continue` to deepen an existing investigation
- The continue workflow reads previous SUMMARY.md and injects context into new agents

## Principles

### Conversational Refinement
Never assume you understand the topic. Always:
1. Restate what you understood
2. Ask for confirmation using `ask_user`
3. Only proceed after explicit user approval

### Progressive Disclosure
Start broad, then refine:
1. What do you want to learn about?
2. What is your current knowledge level?
3. Who is the target audience?
4. What specific aspects interest you most?
5. What language should the material be in?

### Educational Quality
- Every factual claim must have an identifiable source
- Controversial topics include multiple perspectives
- Content adapts to the specified education level and audience
- Assessment materials align with learning objectives

### Request File Contract
Every investigation produces a request file in `.requests/` with:
- Full frontmatter (name, topic, language, status, education-level, target-audience, etc.)
- Detailed body describing the educational scope
- Updated by the orchestrator as the investigation progresses

## Context Files
- See [workflows/investigate.md](workflows/investigate.md) for the investigation workflow
- See [workflows/orchestrate.md](workflows/orchestrate.md) for agent orchestration
- See [../research/workflows/continue.md](../research/workflows/continue.md) for deepening investigations
- See [../research/workflows/status.md](../research/workflows/status.md) for checking status
- See [../research/workflows/archive.md](../research/workflows/archive.md) for archival

## Guardrails

This skill and all agents it triggers are **read-only educational tools**.
- The ONLY writable directory is `.research/` and `.requests/`
- No code execution that modifies the system
- No privilege escalation
- No network requests to external APIs on behalf of targets
- No destructive filesystem operations
- Every factual claim requires an identifiable source
- Controversial topics must include multiple perspectives
- Medical/legal/financial topics include appropriate disclaimers
