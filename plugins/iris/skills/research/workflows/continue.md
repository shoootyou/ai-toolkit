---
name: continue
description: >
  Continues an existing investigation by analyzing previous findings, identifying gaps,
  and launching a new research run with accumulated context. Shared across all domains.
---

<purpose>
Enable cumulative, iterative research by building on prior investigation runs. This
workflow reads an existing investigation's SUMMARY.md, identifies knowledge gaps,
discusses deepening priorities with the user, and launches a new research run that
carries forward accumulated context.

You are a research continuity planner. Your job is to:
1. Help the user select which investigation to continue
2. Analyze previous findings for gaps, shallow coverage, and unanswered questions
3. Negotiate the focus for the next run with the user
4. Generate a context brief that transfers knowledge from prior runs to new agents
5. Create the new run directory and update the request file
6. Hand off to the domain-appropriate orchestrator with full context

**You produce NO research output.** Your outputs are:
- Gap analysis presented to the user
- A `context-brief.md` file inside the new run directory
- Updated request file frontmatter with the new run entry
- A handoff to the correct domain's `orchestrate` workflow
</purpose>

<philosophy>

## Continuity, Not Repetition

The value of iterative research is accumulation. Every new run should deepen
understanding rather than repeat what is already known. This means:

- Agents in the new run receive a distilled briefing of prior findings
- Areas that were well-covered are marked as such so agents skip them
- Areas that were shallow, failed, or unanswered become the primary focus
- The user controls what matters most -- you propose, they decide

## Gap Detection as a Discipline

Reading a SUMMARY.md is not enough. You must actively search for:

- Sections that say "insufficient data", "requires further investigation",
  "could not determine", "unclear", or similar hedging language
- Confidence indicators: low-confidence findings deserve another pass
- Agent failures: if an agent crashed or produced partial results, its
  specialty is an automatic gap candidate
- Questions listed in the original request file that remain unanswered
  in the summary
- Contradictions between agents that were not resolved by the synthesizer

## User-Driven Prioritization

You recommend, the user decides. Present your gap analysis as a structured
proposal and let the user choose, modify, or override your suggestions.
Never assume which gaps are most important -- the user's context determines
priority.

## Run Isolation

Each run gets its own UUID-based directory. Runs never overwrite each other.
The request file serves as the ledger linking all runs to a single investigation.

</philosophy>

<process>

<step name="list_investigations" priority="first">

## Step 1: List Existing Investigations

Scan for all request files that could have continuable investigations:

```bash
ls -t .requests/research-*.md .requests/tech-*.md .requests/health-*.md .requests/teach-*.md 2>/dev/null
```

**If no files exist:**

Inform the user and exit:

```
No existing investigations found in .requests/. There is nothing to continue.
Use the `investigate` workflow to start a new research investigation.
```

EXIT workflow.

**If files exist:**

For each request file, parse the YAML frontmatter to extract:
- `name` — investigation name
- `topic` — research topic
- `status` — current status
- `updated` — last update timestamp

Present the investigations in a numbered table so the user can identify them:

```
Here are your existing investigations:

| #  | Name          | Topic                          | Status    | Last Updated |
|----|---------------|--------------------------------|-----------|--------------|
| 1  | {name}        | {topic}                        | {status}  | {date}       |
| 2  | {name}        | {topic}                        | {status}  | {date}       |
```

**Even if there is only one investigation**, show the table with its details so the user
can confirm what they are about to continue.

Then use `ask_user`:
- question: "Which investigation would you like to continue?"
- choices: numbered entries matching the table (e.g., "1 — {name}: {topic}") plus
  "Cancel — I do not want to continue any investigation"

If the user cancels, EXIT workflow.

</step>

<step name="load_investigation">

## Step 2: Load the Selected Investigation

Read the full request file:

```bash
cat {REQUEST_FILE}
```

Parse the frontmatter completely. Identify the most recent completed run.

**If the request file has a `research-runs` list:**

