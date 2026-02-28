---
name: fitfoundry
description: >
  FitFoundry is an AI-assisted job search workflow. Use this skill whenever
  the user wants to find jobs, search job boards, score listings against their
  profile, set up their job search workspace, run ProfileBuilder to create a
  career profile, generate tailored resumes or cover letters, or do anything
  related to a job search. Trigger on phrases like: find me a job, search for
  jobs, set up job search, run FitFoundry, job board, career profile, score
  jobs, ghost job check, Indeed search, Dice search, Climatebase,
  80000 Hours, Breakthrough Energy, LaunchKit, tailored resume, cover letter
  for a job, or any mention of FitFoundry.
---

# FitFoundry — AI-Assisted Job Search

FitFoundry is a three-stage job search system that runs inside Cowork:

- **Stage 1 — ProfileBuilder:** A guided interview that builds your career profile. Accepts resumes and cover letters to pre-fill answers. Produces `MY-PROFILE.md` and `QUICK-REFERENCE.md`.
- **Stage 2 — FitFoundry:** Searches job boards, scores each listing against your profile (1–10), verifies postings aren't ghost jobs, and saves structured results.
- **Stage 3 — LaunchKit:** Takes your scored results and generates tailored resumes and cover letters for each job you want to pursue.

This skill handles setup automatically. It checks what's already in place, fills in anything missing, and gets you to searching as quickly as possible.

---

## WORKFLOW PROMPT

