# Indeed Job Search Workflow

**Paste this prompt into a new Cowork session to run an Indeed search. The Indeed MCP connector must be active before the session starts — connecting mid-session will not work.**

---

## WORKFLOW PROMPT

```
Read the environment configuration from .env to load PROFILE_PATH, QUICK_REFERENCE_PATH, and OUTPUT_FOLDER.

Read my full professional profile from the path specified in PROFILE_PATH. Hold it in context for the entire workflow — you will use it to assess fit and interest for every job.

Read my quick reference from the path specified in QUICK_REFERENCE_PATH. Hold it in context for the entire workflow — it contains your scoring calibration table, credentials, and board-specific search settings.

Today's date is [AUTO-FILL FROM SYSTEM].

---

STEP 0 — CONFIRM CONNECTOR AND SET PARAMETERS

Confirm the Indeed MCP connector is available by attempting to call get_resume(). If it fails, stop and tell the user the connector is not active — they will need to start a new session with the connector connected.

If successful, note any experience retrieved from the Indeed profile and hold it as supplementary context for scoring.

Set the following run parameters from the Indeed section of QUICK_REFERENCE_PATH:
- LOCATION: Boston, MA
- EMPLOYMENT_TYPE: fulltime
- OUTPUT_FOLDER: from .env (default ./results/)
- OUTPUT_FILENAME: [YYYY-MM-DD]-Indeed.md
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
- Application URL
- Queries that matched it (for deduplication tracking)

Do not call get_job_details yet. Complete all three searches and deduplicate first.

Report the total count: [N] unique jobs found across [N] searches.

---

STEP 2 — PRE-SCREEN BY TITLE

Before visiting any job details, pre-screen the deduplicated list by title against my profile.

Exclude any job where the title clearly signals a poor fit:
- Titles indicating pure software engineering (Software Engineer, SWE, Developer)
- Titles indicating early-career level (Associate, Entry Level, Junior, I/II without Senior context)
- Titles indicating deep technical specialty I don't have (Principal Scientist, Research Scientist, etc.)
- Titles with no ops/PM relevance (Recruiter, Designer, Sales Rep, etc.)

Keep anything ambiguous — err on the side of inclusion at this stage. A borderline title gets included; a clearly irrelevant title gets excluded.

Report: [N] jobs remaining after title pre-screen. List excluded titles briefly so I can verify nothing was wrongly dropped.

---

STEP 3 — SAVE THE MASTER LIST

Create the file: OUTPUT_FOLDER/[YYYY-MM-DD]-Indeed.md

Format as a markdown table with all pre-screened jobs:

| # | Title | Company | Location | Salary | Link |
|---|-------|---------|----------|--------|------|

Number rows sequentially. Save the file before proceeding.

---

STEP 4 — FETCH DETAILS AND SCORE

For each job remaining after pre-screen, call get_job_details(job_id) to retrieve the full description, requirements, qualifications, and company information.

Assess each job against my profile for two things:
- Interest: Would I genuinely want this role given my goals and preferences?
- Fit: Do my background and skills match what the posting requires?

Score each job 1–10 using the calibration table from QUICK_REFERENCE_PATH:
- 9–10: Dream job — create individual file
- 7–8: Strong fit — create individual file
- 6: Partial fit, one significant gap — create individual file
- 5: Multiple gaps or weak alignment — master list only
- 4: Marginal — master list only
- ≤3: Poor fit — skip entirely

Update the master list table to add a Score column. Do not create individual files yet — complete scoring for all jobs first, then create files in a single pass.

---

STEP 5 — COMPANY RESEARCH FOR TOP SCORES

For each job scoring 7 or higher, call get_company_data(company) to retrieve Indeed's data on employee satisfaction, compensation, culture, management, and reviews. Hold this data in context — you will include a Company section in the individual file.

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
**Source:** Indeed (MCP)
**Apply:** [Application URL]
**Found:** [YYYY-MM-DD]

**Company career site:** [leave blank — filled in Step 7]
**Ghost job check:** [leave blank — filled in Step 7]

---

## Why This Role

[2–3 sentences on why this role matches my profile — specific about skill alignment and what appeals based on my stated goals and preferences.]

## Potential Gaps

[1–2 sentences on any requirements I don't fully meet. Write "None identified" if there are no meaningful gaps.]

## Company Notes

[If get_company_data was called: 2–3 sentences summarizing Indeed's data on culture, satisfaction, and compensation. If not called (score < 7): omit this section.]

---

## Full Job Description

[Full description text as returned by get_job_details.]

---

---

STEP 7 — GHOST JOB CHECK

For each individual job file created in Step 6, attempt to verify the posting on the company's own career site.

How to find the company's ATS:
1. Visit the company's website and look for a Careers or Jobs link.
2. Check where that link points — the subdomain identifies the ATS:
   - *.wd5.myworkdayjobs.com → Workday (use CXS POST API — see JOB-BOARD-SITE-NOTES.md)
   - jobs.lever.co/* → Lever (navigate to jobs.lever.co/[slug] and search by title)
   - boards.greenhouse.io/* → Greenhouse (query boards-api.greenhouse.io/v1/boards/[slug]/jobs)
   - jobs.ashbyhq.com/* → Ashby (navigate and search)
3. Use Puppeteer for all career site navigation — the VM egress proxy blocks most career sites, but Puppeteer bypasses it.
4. If the career site is blocked even by Puppeteer (e.g., Cloudflare protection), use Claude in Chrome as a fallback if the extension is connected.

Update each individual file header with one of:

If confirmed:
**Company career site:** [direct URL to the posting on the company's ATS]
**Ghost job check:** ✅ Confirmed live on [ATS name] ([YYYY-MM-DD])

If not found:
**Company career site:** [company careers page URL if found]
**Ghost job check:** ⚠️ Possible Ghost Job — posting not found on company career site ([YYYY-MM-DD])

If career site inaccessible:
**Company career site:** [URL attempted, if known]
**Ghost job check:** ❓ Unverified — career site inaccessible ([YYYY-MM-DD])

Keep all files regardless of ghost job status.

---

STEP 8 — SUMMARY REPORT

Append a summary section to the bottom of the master list file (OUTPUT_FOLDER/[YYYY-MM-DD]-Indeed.md):

---

## Session Summary

- Searches run: [queries]
- Total unique jobs found: [N]
- Jobs passing title pre-screen: [N]
- Jobs scored: [N]
- Jobs skipped (score ≤ 3 or poor fit): [N]
- Individual files created (score 6+): [N]

### Scored Jobs (sorted high to low)

| Score | Title | Company | Ghost Check | File |
|-------|-------|---------|-------------|------|

---

## Session Notes

[Any quirks encountered: duplicate job IDs, unexpected titles, ATS systems encountered during ghost job check, connector behavior, etc.]

---

Confirm when all files have been saved.
```

---

## NOTES FOR FUTURE RUNS

- **Connector must be active at session start.** If `get_resume()` fails at Step 0, stop — restart the session with the connector connected.
- **Deduplication is essential.** The three queries overlap heavily. Always complete all searches before pulling any job details.
- **get_company_data** is most useful for companies you haven't heard of — skip it for well-known employers where you already have a view.
- **Update Last Run date** in `JOB-BOARD-SITE-NOTES.md` Board #14 after each run.