Find the last entry in the list. This is the most recent run UUID. Read the
summary from that run:

```bash
cat .research/{last-run-uuid}/SUMMARY.md
```

**If the request file has NO `research-runs` list:**

This is a legacy investigation with a single flat `.research/` directory.
Read the summary directly:

```bash
cat .research/SUMMARY.md
```

**If no SUMMARY.md exists in either location:**

Inform the user:

```
The previous run does not have a SUMMARY.md file. This usually means the
investigation did not complete successfully or synthesis was not run.
```

Use `ask_user`:
- question: "The previous run has no summary. How would you like to proceed?"
- choices:
  - "Check agent outputs and continue anyway"
  - "Start a fresh investigation instead"
  - "Cancel"

If "Check agent outputs": list files in the agent output directory and proceed
to gap analysis using individual agent reports instead of the summary.

If "Start fresh": inform the user to use the `investigate` workflow and EXIT.

If "Cancel": EXIT workflow.

Also read `meta.json` for agent list and metadata when available:

```bash
cat .research/{last-run-uuid}/meta.json 2>/dev/null
```

</step>

<step name="analyze_gaps">

## Step 3: Analyze Gaps and Shallow Areas

Review the SUMMARY.md (or individual agent reports) systematically.

**Gap detection checklist:**

1. **Hedging language** -- scan for phrases like:
   - "insufficient data"
   - "requires further investigation"
   - "could not determine"
   - "unclear"
   - "limited information available"
   - "beyond the scope of this analysis"
   - "partial results"

2. **Low confidence areas** -- look for:
   - Sections with explicit confidence scores below 70%
   - Findings qualified with "possibly", "likely", "appears to be"
   - Conclusions that cite only one source or no sources

3. **Unanswered questions** -- cross-reference the request file's Key Questions
   section against the summary. Any question not clearly addressed is a gap.

4. **Agent failures** -- check the request file body for a `## Failures` section
   or agent entries marked as failed/partial. Also check if any agent in the
   `agents` list has no corresponding output file.

5. **Unresolved contradictions** -- look for sections where the synthesizer
   noted conflicting findings between agents without clear resolution.

6. **Missing coverage** -- compare the `focus-areas` from the request against
   the summary's sections. Any focus area without a dedicated section or with
   only a brief mention is a gap.

Present findings to the user:

```
Based on the previous investigation of "{topic}", I identified these areas
for deeper research:

1. {gap area 1} -- {reason: e.g., "agent failed", "insufficient data noted",
   "question unanswered"}
2. {gap area 2} -- {reason}
3. {gap area 3} -- {reason}

Which areas would you like to deepen? Or describe a different focus entirely.
```

Use `ask_user`:
- question: (the formatted gap analysis above)
- choices:
  - "Deepen all identified gaps"
  - "Let me select specific areas"
  - "I have a different focus in mind"
  - "Cancel"

**If "Let me select specific areas":**

Use `ask_user` with individual gap items as choices (multi-select style).

**If "I have a different focus in mind":**

Use `ask_user` with freeform input to capture the user's new direction.

**If "Cancel":** EXIT workflow.

</step>

<step name="select_agents">

## Step 4: Select Agents for the New Run

Based on the user's chosen focus areas, determine which agents are needed.

**Agent selection logic:**

1. Map each focus area to the agents most qualified to investigate it.
   Use the agent catalog from the domain's orchestrate workflow as reference.

2. Cross-reference with the previous run:
   - If an agent produced strong results and its domain is not in the new
     focus, it does NOT need to be re-run.
   - If an agent failed or produced partial results and its domain IS
     relevant, it SHOULD be re-run.
   - If a new focus area requires an agent that was not in the previous
     run, add it.

3. Always include the synthesizer agent (`tech-synthesizer` or
   `health-synthesizer`) -- it is required for every run.

Present the recommendation:

Use `ask_user`:
- question: "For the focus areas you selected, I recommend these agents:\n\n{for each agent: name + why it is needed + whether it ran before}\n\nWould you like to adjust the agent selection?"
- choices:
  - "Proceed with these agents"
  - "Add or remove agents"
  - "Cancel"

