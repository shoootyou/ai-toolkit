---
name: research
description: IRIS Research router. USE WHEN the user wants to research or investigate ANY topic. Analyzes the topic, determines the appropriate domain (tech or health), and delegates to the correct domain-specific skill. For learning/educational content, redirects to /iris:teach. Users can also go directly to /iris:tech or /iris:health.
tools: Read, Write, Edit, Bash, Grep, Glob, Task
---

# IRIS Research Router

Intelligent routing skill that serves as the universal entry point for all IRIS
investigations. It analyzes the user's topic, determines the appropriate research
domain, and delegates to the correct domain-specific skill.

## Available Domains

| Domain | Skill | Agents | Scope |
|--------|-------|--------|-------|
| **Tech** | `/iris:tech` | 27 agents | Software, libraries, CLIs, APIs, frameworks, DevOps, infrastructure |
| **Health** | `/iris:health` | 25 agents | Clinical, pharmaceutical, epidemiology, public health, bioethics, health policy |

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **Route** | "research", "investigate", "analyze", "study" | `workflows/route.md` |
| **Continue** | "continue", "deepen", "iterate" | `workflows/continue.md` |
| **Status** | "status", "list investigations" | `workflows/status.md` |
| **Archive** | "archive research" | `workflows/archive.md` |

## How It Works

1. **User invokes `/iris:research`** with a topic or intent
2. **Route workflow** activates:
   - Analyzes the topic for domain signals
   - If domain is clear → delegates immediately
   - If ambiguous → asks the user to clarify
   - Hands off to the appropriate domain skill

## Domain Detection

The router uses keyword and context analysis to determine the domain:

### Tech Signals
- Software names, programming languages, frameworks
- CLI tools, APIs, libraries, packages
- DevOps, CI/CD, infrastructure, cloud
- Security vulnerabilities, code analysis
- Performance, benchmarks, architecture

### Health Signals
- Diseases, conditions, symptoms, treatments
- Drugs, pharmaceuticals, clinical trials
- Medical devices, diagnostics
- Public health, epidemiology, prevention
- Regulatory (FDA, EMA, WHO), compliance (HIPAA)
- Mental health, nutrition, genomics

### Ambiguous Topics
Some topics could belong to either domain:
- "security" → tech (cybersecurity) or health (patient safety)?
- "compliance" → tech (SOC2, GDPR) or health (HIPAA, FDA)?
- "AI" → tech (implementation) or health (AI in healthcare)?

When ambiguous, the router asks the user using `ask_user` with the two domain options.

## Principles

### Never Assume
If the domain is not obvious from the topic, always ask. A wrong delegation wastes
time and produces irrelevant results.

### Quick Handoff
The router should not do research itself. Its job is to understand the topic and
delegate as fast as possible. Minimize conversational turns.

### Direct Access
Power users who know their domain can skip the router:
- `/iris:tech investigate "Node.js"` → goes directly to tech skill
- `/iris:health investigate "diabetes"` → goes directly to health skill

## Guardrails

- This skill does NOT perform research — it only routes
- The ONLY writable directory is `.requests/` (for logging routing decisions)
- No code execution that modifies the system
- No privilege escalation
- No destructive filesystem operations