```
Today's date is [AUTO-FILL FROM SYSTEM].

Read this file in full before taking any action. Then follow the steps below in order.

---

STEP 0 — STATUS CHECK

Check the current workspace for the following. Report the status of each item
clearly before proceeding. Use ✅ for ready and ⚠️ for missing or incomplete.

1. FitFoundry workspace files
   → Check whether FITFOUNDRY-WORKFLOW.md exists in the workspace.
   → If it exists, all workspace files are assumed present. Mark ✅.
   → If it does not exist, mark ⚠️ WORKSPACE FILES MISSING.

2. Career profile
   → Read MY-PROFILE.md (if it exists).
   → If the file does not exist, or if it contains the placeholder text
     "[Your Name]", mark ⚠️ PROFILE NOT SET UP.
   → Otherwise mark ✅ PROFILE READY.

3. Connectors
   → Check whether the Indeed MCP tools (search_jobs, get_job_details) are
     available in this session.
   → If available, mark ✅ INDEED CONNECTOR ACTIVE.
   → If not available, mark ⚠️ INDEED CONNECTOR NOT ACTIVE.
   → Check whether Dice MCP tools are available.
   → If available, mark ✅ DICE CONNECTOR ACTIVE.
   → If not available, mark ⚠️ DICE CONNECTOR NOT ACTIVE.
   → Check whether Puppeteer tools (puppeteer_navigate, puppeteer_evaluate)
     are available.
   → If available, mark ✅ PUPPETEER ACTIVE.
   → If not available, mark ⚠️ PUPPETEER NOT ACTIVE (secondary boards unavailable).

Print the full status summary to the user. Then route:
- If ⚠️ WORKSPACE FILES MISSING → go to STEP 1.
- Else if ⚠️ PROFILE NOT SET UP → go to STEP 2.
- Else if both ⚠️ INDEED and ⚠️ DICE CONNECTOR NOT ACTIVE → go to STEP 3.
- Else → go to STEP 4 (ready to search).

If the user already told you what they want to do (e.g. "run Indeed") and the
relevant components are ready, skip ahead to STEP 4 directly.

---

STEP 1 — BOOTSTRAP WORKSPACE FILES

Only run this step if FITFOUNDRY-WORKFLOW.md was not found in the workspace.

Tell the user:

  "Your workspace doesn't have the FitFoundry files yet. I can download them
   from GitHub and set everything up automatically. This takes about 30 seconds
   and only happens once.

   After setup, your workspace will contain all the workflow files, a template
   profile to fill in, and a results folder for your job search output.

   Ready to set up? (yes / no)"

Wait for confirmation before proceeding.

Once confirmed, download each file below from GitHub using WebFetch.
Base URL: https://raw.githubusercontent.com/ecodad/fit_foundry/main/

Files to download and write to the workspace root:
- FITFOUNDRY-WORKFLOW.md
- PROFILEBUILDER.md
- CAREER-PROFILE-INTERVIEW.md
- JOB-BOARD-SITE-NOTES.md
- LAUNCHKIT.md
- REQUIREMENTS.md
- SETUP.md
- README.md
- .env.example
- MY-PROFILE.md
- QUICK-REFERENCE.md

Files to download and write to sites/ subfolder:
- sites/Indeed.md
- sites/Dice.md
- sites/Climatebase.md
- sites/80kHours.md
- sites/BreakthroughEnergy.md
- sites/YCombinator.md
- sites/WorkOnClimate.md
- sites/Draper.md
- sites/Formlabs.md
- sites/Wellfound.md

Rules for downloading:
- Skip any file that already exists in the workspace — never overwrite.
- If a download fails, note the filename and continue. Report all failures at
  the end.
- FITFOUNDRY-WORKFLOW.md, PROFILEBUILDER.md, CAREER-PROFILE-INTERVIEW.md,
  and JOB-BOARD-SITE-NOTES.md are required. If any of these fail, stop and
  tell the user.

After downloading:
- Create the results/ directory if it doesn't exist (use Bash: mkdir -p results).
- Write a .env file with these default values if .env does not already exist:

  PROFILE_PATH=./MY-PROFILE.md
  QUICK_REFERENCE_PATH=./QUICK-REFERENCE.md
  OUTPUT_FOLDER=./results/
  ALGOLIA_APP_ID=
  ALGOLIA_API_KEY=
  ALGOLIA_INDEX=jobs_prod

Tell the user:

  "✅ Setup complete. Your workspace is ready.

   [List what was downloaded. Note any files that were skipped or failed.]

   Next: I'll help you set up your career profile."

Then continue to STEP 2.

---

STEP 2 — PROFILE SETUP

Only run this step if MY-PROFILE.md is missing or still contains "[Your Name]".

Tell the user:

  "Before searching for jobs, FitFoundry needs to know about you — your skills,
   experience, and what you're looking for. I'll ask you a series of questions
   and build your profile automatically.

   If you have a resume or cover letter, you can share the file path or upload
   it now. I'll use it to pre-fill what I can and skip questions I can already
   answer.

   Two options for the interview:
   • Quick setup (~5 min) — covers the essentials. Gets you searching fast.
   • Full setup (~40 min) — includes resume review + thorough interview.
     Produces better job scoring, especially for culture-fit roles.

   Do you have a resume or cover letters to provide? And which interview
   would you prefer — Quick or Full?"

Wait for the user's answer.

If the user provides a resume or cover letter path:
- Read each file in full.
- Extract: job titles, companies, tenure, skills, education, location,
  any stated objective or summary.
- Hold extracted content in context. Do not generate the profile yet.

Then read CAREER-PROFILE-INTERVIEW.md and hold the full question set in context.

Set MODE = Quick or Full based on the user's choice.

Work through the question set for the selected MODE:
- Ask one topic block at a time (not one question at a time).
- If resume content clearly answers a question, state the inference and ask
  the user to confirm or correct — do not ask the question from scratch.
- If the resume partially answers something, use it as a starting point and
  ask only for what's missing.
- Every question either gets answered by resume inference + confirmation,
  or by asking the user directly. Do not skip questions.

After the final question block, say:
  "That's everything I need. Give me a moment to put your profile together."

Present a compact summary of all inferred and stated answers organized by
section. Flag any that feel uncertain. Ask:
  "Does this look right? Correct anything before I generate your files."

Wait for confirmation. Then:
- Write MY-PROFILE.md using the structure defined in PROFILEBUILDER.md
  (STEP 4 — GENERATE MY-PROFILE.md).
- Write QUICK-REFERENCE.md using the structure defined in PROFILEBUILDER.md
  (STEP 5 — GENERATE QUICK-REFERENCE.md).

Tell the user:

  "✅ Your profile is ready.

   One thing to do before searching: open QUICK-REFERENCE.md and fill in the
   Board-Specific Search Settings section — the keywords to use on each board.
   These can't be inferred from the interview, so they need to be set manually.
   You can do this now or after your first search.

   Next: I'll check your connectors."

Then continue to STEP 3.

---

STEP 3 — CONNECTOR SETUP

Only run this step for connectors that were marked ⚠️ in the status check.

Explain what connectors are, in plain English:

  "Connectors are pre-built integrations that let FitFoundry access job boards
   officially — using the board's own API rather than scraping. Indeed and Dice
   both have official connectors for Cowork.

   I'll walk you through adding the ones you need. Each one takes about
   2 minutes to set up."

--- Indeed ---

If Indeed is not active:

  "Indeed is the largest US job board and FitFoundry's primary board.

   To add the Indeed connector:
   1. Open Cowork Settings (gear icon, top right of the app)
   2. Click 'Connected Services' or 'Integrations'
   3. Find 'Indeed' in the list and click Connect
   4. Sign into your Indeed account when prompted

   ⚠️ IMPORTANT: After connecting, you must start a NEW Cowork session.
   Connectors don't activate until the session is restarted. When you open
   a new session, just ask FitFoundry to start — it will pick up from here.

   Let me know when you've connected Indeed, or if you'd like to skip it
   and proceed with other boards."

Wait for the user to confirm they've connected Indeed or chosen to skip.

--- Dice ---

If Dice is not active:

  "Dice is a tech-focused job board — good for engineering and technical roles.

   To add the Dice connector:
   1. Open Cowork Settings (gear icon, top right)
   2. Click 'Connected Services' or 'Integrations'
   3. Find 'Dice' in the list and click Connect
   4. Sign into your Dice account when prompted

   ⚠️ IMPORTANT: Same as above — you'll need a new Cowork session after
   connecting for it to activate.

   Let me know when you've connected Dice, or if you'd like to skip it."

Wait for the user to confirm or skip.

--- Puppeteer (secondary boards) ---

If Puppeteer is not active and the user wants to use Climatebase, 80,000 Hours,
Breakthrough Energy, Y Combinator, The Engine, Draper, or Formlabs:

  "The secondary job boards (Climatebase, 80,000 Hours, etc.) use a browser
   automation tool called Puppeteer. It runs in the background — you won't see
   a browser window — and lets FitFoundry navigate those sites automatically.

   Puppeteer requires Node.js on your computer.

   Do you have Node.js installed? You can check by opening a terminal and
   typing: node --version

   If you're not sure, say no and I'll help you check."

If the user confirms Node.js is present (or after helping them verify):

  "To add Puppeteer:
   1. Open Cowork Settings → MCP Servers
   2. Click 'Add Server'
   3. In the command field, enter exactly:
      npx -y @modelcontextprotocol/server-puppeteer
   4. Click Save

   ⚠️ New session required after adding, same as the connectors above."

--- After connector setup ---

If the user has connected one or more items and needs to restart:

  "You're all set. Start a new Cowork session, then come back to FitFoundry.
   I'll detect your new connectors automatically and take you straight to
   the job board menu."

If all needed connectors are now active (or the user chose to proceed with
what's available):

  "✅ Connectors ready. Let's start searching."

Continue to STEP 4.

---

STEP 4 — SEARCH

Read FITFOUNDRY-WORKFLOW.md and hold it in context. It contains the complete
workflow for running a job board search.

Present the board menu:

---

**Which board would you like to search?**

**Primary (recommended — official connectors, no ToS concerns):**
1. Indeed — largest US job board; best for broad searches
2. Dice — tech-focused; good for engineering and technical roles

**Secondary (mission-driven and specialized; requires Puppeteer):**
3. Climatebase — climate and sustainability jobs
4. 80,000 Hours — high-impact careers; curated, lower volume
5. Breakthrough Energy (BEV Jobs) — energy transition; ~767 listings
6. Breakthrough Energy (BEF Jobs) — energy transition; ~77 listings
7. Y Combinator — startup jobs from YC-backed companies
8. Draper Laboratory — local research institution (note: most roles require clearance)
9. Formlabs — direct company board (example of single-company search)

**⚠️ Caution (Terms of Service concerns — use at your own discretion):**
10. LinkedIn — requires Claude in Chrome + active login; automated scraping may violate ToS
11. Wellfound — requires Claude in Chrome + active login; automated scraping may violate ToS

**Other:**
- Say "new board" to add a board not on this list

---

Also ask:
- What work type? Remote / Hybrid / In-Person / All
- Any label to add to the output filename? (e.g., "Remote", "Climate") —
  suggest a default based on the board and work type.

Wait for the user's selections, then proceed with FITFOUNDRY-WORKFLOW.md
from STEP 1 (SCRAPE THE JOB BOARD), using the selected board's site file
from sites/[BoardName].md.

If the user selects a Caution board, acknowledge the choice and add:
  "Just to confirm — you're choosing to run [LinkedIn/Wellfound]. I'll proceed,
   but note that automated use of this board may conflict with their Terms of
   Service. Continuing on your instruction."

If the user selects "new board":
  "What's the URL of the board you'd like to add? I'll navigate to it, figure
   out how it's structured, and create a site file so it can be used in future
   sessions too."

---

STEP 5 — LAUNCHKIT (application materials)

If the user asks to generate a resume or cover letter for a specific job,
or to process results from a completed FitFoundry run:

Read LAUNCHKIT.md and follow the workflow there.

LaunchKit scans your results/ folder, presents scored jobs as a numbered list,
and generates tailored .docx resumes and cover letters for each job you select.

You will need:
- At least one base resume (.docx or .pdf) in the workspace
- At least one base cover letter (.docx or .pdf) in the workspace
- Completed FitFoundry results in results/ from a previous Stage 2 run
```

---

## Adding a New Board

To contribute a new board to FitFoundry:

1. Create a new file at `sites/[BoardName].md` using the standard format
   from `JOB-BOARD-SITE-NOTES.md` (Site File Format section).
2. Add a row to the appropriate section of the Known Boards table in
   `JOB-BOARD-SITE-NOTES.md`.
3. Record any lessons learned in the Improvement Log in `FITFOUNDRY-WORKFLOW.md`.
4. Submit a pull request to the GitHub repo.

The modular site-file architecture means adding a board only requires those
three files — no changes to the core workflow are needed.

---

## Notes for Contributors

GitHub repo: **https://github.com/ecodad/fit_foundry**

This SKILL.md is the single install file for Cowork. Place it at:
`~/.config/cowork/skills/fitfoundry/SKILL.md` (or wherever your Cowork
installation stores skills).

On first run it downloads the full FitFoundry directory structure from the
repo above. If you have already cloned the repo into your workspace, the
bootstrap step will detect the existing files and skip them.

Pull requests for new boards, improved site files, and workflow enhancements
are welcome.
