# FitFoundry — Job Board Search

> **Stage 2 of 3.** FitFoundry is part of a three-stage workflow:
> - **Stage 1 — ProfileBuilder** (`PROFILEBUILDER.md`): Run once to create your profile and quick reference from an interview. Re-run when your situation changes.
> - **Stage 2 — FitFoundry** (this file): Run repeatedly to search job boards and score listings against your profile.
> - **Stage 3 — LaunchKit** (`LAUNCHKIT.md`): Run after reviewing results to generate tailored resumes and cover letters for jobs you want to pursue.
>
> Complete Stage 1 before your first FitFoundry run.

**Paste this prompt into Cowork each time you run it. The agent will guide you through board selection before starting.**

---

## WORKFLOW PROMPT

```
Read the environment configuration from .env to load PROFILE_PATH, QUICK_REFERENCE_PATH, and OUTPUT_FOLDER.

Read my full professional profile from the path specified in PROFILE_PATH. Hold it in context for the entire session — you will use it to assess fit and interest for every job.

Read my quick reference from the path specified in QUICK_REFERENCE_PATH. Hold it in context for the entire session — it contains your scoring calibration table, credentials, and board-specific search settings.

Read JOB-BOARD-SITE-NOTES.md and hold it in context.

Today's date is [AUTO-FILL FROM SYSTEM].

---

STEP 0 — SESSION SETUP

Present the known job boards from JOB-BOARD-SITE-NOTES.md as a numbered list showing: board label, last run date, and run type.

Ask the user:
1. Which board do you want to run? (pick a number, or say "new board" and provide the URL)
2. What run type? (Remote / In-Person / All)
3. Any label suffix to add to the output filename? Suggest a default based on run type.

Wait for answers before proceeding.

Set variables:
- JOB_BOARD_NAME: board label (e.g., Indeed, Climatebase)
- RUN_LABEL: full filename label (e.g., Indeed-Remote, ClimateBase-In-Person)
- WORK_TYPE: from user answer
- OUTPUT_FOLDER: from .env (default ./results/)
- DEFAULT_LOCATION: from QUICK_REFERENCE_PATH → Candidate Profile → Default search location

Load sites/[BoardName].md for the selected board.

Check the Scraping method field in the loaded sites/ file:
- If it contains "MCP connector" → follow SECTION A (MCP Connector Flow) below.
- Otherwise → follow SECTION B (Puppeteer Flow) below.

---

SECTION A — MCP CONNECTOR FLOW
(For boards whose sites/ file shows Scraping method: MCP connector)

STEP A1 — CONFIRM CONNECTOR AND SET PARAMETERS

Read the MCP Workflow Notes section in the loaded sites/ file for this board — it describes the connector validation step, search tool, and any board-specific differences from this generic flow.

Confirm the connector is active using the validation method described in the sites/ file. If validation fails, stop and tell the user the connector is not active — they will need to start a new session with the connector connected before proceeding.

Set search parameters from QUICK_REFERENCE_PATH (the section for this board):
- QUERY_LIST: keywords for this board
- SEARCH_LOCATION: board-specific location if set, otherwise DEFAULT_LOCATION
- EMPLOYMENT_TYPE: as specified in the board's QUICK_REFERENCE_PATH section
- OUTPUT_FILENAME: [YYYY-MM-DD]-[JOB_BOARD_NAME].md

---

STEP A2 — SEARCH AND COLLECT

Run each keyword in QUERY_LIST using the board's search tool. Collect all results into a single deduplicated list using the job ID as the unique key. Where the same job appears across multiple queries, keep one entry and note which queries matched it.

For each result, record the fields listed in the sites/ file Per-Job Fields table.

Report: [N] unique jobs found across [N] searches.

---

STEP A3 — PRE-SCREEN BY TITLE

Before fetching full descriptions, review the deduplicated title list against the target roles and "Not a fit" constraints in your profile. Exclude titles that clearly do not match — err on the side of inclusion for anything ambiguous.

Also apply any board-specific pre-screen logic described in the sites/ file MCP Workflow Notes (e.g., flagging staffing firm postings by employerType on Dice).

Report: [N] jobs remaining after pre-screen. List excluded titles briefly for verification.

---

STEP A4 — SAVE THE MASTER LIST

Create the file: OUTPUT_FOLDER/[YYYY-MM-DD]-[JOB_BOARD_NAME].md

Use the master list column format from the sites/ file MCP Workflow Notes. Save before proceeding.

---

STEP A5 — FETCH FULL DESCRIPTIONS

Use the description retrieval method described in the sites/ file MCP Workflow Notes. Retrieve descriptions only for jobs that survived the title pre-screen.

---

STEP A6 — SCORE

Assess each job against PROFILE_PATH for interest and fit. Score 1–10 using the calibration table from QUICK_REFERENCE_PATH. Update the master list table with a Score column.

Do not create individual files yet — complete scoring for all jobs first.

---

STEP A7 — OPTIONAL ENRICHMENT (board-specific)

Check the sites/ file MCP Workflow Notes for any enrichment step that applies to this board (e.g., company research via get_company_data on Indeed for scores ≥7). If described, run it for qualifying jobs before creating individual files.

---

STEP A8 — CREATE INDIVIDUAL FILES

For each job scoring 6 or higher, create a markdown file.

Filename: OUTPUT_FOLDER/[Company]_[JobTitle]_[YYYY-MM-DD].md
Replace spaces with underscores. Remove special characters.

Use the Individual File Format from the COMMON REFERENCE section below, plus any additional fields listed in the sites/ file MCP Workflow Notes for this board.

Then proceed to STEP GHOST and STEP SUMMARY in the COMMON REFERENCE section.

---

SECTION B — PUPPETEER FLOW
(For all boards that use Puppeteer JS, Puppeteer API, or Claude in Chrome)

STEP B1 — SCRAPE THE JOB BOARD

Navigate to the board URL from the sites/ file. Apply the site-specific scraping approach documented there.

Scroll through all available listings (paginate if needed). For each listing, collect:
- Job Title
- Company
- Location (as listed)
- Workplace type (In-person / Hybrid / Remote)
- Brief description (first 2–3 sentences or the posted summary)
- Direct link to the full job posting

Filter: keep only listings where location matches DEFAULT_LOCATION OR workplace type matches WORK_TYPE. Discard contract, part-time, and internship listings.

---

STEP B2 — SAVE THE MASTER LIST

Create the file: OUTPUT_FOLDER/[YYYY-MM-DD]-[RUN_LABEL].md

Format as a markdown table:
| # | Title | Company | Location | Workplace | Description | Link |
|---|-------|---------|----------|-----------|-------------|------|

Number rows sequentially. Save before proceeding.

---

STEP B3 — SCORE EACH JOB

For each job in the master list:
1. Navigate to the full posting link and read the complete job description.
2. Assess interest (would I genuinely want this role?) and fit (does my background match?) against PROFILE_PATH.
3. If both are positive, assign a score 1–10 using the calibration table in QUICK_REFERENCE_PATH.
4. If interest or fit is clearly negative, skip — do not create an individual file.

---

STEP B4 — CREATE INDIVIDUAL FILES

For each job that received a score, create a markdown file using the Individual File Format from the COMMON REFERENCE section below.

Then proceed to STEP GHOST and STEP SUMMARY.

---

COMMON REFERENCE

STEP GHOST — GHOST JOB CHECK

For each individual job file created, attempt to verify the posting on the company's own career site.

How to find and query the company's ATS:
1. Visit the company's website and look for a Careers or Jobs link.
2. Identify the ATS platform from the URL pattern and query it using the approach documented in `JOB-BOARD-SITE-NOTES.md` → **ATS Identification Tip** (covers Workday CXS API, Lever, Greenhouse, Ashby, and others).
3. Use Puppeteer for all career site navigation — the VM egress proxy blocks most career sites, but Puppeteer bypasses it.
4. If the career site is blocked even by Puppeteer (e.g., Cloudflare-protected pages), use Claude in Chrome as a fallback if the extension is connected.

For MCP connector boards: skip this step for postings flagged as staffing firm listings — the ATS is the staffing firm's, not the employer's.

Update each individual job file header with one of:

If confirmed:
**Company career site:** [direct URL to the posting on the company's ATS]
**Ghost job check:** ✅ Confirmed live on [ATS name] ([YYYY-MM-DD])

If not found:
**Company career site:** [company careers page URL if found]
**Ghost job check:** ⚠️ Possible Ghost Job — posting not found on company career site ([YYYY-MM-DD])

If inaccessible:
**Company career site:** [URL attempted, if known]
**Ghost job check:** ❓ Unverified — career site inaccessible ([YYYY-MM-DD])

Keep all files regardless of outcome. The check is advisory.

---

STEP SUMMARY — SESSION SUMMARY REPORT

Append to the bottom of the master list file:

---

## Session Summary

- Board: [JOB_BOARD_NAME]
- Total listings reviewed: [N]
- Listings passing filter/pre-screen: [N]
- Jobs scored: [N]
- Jobs skipped (low interest, poor fit, or score ≤ 3): [N]
- Individual files created (score ≥ 6): [N]

### Scored Jobs (sorted high to low)

| Score | Title | Company | Ghost Check | File |
|-------|-------|---------|-------------|------|

---

## Session Notes

[Scraping quirks, unexpected results, new ATS systems encountered, connector behavior, selector changes, or anything worth logging for future runs.]

---

Confirm when all files have been saved.

---

INDIVIDUAL FILE FORMAT

Filename: OUTPUT_FOLDER/[Company]_[JobTitle]_[YYYY-MM-DD].md

---

# [Job Title] — [Company]

**Score:** [X/10] — [one sentence rationale]

**Company:** [Company]
**Title:** [Job Title]
**Location:** [Location] | **Workplace:** [In-person / Hybrid / Remote]

**Apply:** [direct link to the job posting]
**Company career site:** [leave blank — filled in ghost job check]
**Ghost job check:** [leave blank — filled in ghost job check]

[Include additional fields listed in the board's sites/ file MCP Workflow Notes, if applicable — e.g., Source, Type, Found, Staffing firm posting, Company Notes for Indeed high scores.]

---

## Why This Role

[2–3 sentences on why this role matches the profile — specific about skill alignment and what appeals based on stated preferences.]

## Potential Gaps

[1–2 sentences on any requirements not fully met. Write "None identified" if there are no meaningful gaps.]

---

## Full Job Description

[Full description text as scraped or retrieved.]

---
```

