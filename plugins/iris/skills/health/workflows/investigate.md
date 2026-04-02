---
name: investigate
description: >
  Main health investigation workflow. Guides the user through topic definition,
  verifies healthcare scope, captures geographic/regulatory jurisdiction, enforces
  evidence-based requirements, and launches the multi-agent health research pipeline.
---

<purpose>
Guide the user through defining a healthcare research investigation, capture all
parameters needed by the health orchestrator, generate the request file with
`domain: health`, and hand off to orchestration.

You are a thinking partner helping the user crystallize what they want to investigate
in the healthcare domain. Your job is to ensure the orchestrator and health researcher
agents have crystal-clear, evidence-oriented direction.

**Healthcare scope verification is mandatory.** Before proceeding, you must confirm
the topic falls within healthcare, medical, clinical, pharmaceutical, public health,
epidemiological, or health-adjacent domains. If the topic is not healthcare-related,
redirect the user to the appropriate skill.
</purpose>

<philosophy>

## User = Requester, Agent = Clarifier

The user knows:
- What health topic they want to learn about
- Why they care about this topic
- What they plan to do with the findings
- Specific questions they want answered

The user may NOT know:
- How to phrase their health investigation precisely
- What aspects are researchable vs speculative in healthcare
- What the health research agents can and cannot discover
- How to scope the investigation for best results
- Which regulatory jurisdictions are relevant
- What evidence levels exist (systematic review > RCT > cohort > case > expert opinion)

**Your job:** Help the user articulate their intent clearly, not assume it.

## Confirm Everything

After every substantive user input:
1. Restate what you understood in your own words
2. Highlight any assumptions you are making
3. Ask for explicit confirmation before proceeding
4. If the user corrects you, restate again until confirmed

This is NOT optional. Every step requires confirmation.

## Scope Control

Health research investigations work best when focused. If the user asks for something
too broad, help them narrow it:

**Too broad:** "Investigate diabetes"
**Better:** "Investigate current evidence for GLP-1 receptor agonists in type 2 diabetes management: efficacy, safety, and cost-effectiveness"

**Too broad:** "Research mental health treatments"
**Better:** "Compare CBT vs pharmacotherapy for treatment-resistant depression in adults: clinical outcomes, adherence rates, and long-term relapse data"

**Too broad:** "Look into vaccines"
**Better:** "Evaluate mRNA vaccine platform safety data for respiratory pathogens: adverse event profiles, long-term follow-up, and regulatory approval timelines"

## Evidence-Based Framing

When helping the user refine their topic, guide them toward questions that can be
answered with evidence:

- Prefer questions about efficacy, safety, mechanisms, outcomes, guidelines
- Steer away from questions that require clinical judgment or personalized advice
- Remind the user that all findings will be graded by evidence level
- Clarify that the investigation is informational, NOT medical advice

</philosophy>

<process>

<step name="check_existing_research" priority="first">

Before anything else, check if existing research request files exist:

```bash
ls -la .requests/research-*.md 2>/dev/null
```

**If matching request files are found:**

Use `ask_user` to present options:
- question: "Existing health research investigation(s) found in `.requests/`. What would you like to do?"
- choices:
  - "Start a new investigation" -- creates a fresh request and run
  - "Continue an existing investigation" -- adds a new research run to an existing request
  - "View existing investigations first" -- shows current state before deciding

**If "Start a new investigation":**
Proceed to the `gather_topic` step. A new request file and run folder will be created.

**If "Continue an existing investigation":**
List the existing request files with their topics:
```bash
for f in .requests/research-*.md; do
  echo "---"
  echo "File: $f"
  head -20 "$f" | grep -E "^(name|topic|subject|status|domain):"
done
```
Let the user pick which investigation to continue, then delegate to
`../../research/workflows/continue.md`, passing the selected `REQUEST_FILE`.
EXIT this workflow after delegation.

**If "View existing investigations first":**
```bash
for f in .requests/research-*.md; do
  echo "---"
  echo "File: $f"
  head -20 "$f" | grep -E "^(name|topic|subject|status|jurisdiction|domain):"
done
```
Then return to the question.

</step>

<step name="gather_topic">

Ask the user what they want to investigate.

