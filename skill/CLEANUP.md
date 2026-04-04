# FitFoundry — Cleanup

**Run this when you want to reorganize your results folder, or to migrate from the flat results structure used before the 2026-04-04 core update.**

---

## WORKFLOW PROMPT

```
Read .env to load OUTPUT_FOLDER.

Read fitfoundry-history.json from the workspace root. Note the last_cleanup date if present.

---

STEP 0 — DETECT STRUCTURE

Inspect OUTPUT_FOLDER. Determine which structure is present:

A. **New structure** — OUTPUT_FOLDER contains subfolders: runs/, inbox/, shortlisted/, applied/, declined/, closed/
B. **Legacy structure** — OUTPUT_FOLDER contains individual job MD files (matching [Company]_[JobTitle]_[YYYY-MM-DD].md) or master list MD files (matching [YYYY-MM-DD]-*.md) directly in the root, not in subfolders.
C. **Mixed** — some files are in subfolders, some are not (partial migration or new files that landed flat due to a pre-update run).

Report which structure was detected and what you found (file counts by type). Then proceed to the appropriate step.

---

STEP 1A — MIGRATE LEGACY FILES (only if Structure B or C detected)

Create the following subfolders inside OUTPUT_FOLDER if they do not already exist:
- runs/
- inbox/
- shortlisted/
- applied/
- declined/
- closed/

**Move master list files:**
Any file in the root of OUTPUT_FOLDER matching the pattern [YYYY-MM-DD]-*.md is a run master list. Move it to OUTPUT_FOLDER/runs/.

**Move individual job report files:**
Any file in the root of OUTPUT_FOLDER matching the pattern [Company]_[JobTitle]_[YYYY-MM-DD].md is an individual job report. Read its Status: field from the header (first 10 lines of the file).

- If no Status: field is present, treat the status as `inbox`.
- Map the status value to the correct target folder:

  | Status value                    | Target folder  |
  |---------------------------------|----------------|
  | inbox (or missing)              | inbox/         |
  | shortlisted                     | shortlisted/   |
  | applied                         | applied/       |
  | interviewing                    | applied/       |
  | offer                           | applied/       |
  | declined                        | declined/      |
  | rejected                        | closed/        |
  | withdrawn                       | closed/        |
  | report-requested                | inbox/         |

Move each file to its target folder. Do not move files that are already inside a subfolder.

**Move existing application folders:**
If OUTPUT_FOLDER contains subdirectories named [Company]_[JobTitle] (without a date segment) that contain .docx files, these are LaunchKit application folders. Move them to OUTPUT_FOLDER/shortlisted/ unless the job report inside them has a Status field indicating otherwise (applied → applied/, etc.).

Report how many files and folders were moved in total.

---

STEP 1B — SYNC STATUS TO FOLDER (for new or mixed structure)

Scan all individual job report files across all subfolders of OUTPUT_FOLDER. For each file:

1. Read the Status: field from the header (first 10 lines).
2. Determine the expected folder based on the mapping table above.
3. Compare expected folder to the file's current location.
4. If the file is in the wrong folder, move it to the correct one.
   - If the target folder does not exist, create it.
   - If the file is inside an application folder (shortlisted/Company_JobTitle/), move the entire application folder, not just the report.

After scanning all files, report:
- Files already in the correct folder: [N]
- Files moved: [N], with a list of what moved where

If nothing needed to move, say so.

---

STEP 2 — CONFIRM AND UPDATE HISTORY

Summarize the current folder contents:

| Folder       | Files |
|--------------|-------|
| runs/        | [N] master list files |
| inbox/       | [N] job reports |
| shortlisted/ | [N] job reports / [N] application folders |
| applied/     | [N] application folders |
| declined/    | [N] job reports |
| closed/      | [N] application folders / job reports |

Update fitfoundry-history.json:
- Set last_cleanup to today's date (YYYY-MM-DD).
- Append a record to cleanup_history: { "date": "[YYYY-MM-DD]", "files_moved": [N], "notes": "[brief summary]" }

Confirm when complete.
```

---

## NOTES

- Cleanup is safe to run multiple times. It is idempotent — running it again after a clean run will report "0 files moved."
- Files inside application folders (shortlisted/Company_JobTitle/) are moved as a unit. The report file, resume, and cover letter stay together.
- The Status: field in each job report's header is the single source of truth for where a file belongs. Update that field manually when your job status changes, then run Cleanup to sync folders.
- Cleanup does not delete any files. It only moves them.
