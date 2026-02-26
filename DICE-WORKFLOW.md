# Dice Job Search Workflow

**Paste this prompt into a new Cowork session to run a Dice search. The Dice MCP connector must be active before the session starts — connecting mid-session will not work.**

**Key difference from Indeed:** Dice MCP only provides `search_jobs`. Full job descriptions are retrieved via Puppeteer on individual job URLs returned by the search.

---

## WORKFLOW PROMPT

```
Read the environment configuration from .env to load PROFILE_PATH, QUICK_REFERENCE_PATH, and OUTPUT_FOLDER.

Read my full professional profile from the path specified in PROFILE_PATH. Hold it in context for the entire workflow — you will use it to assess fit and interest for every job.

Read my quick reference from the path specified in QUICK_REFERENCE_PATH. Hold it in context for the entire workflow — it contains your scoring calibration table and board-specific search settings.

Today's date is [AUTO-FILL FROM SYSTEM].

---

STEP 0 — CONFIRM CONNECTOR AND SET PARAMETERS

Confirm the Dice MCP connector is available by attempting to call search_jobs with a minimal test query. If it fails, stop and tell the user the connector is not active — they will need to start a new session with the connector connected.

Set the following run parameters from the Dice section of QUICK_REFERENCE_PATH:
- LOCATION: Boston, MA
- EMPLOYMENT_TYPE: FULLTIME
- OUTPUT_FOLDER: from .env (default ./results/)
- OUTPUT_FILENAME: [YYYY-MM-DD]-Dice.md
- QUERY_LIST: ["technical program manager", "operations program manager", "senior program manager"]

---

STEP 1 — SEARCH AND COLLECT

Run each query in QUERY_LIST using search_jobs with LOCATION and EMPLOYMENT_TYPE. Collect all results into a single deduplicated list using job ID as the unique key. Where the same job appears across multiple queries, keep one entry and note which queries matched it.

For each result, record:
- Job ID
- Title
- Company
- Location (as returned)
- Salary (if listed)
- Job URL
- Queries that matched it

Do not fetch any job descriptions yet. Complete all three searches and deduplicate first.

Report: [N] unique jobs found across [N] searches.

---

STEP 2 — PRE-SCREEN BY TITLE

Before fetching any descriptions, pre-screen the deduplicated list by title against my profile.

Exclude titles that clearly signal a poor fit:
- Pure software engineering (Software Engineer, SWE, Developer, Architect without program/ops context)
- Early-career indicators (Associate, Entry Level, Junior, I/II without Senior context)
- Deep technical specialty I don't have (Principal Scientist, Research Scientist, Data Scientist)
- No ops/PM relevance (Recruiter, Designer, Account Executive, Sales Rep)

Also flag — but keep — any posting where the company name appears to be a staffing firm or recruiter rather than the direct employer. These are lower priority but worth reviewing.

Keep anything ambiguous. Err on the side of inclusion.

Report: [N] jobs remaining after title pre-screen. List excluded titles briefly for verification.

---

STEP 3 — SAVE THE MASTER LIST

Create the file: OUTPUT_FOLDER/[YYYY-MM-DD]-Dice.md

Format as a markdown table with all pre-screened jobs:

| # | Title | Company | Location | Salary | Staffing Firm? | Link |
|---|-------|---------|----------|--------|----------------|------|

Mark the Staffing Firm? column Yes/No/Unknown based on whether the posting company appears to be a recruiter. Save the file before proceeding.

---

STEP 4 — FETCH FULL DESCRIPTIONS VIA PUPPETEER

For each job remaining after pre-screen, navigate to its job URL using Puppeteer to retrieve the full description.

Use the selector: [data-cy="jobDescription"]
If that selector returns nothing, fall back to: .job-description, [class*="description"]
If both fail, capture document.body.innerText and extract the description section manually.

Note: Dice is a React app — if the page returns empty on first load, wait 1–2 seconds and re-query the selector.

---

STEP 5 — SCORE

Assess each job against my profile for interest and fit. Score 1–10 using the calibration table from QUICK_REFERENCE_PATH:
- 9–10: Dream job — create individual file
- 7–8: Strong fit — create individual file
- 6: Partial fit, one significant gap — create individual file
- 5: Multiple gaps or weak alignment — master list only
- 4: Marginal — master list only
- ≤3: Poor fit — skip

Score all jobs before creating any individual files. Update the master list table with a Score column.

---

STEP 6 — CREATE INDIVIDUAL FILES

For each job scoring 6 or higher, create a markdown file.

Filename format: OUTPUT_FOLDER/[Company]_[JobTitle]_[YYYY-MM-DD].md
Replace spaces with underscores. Remove special characters.

File contents:

---

# [Job Title] — [Company]

**Score:** [X/10]
**Company:** [Company]
**Title:** [Job Title]
**Location:** [Location]
**Salary:** [Salary if listed, otherwise "Not listed"]
**Type:** Full-time
**Source:** Dice (MCP)
**Apply:** [Job URL]
**Found:** [YYYY-MM-DD]
**Staffing firm posting:** [Yes / No / Unknown]

**Company career site:** [leave blank — filled in Step 7]
**Ghost job check:** [leave blank — filled in Step 7]

---

## Why This Role

[2–3 sentences on why this role matches my profile — specific about skill alignment and what appeals.]

## Potential Gaps

[1–2 sentences on requirements I don't fully meet. Write "None identified" if none.]

---

## Full Job Description

[Full description text as retrieved via Puppeteer.]

---

---

STEP 7 — GHOST JOB CHECK

For each individual job file created in Step 6, attempt to verify the posting on the company's own career site. Skip this step for postings flagged as staffing firm listings — the ATS is the staffing firm's, not the employer's.

How to find the company's ATS:
1. Visit the company's website and look for a Careers or Jobs link.
2. Check the subdomain to identify the ATS:
   - *.wd5.myworkdayjobs.com → Workday (use CXS POST API — see JOB-BOARD-SITE-NOTES.md)
   - jobs.lever.co/* → Lever (navigate to jobs.lever.co/[slug] and search by title)
   - boards.greenhouse.io/* → Greenhouse (query boards-api.greenhouse.io/v1/boards/[slug]/jobs)
   - jobs.ashbyhq.com/* → Ashby (navigate and search)
3. Use Puppeteer for all career site navigation — the VM egress proxy blocks most sites but Puppeteer bypasses it.
4. If Puppeteer is blocked (Cloudflare, etc.), use Claude in Chrome as a fallback if the extension is connected.

Update each individual file header with one of:

If confirmed:
**Company career site:** [direct URL to the posting]
**Ghost job check:** ✅ Confirmed live on [ATS name] ([YYYY-MM-DD])

If not found:
**Company career site:** [company careers page URL if found]
**Ghost job check:** ⚠️ Possible Ghost Job — posting not found on company career site ([YYYY-MM-DD])

If inaccessible:
**Company career site:** [URL attempted, if known]
**Ghost job check:** ❓ Unverified — career site inaccessible ([YYYY-MM-DD])

Keep all files regardless of outcome.

---

STEP 8 — SUMMARY REPORT

Append a summary section to the bottom of OUTPUT_FOLDER/[YYYY-MM-DD]-Dice.md:

---

## Session Summary

- Searches run: [queries]
- Total unique jobs found: [N]
- Jobs passing title pre-screen: [N]
- Staffing firm postings flagged: [N]
- Jobs scored: [N]
- Jobs skipped (score ≤ 3): [N]
- Individual files created (score 6+): [N]

### Scored Jobs (sorted high to low)

| Score | Title | Company | Staffing Firm? | Ghost Check | File |
|-------|-------|---------|----------------|-------------|------|

---

## Session Notes

[Puppeteer selector issues, unexpected results, ATS systems encountered, connector behavior, etc.]

---

Confirm when all files have been saved.
```

---

## NOTES FOR FUTURE RUNS

- **Connector must be active at session start.** If Step 0 search fails, restart the session.
- **No `get_job_details` in Dice MCP.** Puppeteer is required for full descriptions. If the `[data-cy="jobDescription"]` selector breaks, re-detect the correct selector from the live page and update `sites/Dice.md`.
- **Staffing firm noise.** Dice has a higher proportion of recruiter/staffing firm postings than Indeed. Flagging these in the master list helps prioritise direct-employer postings.
- **Update Last Run date** in `JOB-BOARD-SITE-NOTES.md` Board #15 after each run.
