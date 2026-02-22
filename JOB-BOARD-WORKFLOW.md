# Job Board Review Workflow

**Paste this prompt into Cowork each time you run it. The agent will guide you through site selection before starting.**

---

## WORKFLOW PROMPT

```
Read the environment configuration from .env to load PROFILE_PATH and OUTPUT_FOLDER.

Read my professional profile from the path specified in PROFILE_PATH. Hold it in context for the entire workflow — you will use it to assess every job.

Read JOB-BOARD-SITE-NOTES.md and hold it in context. You will use the per-site scraping notes when you reach Step 1.

Today's date is [AUTO-FILL FROM SYSTEM].

---

STEP 0 — SITE SELECTION

Present the user with the list of known job boards from JOB-BOARD-SITE-NOTES.md. Format it as a numbered list showing: board label, last run date, and run type.

Ask the user:
1. Which board do you want to run? (pick a number, or say "new board" and provide the URL)
2. What run type? (e.g., Remote, In-Person, All)
3. Any label suffix to add to the filename? (e.g., "In-Person", "Remote") — suggest a sensible default based on the run type.

Wait for the user's answers before proceeding.

Set variables based on the user's selection:
- JOB_BOARD_URL: from site notes or user-provided
- JOB_BOARD_NAME: board label (e.g., ClimateBase, GreentownLabs)
- RUN_LABEL: full filename label (e.g., ClimateBase-In-Person)
- TARGET_LOCATION: Greater Boston Area (confirm with user if running a new board)
- WORK_TYPE: Full-time
- OUTPUT_FOLDER: from .env (OUTPUT_FOLDER), default ./results/

If the user selects a known board, load the corresponding scraping notes from JOB-BOARD-SITE-NOTES.md before proceeding to Step 1.

---

STEP 1 — SCRAPE THE JOB BOARD

Navigate to the JOB_BOARD_URL set in Step 0. Apply the site-specific scraping approach from JOB-BOARD-SITE-NOTES.md for the selected board.

Scroll through all available listings on the page (paginate if needed). For each listing, collect:
- Job Title
- Company
- Location (as listed)
- Workplace type (In-person / Hybrid / Remote)
- Brief description (first 2–3 sentences or the posted summary)
- Direct link to the full job posting

Filter: Keep only listings where location includes TARGET_LOCATION OR workplace type matches the selected run type. Discard anything that is contract, part-time, or internship.

---

STEP 2 — SAVE THE MASTER LIST

Create the file: OUTPUT_FOLDER/[YYYY-MM-DD]-[RUN_LABEL].md

Format it as a markdown table:

| # | Title | Company | Location | Workplace | Description | Link |
|---|-------|---------|----------|-----------|-------------|------|

Number each row sequentially. Save the file.

---

STEP 3 — ASSESS EACH JOB AGAINST MY PROFILE

For each job in the master list:

1. Navigate to the full job posting link to read the complete job description.
2. Assess two things based on my profile:
   - **Interest:** Would I genuinely want this role?
   - **Fit:** Do my background and skills align with what they are asking for?
3. If BOTH interest and fit are positive, assign a score from 1–10:
   - 10 = Dream job (aligns with my ideal role description perfectly)
   - 7–9 = Strong interest, good fit, worth pursuing
   - 4–6 = Acceptable role, reasonable fit, would consider
   - 1–3 = Would do it to get by, but not excited
   - Skip (do not create a file) = Low interest OR poor fit

---

STEP 4 — CREATE AN INDIVIDUAL FILE FOR EACH SCORED JOB

For each job that received a score, create a markdown file.

Filename format: [OUTPUT_FOLDER]/[Company]_[JobTitle]_[YYYY-MM-DD].md

Replace spaces with underscores. Remove special characters from filename.

File contents:

---

# [Job Title] — [Company]

**Score:** [X/10] — [one sentence rationale]

**Location:** [location] | **Workplace:** [In-person / Hybrid / Remote]

**Apply:** [direct link to the job posting]

---

## Why This Role

[2–3 sentences on why this role matches my profile — be specific about skill alignment and what appeals about the role based on my stated preferences.]

## Potential Gaps

[1–2 sentences on any requirements in the posting I do not fully meet. Write "None identified" if there are no meaningful gaps.]

---

## Full Job Description

[Paste the complete job description text as scraped from the posting.]

---

---

STEP 5 — SUMMARY REPORT

Append a summary section to the bottom of the master list file (OUTPUT_FOLDER/[YYYY-MM-DD]-[RUN_LABEL].md):

---

## Session Summary

- Total listings reviewed: [N]
- Listings passing location/type filter: [N]
- Jobs scored (interest + fit positive): [N]
- Jobs skipped (low interest or poor fit): [N]

### Scored Jobs (sorted high to low)

| Score | Title | Company | File |
|-------|-------|---------|------|

---

Confirm when all files have been saved.
```

---

## IMPROVEMENT LOG

### 2026-02-22 — Climatebase In-Person Run (Lessons Learned)

**Scraping approach that worked on Climatebase:**
The job listings are rendered in a React app. Standard link selectors (`a[href*="/jobs/"]`) returned nothing. The working approach was:
1. Find the scrollable container by detecting elements with `scrollHeight > clientHeight` — the job list container was a `div` with class pattern `sc-3d1ac256-2`.
2. Scroll the container (not `document.body`) incrementally to trigger lazy loading.
3. Extract cards using the stable CSS class `.Card_Information__HkDL9`.
4. Parse fields from the card's inner HTML: `h2 span` for title, `ul li` items for company/location/workplace/job type, `.Card_Details__NXC2i` for description.
5. Walk up the DOM from each card to find the parent `<a>` tag for the job link.