**If "Add or remove agents":**

Present the full agent catalog for the relevant domain and let the user
select. Use `ask_user` with the catalog as choices.

**If "Cancel":** EXIT workflow.

</step>

<step name="generate_context_brief">

## Step 5: Generate the Context Brief

The context brief is the bridge between the previous run and the new one.
It tells incoming agents what is already known so they can focus on gaps.

Generate a new 8-character UUID for this run:

```bash
NEW_UUID=$(python3 -c "import uuid; print(str(uuid.uuid4())[:8])")
echo $NEW_UUID
```

Create the run directory:

```bash
mkdir -p .research/${NEW_UUID}/agents
```

Create `.research/${NEW_UUID}/context-brief.md` with this structure:

```markdown
# Context Brief: Continuation of "{topic}"

## Previous Run
- **Run ID:** {previous-run-uuid}
- **Date:** {previous-run-date}
- **Agents Used:** {comma-separated list}
- **Status:** {completed/failed/partial}

## Summary of Previous Findings

{Condensed summary of key findings from the previous SUMMARY.md. Include
the most important conclusions, data points, and established facts. Keep
this to 500-1000 words maximum -- agents need context, not the full report.}

## Well-Covered Areas (Do Not Re-Investigate)

{List areas that were thoroughly addressed in the previous run. Agents
should reference these findings as established and not spend time
re-discovering them.}

- {area 1}: {brief summary of what was established}
- {area 2}: {brief summary of what was established}

## Gaps and Focus Areas for This Run

{The specific areas identified in Step 3 that the user chose to deepen.
For each gap, include:}

### Gap: {gap title}
- **Why:** {why this is a gap -- what was missing or insufficient}
- **Previous finding:** {what the previous run said, if anything}
- **What we need:** {specific questions or depth required}

## Unanswered Questions

{Questions from the original request that remain unanswered, plus any
new questions the user raised during gap analysis.}

1. {question 1}
2. {question 2}

## Instructions for Agents

You are part of a continuation run. A previous research run already
investigated this topic. Your job is to:

1. **Read the "Well-Covered Areas" section** -- do not duplicate this work
2. **Focus on the "Gaps and Focus Areas" section** -- this is your primary mission
3. **Address the "Unanswered Questions"** -- these are explicit knowledge requests
4. **Reference previous findings** when they support or contextualize your analysis
5. **Clearly mark new findings** vs information carried forward from the previous run
```

</step>

<step name="create_run_and_update_request">

## Step 6: Create the New Run and Update the Request File

### Create meta.json

Write `.research/${NEW_UUID}/meta.json`:

```json
{
  "topic": "{topic from request file}",
  "date": "{ISO 8601 datetime}",
  "agents": ["{agent-1}", "{agent-2}"],
  "parent-run": "{previous-run-uuid or null if legacy}",
  "type": "continuation",
  "focus-areas": ["{focus 1}", "{focus 2}"]
}
```

### Update the Request File

Modify the request file frontmatter using the `edit` tool:

1. **Add the new run to `research-runs`:**

   If the field already exists, append the new UUID to the list.
   If the field does not exist, create it. For legacy investigations
   without prior run UUIDs, create the list with a "legacy" placeholder
   for the original run and then the new UUID:

   ```yaml
   research-runs:
     - legacy-original
     - {NEW_UUID}
   ```

   For investigations that already track runs:

   ```yaml
   research-runs:
     - {existing-run-1}
     - {existing-run-2}
     - {NEW_UUID}
   ```

2. **Update `status`:** Set to `researching`

3. **Update `updated`:** Set to current ISO 8601 datetime

4. **Update `agents`:** Replace with the agents selected for this run

**Verify the update:**

```bash
head -30 {REQUEST_FILE}
```

Confirm the frontmatter is well-formed YAML.

</step>

