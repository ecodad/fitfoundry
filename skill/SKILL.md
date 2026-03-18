---
name: fitfoundry
description: >
  FitFoundry is an AI-assisted job search workflow for Cowork. Use this skill whenever the user wants to search for jobs, run a job board scrape, review job listings, score postings against their profile, check for ghost jobs, or set up their job search workspace. Trigger whenever the user mentions: job search, job board, run FitFoundry, review listings, score jobs, ghost job check, search a specific job board by name, or setting up a job search workflow. Also trigger if the user pastes or references FITFOUNDRY-WORKFLOW.md or JOB-BOARD-WORKFLOW.md.
---

# FitFoundry — AI-Assisted Job Search

FitFoundry is a three-stage workflow for finding and applying to jobs:

- **Stage 1 — ProfileBuilder:** Interviews the user and generates a career profile (`MY-PROFILE.md`) and search settings (`QUICK-REFERENCE.md`). Run once before the first search, and re-run any time priorities change.
- **Stage 2 — FitFoundry:** Scrapes job boards, filters and scores listings against the profile, verifies postings are real (ghost job check), and saves structured output. Run as often as needed.
- **Stage 3 — LaunchKit:** Takes scored job reports from Stage 2 and generates a tailored resume and cover letter (`.docx`) for each selected role.

Full system requirements are in `REQUIREMENTS.md`. Setup instructions are in `SETUP-STAGE1.md` and `SETUP-STAGE2.md`. Quick-reference prompts are in `QUICKSTART.md`.

---

## First-Time Setup

Setup takes two sessions with a Claude Desktop restart between them.

**Session 1** copies workflow files to the workspace, installs the Puppeteer MCP server, and connects job board accounts (MCP connectors and optionally Claude in Chrome for login-gated boards). Ends with a required restart.

**Session 2** verifies everything is active, then runs ProfileBuilder to create the career profile and search settings.

Full instructions for both sessions are in `SETUP-SESSIONS.md`.

**When a user says "Set up FitFoundry — Session 1":**
Read `SETUP-SESSIONS.md` and follow the SESSION 1 instructions exactly.

**When a user says "Set up FitFoundry — Session 2":**
Read `SETUP-SESSIONS.md` and follow the SESSION 2 instructions exactly.

Do not attempt to download any files from GitHub or the internet during setup. All files needed for bootstrap are already in the skill directory.

---

## Stage 1 — Running ProfileBuilder

**When a user says "Run ProfileBuilder" or wants to update their profile:**

Read `PROFILEBUILDER.md` from the workspace and execute it from STEP 0 through STEP 6. ProfileBuilder reads `CAREER-PROFILE-INTERVIEW.md` for the question set and writes `MY-PROFILE.md` and `QUICK-REFERENCE.md` on completion.

---

## Stage 2 — Running a Job Search

**When a user says "Run a FitFoundry job search" (or similar):**

1. Read `.env` from the workspace to get `PROFILE_PATH`, `QUICK_REFERENCE_PATH`, and `OUTPUT_FOLDER`.
2. **Pre-flight connector check.** Before loading any workflow files, verify tool availability and report status to the user:
   - Check for `puppeteer_navigate` (Puppeteer MCP server)
   - Check for Indeed MCP tools (`search_jobs` from the Indeed connector)
   - Check for Dice MCP tools (`search_jobs` from the Dice connector)
   - Check for Claude in Chrome (`tabs_context_mcp` or similar)

   Report a short status line, e.g.: "Tools active: Puppeteer ✅ | Indeed ✅ | Dice ❌ | Chrome ❌"

   If Puppeteer is missing, warn the user that most boards will not work and offer to help install it. If a connector is missing, note which boards are affected but continue — the user may not need that board this session.
3. Read the career profile from `PROFILE_PATH`. Hold in context for the full session.
4. Read the quick reference from `QUICK_REFERENCE_PATH`. Hold in context — it contains scoring calibration and board-specific search settings.
5. Read `JOB-BOARD-SITE-NOTES.md` from the workspace. Hold in context.
6. Read `FITFOUNDRY-WORKFLOW.md` and execute it from STEP 0.

The workflow will ask which board to run and what run type (Remote / Hybrid / In-Person / All), then scrape → filter → score → ghost job check → save results.

---

## Stage 3 — Running LaunchKit

**When a user says "Run LaunchKit" or wants to generate application materials:**

Read `LAUNCHKIT.md` from the workspace and execute it from STEP 0. LaunchKit reads the career profile and scored job report files from `OUTPUT_FOLDER`, then generates a tailored resume and cover letter (`.docx`) for each selected job.

Requires the `docx` Cowork skill. Requires base resume and cover letter files (`.docx` or `.pdf`) to be present in the workspace.

---

## Supported Job Boards (Stage 2)

The full list of supported boards, their authentication requirements, scraping methods, and site files is maintained in `JOB-BOARD-SITE-NOTES.md` → **Known Boards**. That file is the single source of truth for board registration. When adding a new board, update only `JOB-BOARD-SITE-NOTES.md` and add a corresponding `sites/[BoardName].md` file — no changes to this skill file are needed.

---

## Key Files

| File | Stage | Purpose |
|------|-------|---------|
| `SETUP-SESSIONS.md` | Setup | Agent instructions for Session 1 and Session 2 |
| `QUICKSTART.md` | All | Copy-paste prompts and troubleshooting |
| `REQUIREMENTS.md` | Reference | Full system requirements for all three stages |
| `PROFILEBUILDER.md` | 1 | ProfileBuilder workflow prompt |
| `CAREER-PROFILE-INTERVIEW.md` | 1 | Interview question bank used by ProfileBuilder |
| `MY-PROFILE.md` | 1, 2, 3 | Generated career profile — source of truth |
| `QUICK-REFERENCE.md` | 1, 2 | Scoring calibration and board-specific search settings |
| `FITFOUNDRY-WORKFLOW.md` | 2 | Job search workflow prompt |
| `JOB-BOARD-SITE-NOTES.md` | 2 | Per-board scraping notes and board index |
| `sites/[BoardName].md` | 2 | Individual board configuration — loaded at runtime |
| `LAUNCHKIT.md` | 3 | Application asset generator workflow prompt |

---

## Adding a New Board

1. Run a job search session and select "new board" at the site selection step.
2. Create a new file in `sites/` using the standard format from `JOB-BOARD-SITE-NOTES.md`.
3. Add a row to the Known Boards table in `JOB-BOARD-SITE-NOTES.md`.
4. Record lessons learned in the Improvement Log in `FITFOUNDRY-WORKFLOW.md`.
5. Update `README.md` with the new board.

---

## Output Format (Stage 2)

Results go to `OUTPUT_FOLDER` (set in `.env`):

- **Master list:** `YYYY-MM-DD-BoardName.md` — table of all listings + session summary
- **Individual files:** `Company_JobTitle_YYYY-MM-DD.md` — score, rationale, ghost job status, full job description

Scoring thresholds (adjustable in `QUICK-REFERENCE.md`):

| Score | Meaning | Output |
|-------|---------|--------|
| 9–10 | Dream job | Individual file |
| 7–8 | Strong fit | Individual file |
| 6 | Partial fit — one gap | Individual file |
| 5 | Multiple gaps | Master list only |
| ≤ 4 | Poor fit | Skip |
