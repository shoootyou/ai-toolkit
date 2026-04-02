---
name: investigate
description: Main investigation workflow. Guides the user through topic definition, parameter configuration, and launches the multi-agent research pipeline.
---

<purpose>
Guide the user through defining a research investigation, capture all parameters needed
by the tech-orchestrator, generate the request file, and hand off to orchestration.

You are a thinking partner helping the user crystallize what they want to investigate.
Your job is to ensure the orchestrator and researcher agents have crystal-clear direction.
</purpose>

<philosophy>

## User = Requester, Agent = Clarifier

The user knows:
- What they want to learn about
- Why they care about this topic
- What they plan to do with the findings
- Specific questions they want answered

The user may NOT know:
- How to phrase their investigation precisely
- What aspects are researchable vs speculative
- What the research agents can and cannot discover
- How to scope the investigation for best results

**Your job:** Help the user articulate their intent clearly, not assume it.

## Confirm Everything

After every substantive user input:
1. Restate what you understood in your own words
2. Highlight any assumptions you are making
3. Ask for explicit confirmation before proceeding
4. If the user corrects you, restate again until confirmed

This is NOT optional. Every step requires confirmation.

## Scope Control

Research investigations work best when focused. If the user asks for something too
broad, help them narrow it:

**Too broad:** "Investigate AI tools"
**Better:** "Investigate the GitHub Copilot CLI binary: architecture, capabilities, and ecosystem"

**Too broad:** "Research JavaScript frameworks"
**Better:** "Compare Next.js and Remix for server-side rendering in production environments"

</philosophy>

<process>

<step name="check_existing_research" priority="first">

Before anything else, check if previous investigations exist:

```bash
ls .requests/research-*.md 2>/dev/null
```

**If request files are found:**

Use `ask_user` to present options:
- question: "Existing research investigations were found in `.requests/`. What would you like to do?"
- choices:
  - "Start a new investigation" -- proceeds with normal flow below
  - "Continue an existing investigation" -- delegates to the continue workflow

**If "Continue an existing investigation":**

Delegate to the shared continue workflow at `../../research/workflows/continue.md`.
Pass control entirely; do not proceed with the remaining steps of this workflow.

**If "Start a new investigation":**

Proceed to the next step (gather_topic).

**If NO request files are found:**

Proceed directly to the next step.

</step>

<step name="gather_topic">

Ask the user what they want to investigate.

Use `ask_user`:
- question: "What topic would you like to investigate? Please describe what you want to research and any specific questions you want answered."
- allow_freeform: true

**After receiving the response:**

1. Parse the user's input for:
   - **Primary subject** -- what is being investigated
   - **Specific aspects** -- what dimensions interest them
   - **Key questions** -- what they want answered
   - **Context** -- why they care, what they will do with the results

2. Restate your understanding using `ask_user`:
   - question: "Here is what I understood from your request:\n\n**Subject:** {subject}\n**Focus areas:** {aspects}\n**Key questions:**\n{numbered questions}\n\nIs this correct?"
   - choices:
     - "Yes, that is correct"
     - "Partially correct, let me clarify"
     - "No, let me rephrase"

3. If not confirmed, iterate until the user confirms.

</step>

<step name="determine_agents">

Based on the confirmed topic, determine which research agents are relevant.

**Agent selection logic:**

| If the subject is... | Agents to use |
|---------------------|--------------|
| A binary/executable/CLI tool | ALL four agents |
| A library/package (no binary) | architecture, documentation, ecosystem |
| A concept/protocol/standard | architecture, documentation, ecosystem |
| A comparison of tools | ecosystem (primary), documentation, architecture |
| A documentation review | documentation (primary), ecosystem |

Present the selected agents to the user:

Use `ask_user`:
- question: "Based on your topic, I recommend using these research agents:\n\n{for each agent: name + one-line description + why relevant}\n\nWould you like to adjust the agent selection?"
- choices:
  - "Proceed with these agents"
  - "Add/remove agents"

</step>

<step name="gather_language">

Ask the user for their preferred synthesis language.

Use `ask_user`:
- question: "The research agents always write their findings in English. In what language would you like the final synthesized report?"
- choices:
  - "English"
  - "Spanish (Espanol)"
  - "Portuguese (Portugues)"
  - "French (Francais)"
  - "Other (specify)"

</step>

<step name="confirm_investigation">

Present a complete summary before launching:

Use `ask_user`:
- question: "Here is the complete investigation plan:\n\n**Topic:** {topic}\n**Subject:** {subject}\n**Focus Areas:** {aspects}\n**Key Questions:**\n{questions}\n**Research Agents:** {agent list}\n**Synthesis Language:** {language}\n\nShall I proceed with this investigation?"
- choices:
  - "Yes, launch the investigation"
  - "I want to make changes"
  - "Cancel"

