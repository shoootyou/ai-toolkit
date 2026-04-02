# AGENTS.md

## Project Overview

**IRIS** (Integrated Research Intelligence System) is a multi-agent, multi-domain plugin for AI coding assistants. It operates in two independent modes:

- **Research** — Investigates any topic from multiple expert perspectives, producing a synthesized report. Routes between **tech** (27 agents) and **healthcare** (25 agents) domains.
- **Learn** — Creates adaptive educational material on any topic using **6 pedagogical agents** that collaborate to produce a complete learning resource.

The system works in four layers:
1. **Router layer** — intelligent entry point (`skills/research/SKILL.md`) that analyzes the topic and delegates to tech or health. The teach/learn mode is invoked directly, not routed.
2. **Skill layer** — domain-specific entry points (`skills/tech/SKILL.md`, `skills/health/SKILL.md`, `skills/teach/SKILL.md`) that gather topic details and preferences
3. **Orchestration layer** — coordinates agent spawning, monitors completion, triggers synthesis
4. **Agent layer** — 52 specialized investigators + 6 educational agents, each producing one report in `.research/agents/`

See `README.md` for human-facing documentation, installation, and quick start.

## Directory Structure

```
iris/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── agents/                      # 58 agent definition files
│   ├── tech-*.md                # 27 tech agents
│   ├── health-*.md              # 25 health agents
│   ├── teacher-*.md             # 6 education agents
│   └── workflows/               # Extension files for agents exceeding 30K chars
├── skills/
│   ├── research/                # Router skill (entry point)
│   │   ├── SKILL.md
│   │   └── workflows/
│   │       └── route.md         # Topic analysis and delegation
│   ├── tech/                    # Tech research skill
│   │   ├── SKILL.md
│   │   └── workflows/
│   │       ├── investigate.md
│   │       └── orchestrate.md
│   └── health/                  # Healthcare research skill
│       ├── SKILL.md
│       └── workflows/
│           ├── investigate.md
│           └── orchestrate.md
│   └── teach/                   # Educational content skill
│       ├── SKILL.md
│       └── workflows/
│           ├── investigate.md
│           └── orchestrate.md
├── .requests/                   # Investigation tracking files (GUID-based)
├── .research/                   # Active research output (ONLY writable dir)
│   ├── {8-char-uuid}/           # One folder per research run
│   │   ├── agents/              # Individual agent reports
│   │   ├── SUMMARY.md           # Synthesized report for this run
│   │   ├── context-brief.md     # Context from previous run (continuations)
│   │   └── meta.json            # Run metadata backup
│   └── index.md                 # Catalog of active investigations
├── .research-archives/          # Archived investigations (zip)
├── AGENTS.md                    # This file
└── README.md                    # Human-facing docs
```

**Write permissions:** Agents may ONLY write to `.research/{run-uuid}/`. The orchestrate workflow may update `.requests/` frontmatter. Nothing else is writable.

## Agent System

### Agent File Structure

Each agent is a Markdown file with YAML frontmatter and XML blocks:

```yaml
---
name: tech-{specialty}
description: What this agent investigates. Spawned by the orchestrate workflow.
tools: Read, Bash, Grep, Glob
---
```

XML blocks appear in this order:

| Block | Purpose | Required |
|-------|---------|----------|
| `<role>` | Identity, context reception, quality expectations | Yes |
| `<io_contract>` | Input variables, output path, verification rules | Yes |
| `<philosophy>` | Investigation methodology, evidence standards | Yes |
| `<investigation_techniques>` | Domain-specific commands and analysis methods | Yes |
| `<output_format>` | Exact markdown structure for the output file | Yes |
| `<guardrails>` | Security constraints (see Guardrails section) | Yes |

### io_contract Standard

Every agent receives these variables from the orchestrator:

| Variable | Required | Description |
|----------|----------|-------------|
| `INVESTIGATION_TOPIC` | Yes | Full topic being investigated |
| `RESEARCH_SUBJECT` | Yes | Primary subject/target |
| `FOCUS_AREAS` | Yes | Specific areas to investigate |
| `OUTPUT_PATH` | Yes | Exact file path for output (includes UUID run folder) |
| `CROSS_REFERENCES` | No | Paths to other agents' output |
| `CONTEXT_BRIEF` | No | Path to context-brief.md (for continuation runs) |

