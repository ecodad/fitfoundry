# Core Delta: 2026-04-04

Apply this delta using the instructions in `DELTA-APPLY-CORE.md` at the repo root.

This is a core functionality delta. It updates workflow files and adds maintenance
tools. It does not contain site files — for site updates, see `deltas/YYYY-MM-DD/`.

---

## Summary

This release introduces three enhancements:

1. **Results folder structure** — individual job reports and master list files now
   land in organized subfolders from the start of each run (`inbox/` and `runs/`
   respectively). LaunchKit outputs go to `shortlisted/`.

2. **Status field** — each individual job report gets a machine-readable header block
   (`Status`, `Score`, `Board`, `Date`) that drives the new maintenance workflows.

3. **Confusion Matrix and Cleanup maintenance tools** — a weekly feedback loop to
   measure scoring accuracy and improve the career profile, plus a cleanup tool to
   migrate legacy flat structures and sync files to the correct folder by status.

---

## Updated Files

Replace these files in your `skill/` directory. They contain no personal data and
are safe to overwrite directly. See `DELTA-APPLY-CORE.md` for instructions.

### `FITFOUNDRY-WORKFLOW.md`

- Master list files now write to `OUTPUT_FOLDER/runs/` (created automatically if missing).
- Individual job reports now write to `OUTPUT_FOLDER/inbox/` (created automatically if missing).
- Individual file format gains a header block at the top of each file:
  ```
  Status: inbox
  Score: [X]
  Board: [BoardName]
  Date: [YYYY-MM-DD]
  ```
- Date format enforcement added: all dates written to any file must be YYYY-MM-DD.
  Relative dates (e.g., "3 days ago") must be converted to absolute before saving.
- New **STEP REVIEW** added at the end of each run: presents the full scored jobs
  table and offers to "Generate full report" for any master-list-only job the user
  wants to examine further. Generated reports land in `inbox/` with
  `Status: report-requested` (the False Negative signal for the Confusion Matrix).

### `LAUNCHKIT.md`

- Now scans `OUTPUT_FOLDER/inbox/` for unprocessed job reports instead of the flat
  `OUTPUT_FOLDER/` root.
- Application folders are now created in `OUTPUT_FOLDER/shortlisted/[Company]_[JobTitle]/`
  instead of `OUTPUT_FOLDER/[Company]_[JobTitle]/`.
- When LaunchKit processes a job, it updates `Status: inbox` → `Status: shortlisted`
  in the file header. The old `Application Status` block is replaced by the Status field.
- Next Steps instructions updated to reflect status-based folder workflow.

### `SKILL.md`

- CM weekly check added to Stage 2 startup: if `last_confusion_matrix` in
  `fitfoundry-history.json` is null or more than 7 days old, a brief advisory
  reminder is surfaced at the start of the session.
- Key Files table updated with new maintenance files.
- New Maintenance Workflows section added with triggers for Cleanup and Confusion Matrix.
- Output Format section updated to document the new subfolder structure and Status field.

> **Existing users:** `SKILL.md` is read from the installed skill bundle, not the workspace.
> These changes only take effect after reinstalling the `.skill` file in Claude Desktop.
> This is optional — all other files in this delta apply immediately without a reinstall.

---

## New Files

Copy these files into your `skill/` directory.

### `CLEANUP.md`

A manual maintenance workflow that:
- Detects whether your results folder uses the new subfolder structure or the legacy flat layout.
- Migrates legacy flat files: master list MDs → `runs/`, individual job reports → the correct subfolder based on their `Status:` field (or `inbox/` if no Status field is present).
- For new-structure workspaces: scans all subfolders and moves any files whose `Status:` field does not match their current folder.
- Updates `fitfoundry-history.json` with the cleanup date.

### `CONFUSION-MATRIX.md`

A weekly maintenance workflow that:
- Reads all individual job reports across all result subfolders and classifies them as TP, FP, FN, or unreviewed (inbox).
- Reads run files in `runs/` to count True Negatives (master-list-only jobs never revisited).
- Computes precision and recall. Surfaces patterns in false positives and false negatives.
- Generates specific, quoted suggestions for changes to `QUICK-REFERENCE.md` and `MY-PROFILE.md`.
- Applies suggestions only after user approval.
- Updates `fitfoundry-history.json` with the run date and metrics.

### `fitfoundry-history.json`

Template tracking file for last-run dates and run history. **Only copy this file if it
does not already exist in your workspace root.** If it already exists, do not overwrite it.

---

## Migration Step (Required for Existing Workspaces)

After applying this delta, run Cleanup once to migrate your existing results:

> Run FitFoundry Cleanup

Cleanup will detect your flat `results/` structure, create the new subfolders, and
move files to their correct locations. Individual job reports without a `Status:` field
will be moved to `inbox/`. Master list files will move to `runs/`.

This step is one-time. After migration, all future FitFoundry runs will write to the
correct subfolders automatically.