Use `ask_user`:
- question: "What health topic would you like to investigate? Please describe the healthcare subject you want to research and any specific questions you want answered.\n\n*Examples: disease mechanisms, treatment comparisons, drug safety profiles, public health interventions, regulatory pathways, epidemiological trends, health policy analysis.*"
- allow_freeform: true

**After receiving the response:**

1. Parse the user's input for:
   - **Primary subject** — what is being investigated (disease, treatment, drug, device, policy, etc.)
   - **Specific aspects** — what dimensions interest them (clinical, regulatory, economic, epidemiological, etc.)
   - **Key questions** — what they want answered
   - **Context** — why they care, what they will do with the results

2. Restate your understanding using `ask_user`:
   - question: "Here is what I understood from your request:\n\n**Subject:** {subject}\n**Focus areas:** {aspects}\n**Key questions:**\n{numbered questions}\n\nIs this correct?"
   - choices:
     - "Yes, that is correct"
     - "Partially correct, let me clarify"
     - "No, let me rephrase"

3. If not confirmed, iterate until the user confirms.

</step>

<step name="verify_healthcare_scope">

After confirming the topic, verify it falls within the healthcare domain.

**Healthcare scope includes:**
- Diseases, conditions, disorders (any ICD-classified condition)
- Treatments, therapies, interventions (pharmacological, surgical, behavioral, complementary)
- Drugs, pharmaceuticals, biologics, vaccines
- Medical devices and diagnostics
- Public health interventions and policy
- Epidemiology and population health
- Health systems, financing, and delivery
- Clinical trials and research methodology
- Regulatory frameworks (FDA, EMA, WHO, national health authorities)
- Bioethics and research ethics
- Health informatics and digital health
- Environmental and occupational health
- Nutrition and lifestyle medicine
- Genomics, precision medicine, and biomarkers
- Mental health and behavioral health
- Maternal, child, and reproductive health
- Rehabilitation and disability

**If the topic is NOT within healthcare scope:**
- Inform the user: "This topic appears to be outside the healthcare domain. The health research skill is designed for medical, clinical, pharmaceutical, public health, and health-related investigations."
- Suggest the appropriate skill (e.g., tech research for technology topics)
- EXIT workflow

**If the topic is borderline** (e.g., health-adjacent technology, fitness, wellness):
Use `ask_user`:
- question: "Your topic touches on health but also extends into other domains. Would you like to:\n\n1. Proceed with a health-focused investigation (emphasizing clinical evidence, regulatory aspects, and health outcomes)\n2. Reframe the investigation to focus purely on the health dimension"
- choices:
  - "Proceed with health focus"
  - "Let me reframe the topic"

</step>

<step name="gather_jurisdiction">

Ask about geographic and regulatory jurisdiction.

Use `ask_user`:
- question: "Health research is often jurisdiction-dependent (e.g., FDA vs EMA approval status, national treatment guidelines, insurance coverage). Is there a specific geographic or regulatory jurisdiction you want this investigation to focus on?"
- choices:
  - "United States (FDA, CMS, CDC)"
  - "European Union (EMA, ECDC)"
  - "United Kingdom (MHRA, NICE, NHS)"
  - "Global / WHO perspective"
  - "Latin America (PAHO, ANVISA, COFEPRIS)"
  - "Multiple jurisdictions (specify)"
  - "No specific jurisdiction — general evidence"

If the user selects "Multiple jurisdictions" or provides custom input, capture the specifics.

</step>

<step name="determine_agents">

Based on the confirmed topic, determine which health research agents are relevant.

**Agent selection logic:**

| If the subject is... | Agents to use |
|---------------------|--------------|
| Disease/condition investigation | clinical-evidence, epidemiological, pharmaceutical, clinical-trials, comparative, patient-safety |
| Drug/treatment research | pharmaceutical, clinical-trials, regulatory, clinical-evidence, economics, patient-safety |
| Medical device evaluation | medical-device, regulatory, patient-safety, clinical-trials, economics |
| Public health topic | public-health, epidemiological, environmental, equity, policy, infectious-disease |
| Health policy/systems | policy, economics, compliance, equity, informatics, comparative |
| Mental health topic | mental-health, clinical-evidence, pharmaceutical, rehabilitation, public-health |
| Regulatory/compliance | regulatory, compliance, bioethics, policy, clinical-trials |
| Digital health/telemedicine | telemedicine, informatics, regulatory, trend, compliance |
| Genomics/precision medicine | genomics, pharmaceutical, clinical-trials, trend, bioethics |
| Full evaluation | ALL 25 agents |