Output rules:
1. Use `OUTPUT_PATH` exactly as provided; fall back to `.research/{RUN_ID}/agents/{agent-name}.md`
2. Run `mkdir -p` on the parent directory before writing
3. Write the output file **INCREMENTALLY**, section by section — create with `cat > file`, then append each subsequent section with `cat >> file`. Never write the entire file in a single operation.
4. Verify file exists after writing with `ls -la {OUTPUT_PATH}`
5. Write in English always — the synthesizer handles translation

### Tech Domain — 27 Agents

#### Technical
| Agent | Specialty |
|-------|-----------|
| `binary-analyst` | File structure, symbols, build info, runtime detection |
| `architecture-analyst` | Design patterns, modularity, data flow |
| `source-code-analyst` | Code structure, quality, dependencies, tests |
| `configuration-specialist` | Config files, env vars, flags, defaults |
| `performance-profiler` | CPU, memory, disk, startup time |
| `documentation-analyst` | Help systems, man pages, completeness |
| `dependency-analyst` | Dependency tree, versions, conflicts |
| `extensibility-analyst` | Plugin systems, hooks, extension APIs |

#### Security
| Agent | Specialty |
|-------|-----------|
| `vulnerability-analyst` | CVEs, OWASP, attack surfaces |
| `supply-chain-analyst` | Provenance, trust chains, SBOM, signing |
| `cryptography-analyst` | TLS, encryption, tokens, key management |

#### Network & API
| Agent | Specialty |
|-------|-----------|
| `network-traffic-analyst` | Protocols, ports, TLS, DNS, proxy support |
| `api-communication-analyst` | HTTP/gRPC/WebSocket endpoints, payloads |
| `api-surface-analyst` | API contracts, versioning, schemas |

#### DevOps
| Agent | Specialty |
|-------|-----------|
| `cicd-pipeline-analyst` | CI/CD workflows, automation, release patterns |
| `infrastructure-analyst` | Containers, Docker, IaC, deployment |

#### UX & Accessibility
| Agent | Specialty |
|-------|-----------|
| `usability-researcher` | User flows, CLI UX, error messages |
| `accessibility-auditor` | A11y, WCAG, NO_COLOR, screen readers |
| `i18n-analyst` | Internationalization, localization |

#### Business & Market
| Agent | Specialty |
|-------|-----------|
| `market-analyst` | Market position, audience, pricing |
| `comparative-analyst` | Weighted scoring, trade-offs, alternatives |
| `trend-analyst` | Adoption trajectory, lifecycle stage |
| `compliance-reviewer` | Licensing, privacy, terms, export |
| `community-analyst` | Contributors, governance, adoption |
| `integration-analyst` | Interoperability, IDE/CI integrations |
| `qa-strategist` | Test coverage, frameworks, quality gates |

#### Synthesis
| Agent | Specialty |
|-------|-----------|
| `synthesizer` | Cross-references all findings, resolves conflicts, translates |

### Health Domain — 25 Agents

These agents carry strict evidence-based guardrails. All output must cite verified sources.

#### Clinical & Evidence
| Agent | Specialty |
|-------|-----------|
| `health-clinical-evidence-analyst` | Systematic reviews, RCTs, meta-analyses, GRADE methodology |
| `health-clinical-trials-analyst` | ClinicalTrials.gov, phases, endpoints, enrollment |
| `health-pharmaceutical-analyst` | Drug mechanisms, interactions, formularies, approvals |
| `health-medical-device-analyst` | Device classifications, 510(k), PMA, recalls |

#### Public Health & Epidemiology
| Agent | Specialty |
|-------|-----------|
| `health-epidemiological-analyst` | Disease surveillance, outbreak analysis, incidence |
| `health-public-health-analyst` | Population health, prevention, community interventions |
| `health-infectious-disease-analyst` | Pathogens, transmission, antimicrobial resistance |
| `health-environmental-analyst` | Environmental exposures, occupational health |

#### Policy & Regulatory
| Agent | Specialty |
|-------|-----------|
| `health-regulatory-analyst` | FDA, EMA, WHO frameworks, approval pathways |
| `health-policy-analyst` | Health policy analysis, reform, systems comparison |
| `health-compliance-reviewer` | HIPAA, clinical guidelines compliance, accreditation |
| `health-economics-analyst` | Cost-effectiveness, HTA, value-based care, QALY |