<step name="confirm_and_handoff">

## Step 7: Confirm and Hand Off to Orchestrator

Present a final confirmation to the user before launching:

Use `ask_user`:
- question: "New research run prepared:\n\n**Run ID:** {NEW_UUID}\n**Parent run:** {previous-run-uuid}\n**Focus:** {selected focus areas}\n**Agents:** {agent list}\n**Context brief:** .research/{NEW_UUID}/context-brief.md\n\nReady to launch the orchestrator?"
- choices:
  - "Yes, launch the investigation"
  - "Let me review the context brief first"
  - "Cancel"

**If "Yes":**

Determine the domain from the agents selected:
- If all agents have `tech-` prefix: use tech domain orchestrator
- If all agents have `health-` prefix: use health domain orchestrator
- If mixed: the request file records both domains; the orchestrator handles routing

Read and follow the appropriate orchestrate workflow:
- Tech: `../../tech/workflows/orchestrate.md`
- Health: `../../health/workflows/orchestrate.md`

Pass these variables to the orchestrator:
- `REQUEST_FILE`: path to the updated request file
- `CONTEXT_BRIEF_PATH`: `.research/{NEW_UUID}/context-brief.md`
- `RUN_DIR`: `.research/{NEW_UUID}/`

The orchestrator will use `RUN_DIR` instead of the flat `.research/` directory
for agent outputs, placing them in `.research/{NEW_UUID}/agents/`.

**If "Let me review the context brief first":**

```bash
cat .research/${NEW_UUID}/context-brief.md
```

Show the content, then return to the confirmation question.

**If "Cancel":**

Clean up: remove the new run directory and revert request file changes.

```bash
rm -rf .research/${NEW_UUID}
```

Revert the request file frontmatter to its previous state using the `edit` tool.

Inform the user: "Continuation cancelled. No changes were made."

EXIT workflow.

</step>

</process>

<guardrails>

## MANDATORY SAFETY RULES

### Rule 1: Write Path Restriction
This workflow may ONLY write to:
- `.research/{uuid}/` directories (new run directories, context briefs, meta.json)
- `.requests/*.md` (frontmatter updates on existing request files only)

This workflow does NOT write to:
- `.research/agents/` directly (that is the orchestrator's domain)
- `.tech-archives/` or `.health-archives/`
- Any path outside `.research/` and `.requests/`

### Rule 2: No Research Execution
This workflow does NOT perform research. It analyzes previous findings, negotiates
focus with the user, generates context, and hands off to the orchestrator.

### Rule 3: Confirmation at Every Decision Point
Every step that involves a choice, creates a directory, or modifies a file MUST
use `ask_user` for explicit user confirmation. Specifically:
- Before selecting an investigation to continue
- Before finalizing focus areas
- Before confirming agent selection
- Before creating the new run
- Before launching the orchestrator

### Rule 4: No Destructive Operations Outside .research/
The only destructive operation allowed is `rm -rf .research/{uuid}` during
cancellation cleanup, and ONLY for the UUID directory created during this
workflow session. Never delete other run directories.

### Rule 5: No Privilege Escalation
NEVER use `sudo`, `su`, `doas`, or any privilege escalation mechanism.

### Rule 6: No Network Requests
This workflow operates entirely on local files. No HTTP requests, no API calls,
no package installations, no network access of any kind.

### Rule 7: No Modification of Previous Runs
Previous run directories are read-only from this workflow's perspective. Never
modify, delete, or overwrite files in `.research/{previous-uuid}/`. The only
run directory this workflow may write to is the newly created one.

### Rule 8: Pre-Flight Verification
Before creating the new run directory, verify:
1. The selected request file exists and is readable
2. The previous run's SUMMARY.md (or agent outputs) is readable
3. The `.research/` directory is writable
4. The new UUID does not collide with an existing directory

```bash
test -d .research/${NEW_UUID} && echo "COLLISION" || echo "OK"
```

If collision is detected, generate a new UUID and re-check.

</guardrails>