Present the selected agents to the user:

Use `ask_user`:
- question: "Based on your topic, I recommend using these health research agents:\n\n{for each agent: name + one-line description + why relevant}\n\nWould you like to adjust the agent selection?"
- choices:
  - "Proceed with these agents"
  - "Add/remove agents"

</step>

<step name="gather_language">

Ask the user for their preferred synthesis language.

Use `ask_user`:
- question: "The health research agents always write their findings in English. In what language would you like the final synthesized report?"
- choices:
  - "English"
  - "Spanish (Español)"
  - "Portuguese (Português)"
  - "French (Français)"
  - "Other (specify)"

</step>

<step name="evidence_requirements">

Remind the user about the evidence-based framework.

Use `ask_user`:
- question: "This investigation will follow strict evidence-based methodology:\n\n• **Sources**: Only official health organizations (WHO, FDA, EMA, NIH, CDC) + peer-reviewed literature (PubMed, Cochrane, NEJM, Lancet) + university research with DOI\n• **Evidence grading**: Every finding classified (systematic review > RCT > cohort > case study > expert opinion)\n• **Date sensitivity**: Sources older than 3 years will be flagged\n• **Mandatory disclaimer**: All outputs include 'This is research output, NOT medical advice'\n• **No prescriptions**: The investigation will never recommend dosages, treatments, or diagnoses\n\nDo you understand and accept these parameters?"
- choices:
  - "Yes, proceed with evidence-based methodology"
  - "I have questions about this framework"

If the user has questions, answer them before proceeding.

</step>

<step name="confirm_investigation">

Present a complete summary before launching:

Use `ask_user`:
- question: "Here is the complete health investigation plan:\n\n**Topic:** {topic}\n**Subject:** {subject}\n**Focus Areas:** {aspects}\n**Key Questions:**\n{questions}\n**Jurisdiction:** {jurisdiction}\n**Research Agents:** {agent list}\n**Synthesis Language:** {language}\n**Evidence Framework:** Peer-reviewed sources only, graded by evidence level\n\nShall I proceed with this investigation?"
- choices:
  - "Yes, launch the investigation"
  - "I want to make changes"
  - "Cancel"

</step>

<step name="generate_request_file">

Generate the request file and initial run folder:

```bash
# Generate full GUID as request ID
REQUEST_ID=$(uuidgen | tr '[:upper:]' '[:lower:]')
SHORT_ID=$(echo "$REQUEST_ID" | cut -c1-8)
REQUEST_FILE=".requests/research-${SHORT_ID}.md"
mkdir -p .requests

# Generate run UUID and create run folder
RUN_ID=$(uuidgen | tr '[:upper:]' '[:lower:]' | cut -c1-8)
RUN_DIR=".research/${RUN_ID}"
mkdir -p "${RUN_DIR}/agents"
```

Create the request file with this structure:

```markdown
---
name: "{descriptive name}"
topic: "{research topic}"
subject: "{primary subject}"
language: "{synthesis language}"
domain: health
jurisdiction: "{geographic/regulatory jurisdiction}"
evidence-requirements: "{evidence level preference, e.g. Level I-III preferred}"
status: researching
created: "{ISO 8601 datetime}"
updated: "{ISO 8601 datetime}"
request-id: "{REQUEST_ID}"
research-runs:
  - id: {RUN_ID}
    status: researching
    date: "{ISO 8601 datetime}"
    agents: [health-{agent-1}, health-{agent-2}]
focus-areas:
  - "{area 1}"
  - "{area 2}"
agents:
  - health-{agent-1}
  - health-{agent-2}
  - health-{agent-3}
---

# Health Research Request: {descriptive name}

## Topic
{Full topic description}

## Subject
{Primary subject being investigated}

## Jurisdiction
{Geographic/regulatory context and why it matters for this investigation}

## Focus Areas
{Numbered list of focus areas}

## Key Questions
{Numbered list of specific questions to answer}

## Evidence Requirements
- Sources: Official health organizations + peer-reviewed literature with DOI
- Grading: Systematic review > RCT > cohort study > case study > expert opinion
- Date threshold: Flag sources older than 3 years
- Conflict of interest: Note industry-funded studies
- Retraction check: Verify cited papers are not retracted

## Agent Configuration
{For each agent: name, focus area, output path in RUN_DIR/agents/}

## Parameters
- Domain: Health
- Jurisdiction: {jurisdiction}
- Synthesis Language: {language}
- Research Language: English (always)
- Run Directory: {RUN_DIR}
- Agent Output Directory: {RUN_DIR}/agents/
```

