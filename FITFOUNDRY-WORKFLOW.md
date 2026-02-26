# FitFoundry Workflow

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
(For boards whose sites/ file shows Scraping method: MCP connector — currently Indeed and Dice)

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

How to find the company's ATS:
1. Visit the company's website and look for a Careers or Jobs link.
2. Check the subdomain to identify the ATS:
   - *.wd5.myworkdayjobs.com or *.wd1.myworkdayjobs.com → Workday
   - jobs.lever.co/* → Lever
   - boards.greenhouse.io/* → Greenhouse
   - jobs.ashbyhq.com/* → Ashby
3. Workday: use the undocumented CXS POST API — POST to /wday/cxs/[company-slug]/[board-slug]/jobs with { "searchText": "[job title]", "limit": 20, "offset": 0, "appliedFacets": {}, "returnFacets": false }. See Improvement Log below for working examples.
4. Lever: navigate to jobs.lever.co/[company-slug] and search page text for the job title.
5. Greenhouse: query https://boards-api.greenhouse.io/v1/boards/[company-slug]/jobs?content=false and filter by title.
6. Use Puppeteer for all career site navigation — the VM egress proxy blocks most career sites, but Puppeteer bypasses it.
7. If the career site is blocked even by Puppeteer (e.g., Cloudflare), use Claude in Chrome as a fallback if the extension is connected.

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

### 2026-02-25 — Ghost Job Check Process (First Run)

Verified three scored jobs against company career sites. All three were confirmed live. Key findings:

**Boston Dynamics** uses Workday (`bostondynamics.wd1.myworkdayjobs.com`). The Workday CXS POST API works without authentication — POST to `/wday/cxs/bostondynamics/Boston_Dynamics/jobs` with `{ searchText: "Technical Program Manager", limit: 20, offset: 0, appliedFacets: {}, returnFacets: false }`. Job confirmed at req ID `R2354`.

**Commonwealth Fusion Systems** uses Lever (`jobs.lever.co/cfsenergy`). Navigate directly and search page text for job title. Posting found at `jobs.lever.co/cfsenergy/b2c21785-e359-410c-b176-8e3d1b3a3911`. Note: CFS's Greenhouse board (`boards.greenhouse.io/cfsenergy`) returned zero jobs — they are on Lever only.

**Johnson Controls** uses Workday (`jci.wd5.myworkdayjobs.com/JCI`). Same CXS POST API pattern. Job confirmed at req ID `WD30260051`. The company's public careers site (`jobs.johnsoncontrols.com`) redirects to Workday for actual job search.

**ATS discovery tip:** If you don't know which ATS a company uses, navigate to their careers page and check where the "Apply" or job listing links point. The subdomain almost always identifies the platform.

**Egress proxy limitation:** Career sites (`jobs.lever.co`, company career pages, etc.) are typically blocked by the VM's network egress proxy. Always use Puppeteer — not Bash, curl, WebFetch, or Python requests — for career site navigation. Puppeteer bypasses the proxy.

---

### 2026-02-22 — Climatebase In-Person Run (Lessons Learned)

**Scraping approach that worked:**
The job listings are rendered in a React app. Standard link selectors (`a[href*="/jobs/"]`) returned nothing. Working approach:
1. Find the scrollable container by detecting elements with `scrollHeight > clientHeight` — the job list container was a `div` with class pattern `sc-3d1ac256-2`.
2. Scroll the container (not `document.body`) incrementally to trigger lazy loading.
3. Extract cards using the stable CSS class `.Card_Information__HkDL9`.
4. Parse fields from the card's inner HTML: `h2 span` for title, `ul li` items for company/location/workplace/job type, `.Card_Details__NXC2i` for description.
5. Walk up the DOM from each card to find the parent `<a>` tag for the job link.

**Location filter:** The URL parameter `remote=false` did NOT reliably exclude remote jobs after applying a location filter via the UI. Use the location input field (`id="locationNew-desktop"`) and set Workplace Preference sidebar checkboxes (Hybrid + In-person) manually. Applying the location filter triggers a job alert signup modal — dismiss it before proceeding.

**Cookie banner:** Find the Reject button via `Array.from(document.querySelectorAll('button')).find(b => b.textContent.trim() === 'Reject')` and click it.

**Pre-screening at scale:** With 143 filtered listings, visiting every posting is impractical. A keyword pre-screen on job titles reduced 143 → 14 candidates before visiting any individual postings.

**Puppeteer variable scope:** `puppeteer_evaluate` calls share a JavaScript context within a session. Use anonymous expressions or unique variable names per call to avoid `Identifier already declared` errors.

---

### 2026-02-22 — Greentown Labs Run (Lessons Learned)

**Scraping approach:** Standard WordPress/static page — no lazy loading. Job cards are `<a class="card x-between flex y-center">` elements containing `<h3>` (title) and `<h4>` (company). Walk backwards through siblings to associate cards with their category headings. 65 jobs in one pass, no pagination.

**Location filter:** The `location=boston` URL parameter has no effect. The board mixes Boston and Houston members with no location marker on listing cards. Determine location by visiting each individual posting. Most member companies are Boston-area.

**LinkedIn short URLs:** Several Greentown listings link to `lnkd.in/` shortened URLs that redirect through a LinkedIn warning page. Navigate to the final destination directly, not through LinkedIn.

**Dover ATS:** Dover-hosted job postings (`app.dover.com`) are blocked by Cloudflare bot protection and the network egress proxy. Cannot be fetched via Puppeteer or WebFetch. Mark as ❓ Unverified.

---

## TIPS FOR REUSE

- **To run a different board:** Select it at Step 0. The workflow loads that board's sites/ file and adapts automatically.
- **To adjust filters:** Edit the filter criteria in QUICK_REFERENCE_PATH for the relevant board section.
- **To adjust scoring:** Edit the scoring calibration table in QUICK_REFERENCE_PATH.
- **Your profile is the source of truth.** Update PROFILE_PATH content when your situation changes — the workflow adapts automatically.
- **To add a new board:** Select "new board" at Step 0, provide the URL, and document the scraping approach in a new `sites/` file using the standard format in `JOB-BOARD-SITE-NOTES.md`. Add a row to the Known Boards table there, then log the approach in the Improvement Log above.
