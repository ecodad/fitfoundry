# FitFoundry Setup Guide

Everything you need to get FitFoundry running for the first time. No technical background required.

---

## What You Need Before Starting

| Requirement | Notes |
|-------------|-------|
| **Claude Desktop with Cowork** | Download from anthropic.com if not already installed |
| **Node.js** | Required for the Puppeteer browser automation tool. Download the LTS version from nodejs.org. If you are not sure whether it is already installed, the setup process will check for you. |
| **Indeed account** | Free, at indeed.com. Required to search Indeed. |
| **Dice account** | Free, at dice.com. Required to search Dice. Skip if you don't plan to use it. |
| **Wellfound or LinkedIn account** | Required only if you plan to search those boards. Also requires the Claude in Chrome browser extension (see Session 1). |
| **Google Chrome** | Required only for LinkedIn and Wellfound searches. |

---

## Step 1 — Install the FitFoundry Skill

FitFoundry runs as a skill inside Cowork. Installing it means placing the skill folder in the right location so Cowork can find it.

**Option A — Plugin Marketplace (easiest)**

If FitFoundry appears in your Cowork plugin marketplace, install it from there and skip to Step 2.

**Option B — Manual Install**

1. Download the FitFoundry ZIP from the releases page: go to **github.com/ecodad/fitfoundry** in your browser, click **Releases** on the right side, then click **Assets** under the latest release and download the ZIP.

   > Do this in your browser — Claude Desktop cannot reach GitHub on your behalf.

2. Extract the ZIP. You will get a folder called `fitfoundry-main` (or similar). Open it and confirm you can see `SKILL.md` and a `sites/` folder inside.

3. Find your Claude skills folder:
   - **Windows:** `C:\Users\[YourName]\AppData\Roaming\Claude\skills\`
   - **Mac:** `~/Library/Application Support/Claude/skills/`

   The `AppData` folder on Windows is hidden by default. In File Explorer, paste the path directly into the address bar and press Enter.

   If the `skills` folder does not exist, create it.

4. Copy the extracted folder into the skills folder and rename it from `fitfoundry-main` to `fitfoundry`. The result should look like this:

   ```
   Claude/
   └── skills/
       └── fitfoundry/
           ├── SKILL.md
           ├── CAREER-PROFILE-INTERVIEW.md
           ├── FITFOUNDRY-WORKFLOW.md
           ├── PROFILEBUILDER.md
           ├── LAUNCHKIT.md
           ├── ... (other files)
           └── sites/
   ```

5. Restart Claude Desktop. The FitFoundry skill will now be available in Cowork.

---

## Step 2 — Create Your Job Search Folder

Create a new folder somewhere on your computer for your personal job search files. This is separate from the skill folder — it will hold your profile, search settings, and job results.

Name it whatever you like. You will select this folder in Cowork at the start of every session.

---

## Step 3 — Run Session 1 (Workspace, Puppeteer, and Connectors)

1. Open Claude Desktop and click **Cowork**.
2. Click the folder icon and select the folder you created in Step 2.
3. Paste this prompt:

> Set up FitFoundry — Session 1

Claude will:
- Copy the FitFoundry workflow files from the skill into your folder (no internet download needed)
- Check whether Node.js is installed and guide you to install it if not
- Install the Puppeteer MCP server (browser automation — required for most job boards) and walk you through registering it in Claude Desktop settings
- Walk you through connecting your Indeed and Dice accounts
- Optionally set up Claude in Chrome for LinkedIn and Wellfound
- Tell you to restart Claude Desktop when everything is done

**At the end of Session 1, restart Claude Desktop before continuing.** Both Puppeteer and the connectors require a restart to activate.

---

## Step 4 — Run Session 2 (Profile Setup)

After restarting Claude Desktop:

1. Open Cowork and select the same folder.
2. Paste this prompt:

> Set up FitFoundry — Session 2

Claude will verify your tools are active, then run **ProfileBuilder** — a structured interview that creates your career profile (`MY-PROFILE.md`) and search settings (`QUICK-REFERENCE.md`). You can provide a resume to speed up the interview; it will pre-fill what it can and only ask about the rest.

You choose interview depth:
- **Quick (~5 minutes):** Covers role targets, skills, location, and hard constraints. Sufficient for basic matching.
- **Full (~20 minutes):** Adds culture, motivations, and work style. Improves scoring quality for borderline roles.

When the interview is done, open `QUICK-REFERENCE.md` in your folder and fill in the **Board-Specific Search Settings** near the bottom — keywords and location for each board you plan to use.

---

## Step 5 — Add Your Base Documents (for LaunchKit)

If you plan to use LaunchKit (Stage 3) to generate tailored resumes and cover letters, copy your current resume and a sample cover letter into your job search folder. Accepted formats: `.docx` and `.pdf`. You can do this any time before running LaunchKit.

---

## You Are Ready

Open a new Cowork session, select your folder, and say:

> Run a FitFoundry job search

Your `QUICKSTART.md` has all the prompts you need going forward, plus a troubleshooting section for common issues.

---

## Troubleshooting

**How to restart Claude Desktop**

- **Windows:** Right-click the Claude icon in the system tray (the small icons in the bottom-right corner of the taskbar) → click **Quit**. The icon may be hidden behind the ^ arrow. Then reopen from your desktop shortcut or Start menu.
- **Mac:** Click the Claude icon in the menu bar (top-right of screen) → **Quit Claude**. Then reopen from Applications or Dock.

---

**Claude Desktop froze and won't respond**

- **Windows:** Press Ctrl+Alt+Delete → Task Manager → find Claude Desktop → End Task. Then reopen normally.
- **Mac:** Press Cmd+Option+Esc → Force Quit → Claude Desktop.

Your workspace files are saved to disk and will be intact when you reopen.

---

**The Settings gear icon or MCP Servers section is not where described**

Claude Desktop's interface changes between versions. Look for a gear or cog icon anywhere on the left sidebar, or check the application menu at the top of the screen under Preferences or Settings. Once in Settings, look for sections called Connectors, MCP Servers, or Developer.

---

**A connector shows as disconnected after restart**

Open Settings → Connectors and click Connect next to the affected connector. Log in again. You do not need to restart a second time — it will activate in the current session.

---

**Puppeteer is not active after restart**

Open Settings → MCP Servers and confirm the Puppeteer entry is present. If it is missing, open a new Cowork session and say "Help me reinstall the Puppeteer MCP server" — Claude will walk you through it.

---

## What Gets Saved Where

| File | Location | Contains personal data? |
|------|----------|------------------------|
| `MY-PROFILE.md` | Your job search folder | Yes — do not share |
| `QUICK-REFERENCE.md` | Your job search folder | Yes — do not share |
| `.env` | Your job search folder | Yes (file paths, API keys) — do not share |
| `results/` | Your job search folder | Yes — do not share |
| All workflow `.md` files | Your job search folder | No — generic, safe to share |
| `SKILL.md` + `sites/` | Claude skills folder | No — do not edit directly |

Your personal files stay on your computer. They are never uploaded anywhere.
