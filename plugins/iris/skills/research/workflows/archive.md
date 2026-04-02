---
name: archive
description: >
  Archives completed research investigations to .research-archives/.
  Supports archiving individual runs or entire investigations. Shared across domains.
---

<purpose>
Archive completed research investigations by creating zip archives in
`.research-archives/` and cleaning up `.research/` directories. This is the
unified archive workflow for all domains (tech and health).

Request files in `.requests/` are permanent records and are NEVER deleted.
Their status field is updated to `archived` when all associated runs have
been archived.
</purpose>

<process>

<step name="list_archivable_investigations">

Scan request files for completed investigations:

```bash
echo "=== Completed Investigations ==="
grep -l "status: completed" .requests/research-*.md 2>/dev/null
```

For each matching request file, extract the topic, UUID(s), and last-run date
from the frontmatter and the `research-runs` list.

Present to user with `ask_user`:
- question:
  ```
  Completed investigations available for archival:

  1. [Topic A] -- N runs, last: YYYY-MM-DD
  2. [Topic B] -- N runs, last: YYYY-MM-DD
  ```
- choices:
  - "1" (or the number of the investigation)
  - "Archive all completed"
  - "Cancel"

If no completed investigations are found:
- Inform the user: "No completed investigations found. Nothing to archive."
- EXIT workflow

</step>

<step name="confirm_scope">

For the selected investigation, list all runs with details:

```bash
# For each UUID in the investigation's research-runs list
INVESTIGATION_DIR=".research/{uuid}"
echo "=== Run: {uuid} ==="
echo "Date: {date}"
find "${INVESTIGATION_DIR}" -type f 2>/dev/null | wc -l
du -sh "${INVESTIGATION_DIR}" 2>/dev/null
```

Present to user with `ask_user`:
- question:
  ```
  Investigation: {topic}
  Runs:
  - {uuid-1} ({date}) -- {file_count} files, {size}
  - {uuid-2} ({date}) -- {file_count} files, {size}

  Archive all runs or select specific ones?
  ```
- choices:
  - "Archive all runs"
  - "Select specific runs"
  - "Cancel"

If "Select specific runs", present a numbered list of runs and let the user
choose which to archive.

</step>

<step name="create_archive">

For each selected UUID, create a zip archive:

```bash
mkdir -p .research-archives

cd .research/{uuid} && zip -r "../../.research-archives/{topic-slug}-{uuid}.zip" . && cd ../..
```

Verify each archive was created successfully:

```bash
ls -la ".research-archives/{topic-slug}-{uuid}.zip"
```

If zip creation fails for any run, report the error and do NOT proceed with
cleanup for that run. Continue with remaining runs.

</step>

<step name="clean_up">

Before deleting, confirm with `ask_user`:
- question:
  ```
  Archives created successfully:
  - {topic-slug}-{uuid-1}.zip ({size})
  - {topic-slug}-{uuid-2}.zip ({size})

  Remove the original folders from .research/?
  ```
- choices:
  - "Yes, remove archived folders"
  - "No, keep originals"
  - "Cancel"

If user confirms removal:

```bash
rm -rf .research/{uuid}
```

Update the request file in `.requests/`:
- If ALL runs for the investigation were archived, set `status: archived`
- If only some runs were archived, update individual entries in
  `research-runs` to `status: archived`

Request files are NEVER deleted. They remain as permanent records.

</step>

<step name="confirm_completion">

Report results to the user:

```
Archived:
- {uuid-1} -> .research-archives/{topic-slug}-{uuid-1}.zip ({size})
- {uuid-2} -> .research-archives/{topic-slug}-{uuid-2}.zip ({size})

Remaining active runs: {count}
Investigation status: {archived | partial}
```

Use `ask_user`:
- question: "Archive complete. What would you like to do next?"
- choices:
  - "Archive another investigation"
  - "View all archives"
  - "That's all for now"

If "View all archives":
```bash
echo "=== Research Archives ==="
ls -lh .research-archives/*.zip 2>/dev/null
```

</step>

</process>

<guardrails>

### Rule 1: Write Path Restriction
This workflow may ONLY write to:
- `.research-archives/*.zip` (archive creation)

This workflow may ONLY delete from:
- `.research/{uuid}/` (removing archived investigation folders)

This workflow may ONLY modify:
- `.requests/research-*.md` (updating status fields)

### Rule 2: Request File Preservation
Request files in `.requests/` are NEVER deleted. They are permanent records.
Only the `status` field (and individual run statuses) may be updated.

### Rule 3: Confirmation Required
ALWAYS use `ask_user` to get explicit user confirmation before:
- Creating archive files
- Deleting folders from `.research/`
- Updating request file status

### Rule 4: Archive Immutability
Once a zip archive is created in `.research-archives/`, it MUST NOT be
modified, overwritten, or deleted by this workflow.

### Rule 5: No Privilege Escalation
NEVER use `sudo`, `su`, `doas`, or any privilege escalation command.

### Rule 6: No Network Requests
This workflow operates entirely on local files. No network access is needed
or permitted.

### Rule 7: Verify Before Delete
ALWAYS verify that the zip archive exists and has a non-zero size before
removing the source folder from `.research/`.

</guardrails>
