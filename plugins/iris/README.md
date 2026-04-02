# IRIS — Integrated Research Intelligence System

A multi-agent plugin for AI coding agents with **58 specialized agents** operating in two independent modes: **Research** (investigate any topic) and **Learn** (create adaptive educational material).

Compatible with [GitHub Copilot](https://github.com/features/copilot), [Claude Code](https://code.claude.com), and any agent supporting the plugin marketplace protocol.

## What is IRIS?

IRIS is like having a research department and a teaching team at your fingertips.

- **Research Mode** — Deploy up to 27 parallel agents that examine a topic from every angle (architecture, security, APIs, performance, compliance, market position, etc.), then synthesize findings into a comprehensive report.
- **Learn Mode** — Six pedagogical agents collaborate to create educational material adapted to your level, style, and language on any topic.

## Quick Start

```bash
# Install via marketplace
/plugin marketplace add shoootyou/ai-toolkit
/plugin install iris@ai-toolkit

# Research: let IRIS pick the right domain
/iris:research investigate "GitHub Copilot CLI"

# Research: go directly to a domain
/iris:tech investigate "Node.js performance"
/iris:health investigate "diabetes management protocols"

# Learn: create educational material on any topic
/iris:teach "quantum computing"

# Check investigation status
/iris:tech status

# Continue a previous investigation
/iris:tech continue
```

## Skills

| Skill | Command | Mode | Description |
|-------|---------|------|-------------|
| Research | `/iris:research` | Research | Intelligent router — analyzes topic and delegates to tech or health |
| Tech | `/iris:tech` | Research | Software and technology (27 agents) |
| Health | `/iris:health` | Research | Healthcare with evidence-based guardrails (25 agents) |
| Teach | `/iris:teach` | Learn | Adaptive educational content (6 agents) |

## How It Works

### Research Mode

```
User → /iris:research → Router → Delegates to domain
                                      │
                        ┌──────────────┼──────────────┐
                        ▼                             ▼
                  /iris:tech                    /iris:health
                  27 agents                    25 agents
                        │                             │
                  Synthesis                    Synthesis
                        ▼                             ▼
              .research/SUMMARY.md        .research/SUMMARY.md
```

1. **You describe what to investigate** — IRIS asks clarifying questions
2. **The orchestrator selects agents** — from the domain's roster
3. **Agents investigate in parallel** — each from their specialty
4. **The synthesizer combines findings** — resolving conflicts, translating
5. **You get a report** — in `.research/SUMMARY.md` with sub-reports

### Learn Mode

```
User → /iris:teach → Investigate (gather preferences)
                          │
                    Orchestrate (6 agents in parallel)
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
         Researcher  Curriculum  Content
                     Designer   Creator
              ▼           ▼           ▼
         Fact-Checker Assessor  Synthesizer
                          │
              .research/SUMMARY.md
              (Complete learning resource)
```

1. **You describe what to learn** — level, style, language
2. **Six agents collaborate** — research, structure, create, assess, verify
3. **You get a learning document** — adapted to your preferences

## The 58 Agents

### Tech Domain — 27 Agents

#### Technical
| Agent | Specialty |
|-------|-----------|
| `binary-analyst` | File structure, symbols, build info, runtime detection |
| `architecture-analyst` | Design patterns, modularity, data flow, structure |
| `source-code-analyst` | Code structure, quality, dependencies, tests |
| `configuration-specialist` | Config files, env vars, flags, defaults, precedence |
| `performance-profiler` | CPU, memory, disk, startup time, efficiency |
| `documentation-analyst` | Help systems, man pages, completeness, accuracy |
| `dependency-analyst` | Dependency tree, versions, conflicts, lockfiles |
| `extensibility-analyst` | Plugin systems, hooks, extension APIs |

#### Security
| Agent | Specialty |
|-------|-----------|
| `vulnerability-analyst` | CVEs, OWASP, attack surfaces, binary hardening |
| `supply-chain-analyst` | Provenance, trust chains, SBOM, signing |
| `cryptography-analyst` | TLS, encryption, tokens, key management |

#### Network & API
| Agent | Specialty |
|-------|-----------|
| `network-traffic-analyst` | Protocols, ports, TLS, DNS, proxy support |
| `api-communication-analyst` | HTTP/gRPC/WebSocket endpoints, payloads, auth |
| `api-surface-analyst` | API contracts, versioning, schemas, deprecation |

#### DevOps
| Agent | Specialty |
|-------|-----------|
| `cicd-pipeline-analyst` | CI/CD workflows, automation, release patterns |
| `infrastructure-analyst` | Containers, Docker, IaC, deployment, scaling |

#### UX & Accessibility
| Agent | Specialty |
|-------|-----------|
| `usability-researcher` | User flows, CLI UX, error messages, learning curve |
| `accessibility-auditor` | A11y, WCAG, NO_COLOR, screen readers |
| `i18n-analyst` | Internationalization, localization, multi-language |

#### Business & Market
| Agent | Specialty |
|-------|-----------|
| `market-analyst` | Market position, audience, pricing, business model |
| `comparative-analyst` | Weighted scoring, trade-offs, alternatives |
| `trend-analyst` | Adoption trajectory, lifecycle stage, future outlook |
| `compliance-reviewer` | Licensing, privacy, terms, export restrictions |
| `community-analyst` | Contributors, governance, adoption, health metrics |
| `integration-analyst` | Interoperability, IDE/CI integrations, compatibility |
| `qa-strategist` | Test coverage, frameworks, quality gates |

#### Synthesis
| Agent | Specialty |
|-------|-----------|
| `tech-synthesizer` | Cross-references all tech findings, resolves conflicts |

### Health Domain — 25 Agents

#### Clinical & Evidence
| Agent | Specialty |
|-------|-----------|
| `clinical-evidence-analyst` | Systematic reviews, RCTs, meta-analyses, GRADE |
| `clinical-trials-analyst` | Trial phases, endpoints, enrollment |
| `pharmaceutical-analyst` | Drug mechanisms, interactions, approvals |
| `medical-device-analyst` | Device classifications, 510(k), PMA, recalls |

#### Public Health & Epidemiology
| Agent | Specialty |
|-------|-----------|
| `epidemiological-analyst` | Disease surveillance, outbreak analysis |
| `public-health-analyst` | Population health, prevention |
| `infectious-disease-analyst` | Pathogens, transmission, AMR |
| `environmental-analyst` | Environmental exposures, occupational health |

#### Policy & Regulatory
| Agent | Specialty |
|-------|-----------|
| `regulatory-analyst` | FDA, EMA, WHO regulatory frameworks |
| `policy-analyst` | Health policy analysis, systems comparison |
| `compliance-reviewer` | HIPAA, clinical guidelines compliance |
| `economics-analyst` | Cost-effectiveness, HTA, value-based care |

#### Specialized Domains
| Agent | Specialty |
|-------|-----------|
| `mental-health-analyst` | Psychiatry, psychology, therapy modalities |
| `nutrition-analyst` | Dietary guidelines, evidence-based nutrition |
| `genomics-analyst` | Precision medicine, pharmacogenomics |
| `maternal-health-analyst` | Obstetrics, perinatal, neonatal health |
| `rehabilitation-analyst` | Physical/occupational therapy, recovery |

#### Health Systems & Technology
| Agent | Specialty |
|-------|-----------|
| `informatics-analyst` | EHR, FHIR, health data interoperability |
| `telemedicine-analyst` | Remote care, digital therapeutics |
| `patient-safety-analyst` | Adverse events, medication errors |

#### Analysis & Trends
| Agent | Specialty |
|-------|-----------|
| `comparative-analyst` | Cross-country health system comparison |
| `trend-analyst` | Emerging health trends, digital health |
| `equity-analyst` | Health disparities, social determinants |
| `bioethics-reviewer` | Medical ethics, informed consent |

#### Synthesis
| Agent | Specialty |
|-------|-----------|
| `health-synthesizer` | Cross-references health findings, applies GRADE |

### Education Domain — 6 Agents

| Agent | Role |
|-------|------|
| `teacher-researcher` | Gathers authoritative sources on any topic |
| `teacher-curriculum-designer` | Builds structured learning paths with prerequisites |
| `teacher-content-creator` | Writes explanations, analogies, examples |
| `teacher-assessor` | Creates exercises, quizzes, Socratic questions |
| `teacher-fact-checker` | Validates sources, detects bias, flags misinformation |
| `teacher-synthesizer` | Produces the final unified learning document |

## Plugin Structure

```
iris/
├── .claude-plugin/
│   └── plugin.json
├── agents/                     # 58 agent definitions
│   ├── tech-*.md               # 27 tech agents
│   ├── health-*.md             # 25 health agents
│   └── teacher-*.md            # 6 education agents
├── skills/
│   ├── research/               # Router (research mode entry)
│   │   ├── SKILL.md
│   │   └── workflows/
│   │       ├── route.md        # Domain detection + delegation
│   │       ├── continue.md     # Continue previous investigation
│   │       ├── status.md       # Investigation dashboard
│   │       └── archive.md      # Archive completed research
│   ├── tech/                   # Tech domain skill
│   │   ├── SKILL.md
│   │   └── workflows/
│   │       ├── investigate.md
│   │       └── orchestrate.md
│   ├── health/                 # Health domain skill
│   │   ├── SKILL.md
│   │   └── workflows/
│   │       ├── investigate.md
│   │       └── orchestrate.md
│   └── teach/                  # Learn mode skill
│       ├── SKILL.md
│       └── workflows/
│           ├── investigate.md
│           └── orchestrate.md
├── .requests/                  # Investigation tracking files
├── .research/                  # Active research output
├── .research-archives/         # Archived investigations (zip)
├── AGENTS.md
└── README.md
```

## Output Structure

After a research investigation:

```
.research/{run-uuid}/
├── SUMMARY.md              # Synthesized report
└── agents/                 # Individual agent outputs
    ├── tech-binary-analyst.md
    ├── tech-architecture-analyst.md
    └── ... (one per selected agent)
```

After a learning session:

```
.research/{run-uuid}/
├── SUMMARY.md              # Complete learning document
└── agents/
    ├── teacher-researcher.md
    ├── teacher-curriculum-designer.md
    ├── teacher-content-creator.md
    ├── teacher-assessor.md
    ├── teacher-fact-checker.md
    └── teacher-synthesizer.md
```

## Safety & Guardrails

All IRIS agents are **read-only researchers**. They cannot:

- Write outside `.research/` directory
- Execute the target tool's primary functions
- Modify system configuration
- Make network requests
- Use elevated privileges (sudo)
- Install or uninstall packages
- Modify source code or git history

Every agent has a mandatory pre-flight checklist that validates each command before execution.

### Healthcare-Specific Guardrails

- Only verified sources (WHO, FDA, EMA, NIH, CDC, PubMed, Cochrane)
- Mandatory "not medical advice" disclaimer in every output
- Evidence grading using GRADE methodology
- Sources older than 3 years flagged as potentially outdated
- No prescriptions, dosages, or personal health recommendations
- Zero patient data processing

### Education-Specific Guardrails

- Source grading (academic, institutional, expert, community)
- Bias detection in educational materials
- Age-appropriate content filtering
- Controversial topic handling with multiple perspectives
- Citation of primary sources for all factual claims

## Multi-Language Support

Agents always research in English for consistency. The synthesizer translates the final report to your preferred language while preserving technical terms, code blocks, and commands.

Supported: Any language — just tell IRIS during setup.

## License

MIT