---

## IMPROVEMENT LOG

### Ghost Job Check — ATS Patterns Reference

See `JOB-BOARD-SITE-NOTES.md` → **ATS Identification Tip** for the authoritative ATS patterns reference (Workday CXS API, Lever, Greenhouse, and the full URL pattern table). That section is the single source of truth for ATS detection and query patterns.

**Egress proxy reminder:** Career sites are typically blocked by the VM's network egress proxy. Always use Puppeteer for career site navigation. If Puppeteer is also blocked (e.g., Cloudflare-protected pages), fall back to Claude in Chrome if the extension is connected.

---

### General Browser Scraping Notes

See `JOB-BOARD-SITE-NOTES.md` → **General Browser Scraping Notes** for Puppeteer variable scope, LinkedIn short URL handling, Dover ATS limitations, and other cross-board scraping reference material.

---

## TIPS FOR REUSE

- **To run a different board:** Select it at Step 0. The workflow loads that board's sites/ file and adapts automatically.
- **To adjust filters:** Edit the filter criteria in QUICK_REFERENCE_PATH for the relevant board section.
- **To adjust scoring:** Edit the scoring calibration table in QUICK_REFERENCE_PATH.
- **Your profile is the source of truth.** Update PROFILE_PATH content when your situation changes — the workflow adapts automatically.
- **To add a new board:** Select "new board" at Step 0, provide the URL, and document the scraping approach in a new `sites/` file using the standard format in `JOB-BOARD-SITE-NOTES.md`. Add a row to the Known Boards table there, then log the approach in the Improvement Log above.
