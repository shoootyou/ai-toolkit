---
name: health
description: IRIS Health Research skill. USE WHEN investigating healthcare, medical, clinical, pharmaceutical, public health, epidemiological, or any health-related topic. Guides users through defining a health research topic and launching a multi-agent investigation with 25 specialized healthcare research agents. Enforces strict evidence-based guardrails with verified sources only.
tools: Read, Write, Edit, Bash, Grep, Glob, Task
---

# IRIS Health Research

Healthcare research skill. This is the domain-specific skill for investigating
health and medical topics: clinical evidence, pharmaceuticals, epidemiology,
regulatory frameworks, public health, bioethics, health informatics, and any
health-related subject. It guides users through topic definition, parameter
configuration, and investigation launch via conversational interaction.

**All outputs include mandatory disclaimers and evidence grading.**

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **Investigate** | "health research", "investigate disease", "analyze treatment" | `workflows/investigate.md` |
| **Orchestrate** | (internal — invoked by investigate) | `workflows/orchestrate.md` |
| **Continue** | "continue", "deepen" | `../research/workflows/continue.md` (shared) |
| **Status** | "health status", "check investigation" | `../research/workflows/status.md` (shared) |
| **Archive** | "archive health research" | `../research/workflows/archive.md` (shared) |

## How It Works

1. **User invokes IRIS health research** with a healthcare topic
2. **Investigate workflow** activates:
   - Checks for existing investigations (offers to continue or start new)
   - Gathers topic through conversational refinement
   - Verifies the topic is within healthcare scope
   - Confirms understanding with the user
   - Generates a request file in `.requests/` with GUID-based ID
   - Creates a UUID run folder in `.research/{uuid}/agents/`
   - Transitions to the `orchestrate` workflow
3. **Orchestrate workflow takes over:**
   - Spawns health researcher agents in parallel
   - Monitors completion (mandatory wait gate — never skips agents)
   - Spawns the health-synthesizer agent
   - Status tracked in request file (`researching` → `synthesizing` → `completed`)
4. **Results appear** in `.research/{uuid}/SUMMARY.md` with disclaimers and evidence grading

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
1. What health topic do you want to investigate?
2. What specific aspects interest you most? (clinical, regulatory, economic, etc.)
3. What questions should the investigation answer?
4. Is there a specific geographic/regulatory jurisdiction?
5. What language should the final report be in?

### Evidence-Based Research
All health investigations must:
- Source information ONLY from official health organizations and peer-reviewed literature
- Grade every finding by evidence level (systematic review > RCT > cohort > case > expert opinion)
- Note publication dates and flag sources older than 3 years
- Include mandatory disclaimer in every output

### Request File Contract
Every investigation produces a request file in `.requests/` with:
- Full frontmatter (name, topic, language, status, domain: health)
- Detailed body describing the investigation scope
- Updated by the orchestrator as the investigation progresses

## Context Files
- See [workflows/investigate.md](workflows/investigate.md) for the main investigation workflow
- See [workflows/orchestrate.md](workflows/orchestrate.md) for agent orchestration
- See [workflows/archive.md](workflows/archive.md) for the archival workflow
- See [workflows/status.md](workflows/status.md) for checking investigation status

## Healthcare Guardrails

This skill enforces **strict healthcare research guardrails** in addition to standard IRIS guardrails:

### Standard IRIS Guardrails
- The ONLY writable directory is `.research/` and `.requests/`
- No code execution that modifies the system
- No privilege escalation
- No network requests to external APIs on behalf of targets
- No destructive filesystem operations

### Healthcare-Specific Guardrails
1. **Verified sources only** — ONLY official health organizations (WHO, FDA, EMA, NIH, CDC, PAHO) + peer-reviewed research (PubMed, Cochrane, NEJM, Lancet) + university research with DOI
2. **Mandatory disclaimer** — Every output must include: "This is research output, NOT medical advice. Consult a healthcare professional."
3. **Evidence grading** — Classify every finding: systematic review > RCT > cohort study > case study > expert opinion
4. **Date sensitivity** — Note publication date of every source; flag if older than 3 years
5. **No prescriptions** — Never recommend dosages, treatments, or diagnoses
6. **No personal health advice** — Research is informational, never prescriptive
7. **Explicit jurisdiction** — Always clarify geographic and regulatory context
8. **Conflict of interest** — Note if studies were industry-funded
9. **Retraction awareness** — Verify cited papers have not been retracted or corrected
10. **Bias detection** — Flag methodological limitations (sample size, study design, population)
11. **Zero patient data** — Never include patient-identifiable information
