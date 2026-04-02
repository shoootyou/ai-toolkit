---
name: validate
description: >
  Workflow for validating plugin structure against marketplace conventions.
  Checks required files, naming conventions, size limits, guardrails presence,
  and cross-file consistency. Reports issues with severity levels.
---

# Validate Plugin Workflow

Audits a plugin directory against the AI Toolkit marketplace conventions defined
in CONTRIBUTING.md. Produces a structured report with severity-classified issues.

## Pre-Flight

1. Read `CONTRIBUTING.md` from the repository root to load current conventions
2. Verify the target path exists and contains a plugin structure
3. If no path is specified, list available plugins and ask the user to choose

## Step 1 — Identify Target

Parse the user's input to determine the plugin path:

- `/scaffold validate iris` → `plugins/iris/`
- `/scaffold validate plugins/iris` → `plugins/iris/`
- `/scaffold validate` (no target) → list plugins, ask user

Verify the directory exists. If not, report error and stop.

## Step 2 — Required Files Check

Check for the presence of required files per CONTRIBUTING.md:

| File | Required | Severity if Missing |
|------|----------|-------------------|
| `.claude-plugin/plugin.json` | Yes | HIGH |
| `README.md` | Yes | HIGH |
| `AGENTS.md` | Recommended | MEDIUM |
| `CLAUDE.md` | Recommended | LOW |

For each file found, verify it is non-empty (> 100 characters for meaningful content).

### plugin.json Validation

If plugin.json exists, verify required fields:

```json
{
  "name": "string (required)",
  "description": "string (required)",
  "version": "string, semver format (required)",
  "author": {
    "name": "string (required)"
  },
  "license": "string (required)"
}
```

Report missing fields as HIGH severity.

## Step 3 — Agent Files Validation

If `agents/` directory exists:

### 3.1 Naming Conventions

For each `.md` file in `agents/`:
- Must follow `{prefix}-{role}.md` pattern (kebab-case)
- Prefix should be consistent within a domain (e.g., all `tech-*` or all `health-*`)
- Flag files that don't match the pattern → MEDIUM

### 3.2 Structure Check

For each agent file, verify presence of:

| Element | Required | Check Method |
|---------|----------|--------------|
| YAML frontmatter | Yes | Starts with `---` |
| `name` in frontmatter | Yes | Grep in frontmatter |
| `description` in frontmatter | Yes | Grep in frontmatter |
| `tools` in frontmatter | Yes | Grep in frontmatter |
| `<role>` block | Yes | Grep for `<role>` |
| `<io_contract>` block | Yes | Grep for `<io_contract>` |
| `<philosophy>` block | Yes | Grep for `<philosophy>` |
| `<output_format>` block | Yes | Grep for `<output_format>` |
| `<guardrails>` block | Yes | Grep for `<guardrails>` |

Missing blocks → HIGH for guardrails, MEDIUM for others.

### 3.3 Size Check

For each agent file:
- Count characters: `wc -c < file`
- Over 30,000 → HIGH (exceeds limit)
- Over 25,000 → LOW (approaching limit, consider splitting)

### 3.4 Guardrails Content Check

For each agent file with a `<guardrails>` block, verify it contains at minimum:
- Reference to output directory restriction
- Prohibition of system modification
- Prohibition of privilege escalation

Missing guardrail topics → MEDIUM

## Step 4 — Skill Files Validation

If `skills/` directory exists:

### 4.1 Structure Check

For each skill directory in `skills/`:
- Must contain `SKILL.md` → HIGH if missing
- Must contain `workflows/` directory → MEDIUM if missing
- `SKILL.md` must have YAML frontmatter with `name`, `description`, `tools`
- At least one workflow file must exist in `workflows/`

### 4.2 SKILL.md Content Check

Each SKILL.md should contain:
- Workflow Routing table → MEDIUM if missing
- Guardrails section → HIGH if missing
- How It Works section → LOW if missing

### 4.3 Workflow Files Check

For each workflow `.md` file:
- Must have YAML frontmatter with `name` and `description`
- Should contain numbered steps (## Step N)
- Should contain Error Handling section
- Should contain Guardrails section
- Must be under 30,000 characters

## Step 5 — Cross-Reference Consistency

Check that references between files are consistent:

### 5.1 Agent Catalog Consistency

If an orchestrate workflow exists, compare:
- Agents listed in orchestrate.md catalog vs. actual files in `agents/`
- Flag agents in catalog but not on disk → HIGH
- Flag agents on disk but not in catalog → MEDIUM

### 5.2 SKILL.md ↔ Workflow Consistency

For each SKILL.md:
- Workflows referenced in routing table vs. actual files in `workflows/`
- Flag referenced workflows that don't exist → HIGH
- Flag workflow files not referenced in routing table → LOW

### 5.3 Documentation Consistency

Compare agent/skill counts across:
- `AGENTS.md` stated counts vs. actual files
- `plugin.json` description counts vs. actual files
- `README.md` stated counts vs. actual files

Mismatches → MEDIUM

### 5.4 Marketplace Entry

Check `.claude-plugin/marketplace.json` at the repository root:
- Plugin should have an entry → MEDIUM if missing
- Entry description should match plugin.json description
- Version should match

## Step 6 — Generate Report

Compile all findings into a structured report:

```markdown
# Validation Report: <plugin-name>

## Summary

| Severity | Count |
|----------|-------|
| HIGH     | <n>   |
| MEDIUM   | <n>   |
| LOW      | <n>   |
| PASS     | <n>   |

Total checks: <n>

## HIGH Severity Issues

### [H1] <issue title>
- **File**: <path>
- **Issue**: <description>
- **Fix**: <suggested fix>

## MEDIUM Severity Issues

### [M1] <issue title>
...

## LOW Severity Issues

### [L1] <issue title>
...

## Passed Checks

- <check 1>: PASS
- <check 2>: PASS
...
```

### Severity Definitions

| Level | Meaning |
|-------|---------|
| **HIGH** | Plugin will not work correctly or violates a mandatory convention |
| **MEDIUM** | Plugin works but deviates from recommended practices |
| **LOW** | Minor improvement opportunity, cosmetic or non-functional |
| **PASS** | Check passed successfully |

## Step 7 — Present Results

1. Show the summary table first (severity counts)
2. Then list HIGH issues (these need attention)
3. Use `ask_user` to offer next steps:
   - "Should I show MEDIUM and LOW issues too?"
   - "Should I fix the HIGH issues automatically?"
   - "Should I generate a detailed report file?"

If user wants a report file, write it to the plugin directory:
`plugins/<plugin>/VALIDATION-REPORT.md`

## Error Handling

| Error | Action |
|-------|--------|
| Target directory not found | List available plugins, ask user to choose |
| No agents or skills found | Report as informational, not an error |
| File read permission denied | Skip file, report as warning |
| CONTRIBUTING.md not found | Use built-in convention rules |

## Guardrails

1. This workflow is **read-only** — it never modifies plugin files during validation
2. The only file it may write is the optional `VALIDATION-REPORT.md`
3. Never deletes files, even if they appear to be incorrect
4. Reports findings objectively — does not auto-fix without explicit user request
5. If asked to fix issues, delegates to `create-agent` or `create-skill` workflows
