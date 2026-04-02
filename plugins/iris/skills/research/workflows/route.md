<purpose>
Analyze the user's research topic, determine the appropriate domain (tech or health),
and delegate to the correct domain-specific IRIS skill. This is the intelligent routing
layer that ensures users always reach the right set of specialized agents.

Note: The teach/education domain (/iris:teach) is a separate flow — not routed through
here. Users invoke it directly when they want to learn about a topic rather than
investigate it.
</purpose>

<philosophy>
The router is a traffic director, not a researcher. Its goal is to understand intent
quickly and hand off to the right domain with minimal friction. When in doubt, ask —
a wrong delegation is worse than one extra question.
</philosophy>

<process>

<step name="receive-topic">
## Step 1: Receive the Topic

The user provides a topic or research intent. Capture it exactly as stated.

If the user provides no topic, ask:
```
What topic would you like to investigate?
```

Use `ask_user` with freeform input.
</step>

<step name="analyze-domain">
## Step 2: Analyze Domain Signals

Scan the topic for domain indicators:

### Tech Domain Indicators (→ delegate to /iris:tech)
- Programming languages: Python, JavaScript, Rust, Go, Java, C++, etc.
- Software tools: Git, Docker, Kubernetes, Terraform, Ansible, etc.
- Frameworks/libraries: React, Django, Spring, Express, etc.
- Infrastructure: AWS, Azure, GCP, servers, databases, etc.
- Concepts: API, CLI, SDK, CI/CD, DevOps, microservices, etc.
- Security: CVE, vulnerability, penetration testing, encryption, etc.
- Software names: any recognizable software product

### Health Domain Indicators (→ delegate to /iris:health)
- Medical terms: diagnosis, treatment, therapy, clinical, patient, etc.
- Diseases/conditions: diabetes, cancer, hypertension, COVID, etc.
- Pharmaceuticals: drug names, FDA approval, clinical trials, dosage, etc.
- Health systems: hospital, clinic, EHR, FHIR, telemedicine, etc.
- Public health: epidemic, vaccination, prevention, surveillance, etc.
- Regulatory: FDA, EMA, WHO, HIPAA, clinical guidelines, etc.
- Specialties: cardiology, oncology, psychiatry, pediatrics, etc.
- Research: PubMed, Cochrane, systematic review, meta-analysis, etc.

### Ambiguous Indicators (→ ask user)
These terms could belong to multiple domains depending on context:
- "security" — cybersecurity (tech) vs patient safety (health)
- "compliance" — SOC2/GDPR (tech) vs HIPAA/FDA (health)
- "AI" / "machine learning" — implementation (tech) vs application in healthcare (health)
- "data" — software data (tech) vs clinical data (health)
- "analytics" — software analytics (tech) vs health analytics (health)
- "IoT" — general IoT (tech) vs medical IoT/wearables (health)
- "blockchain" — crypto/fintech (tech) vs health records (health)
- "regulation" — software regulation (tech) vs health regulation (health)

### Educational Intent (→ NOT routed here)
If the user's intent is to **learn** about a topic rather than **investigate** it
(keywords: "learn", "teach", "explain", "understand", "for beginners", "for students"),
inform them that teaching is a separate IRIS flow:

```
It sounds like you want to learn about this topic rather than conduct a research investigation.
For learning, use /iris:teach directly — it has 6 specialized educational agents that adapt
to your level and learning style.
```

Use `ask_user` with choices:
- "Yes, switch to /iris:teach for learning"
- "No, I want a full research investigation"
</step>

<step name="confirm-or-ask">
## Step 3: Confirm or Ask

### If domain is clear (high confidence):
Confirm with the user before delegating:

```
I understand you want to investigate "[topic]". This falls under [tech/health] research.
I'll delegate to /iris:[domain] which has [N] specialized agents for this.
```

Use `ask_user` with choices:
- "Yes, proceed with [domain] research"
- "Actually, I meant [other domain]"
- "Let me clarify the topic"

### If domain is ambiguous:
Ask the user directly:

```
Your topic "[topic]" could be investigated from either perspective:
```

Use `ask_user` with choices:
- "Tech/Software perspective — using 27 tech research agents"
- "Healthcare perspective — using 25 health research agents with evidence-based guardrails"
- "Let me clarify the topic"

### If domain is none of the above:
Inform the user of available domains:

```
IRIS currently supports two research domains:
- /iris:tech — Software and technology (27 agents)
- /iris:health — Healthcare and medical (25 agents)

For learning/educational content, use /iris:teach directly (6 agents).

Your topic "[topic]" doesn't clearly fit either research domain. Would you like to:
```

Use `ask_user` with choices:
- "Try tech research (closest match)"
- "Try health research (closest match)"
- "Cancel — this topic isn't suitable for IRIS"
</step>

<step name="delegate">
## Step 4: Delegate to Domain Skill

Once the domain is confirmed, hand off to the appropriate skill:

### For Tech:
```
Proceeding with tech research. Activating /iris:tech...
```
Read and follow: `../tech/workflows/investigate.md`

### For Health:
```
Proceeding with health research. Activating /iris:health...
```
Read and follow: `../health/workflows/investigate.md`

Pass the original topic and any clarifications gathered during routing.
</step>

</process>

<guardrails>
## Router Guardrails

### Behavioral Rules
1. **Never research** — The router analyzes and delegates, it does not investigate
2. **Minimize turns** — Get to delegation in 1-2 conversational turns maximum
3. **Always confirm** — Never delegate without user confirmation via `ask_user`
4. **Preserve context** — Pass the full topic and user clarifications to the domain skill

### Safety Rules
1. **Read-only** — The router does not write files except routing logs to `.requests/`
2. **No code execution** — No bash commands, no file modifications
3. **No privilege escalation** — No sudo, no system changes
4. **No network requests** — Routing is purely analytical

### Domain Assignment Rules
1. If 3+ tech indicators and 0 health indicators → tech (high confidence)
2. If 3+ health indicators and 0 tech indicators → health (high confidence)
3. If indicators from both domains → ambiguous (ask user)
4. If 1-2 indicators from one domain → confirm with user before delegating
5. If 0 indicators from either → ask user to clarify
</guardrails>
