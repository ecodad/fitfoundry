# Setup Guide

Everything you need to configure FitFoundry before running your first session.

---

## Prerequisites

- [Cowork](https://www.anthropic.com) — Anthropic's agentic desktop tool. All browser automation runs inside Cowork's built-in Puppeteer and Claude in Chrome environments. No additional software installation is required.
- A LinkedIn account (for the LinkedIn board)
- A Wellfound account (for the Wellfound board)
- The Claude in Chrome browser extension (for LinkedIn and Wellfound — see below)

---

## 1. Clone the Repository

```bash
git clone https://github.com/ecodad/fit_foundry.git
cd fit_foundry
```

---

## 2. Configure Your Environment

```bash
cp .env.example .env
```

Open `.env` and set the following variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `PROFILE_PATH` | Path to your personal profile file | `./MY-PROFILE.md` |
| `OUTPUT_FOLDER` | Directory where result files are written | `./results/` |
| `ALGOLIA_APP_ID` | Algolia application ID for the 80,000 Hours board | *(required for that board)* |
| `ALGOLIA_API_KEY` | Algolia read-only API key for the 80,000 Hours board | *(required for that board)* |
| `ALGOLIA_INDEX` | Algolia index name | `jobs_prod` |

**Getting Algolia credentials:** These are public read-only search keys. Navigate to [jobs.80000hours.org](https://jobs.80000hours.org) in Chrome, open DevTools → Network tab, and look for requests to `algolia.net`. The App ID and API Key appear in the request headers. Add them to `.env`. If the board's credentials change between runs, retrieve fresh values the same way.

---

## 3. Fill In Your Profile

Open `MY-PROFILE.md` and complete every section — target role, location preferences, sectors of interest, current experience, and any hard constraints (e.g., no clearance-required roles). The more specific you are, the more accurately the agent can score job fit.

Your profile is the agent's source of truth. Update it whenever your situation or priorities change — the workflow adapts automatically.

---

## 4. Set Up Claude in Chrome (LinkedIn and Wellfound only)

LinkedIn and Wellfound both require an authenticated browser session. Puppeteer cannot bypass their login walls or CAPTCHA systems. The Claude in Chrome browser extension solves this by letting the agent control your real Chrome browser, where you are already logged in.

1. Install the Claude in Chrome extension from the Chrome Web Store.
2. Log into LinkedIn and Wellfound in Chrome before starting a session.
3. When Cowork prompts you to connect the extension, click **Connect** in Chrome.
4. The agent will then be able to navigate LinkedIn and Wellfound within your authenticated session.

**Note:** JavaScript injection is blocked on LinkedIn due to reCAPTCHA iframes. The agent uses the accessibility tree (`read_page`) for extraction instead. This is slower but reliable. See `JOB-BOARD-SITE-NOTES.md` §4 for full details.

---

## 5. Run the Workflow

Open Cowork and paste the full contents of `JOB-BOARD-WORKFLOW.md` into a new session. The agent will:

1. Ask which job board to run and what run type (remote, hybrid, in-person, or all).
2. Scrape and filter listings using the site-specific approach in `JOB-BOARD-SITE-NOTES.md`.
3. Score each job against your profile.
4. Create a master list and individual files for scored jobs.
5. Attempt to verify each scored job on the company's own career site (ghost job check).

You do not need to do anything during the run beyond the initial board selection unless the agent needs login credentials or encounters a CAPTCHA it cannot bypass.

---

## What Is and Is Not Committed

| File / Folder | Committed | Reason |
|---------------|-----------|--------|
| `.env.example` | ✅ Yes | Safe template — no real values |
| `.env` | ❌ No | Contains API keys and personal paths |
| `MY-PROFILE.md` | ✅ Yes | Boilerplate personal profile template |
| `results/` | ❌ No | Contains scraped job descriptions and personal assessments |
| `JOB-BOARD-WORKFLOW.md` | ✅ Yes | Generic workflow prompt |
| `JOB-BOARD-SITE-NOTES.md` | ✅ Yes | Technical scraping reference |
| `README.md` | ✅ Yes | Project overview |
| `SETUP.md` | ✅ Yes | This file |

---

## Folder Structure

```
job-search/
├── .env.example                  ← config template — copy to .env and fill in
├── .env                          ← local config (not committed)
├── .gitignore                    ← excludes .env and results/
├── MY-PROFILE.md                 ← your profile (fill in before first run)
├── README.md                     ← project overview
├── SETUP.md                      ← this file
├── JOB-BOARD-WORKFLOW.md        ← workflow prompt — paste into Cowork each session
├── JOB-BOARD-SITE-NOTES.md     ← per-site scraping reference
└── results/                     ← generated output (not committed)
    ├── YYYY-MM-DD-BoardName.md  ← master list + session summary per run
    └── Company_Role_YYYY-MM-DD.md  ← individual file per scored job
```
