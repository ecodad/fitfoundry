# Applying FitFoundry Core Updates

This guide explains how to apply a FitFoundry **core delta** — an update to the workflow files, maintenance tools, or skill configuration — to your personal workspace using Claude in Cowork.

Core deltas are separate from site deltas. Site deltas (in `deltas/YYYY-MM-DD/`) add or update job board files and the board index. Core deltas (in `deltas/core/YYYY-MM-DD/`) update the workflow logic itself. Apply them independently; order does not matter between the two types.

---

## What Is a Core Delta?

A core delta folder (e.g., `deltas/core/2026-04-04/`) contains:

- `CHANGES.md` — summary of what changed and migration notes
- Updated workflow files (`FITFOUNDRY-WORKFLOW.md`, `LAUNCHKIT.md`, `SKILL.md`, etc.)
- New workflow files (`CLEANUP.md`, `CONFUSION-MATRIX.md`, etc.)
- New template files (`fitfoundry-history.json`, etc.)

Unlike site deltas, core workflow files contain no personal data (no run dates, no profile content). They can be replaced directly without a careful merge — the only exception is noted below for `fitfoundry-history.json`.

---

## Before You Start

1. Identify your active FitFoundry workspace folder (the one that contains `skill/`, `results/`, `resumes/`, and `.env`).
2. Identify the core delta folder you are applying (e.g., `deltas/core/2026-04-04/`).
3. Read `CHANGES.md` in that folder before touching any files. It will tell you exactly what changed and whether any manual steps are needed.

---

## Step 1 — Replace Updated Workflow Files

For each file listed under **Updated Files** in `CHANGES.md`:

1. Copy the file from the core delta folder into your workspace's `skill/` directory, replacing the existing version.
2. No merge is required. These files contain no personal data.

Example: if `FITFOUNDRY-WORKFLOW.md` is listed as updated, copy `deltas/core/2026-04-04/FITFOUNDRY-WORKFLOW.md` into `skill/FITFOUNDRY-WORKFLOW.md`.

---

## Step 2 — Add New Files

For each file listed under **New Files** in `CHANGES.md`:

1. Copy the file from the core delta folder into your workspace's `skill/` directory.
2. If the file already exists in your workspace (you added it manually before this delta), compare the two. Keep whichever is more complete, or merge manually.

**Special case — `fitfoundry-history.json`:** This file tracks your personal run history for the confusion matrix and cleanup tools. If it already exists in your workspace, do not replace it. Only copy it from the delta if it does not yet exist.

---

## Step 3 — Run Cleanup (if this delta introduced folder structure changes)

If `CHANGES.md` notes that this delta introduces a new results folder structure, run the Cleanup workflow once after applying the delta:

Open a Cowork session, select your job search folder, and say:

> Run FitFoundry Cleanup

Cleanup will detect your existing flat results structure and migrate files into the new subfolder layout (runs/, inbox/, shortlisted/, applied/, declined/, closed/). It is safe to run on an empty results folder — it will simply create the subfolders and confirm there is nothing to migrate.

---

## Step 4 — Verify

After applying the delta:

1. Open your `skill/` folder and confirm the new and updated files are present.
2. If `fitfoundry-history.json` was new in this delta, confirm it exists in your workspace root (alongside `.env`).
3. Run a quick FitFoundry session or the Cleanup workflow to confirm everything loads correctly.

---

## Notes for Future Core Deltas

This process is the same for all core deltas. Each core delta is self-contained. If you have missed multiple releases, apply them oldest-first so that a newer delta's changes take precedence. Read each `CHANGES.md` in order — earlier deltas may have migration notes that are prerequisites for later ones.
