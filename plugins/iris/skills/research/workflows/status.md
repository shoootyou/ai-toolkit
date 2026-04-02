---
name: status
description: >
  Shows the status of all IRIS research investigations across all domains.
  Displays run history, agent completion, and disk usage. Shared across domains.
---

<purpose>
  Unified status dashboard for the IRIS research system. Scans all investigation
  request files and research output directories, then presents a cross-domain
  overview of every investigation regardless of whether it originated from the
  tech or health skill. Read-only -- this workflow never writes or modifies files.
</purpose>

<process>

  <step name="scan_investigations">
    ## Step 1: Scan Investigations

    Collect every request file and detect orphan research folders.

    ```bash
    # List all request files sorted by modification time
    ls -t .requests/research-*.md .requests/tech-*.md .requests/health-*.md .requests/teach-*.md 2>/dev/null
    ```

    For each file found, read its YAML frontmatter and extract:
    - `name` -- descriptive investigation name
    - `topic` -- research topic
    - `status` -- one of: initialized, researching, synthesizing, completed, failed
    - `research-runs` or `learning-runs` -- list of run UUIDs (array in frontmatter)
    - `created` -- ISO 8601 timestamp
    - `updated` -- ISO 8601 timestamp

    Determine the domain by inspecting the `agents` list or file prefix:
    - File starts with `tech-` or agents start with `tech-` --> domain is **tech**
    - File starts with `health-` or agents start with `health-` --> domain is **health**
    - File starts with `teach-` or agents start with `teacher-` --> domain is **teach**
    - If mixed or absent --> domain is **unknown**

    Then scan for orphan research folders:

    ```bash
    # List UUID folders under .research/ that are not referenced by any request file
    ls -d .research/*/ 2>/dev/null
    ```

    Compare folder names against all `research-runs` UUIDs collected from request
    files. Any folder not referenced is flagged as an orphan.

    If no request files and no research folders exist, inform the user:

    ```
    No investigations found. Use the research skill to start one.
    ```

    Then EXIT the workflow.
  </step>

  <step name="display_overview">
    ## Step 2: Display Overview

    Present a summary table covering all domains:

    ```
    Active Investigations:

    | # | Topic              | Domain | Status      | Runs | Last Updated |
    |---|--------------------|--------|-------------|------|--------------|
    | 1 | Copilot CLI        | tech   | completed   | 2    | 2026-04-04   |
    | 2 | Diabetes protocols | health | researching | 1    | 2026-04-05   |
    ```

    If orphan folders were detected, append a warning:

    ```
    Orphan research folders (not linked to any request):
      .research/a1b2c3d4/
    ```

    Calculate disk usage:

    ```bash
    du -sh .research/ 2>/dev/null
    du -sh .research-archives/ 2>/dev/null
    ```

    Display totals:

    ```
    Disk usage: .research/ = X MB, .research-archives/ = Y MB
    ```

    Use `ask_user` to offer choices. Build the choice list dynamically based on
    the number of investigations found (one choice per investigation):

    - "View details of investigation #1" (repeat for each)
    - "Continue an investigation" (delegates to continue.md workflow)
    - "Archive old investigations" (delegates to archive.md workflow)
    - "Done"

    Wait for the user's selection before proceeding.
  </step>

  <step name="detailed_view">
    ## Step 3: Detailed View

    If the user selects a specific investigation, read its request file and all
    associated run directories.

    For each run UUID listed in `research-runs`:

    ```bash
    # Check run directory exists and list agent outputs
    ls .research/{uuid}/agents/ 2>/dev/null
    # Count completed agent files
    ls .research/{uuid}/agents/*.md 2>/dev/null | wc -l
    ```

    Check whether a SUMMARY.md exists for the run:

    ```bash
    test -f .research/{uuid}/SUMMARY.md && echo "exists"
    ```

    Present the detailed dashboard:

    ```
    Investigation: Copilot CLI
    Request: .requests/research-550e8400.md
    Domain: tech
    Status: completed
    Created: 2026-04-04T16:50:00Z

    Runs:
    | Run | UUID     | Date             | Agents     | Status    |
    |-----|----------|------------------|------------|-----------|
    | 1   | a1b2c3d4 | 2026-04-04 16:50 | 12 agents  | completed |
    | 2   | e5f6g7h8 | 2026-04-04 18:00 | 8 agents   | completed |

    Latest SUMMARY: .research/e5f6g7h8/SUMMARY.md
    ```

    For health-domain investigations, append an evidence note:

    ```
    Note: This is research output, not medical advice. Verify all findings
    against authoritative clinical sources before any application.
    ```

    After displaying, use `ask_user` with:

    - "View the summary report" (if SUMMARY.md exists for latest run)
    - "View a specific agent's findings" (list agents, let user pick)
    - "Back to overview"
    - "Done"
  </step>

  <step name="disk_usage_warning">
    ## Step 4: Disk Usage Warning

    After displaying the overview or detail view, check total research size:

    ```bash
    du -sm .research/ 2>/dev/null | awk '{print $1}'
    ```

    If the value exceeds 10 (MB), append a warning:

    ```
    Your .research/ directory is X MB. Consider archiving completed
    investigations to free up space.
    ```

    This check runs automatically as part of Step 2 and Step 3 output. It does
    not require a separate user action.
  </step>

</process>

<guardrails>
  ## MANDATORY SAFETY RULES

  ### Rule 1: Read-Only Operations
  This workflow MUST NOT create, modify, or delete any files. It reads request
  files, lists directories, and computes disk usage. Nothing else.

  ### Rule 2: No Destructive Commands
  Do not use `rm`, `mv`, or any command that alters the filesystem. Archival
  and cleanup are handled by the archive workflow, not this one.

  ### Rule 3: No Privilege Escalation
  Do not use `sudo` or any elevated-privilege command. Do not make network
  requests. All data comes from local files.

  ### Rule 4: Health Domain Disclaimer
  When displaying results for health-domain investigations, always include the
  research-output disclaimer. Never present health research findings without it.
</guardrails>