Create `meta.json` inside the run folder:

```bash
cat > "${RUN_DIR}/meta.json" << 'METAEOF'
{
  "topic": "{topic}",
  "subject": "{subject}",
  "domain": "health",
  "jurisdiction": "{jurisdiction}",
  "date": "{ISO 8601 datetime}",
  "agents": ["health-{agent-1}", "health-{agent-2}"],
  "type": "initial"
}
METAEOF
```

</step>

<step name="handoff_to_orchestration">

After the request file and run folder are created, transition to the orchestration workflow.

**Do NOT invoke any agent directly.** Instead, execute the orchestration workflow
directly. The orchestration logic lives in the `orchestrate` workflow of this skill
(`workflows/orchestrate.md`).

**Execution:** Read and follow all steps defined in the `orchestrate` workflow, passing
the required variables as context. The orchestrate workflow will:
1. Read the request file
2. Use the run directory already created
3. Transition status to `researching`
4. Spawn all selected health researcher agents in parallel
5. Monitor their completion
6. Transition status to `synthesizing`
7. Spawn the health-synthesizer agent
8. Finalize the status to `completed`
9. Report results to the user (including mandatory disclaimer)

**Variables to pass to orchestrate workflow:**
- `REQUEST_FILE`: Full path to the request file (e.g., `.requests/research-550e8400.md`)
- `RUN_ID`: The 8-character run UUID (e.g., `b2c3d4e5`)
- `RUN_DIR`: Full path to the run directory (e.g., `.research/b2c3d4e5`)

Inform the user before starting orchestration:
```
ask_user:
  question: "Request file generated at `{REQUEST_FILE}`. Run folder created at
  `{RUN_DIR}`. Starting the orchestration phase -- I will now spawn the health
  research agents and coordinate the investigation. This may take a few minutes.
  Proceed?"
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
- `.requests/*.md` (new request files)
- `.research/*/` (run folders: agents/, meta.json, SUMMARY.md)

### Rule 2: No Research Execution
This workflow does NOT perform research. It gathers parameters and hands off.

### Rule 3: Confirmation Required
EVERY step that uses user input MUST confirm understanding before proceeding.
Use `ask_user` with clear restatements.

### Rule 4: No Destructive Surprises
Before deleting or archiving anything, ALWAYS get explicit user confirmation.

### Rule 5: No Privilege Escalation
NEVER use `sudo`, `su`, `doas`.

## MANDATORY HEALTHCARE GUARDRAILS

### HC-1: Verified Sources Only
ONLY official health organizations (WHO, FDA, EMA, NIH, CDC, PAHO) + peer-reviewed
research (PubMed, Cochrane, NEJM, Lancet) + university research with DOI.

### HC-2: Mandatory Disclaimer
Every output must include: "This is research output, NOT medical advice. Consult a
healthcare professional."

### HC-3: Evidence Grading
Classify every finding: systematic review > RCT > cohort study > case study > expert opinion.

### HC-4: Date Sensitivity
Note publication date of every source; flag if older than 3 years.

### HC-5: No Prescriptions
Never recommend dosages, treatments, or diagnoses.

### HC-6: No Personal Health Advice
Research is informational, never prescriptive.

### HC-7: Explicit Jurisdiction
Always clarify geographic and regulatory context.

### HC-8: Conflict of Interest
Note if studies were industry-funded.

### HC-9: Retraction Awareness
Verify cited papers have not been retracted or corrected.

### HC-10: Bias Detection
Flag methodological limitations (sample size, study design, population).

### HC-11: Zero Patient Data
Never include patient-identifiable information.

</guardrails>