</step>

<step name="generate_request_file">

Generate a GUID-based request file and a run folder.

**1. Generate identifiers:**

```bash
REQUEST_GUID=$(uuidgen | tr '[:upper:]' '[:lower:]')
REQUEST_SHORT=$(echo "${REQUEST_GUID}" | cut -c1-8)
REQUEST_FILE=".requests/research-${REQUEST_SHORT}.md"

RUN_UUID=$(uuidgen | tr '[:upper:]' '[:lower:]' | cut -c1-8)
RUN_DIR=".research/${RUN_UUID}"
```

**2. Create the run folder and meta.json:**

```bash
mkdir -p .requests
mkdir -p "${RUN_DIR}/agents"
```

Write `meta.json` inside the run folder:

```json
{
  "topic": "{topic}",
  "subject": "{subject}",
  "domain": "tech",
  "date": "{ISO-8601 datetime}",
  "agents": ["{agent-1}", "{agent-2}"],
  "type": "initial"
}
```

**3. Create the request file** with this structure:

```markdown
---
name: "{descriptive name}"
topic: "{research topic}"
subject: "{primary subject}"
language: "{synthesis language}"
domain: tech
status: researching
created: "{ISO 8601 datetime}"
updated: "{ISO 8601 datetime}"
request-id: "{REQUEST_GUID}"
research-runs:
  - id: "{RUN_UUID}"
    status: researching
    date: "{ISO 8601 datetime}"
    agents: [{agent-1}, {agent-2}]
focus-areas:
  - "{area 1}"
  - "{area 2}"
agents:
  - {agent-1}
  - {agent-2}
---

# Research Request: {descriptive name}

## Topic
{Full topic description}

## Subject
{Primary subject being investigated}

## Focus Areas
{Numbered list of focus areas}

## Key Questions
{Numbered list of specific questions to answer}

## Agent Configuration
{For each agent: name, focus area, output path inside RUN_DIR}

## Parameters
- Synthesis Language: {language}
- Research Language: English (always)
- Run Directory: {RUN_DIR}
- Agent Output Directory: {RUN_DIR}/agents/
```

</step>

<step name="handoff_to_orchestration">

After the request file is created, transition to the orchestration workflow.

**Do NOT invoke the `tech-orchestrator` agent.** Instead, execute the orchestration
workflow directly. The orchestration logic lives in the `orchestrate` workflow of this
skill (`.github/skills/analyzer/workflows/orchestrate.md`).

**Execution:** Read and follow all steps defined in the `orchestrate` workflow, passing
the `REQUEST_FILE` path as context. The orchestrate workflow will:
1. Read the request file
2. Prepare the research directory
3. Transition status to `researching`
4. Spawn all selected researcher agents in parallel
5. Monitor their completion
6. Transition status to `synthesizing`
7. Spawn the synthesizer agent
8. Finalize the status to `completed`
9. Report results to the user

**Variables to pass to orchestrate workflow:**
- `REQUEST_FILE`: Full path to the request file
  (e.g., `.requests/research-550e8400.md`)
- `RUN_ID`: The 8-char UUID identifying this run (e.g., `a1b2c3d4`)
- `RUN_DIR`: Path to the run folder (e.g., `.research/a1b2c3d4/`)

The orchestrator uses `RUN_DIR` to set the OUTPUT_PATH for each agent.

Inform the user before starting orchestration:
```
ask_user:
  question: "Request file generated at `{REQUEST_FILE}`. Starting the orchestration
  phase — I will now spawn the research agents and coordinate the investigation.
  This may take a few minutes. Proceed?"
  choices:
    - "Yes, launch the investigation"
    - "Wait, I want to review the request file first"
    - "Cancel"
```

If "Yes": Proceed with the orchestrate workflow steps.
If "Wait": Show the request file content, then return to this choice.
If "Cancel": Set status to `failed` with reason "Cancelled by user" and exit.

</step>

</process>

<guardrails>

## MANDATORY SAFETY RULES

### Rule 1: Write Path Restriction
This workflow may ONLY write to:
- `.requests/research-*.md` (new request files)
- `.research/{uuid}/` (run folders, agents, meta.json)

### Rule 2: No Research Execution
This workflow does NOT perform research. It gathers parameters and hands off.

### Rule 3: Confirmation Required
EVERY step that uses user input MUST confirm understanding before proceeding.
Use `ask_user` with clear restatements.

### Rule 4: No Destructive Surprises
Before deleting or archiving anything, ALWAYS get explicit user confirmation.

### Rule 5: No Privilege Escalation
NEVER use `sudo`, `su`, `doas`.

</guardrails>