#### Specialized Domains
| Agent | Specialty |
|-------|-----------|
| `health-mental-health-analyst` | Psychiatry, psychology, therapy modalities |
| `health-nutrition-analyst` | Dietary guidelines, supplementation, evidence-based nutrition |
| `health-genomics-analyst` | Precision medicine, pharmacogenomics, genetic testing |
| `health-maternal-health-analyst` | Obstetrics, perinatal, neonatal health |
| `health-rehabilitation-analyst` | Physical/occupational therapy, recovery protocols |

#### Health Systems & Technology
| Agent | Specialty |
|-------|-----------|
| `health-informatics-analyst` | EHR, FHIR, health data interoperability |
| `health-telemedicine-analyst` | Remote care, digital therapeutics, virtual health |
| `health-patient-safety-analyst` | Adverse events, medication errors, safety culture |

#### Analysis & Trends
| Agent | Specialty |
|-------|-----------|
| `health-comparative-analyst` | Cross-country health system comparison |
| `health-trend-analyst` | Emerging health trends, digital health evolution |
| `health-equity-analyst` | Health disparities, social determinants, access |
| `health-bioethics-reviewer` | Medical ethics, informed consent, research ethics |

#### Synthesis
| Agent | Specialty |
|-------|-----------|
| `health-synthesizer` | Cross-references health findings, applies GRADE, translates |

### Education Domain — 6 Agents

| Agent | Specialty |
|-------|-----------|
| `teacher-researcher` | Gathers information on any topic across all disciplines |
| `teacher-curriculum-designer` | Structures learning paths with prerequisites and progression |
| `teacher-content-creator` | Creates explanations, analogies, examples, visual aids |
| `teacher-assessor` | Designs exercises, quizzes, Socratic questions, rubrics |
| `teacher-fact-checker` | Validates sources, experts, detects biases, flags misinformation |
| `teacher-synthesizer` | Combines all outputs into a unified educational document |

## Skill System

IRIS has two independent modes, each with its own skills:

### Research Mode — `/iris:research`, `/iris:tech`, `/iris:health`

Investigate a topic from multiple expert perspectives. The router delegates to the right domain.

| Skill | Workflow | Trigger | File |
|-------|----------|---------|------|
| **Router** | Route | "research", "investigate" (no domain) | `skills/research/workflows/route.md` |
| **Router** | Continue | "continue", "deepen" | `skills/research/workflows/continue.md` |
| **Router** | Status | "status", "list investigations" | `skills/research/workflows/status.md` |
| **Router** | Archive | "archive research" | `skills/research/workflows/archive.md` |
| **Tech** | Investigate | tech topic detected | `skills/tech/workflows/investigate.md` |
| **Tech** | Orchestrate | (internal) | `skills/tech/workflows/orchestrate.md` |
| **Health** | Investigate | health topic detected | `skills/health/workflows/investigate.md` |
| **Health** | Orchestrate | (internal) | `skills/health/workflows/orchestrate.md` |

### Learn Mode — `/iris:teach`

Create adaptive educational material on any topic. Invoked directly, not through the router.

| Workflow | Trigger | File |
|----------|---------|------|
| **Investigate** | "teach", "learn", "aprender" | `skills/teach/workflows/investigate.md` |
| **Orchestrate** | (internal) | `skills/teach/workflows/orchestrate.md` |

**Humanization:** The teach domain targets non-technical users. User-facing
messages (via `ask_user`) use humanized language: "specialists" instead of "agents",
"team" instead of "pipeline", "learning session" instead of "investigation". Internal
processing, variables, file paths, and metadata retain technical terms. See the User
Communication Guide in `skills/teach/SKILL.md` for the full mapping table.

**Request files:** Teach uses `teach-{8-char-guid}.md` prefix (not `research-*`) and
`learning-runs` key (not `research-runs`) for domain clarity.

All skills use `tools: Read, Write, Edit, Bash, Grep, Glob, Task`.

## Research Workflow

### End-to-End Flow

