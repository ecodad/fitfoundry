# Applying FitFoundry Updates (Delta Packages)

This guide explains how to apply a FitFoundry delta package to your personal workspace using Claude in Cowork. Follow these instructions each time a new delta is released. The process is designed to preserve your personal run history and any site files you have customised or added yourself.

---

## What Is a Delta Package?

A delta package is a folder inside `deltas/` named by release date (e.g. `deltas/2026-03-18/`). Each delta contains:

- `CHANGES.md` — a summary of what has changed in this release
- `sites/` — new site files and updated versions of existing site files
- `JOB-BOARD-SITE-NOTES.md` — an updated copy of the board index

All `Last run` dates in delta files are set to `Never`. This is intentional — the delta does not know when you last ran a board. You preserve your own dates during the merge.

---

## Before You Start

1. Identify your active FitFoundry workspace folder (e.g. `fitfoundry-jon/` or similar). This is the folder that contains your `skill/` directory, your `resumes/`, and your `results/`.
2. Identify the delta folder you are applying (e.g. `deltas/2026-03-18/`).
3. Read `CHANGES.md` in the delta folder so you know what is included before touching any files.

---

## Step 1 — Apply New Site Files

New site files are boards that did not exist in your workspace at all. They are listed under **New Site Files** in `CHANGES.md`.

For each new site file in `delta/sites/`:

1. Check whether a file with the same name already exists in your `skill/sites/` folder.
   - **If it does not exist:** copy the file from the delta directly into your `skill/sites/`. No further action needed.
   - **If it already exists** (you added the board yourself before this delta): compare the two files. Keep whichever version has more recent run history or more complete notes — or manually merge the content. Preserve your `Last run` date.

---

## Step 2 — Apply Updated Site Files

Updated site files are existing boards with substantive changes to extraction logic, known issues, or workflow notes. They are listed under **Updated Site Files** in `CHANGES.md`, with a plain-English summary of what changed.

For each updated site file in `delta/sites/`:

1. Open your existing `skill/sites/[SiteName].md` and note your current `Last run` date.
2. Copy the delta version of the file into your `skill/sites/`, replacing the old version.
3. After copying, update the `Last run` field in your copy to the date you recorded in step 1. Do not leave it as `Never` if you have run the board.

> **Why this matters:** The delta file contains the corrected extraction logic, new quirks, and updated selectors discovered through real runs. Your `Last run` date is personal — it records when you last searched, not when the file was written.

---

## Step 3 — Merge JOB-BOARD-SITE-NOTES.md

The delta includes a full replacement copy of `JOB-BOARD-SITE-NOTES.md`. Applying it requires a careful merge rather than a direct overwrite, because your copy contains your personal `Last run` dates in the Known Boards table.

Perform the following steps:

1. Open both files side by side: your current `skill/JOB-BOARD-SITE-NOTES.md` and the delta's `JOB-BOARD-SITE-NOTES.md`.

2. **Add new rows:** For any board rows in the delta that are not in your copy, add them to the appropriate table section. The `Last run` in the new row will be `Never` — that is correct since you have not run it yet.

3. **Preserve existing Last run dates:** For rows that already exist in your copy, keep your existing `Last run` date. Do not replace your dates with `Never` from the delta.

4. **Apply content changes outside the table:** Any changes below the Known Boards tables (site file format template, adding a new board instructions, ATS identification tips, general browser scraping notes) can be applied by replacing those sections with the delta version. These sections do not contain personal data.

5. **Update method descriptions:** If a row already exists in your copy but the delta shows a changed `Method` column value (e.g. an updated tool or approach), update the method text while keeping your `Last run` date.

---

## Step 4 — Check for Skill File Updates

Occasionally a delta may also include updates to top-level skill files (`SKILL.md`, `FITFOUNDRY-WORKFLOW.md`, `QUICK-REFERENCE.md`, etc.). If any such files are included in the delta root (not inside `sites/`), check `CHANGES.md` for what changed, then apply those changes manually. These files are less likely to contain personal data, but review before overwriting.

---

## Step 5 — Verify

After applying the delta:

1. Open `skill/JOB-BOARD-SITE-NOTES.md` and confirm the new board rows are present and your existing `Last run` dates are intact.
2. Open one of the updated site files and confirm your `Last run` date is present and the new content (quirks, selectors, warnings) is there.
3. Spot-check one new site file to confirm it loaded correctly and has `Last run: Never`.

---

## Notes for Future Deltas

This process is the same regardless of the delta date. Each delta folder is self-contained and cumulative — you can apply them in order if you have missed multiple releases. When applying multiple deltas, apply them oldest-first so that a newer delta's updates to a site file take precedence over an older one.

If you are setting up FitFoundry for the first time from the repo (no existing workspace), you do not need to merge — simply copy all files from the latest delta directly into your `skill/` folder and begin from there.
