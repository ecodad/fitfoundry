# LaunchKit — Application Asset Generator

**Paste this prompt into Cowork when you are ready to prepare resumes and cover
letters for specific jobs. Run after reviewing your FitFoundry results and deciding
which roles to pursue.**

---

## WORKFLOW PROMPT

```
Read .env to load OUTPUT_FOLDER and PROFILE_PATH.
- OUTPUT_FOLDER default: ./results/
- PROFILE_PATH default: ./MY-PROFILE.md

Read the candidate profile from PROFILE_PATH. Hold it in context for the full session.

---

STEP 0 — DISCOVER AVAILABLE JOBS

Scan OUTPUT_FOLDER for individual job files. These are markdown files matching the
pattern: [Company]_[JobTitle]_[YYYY-MM-DD].md

For each file found:
- Read the Score line from the file header.
- Check whether a corresponding application folder already exists in OUTPUT_FOLDER.
  An application folder is a subdirectory named [Company]_[JobTitle] (without the date).
  If the folder exists, LaunchKit has already been run for that job.

Build two lists:
- UNPROCESSED: job files with no corresponding application folder
- ALREADY_PROCESSED: job files that already have an application folder

Present the UNPROCESSED list as a numbered table:

  | # | Score | Title | Company | Date |
  |---|-------|-------|---------|------|

If ALREADY_PROCESSED is non-empty, note them separately:
  "These jobs already have application packages — let me know if you want to
  re-run any of them: [list]"

If no job files are found at all, stop and tell the user:
  "No individual job files found in [OUTPUT_FOLDER]. Run FitFoundry first to
  generate scored job reports, then come back to LaunchKit."

---

STEP 1 — JOB SELECTION

Ask the user:
  "Which jobs would you like to prepare application materials for? Pick by number.
  You can also add notes for specific jobs — for example:
  '2, 5, 8 — for 5, emphasize the hardware background; for 8, I know someone
  on the team so the cover letter should feel warmer.'"

Wait for the user's response.

Parse the response into:
- SELECTED_JOBS: list of selected job file references
- JOB_NOTES: dict of job index → user notes (empty string if none provided)

Confirm the selection:
  "Got it — preparing materials for: [Title | Company] × N jobs. Let me find
  your base documents."

---

STEP 2 — DISCOVER BASE DOCUMENTS

Scan the workspace for resume and cover letter files. Look for:
- File extensions: .docx, .pdf
- Filenames containing keywords: resume, cv, cover, letter, coverletter

Present what was found, grouped by type:

  Resumes found:
  - [filename] ([file extension])
  - ...

  Cover letters found:
  - [filename] ([file extension])
  - ...

If no files are found in either category, ask the user to provide a path or
upload the files before continuing.

If files are found, ask the user to confirm they are correct, or point to
different files if needed.

Hold the confirmed file list as:
- BASE_RESUMES: list of resume file paths
- BASE_COVER_LETTERS: list of cover letter file paths

---

STEP 3 — ASSESS DOCUMENT-JOB PAIRING

For each selected job, determine which base resume and cover letter to use.

Rules:
- If there is exactly one resume and one cover letter, pair all jobs to those.
  No question needed.
- If there are multiple resumes or cover letters, read the job description from
  the job file and the filename/content of each base document. Assess whether a
  clear match exists (e.g., a technical-focused resume for an engineering-heavy
  role, a PM-focused resume for a product role).
- If the match is clear, assign silently.
- If the match is genuinely ambiguous — the job could reasonably use more than
  one base document — flag the job as NEEDS_CLARIFICATION. Do not stop the run.

After assessing all selected jobs, note internally:
- CLEAR_PAIRS: jobs with confident document assignments
- NEEDS_CLARIFICATION: jobs where the right base document is not obvious

Proceed immediately to Step 4 for CLEAR_PAIRS. Do not wait for the user.

---

STEP 4 — GENERATE APPLICATION ASSETS (clear pairs first)

For each job in CLEAR_PAIRS, in order:

4a. Create the application folder.
    Path: OUTPUT_FOLDER/[Company]_[JobTitle]/
    Replace spaces with underscores. Remove special characters.

4b. Move the job report file into the folder.
    Original path: OUTPUT_FOLDER/[Company]_[JobTitle]_[YYYY-MM-DD].md
    New path:      OUTPUT_FOLDER/[Company]_[JobTitle]/[Company]_[JobTitle]_[YYYY-MM-DD].md

    After moving, insert the following status block into the file header,
    immediately after the **Ghost job check:** line:

    **Application Status:** Pending
    **Applied:** —
    **Decision Notes:** —

    Status values:
    - Pending   — assets generated, not yet submitted
    - Applied   — submitted; set Applied to the submission date (YYYY-MM-DD)
    - Declined  — decided not to pursue; record reason in Decision Notes

4c. Read the full job description from the report.
    Read the candidate profile from PROFILE_PATH.
    Read the assigned base resume and cover letter.
    Apply any job-specific notes from JOB_NOTES.

4d. Generate a tailored resume.

    The resume should:
    - Open the docx skill before generating any .docx file.
    - Synthesize content from all provided base resumes — treat them as a pool
      of experience, not a single document to copy.
    - Emphasize skills and experience most relevant to this specific role.
    - De-emphasize or omit experience that is not relevant.
    - Apply any job-specific user notes (e.g., "emphasize hardware background").
    - Be ATS-friendly: clear section headers, no tables for the main body,
      standard fonts.
    - Match the length and structure of the base resume(s) unless the job
      clearly warrants a different approach.

    Save as: OUTPUT_FOLDER/[Company]_[JobTitle]/Resume_[Company]_[JobTitle].docx

4e. Generate a tailored cover letter.

    The cover letter should:
    - Use the docx skill.
    - Synthesize from all provided base cover letters as tone and structure
      reference — do not copy.
    - Open with a specific hook tied to the company or role (not a generic opener).
    - Reference 2–3 concrete accomplishments from the resume that directly
      address the job's key requirements.
    - Address any stated gaps from the job report's "Potential Gaps" section
      if they are bridgeable.
    - Apply job-specific user notes (e.g., "warmer tone — I know someone on
      the team").
    - Be concise: three to four paragraphs, under one page.

    Save as: OUTPUT_FOLDER/[Company]_[JobTitle]/CoverLetter_[Company]_[JobTitle].docx

4f. Report progress after each job:
    "✅ [Company] — [Title]: folder created, resume and cover letter saved."

---

STEP 5 — RESOLVE AMBIGUOUS PAIRINGS

If NEEDS_CLARIFICATION is non-empty, ask all clarification questions at once
after the clear pairs are complete:

  "A few jobs had more than one plausible resume match. Here's what I need:

  [For each ambiguous job:]
  - [Company] — [Title] (score X): I have [Resume A] (emphasizes [summary])
    and [Resume B] (emphasizes [summary]). Which should I use as the primary
    base — or should I draw equally from both?
  "

Wait for the user's responses. Apply the answers and run Step 4 for each
remaining job.

---

STEP 6 — SUMMARY

When all jobs are complete, report:

  ## LaunchKit Complete

  [N] application packages created:

  | Company | Title | Score | Folder |
  |---------|-------|-------|--------|
  | ...     | ...   | ...   | ...    |

  ### Next Steps
  - Review each folder. Resume and cover letter are first drafts — edit before
    submitting.
  - To flag a job as an active pursuit, prefix its folder name with an underscore:
    _[Company]_[JobTitle]/
  - For deeper editing and refinement, open the .docx files in Claude Chat or
    your preferred editor.
  - Re-run LaunchKit any time on additional jobs from your FitFoundry results.
```

---

## NOTES

- LaunchKit detects already-processed jobs by folder presence. If you want to
  regenerate assets for a job, delete or rename its application folder first.
- The underscore prefix convention (_FolderName) is a lightweight way to flag
  active pursuits at a glance in your file system. It has no effect on LaunchKit
  behavior.
- LaunchKit does not submit applications or contact employers. All generated
  files require human review before use.
- If base documents are in PDF format, LaunchKit will read their content but
  output resumes and cover letters as .docx for easy editing. Keep the originals
  in the workspace for reference.
