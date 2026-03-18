# FitFoundry — Quick Start

Copy-pasteable prompts for every stage, plus troubleshooting for common issues.

---

## First-Time Setup

Setup runs across two sessions with a Claude Desktop restart between them.

---

**Session 1 — Workspace, Puppeteer, and connectors** (15–20 minutes)

1. Open Cowork and select your FitFoundry folder.
2. Paste this prompt:

> Set up FitFoundry — Session 1

Claude will copy workflow files from the skill, install the Puppeteer browser automation tool, and walk you through connecting your job board accounts. At the end it will tell you to restart Claude Desktop.

**Restart Claude Desktop before continuing to Session 2.**

---

**Session 2 — Profile setup** (5–25 minutes depending on interview length)

1. After restarting, open Cowork and select the same folder.
2. Paste this prompt:

> Set up FitFoundry — Session 2

Claude will verify your tools are active, then run ProfileBuilder — an interview that creates your career profile and search settings. You can provide a resume to speed it up. You choose Quick (~5 minutes) or Full (~20 minutes) interview depth.

When the interview is done, open `QUICK-REFERENCE.md` in your folder and fill in the board-specific search settings (keywords, location) for each board you plan to use.

---

## Stage 2 — Running a Job Search

Open a new Cowork session, select your FitFoundry folder, and paste this prompt:

> Run a FitFoundry job search

Claude will ask which board to search and what run type (Remote, Hybrid, In-Person, or All), then run the full workflow: scrape → filter → score → ghost job check → save results to your `results/` folder.

---

## Stage 3 — Generating Application Materials

After reviewing your results, open a new Cowork session, select your folder, and paste:

> Run LaunchKit

Claude will show you the scored jobs from your results folder, let you pick which ones to pursue, and generate a tailored resume and cover letter (`.docx`) for each. Make sure your base resume and a sample cover letter are in your workspace folder before running.

---

## Other Prompts

| What you want to do | Prompt to paste |
|---------------------|-----------------|
| Update your profile | "Run ProfileBuilder" |
| Re-run profile for a specific board | "Run ProfileBuilder — quick interview only" |
| Check results from a recent run | "Show me the scored jobs from my last FitFoundry run" |
| Add a job you already applied to | "Add [Company] [Role] to my active applications in QUICK-REFERENCE.md" |
| Add a new job board | "Add a new job board to FitFoundry — the URL is [url]" |

---

## Troubleshooting

**Claude says it cannot find my workspace files**

Make sure you selected your FitFoundry folder when opening Cowork. Look for the folder icon in the Cowork sidebar — click it and select the same folder you used during setup.

---

**Claude says a connector (Indeed, Dice) is not active**

Open Settings (gear icon, bottom-left in Claude Desktop) → Connectors. If the connector shows as disconnected, click Connect and log in again. After reconnecting you do not need to restart — it activates in the current session.

If the Settings icon or Connectors section looks different from this description, look for a gear or cog icon anywhere on the left sidebar, or check the application menu at the top of the screen.

---

**Claude says Puppeteer is not active**

Open Settings → MCP Servers (sometimes under Developer or Advanced). Check that the Puppeteer server entry is present. If it is missing, follow the Puppeteer setup steps in `SETUP-STAGE1.md` or say "Help me reinstall the Puppeteer MCP server" in a new Cowork session.

---

**Claude Desktop froze or became unresponsive**

- **Windows:** Right-click the Claude icon in the system tray (bottom-right of the taskbar) → **Quit**. If the icon is hidden, click the ^ arrow to expand the tray. If that doesn't work: Ctrl+Alt+Delete → Task Manager → find Claude Desktop → End Task.
- **Mac:** Click the Claude icon in the menu bar → **Quit Claude**. If it's unresponsive: Cmd+Option+Esc → Force Quit → Claude Desktop.

Your workspace files are saved to disk and will be intact when you reopen.

---

**I lost progress during the ProfileBuilder interview**

ProfileBuilder saves `MY-PROFILE.md` at the end of the run, not incrementally. If the session crashed mid-interview, open a new Cowork session and say "Run ProfileBuilder" — it will start fresh. The interview is designed to move quickly, especially if you provide your resume upfront.

---

**The connector setup requires a restart but I already closed Session 1**

That is fine — restart Claude Desktop, open a new Cowork session, select your folder, and paste the Session 2 prompt. Claude will check connector and Puppeteer status at the start of Session 2 and flag anything that still needs attention.

---

**I want to search the 80,000 Hours job board**

This board requires Algolia API credentials. Open your `.env` file and fill in `ALGOLIA_APP_ID` and `ALGOLIA_API_KEY`. To get these values: go to jobs.80000hours.org in Chrome, open DevTools (F12) → Network tab, and look for requests to `algolia.net`. The App ID and API Key appear in the request headers. Also add them to the Credentials section of `QUICK-REFERENCE.md`. These are public read-only keys — safe to save locally.

---

**My cover letter sounds like it was written by AI**

After LaunchKit generates a draft, open the `.docx` file and paste the cover letter text into a new Cowork session with this prompt:

> Review this cover letter for phrases that read as AI-generated — especially hedged praise, throat-clearing openers, and buzzword constructions like "sits at the intersection of" or "which is precisely what this role requires." Rewrite any flagged sentences in the first person, grounded in a specific detail from the job description or my background. Do not add new content — just make what is there sound like me.

---

**The search returned very few listings**

Open `QUICK-REFERENCE.md` and check the Board-Specific Search Settings for the board you ran. Broaden keywords, add an industry category, or adjust the location filter. Then run the search again in a new Cowork session.