**Location filter on Climatebase:**
- The URL parameter `remote=false` did NOT reliably exclude remote jobs after applying a location filter via the UI.
- The location input is a react-select field with id `locationNew-desktop`. It can be filled with `puppeteer_fill` and the dropdown option selected by text match via `querySelectorAll` + `.click()`.
- Applying the location filter triggers a "job alert" signup modal — dismiss it before proceeding.
- After setting the location, manually check and set the Workplace Preference sidebar checkboxes (Hybrid + In-person) to restore the remote exclusion filter.

**Cookie banner:**
Climatebase shows a GDPR-style cookie banner on load. Find the Reject button via `Array.from(document.querySelectorAll('button')).find(b => b.textContent.trim() === 'Reject')` and click it.

**Output file naming for multiple runs of the same board:**
Use the label in the filename (e.g., `2026-02-22-ClimateBase-In-Person.md`) to distinguish run types (remote vs. in-person) on the same date.

**Pre-screening candidates before visiting full postings:**
With 143 filtered listings, visiting every posting is impractical. A keyword pre-screen on job titles (include/exclude word lists) reduced 143 → 14 candidates. This saved significant time. For future runs, consider encoding the pre-screen logic into the workflow prompt as a Step 2.5.

**Puppeteer variable scope issue:**
Puppeteer's `puppeteer_evaluate` calls share a JavaScript context within a session. Declaring variables with `const` or `let` in one call prevents re-declaration in a later call. Use anonymous expressions or unique variable names per call to avoid `Identifier already declared` errors.

### 2026-02-22 — Greentown Labs Run (Lessons Learned)

**Scraping approach that worked on Greentown Labs:**
The site is a standard WordPress/static page — no lazy loading or infinite scroll. `document.body.scrollHeight` stabilizes immediately. The structure is:
- Job cards are `<a class="card x-between flex y-center">` elements containing `<h3>` (title) and `<h4>` (company).
- Categories are `<h3>` elements inside `.col-10.jobs` that are NOT inside an `<a>` tag.
- Walk backwards through siblings and parent elements to associate each card with its category heading.
- Total extraction: 65 jobs in one pass, no pagination needed.

**Location filter:**
The URL parameter `location=boston` has no effect — the dropdown only offers "All Locations." The board mixes Greentown Boston (Somerville, MA) and Greentown Houston (Houston, TX) members without marking which is which on the listing cards. Location must be determined by visiting each individual posting. In practice, most member companies are Boston-area, but it is worth noting Houston companies appear in the list.

**LinkedIn short URLs (lnkd.in):**
Several Greentown listings link to `lnkd.in/` shortened URLs. These redirect through a LinkedIn warning page showing the actual destination URL. Navigate to that destination directly (not through LinkedIn) to read the job posting.

**Dover ATS (app.dover.com):**
Dover-hosted job postings are blocked by Cloudflare bot protection and by the network egress proxy. Cannot be fetched via Puppeteer or WebFetch. If a job is on Dover, skip the detailed review or note it as unverifiable.

**Board characteristics:**
- ~60% of listings are IC engineering or software roles — lower PM/leadership density than Climatebase.
- No date filter: all postings are current as of the scrape date, but there is no way to filter for recency. Re-running periodically will capture new member companies.
- NeuroBionics appears as a Greentown member but makes implantable neurotech devices — not climate tech. Worth noting that not all members are strictly climate-focused.
- Board is most useful as a supplement to Climatebase for surfacing early-stage Boston climate startups not yet on larger boards.

---

## TIPS FOR REUSE

- **To run on a different job board:** Change `JOB_BOARD_URL` and `JOB_BOARD_NAME`. Everything else stays the same.
- **To tighten or loosen filters:** Edit the filter criteria in Step 1 (e.g., add salary range if the URL filter supports it, or add specific titles to exclude).
- **To adjust scoring calibration:** Edit the score descriptions in Step 3 to reflect current priorities (e.g., if you become more selective, raise the skip threshold).
- **Your profile is the source of truth.** Update `MY-PROFILE.md` (path set in `.env`) when your situation changes — the workflow will adapt automatically.

## SUGGESTED FOLDER STRUCTURE

```
~/job-search/
├── .env                                 ← local config: profile path, output folder, API keys (NOT committed)
├── .env.example                         ← safe template to share — no real values
├── MY-PROFILE.md                        ← your profile (NOT committed — path set in .env)
├── JOB-BOARD-WORKFLOW.md               ← this file
├── JOB-BOARD-SITE-NOTES.md            ← per-site scraping notes
└── results/                             ← output folder (NOT committed)
    ├── 2026-02-21-LinkedIn.md          ← master list + summary
    ├── Acme_VP_of_Product_2026-02-21.md
    └── BetaCorp_Head_of_Strategy_2026-02-21.md
```

## ON FILE FORMAT FOR DOWNSTREAM AI USE

Individual job files are saved as `.md` (Markdown). This is the best format for feeding back to an AI agent later because:
- It is plain text — no parsing library needed
- Structure (headings, scores, links) is preserved and machine-readable
- You can also batch-feed multiple `.md` files to a future agent (e.g., one that drafts tailored cover letters) without conversion
- If you later need PDF for human review or JSON for a database, Cowork can batch-convert the folder in a single follow-up task