```
User invokes research → router determines domain → investigate workflow gathers topic
  → creates .requests/research-{8-char-guid}.md
  → creates .research/{8-char-uuid}/agents/
  → orchestrate workflow spawns agents in parallel
  → agents write to .research/{uuid}/agents/*.md
  → synthesizer combines into .research/{uuid}/SUMMARY.md
```

### Cumulative Research Flow

```
User invokes continue → selects existing investigation → gap analysis
  → creates new .research/{new-uuid}/agents/
  → generates context-brief.md from previous SUMMARY
  → orchestrate spawns agents with context injection
  → agents build upon previous findings
  → synthesizer combines new + previous into SUMMARY.md
```

### State Machine

The `status` field in the request file follows this progression:

```
initialized → researching → synthesizing → completed
                  │               │
                  └── failed ◄────┘
```

| State | Meaning | Set By |
|-------|---------|--------|
| `initialized` | Request created, not yet started | investigate workflow |
| `researching` | Agents spawned and running | orchestrate workflow |
| `synthesizing` | All research complete, synthesizer running | orchestrate workflow |
| `completed` | Investigation finished successfully | orchestrate workflow |
| `failed` | Unrecoverable error | orchestrate workflow |

Transitions must be sequential. Every transition updates both `status` and `updated` fields.

## Request File Contract

**Filename pattern by domain:**
- Tech/Health: `.requests/research-{8-char-guid}.md`
- Teach: `.requests/teach-{8-char-guid}.md`

**Request ID:** Full GUID (e.g., `550e8400-e29b-41d4-a716-446655440000`). File uses first 8 chars.

**Frontmatter:**

```yaml
---
name: "descriptive investigation name"
topic: "research topic as stated by user"
subject: "primary subject"
language: "English|Spanish|etc."
domain: "tech|health|teach"
status: researching
created: "2026-04-04T16:50:00Z"
updated: "2026-04-04T18:00:00Z"
request-id: "550e8400-e29b-41d4-a716-446655440000"
research-runs:          # tech/health domains
  - id: a1b2c3d4
    status: completed
    date: "2026-04-04T16:50:00Z"
    agents: [tech-binary-analyst, tech-architecture-analyst]
learning-runs:          # teach domain (equivalent to research-runs)
  - id: a1b2c3d4
    status: completed
    date: "2026-04-04T16:50:00Z"
    agents: [teacher-researcher, teacher-curriculum-designer]
# teach domain also includes:
# education-level: "basic|intermediate|advanced"
# target-audience: "children|teenagers|adults|professionals"
# learning-style: "visual|textual|practical|mixed"
# include-exercises: true|false
focus-areas:
  - "area 1"
  - "area 2"
agents:
  - tech-binary-analyst
---
```

**Body:** Contains Topic, Subject, Focus Areas, Key Questions, Agent Configuration, and Parameters sections.

## Guardrails (CRITICAL)

These rules are mandatory for ALL agents and workflows. Non-negotiable.

### Rule 1: Write Path Restriction
Agents may ONLY write to:
- `.research/{run-uuid}/` and subdirectories (research output)
- `.requests/*.md` (only frontmatter updates, only by orchestrate workflow)
- `.research-archives/` (only archive zips, only by archive workflow)

Everything else is BLOCKED — `agents/`, `skills/`, project root, system dirs.

### Rule 2: No Destructive Commands
BLOCKED: `rm` (except `.research/` cleanup with user confirmation), `mv`/`cp` outside `.research/`, `chmod`, `chown`, `sed -i` on non-research files, `sudo`/`su`, `kill`/`pkill`, package managers (`brew`, `apt`, `pip`, `npm install`), git write operations (`git commit`, `git push`, `git rebase`).

### Rule 3: No Privilege Escalation
Never use `sudo`, `su`, `doas`. If investigation requires elevated privileges, document the limitation and move on.

### Rule 4: No Network Requests to Target
No `curl` to target APIs, no `wget` of target resources, no `ssh` to target servers. Reference public docs only.

### Rule 5: Temp File Cleanup
Temp files go in `.research/agents/` (not `/tmp`) and must be removed before completion.

### Rule 6: Violation Response
On any violation: STOP immediately → log in request file → set status to `failed` → EXIT.

