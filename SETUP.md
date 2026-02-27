# Setup Guide

Everything you need to configure FitFoundry before running your first session.

For a full list of tools, connectors, and per-board requirements, see [REQUIREMENTS.md](REQUIREMENTS.md).

---

## Prerequisites

- [Cowork](https://www.anthropic.com) — Anthropic's agentic desktop tool
- Node.js — required for the Puppeteer MCP server. Download from [nodejs.org](https://nodejs.org) if not already installed.
- A LinkedIn account (for the LinkedIn board)
- A Wellfound account (for the Wellfound board)
- The Claude in Chrome browser extension (for LinkedIn and Wellfound — see Step 4)

---

## 1. Clone the Repository

```bash
git clone https://github.com/ecodad/fit_foundry.git
cd fit_foundry
```

---

## 2. Install the Puppeteer MCP Server

Puppeteer is not bundled with Cowork. It must be added as an MCP server before Stage 2 (FitFoundry) will work on any Puppeteer-based board.

Open a Cowork session and tell it you want to install the Puppeteer MCP server — it will guide you through the process. Node.js must be installed first (see Prerequisites).

---

## 3. Configure Your Environment

```bash
cp .env.example .env
```

Open `.env` and set the following variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `PROFILE_PATH` | Path to your personal profile file | `./MY-PROFILE.md` |
| `QUICK_REFERENCE_PATH` | Path to your personal quick reference file | `./QUICK-REFERENCE.md` |
| `OUTPUT_FOLDER` | Directory where result files are written | `./results/` |
| `ALGOLIA_APP_ID` | Algolia application ID for the 80,000 Hours board | *(required for that board)* |
| `ALGOLIA_API_KEY` | Algolia read-only API key for the 80,000 Hours board | *(required for that board)* |
| `ALGOLIA_INDEX` | Algolia index name | `jobs_prod` |

**Getting Algolia credentials:** These are public read-only search keys. Navigate to [jobs.80000hours.org](https://jobs.80000hours.org) in Chrome, open DevTools → Network tab, and look for requests to `algolia.net`. The App ID and API Key appear in the request headers. Only needed if you plan to run the 80,000 Hours board.

---

## 4. Set Up Claude in Chrome (LinkedIn and Wellfound only)

LinkedIn and Wellfound both require an authenticated browser session. Puppeteer cannot bypass their login walls or CAPTCHA systems. The Claude in Chrome extension lets the agent control your real Chrome browser, where you are already logged in.

1. Install the Claude in Chrome extension from the Chrome Web Store.
2. Log into LinkedIn and Wellfound in Chrome before starting a session.
3. When Cowork prompts you to connect the extension, click **Connect** in Chrome.

**Note:** JavaScript injection is blocked on LinkedIn due to reCAPTCHA iframes. The agent uses the accessibility tree for extraction instead. See `JOB-BOARD-SITE-NOTES.md` §4 for details.

---

## 5. Run ProfileBuilder (Stage 1)

Before running FitFoundry, you need a completed profile and quick reference. ProfileBuilder generates both through a structured interview.

Open Cowork and paste the contents of `PROFILEBUILDER.md` into a new session. The agent will:

1. Ask if you have a resume to provide (`.docx` or `.pdf`). Providing one lets it pre-fill answers it can infer.
2. Ask whether you want the quick interview (~5 min) or the full interview (~20 min).
3. Conduct the interview and confirm all extracted answers with you.
4. Write `MY-PROFILE.md` and `QUICK-REFERENCE.md` to the paths set in `.env`.

After ProfileBuilder completes, open `QUICK-REFERENCE.md` and fill in the board-specific search settings for each board you plan to use. These cannot be inferred from the interview and must be set manually.

Re-run ProfileBuilder any time your situation or priorities change — it overwrites the existing files, so back them up first if you want to preserve the originals.

---

## 6. Place Your Base Documents (Stage 3)

Before using LaunchKit, place your base resume(s) and cover letter(s) in the workspace folder. Accepted formats are `.docx` and `.pdf`.

You can provide multiple files of each type — LaunchKit treats them as a pool and synthesizes from all of them when generating tailored versions. Descriptive filenames help (e.g., `resume-pm.docx`, `resume-technical.docx`).

---

## 7. Run the Workflow

**Stage 2 — FitFoundry:** Paste the contents of `FITFOUNDRY-WORKFLOW.md` into a new Cowork session. The agent will ask which board to run and what work type (remote / in-person / all), then proceed automatically.

**Stage 3 — LaunchKit:** After reviewing your results in `OUTPUT_FOLDER`, paste the contents of `LAUNCHKIT.md` into a new Cowork session. The agent will present your scored jobs, ask which to pursue, and generate application materials for each.

You do not need to do anything during a run beyond the initial selections unless the agent encounters a CAPTCHA or needs clarification on document pairing.

---

## What Is and Is Not Committed

| File / Folder | Committed | Reason |
|---------------|-----------|--------|
| `.env.example` | ✅ Yes | Safe template — no real values |
| `.env` | ❌ No | Contains API keys and personal paths |
| `MY-PROFILE.md` | ✅ Yes | Boilerplate template — ProfileBuilder overwrites with your data |
| `QUICK-REFERENCE.md` | ✅ Yes | Boilerplate template — ProfileBuilder overwrites with your data |
| `PROFILEBUILDER.md` | ✅ Yes | Stage 1 workflow prompt |
| `CAREER-PROFILE-INTERVIEW.md` | ✅ Yes | Interview question bank used by ProfileBuilder |
| `FITFOUNDRY-WORKFLOW.md` | ✅ Yes | Stage 2 workflow prompt |
| `LAUNCHKIT.md` | ✅ Yes | Stage 3 workflow prompt |
| `JOB-BOARD-SITE-NOTES.md` | ✅ Yes | Technical scraping reference |
| `REQUIREMENTS.md` | ✅ Yes | Full tool and connector requirements |
| `README.md` | ✅ Yes | Project overview |
| `SETUP.md` | ✅ Yes | This file |
| `results/` | ❌ No | Contains scraped job descriptions and personal assessments |

---

## Folder Structure

```
fit_foundry/
├── .env.example                    ← config template — copy to .env and fill in
├── .env                            ← local config (not committed)
├── .gitignore                      ← excludes .env and results/
├── README.md                       ← project overview
├── SETUP.md                        ← this file
├── REQUIREMENTS.md                 ← full tool and connector requirements
├── CAREER-PROFILE-INTERVIEW.md     ← interview question bank (used by ProfileBuilder)
├── PROFILEBUILDER.md               ← Stage 1 workflow prompt
├── MY-PROFILE.md                   ← your career profile (generated by ProfileBuilder)
├── QUICK-REFERENCE.md              ← scoring settings and board search config
├── FITFOUNDRY-WORKFLOW.md          ← Stage 2 workflow prompt
├── LAUNCHKIT.md                    ← Stage 3 workflow prompt
├── JOB-BOARD-SITE-NOTES.md        ← board index and site file format reference
├── sites/                          ← per-board scraping notes and MCP workflow notes
│   ├── Indeed.md
│   ├── Dice.md
│   └── [BoardName].md ...
└── results/                        ← generated output (not committed)
    ├── YYYY-MM-DD-BoardName.md     ← master list + session summary per run
    ├── Company_Role_YYYY-MM-DD.md  ← individual job file (before LaunchKit)
    └── Company_JobTitle/           ← application folder (created by LaunchKit)
        ├── Company_JobTitle_YYYY-MM-DD.md
        ├── Resume_Company_JobTitle.docx
        └── CoverLetter_Company_JobTitle.docx
```
