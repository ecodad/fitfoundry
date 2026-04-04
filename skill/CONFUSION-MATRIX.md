# FitFoundry — Confusion Matrix

**Run this weekly to measure scoring accuracy, surface patterns in false positives and false negatives, and get suggestions for improving your career profile and quick reference.**

---

## Background

FitFoundry scores every job it finds. The confusion matrix evaluates how well those scores predicted which jobs were actually worth pursuing. Over time, this feedback tightens the scoring calibration and prunes the "Not a Fit" and "Must Have" criteria in your career profile.

| Term | Definition |
|------|------------|
| True Positive (TP) | Scored ≥ 6, you decided to pursue it (shortlisted, applied, interviewing, offer, rejected, or withdrawn) |
| False Positive (FP) | Scored ≥ 6, you decided not to pursue it (declined) |
| False Negative (FN) | Scored ≤ 5 (no individual report), you later requested a full report for it |
| True Negative (TN) | Scored ≤ 5, you never requested a report (correctly excluded) |

Precision = TP / (TP + FP) — of the jobs FitFoundry flagged as worth reviewing, how many were you actually interested in?
Recall = TP / (TP + FN) — of the jobs worth reviewing, how many did FitFoundry catch?

---

## WORKFLOW PROMPT

```
Read .env to load OUTPUT_FOLDER, PROFILE_PATH, and QUICK_REFERENCE_PATH.

Read MY-PROFILE.md from PROFILE_PATH. Hold in context.
Read QUICK-REFERENCE.md from QUICK_REFERENCE_PATH. Hold in context.

Read fitfoundry-history.json from the workspace root.
- Note the last_confusion_matrix date. If it was less than 7 days ago, inform the user and ask whether to proceed anyway.
- Note prior runs in confusion_matrix_history for trend context.

Today's date is [AUTO-FILL FROM SYSTEM].

---

STEP 0 — COLLECT JOB DATA

Scan the following subfolders of OUTPUT_FOLDER for individual job report files
(matching [Company]_[JobTitle]_[YYYY-MM-DD].md):
- inbox/
- shortlisted/ (including files inside application subfolders)
- applied/ (including files inside application subfolders)
- declined/
- closed/

For each file found, read the header block (first 15 lines) and extract:
- Status: field
- Score: field
- Board: field
- Date: field
- Job title and company (from filename)

Build the INDIVIDUAL_JOBS list.

Then scan OUTPUT_FOLDER/runs/ for master list files (matching [YYYY-MM-DD]-*.md).
For each master list file, read the Scored Jobs table in the Session Summary section.
Collect all jobs listed there with a score ≤ 5 that do NOT appear in INDIVIDUAL_JOBS
(i.e., they were never given a full report). These are the True Negatives pool.
Also collect any master list rows marked "(Full report generated)" — these are
already in INDIVIDUAL_JOBS with Status: report-requested.

---

STEP 1 — CLASSIFY

Classify each job in INDIVIDUAL_JOBS:

TP: Status is one of — shortlisted, applied, interviewing, offer, rejected, withdrawn
FP: Status is — declined
FN: Status is — report-requested (was on master list only, user later requested full report)
UNREVIEWED: Status is — inbox (not yet reviewed; exclude from matrix counts)

TN count: number of master list entries with score ≤ 5 that have no individual file
and are not in INDIVIDUAL_JOBS (i.e., correctly excluded, never revisited).

Report:

UNREVIEWED jobs (excluded from matrix): [N]
  Note: [N] jobs in inbox/ have not been reviewed yet and are not counted.
  Review and update their Status fields, then re-run for a more complete picture.

Confusion Matrix:

| Predicted → | Positive (score ≥ 6) | Negative (score ≤ 5) |
|---|---|---|
| Actual Positive | TP: [N] | FN: [N] |
| Actual Negative | FP: [N] | TN: [N] |

Precision: [TP / (TP + FP)] ([X]%)
Recall:    [TP / (TP + FN)] ([X]%)

If TP + FP + FN = 0, stop and report: "Not enough reviewed jobs to compute a meaningful
confusion matrix. Classify some jobs in your inbox and re-run."

---

STEP 2 — PATTERN ANALYSIS

**False Positives (scored ≥ 6, you decided not to pursue):**

List all FP jobs with: Score, Board, Job Title, Company, Date.

Identify patterns across FP jobs. Look for:
- Common title keywords (e.g., "Manager", "Director", "Analyst")
- Common boards (e.g., all FPs from a particular board)
- Common company types (e.g., staffing firms, very large enterprises, early-stage startups)
- Score clustering (e.g., all FPs scored exactly 6 — suggesting the 6-threshold is too low)
- Skill or requirement patterns visible in the job titles

Summarize the top 1–3 patterns found, or state "No clear pattern" if the FPs are diverse.

**False Negatives (scored ≤ 5, you later wanted a full report):**

List all FN jobs with: Score, Board, Job Title, Company, Date.

Identify patterns:
- Were these clustered on a specific board?
- Did the titles or companies suggest a gap in your keyword list?
- Were they near the threshold (score 4–5) or far below it?

Summarize the top 1–2 patterns found, or state "No clear pattern."

**Trend (if prior CM runs exist in confusion_matrix_history):**

Compare current precision and recall to the prior run. Note direction (improving/declining)
and any significant shifts.

---

STEP 3 — GENERATE SUGGESTIONS

Based on the pattern analysis, generate specific, actionable suggestions. Each suggestion
must identify the exact change: which file, which section, and the proposed new or
removed text.

**Possible QUICK-REFERENCE.md suggestions:**
- Raise or lower the individual-file score threshold (e.g., from 6 to 7) if FPs
  cluster at the low end of the range.
- Add or modify a "Not a fit" keyword filter if a title keyword appears in multiple FPs.
- Add or modify a search keyword if a recurring FN title type suggests a gap in coverage.
- Adjust the board-specific score calibration for boards with high FP or FN rates.

**Possible MY-PROFILE.md suggestions:**
- Add a disqualifying criterion to the "Not a Fit" section if a company type or role
  type appears repeatedly in FPs.
- Expand a "Must Have" criterion if FNs suggest that certain role types are being
  undervalued by the current criteria.

Present suggestions in this format:

---
Suggestion [N]: [One-line summary]
File: [MY-PROFILE.md or QUICK-REFERENCE.md]
Section: [exact section name]
Change: [exact proposed addition, removal, or edit — quote the specific text]
Rationale: [one sentence explaining what pattern this addresses]
---

If no changes are warranted (precision and recall both above 80% and no clear patterns),
say so explicitly rather than generating weak suggestions.

---

STEP 4 — USER APPROVAL

Present all suggestions and ask:

"Would you like me to apply any of these changes? Reply with the suggestion numbers
to apply (e.g., '1, 3'), 'all' to apply all of them, or 'none' to skip.
You can also say 'modify [N]' to adjust a suggestion before applying."

Wait for the user's response.

If the user requests modifications, accept the revised text and confirm before applying.

Apply only the approved suggestions. For each applied change:
1. Edit the target file.
2. Confirm: "Applied suggestion [N] — [summary of change]."

If 'none', skip to Step 5.

---

STEP 5 — UPDATE HISTORY

Update fitfoundry-history.json:
- Set last_confusion_matrix to today's date (YYYY-MM-DD).
- Append a record to confusion_matrix_history:
  {
    "date": "[YYYY-MM-DD]",
    "tp": [N],
    "fp": [N],
    "fn": [N],
    "tn": [N],
    "unreviewed": [N],
    "precision": [X.XX],
    "recall": [X.XX],
    "suggestions_generated": [N],
    "suggestions_applied": [N],
    "notes": "[one sentence summary of key finding, if any]"
  }

Confirm when complete and remind the user to re-run ProfileBuilder if significant
changes were made to MY-PROFILE.md so that QUICK-REFERENCE.md is regenerated
from the updated profile.
```

---

## NOTES

- The confusion matrix is only as good as your Status fields. Jobs sitting in inbox/ are excluded from the count. Make a habit of reviewing inbox jobs and updating their status after each FitFoundry run.
- "report-requested" status is the signal for False Negatives. It is set automatically when you use the "Generate full report" feature at the end of a FitFoundry run, or manually if you add a job report for a job that scored below the individual-file threshold.
- Suggestions are always presented for review — no changes are made without your approval.
- If you applied changes to MY-PROFILE.md, re-run ProfileBuilder to keep QUICK-REFERENCE.md in sync.