### Pre-Flight Checklist
Before running any command, verify:
- [ ] Output path is within `.research/`
- [ ] Command is read-only (no writes outside `.research/`)
- [ ] No privilege escalation
- [ ] No network calls to target

## Healthcare Guardrails (CRITICAL)

These additional rules apply to ALL health-* agents. Non-negotiable.

1. **Verified sources only** — WHO, FDA, EMA, NIH, CDC, PubMed, Cochrane ONLY
2. **Mandatory disclaimer** — every output must state this is for informational purposes only, not medical advice
3. **Evidence grading** — use GRADE methodology (systematic review > RCT > cohort > case)
4. **Date sensitivity** — flag sources older than 3 years as potentially outdated
5. **No prescriptions** — never recommend specific medications, dosages, or treatment plans
6. **No personal health advice** — never provide individual health recommendations
7. **Explicit jurisdiction** — regulatory analysis must specify country/region
8. **Conflict of interest** — flag industry-funded studies
9. **Retraction awareness** — check for retracted papers
10. **Bias detection** — assess study designs for common biases
11. **Zero patient data** — never process, store, or reference real patient information

## Agent Anatomy

### Creating a New Agent

1. **Name it:** `tech-{specialty}.md`, `health-{specialty}.md`, or `teacher-{specialty}.md` — prefix matches the domain, kebab-case, descriptive
2. **Create file:** `agents/tech-{specialty}.md`
3. **Write frontmatter:** name, description (end with "Spawned by the orchestrate workflow"), tools
4. **Write `<role>`:** Identity ("You are the **Research {Name}**"), output path, context reception
5. **Write `<io_contract>`:** Input variables table, output rules, verification steps
6. **Write `<philosophy>`:** Investigation approach, evidence standards (what/how/meaning/confidence)
7. **Write `<investigation_techniques>`:** Domain techniques with purpose, commands, interpretation, red flags (5K-10K chars)
8. **Write `<output_format>`:** Markdown template with Executive Summary, Methodology, Findings, Recommendations
9. **Write `<guardrails>`:** Copy the 6 mandatory rules above, add domain-specific rules
10. **Verify size:** `wc -c agents/tech-{specialty}.md` — target 7K-25K chars, max 30K
11. **Register:** Add to the agent catalog in `skills/tech/workflows/orchestrate.md`

### Size Limits

- Target: 7,000–25,000 characters per agent
- Maximum: 30,000 characters
- If over 30K, move techniques to `agents/workflows/{agent-name}-techniques.md` and add a read instruction in the main file
- Extension files inherit parent guardrails — no separate frontmatter needed

## Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| Tech agent file | `tech-{specialty}.md` | `tech-binary-analyst.md` |
| Health agent file | `health-{specialty}.md` | `health-clinical-evidence-analyst.md` |
| Teach agent file | `teacher-{specialty}.md` | `teacher-researcher.md` |
| Agent output | `.research/{uuid}/agents/{agent-name}.md` | `.research/a1b2c3d4/agents/tech-binary-analyst.md` |
| Tech/Health request | `research-{8-char-guid}.md` | `research-550e8400.md` |
| Teach request | `teach-{8-char-guid}.md` | `teach-a3c7e192.md` |
| Run folder | `.research/{8-char-uuid}/` | `.research/a1b2c3d4/` |
| Synthesis | `.research/{uuid}/SUMMARY.md` | Per-run synthesis |
| Context brief | `.research/{uuid}/context-brief.md` | For continuation runs |
| Run metadata | `.research/{uuid}/meta.json` | Backup metadata |
| Archives | `{topic-slug}-{uuid}.zip` | `copilot-cli-a1b2c3d4.zip` |

## Language Policy

- **Research:** Always English — ensures cross-agent consistency
- **Synthesis:** User's chosen language (captured in request file `language` field)
- **System files:** English (agents, skills, workflows, this file)
- **User interaction:** Adaptive — match the user's language

## Archive System

Archival is handled by the shared `skills/research/workflows/archive.md` workflow:

1. User invokes `/iris:research archive`
2. Lists completed investigations
3. User selects which to archive
4. Zips selected run folder(s) to `.research-archives/{topic-slug}-{uuid}.zip`
5. Removes archived UUID folders from `.research/`
6. Updates request file status to `archived`
7. Request files in `.requests/` are NEVER deleted — they are permanent records
