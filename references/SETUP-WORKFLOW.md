# FitFoundry Setup Workflow

Two-session setup. Session 1 builds the workspace, installs Puppeteer, and connects job board accounts. Session 2 runs ProfileBuilder to create the career profile after Claude Desktop has been restarted.

Do not combine sessions. The restart between them is required for connectors and the Puppeteer MCP server to activate.

---

## SESSION 1 — WORKSPACE, PUPPETEER, AND CONNECTORS

**Trigger phrase:** "Set up FitFoundry — Session 1"

---

### S1-STEP 0 — STATUS CHECK

Locate the user's workspace folder (the folder selected in Cowork). Check for each item and report status before doing anything else:

| Item | Check |
|------|-------|
| Workspace files | `FITFOUNDRY-WORKFLOW.md`, `PROFILEBUILDER.md`, `CAREER-PROFILE-INTERVIEW.md`, `LAUNCHKIT.md`, `JOB-BOARD-SITE-NOTES.md`, `REQUIREMENTS.md`, `.env` all exist in the workspace root |
| Profile template | `MY-PROFILE.md` exists |
| Quick reference template | `QUICK-REFERENCE.md` exists |
| Results folder | `results/` directory exists |
| Indeed connector | The `search_jobs` tool from the Indeed MCP is available |
| Dice connector | The `search_jobs` tool from the Dice MCP is available |
| Puppeteer MCP | The `puppeteer_navigate` tool is available |

Report a short table: ✅ in place / ❌ missing / ⚠️ partial. Then work through only the steps below that are needed.

---

### S1-STEP 1 — BOOTSTRAP WORKSPACE

If any workspace files are missing:

1. Locate the FitFoundry skill directory. In the Cowork VM, it is accessible at `.skills/skills/fitfoundry/` relative to the mounted workspace. Do not download anything from the internet — all files come from this local directory.

2. Copy the following files from the skill directory root to the workspace root:
   - `FITFOUNDRY-WORKFLOW.md`
   - `PROFILEBUILDER.md`
   - `CAREER-PROFILE-INTERVIEW.md`
   - `LAUNCHKIT.md`
   - `JOB-BOARD-SITE-NOTES.md`
   - `REQUIREMENTS.md`
   - `QUICKSTART.md`
   - `MY-PROFILE.md`
   - `QUICK-REFERENCE.md`

3. Copy the entire `sites/` directory from the skill to the workspace root.

4. Create a `results/` directory in the workspace root if it does not already exist.

5. Create a `.env` file in the workspace root. **Detect the workspace folder's absolute path automatically** by resolving the mounted workspace directory (the folder the user selected in Cowork). Do not use `./` relative paths — always write fully resolved absolute paths:

```
PROFILE_PATH=[resolved absolute path]/MY-PROFILE.md
QUICK_REFERENCE_PATH=[resolved absolute path]/QUICK-REFERENCE.md
OUTPUT_FOLDER=[resolved absolute path]/results/
ALGOLIA_APP_ID=
ALGOLIA_API_KEY=
ALGOLIA_INDEX=jobs_prod
```

To resolve the path: use the Cowork workspace mount point. In the VM, the workspace is accessible at a known mount path — read the real path from that mount and use it to construct the `.env` values. The user should never need to type or find a path manually.

6. Confirm each item created to the user. Show the resolved paths from `.env` so the user can verify they look correct.

---

### S1-STEP 2 — NODE.JS CHECK

Puppeteer requires Node.js. Check whether it is installed:

Run `node --version` in bash.

- If a version number is returned (e.g., `v20.11.0`): confirm to the user that Node.js is present and move on.
- If the command is not found: tell the user:

---
Node.js is required for the Puppeteer browser automation tool. It is not currently installed on your computer.

To install it:
1. Go to **nodejs.org** in your browser.
2. Download the **LTS** version (the one labeled "Recommended For Most Users").
3. Run the installer and follow the prompts — defaults are fine.
4. Once installed, come back here and say "Node.js is installed" so I can continue.

---

Wait for the user to confirm before proceeding.

---

### S1-STEP 3 — PUPPETEER MCP SERVER

If the `puppeteer_navigate` tool is not available:

Tell the user: "I'm going to install the Puppeteer MCP server now. This is what allows FitFoundry to control a browser window to scrape job listings. It only needs to be installed once."

Run the following in bash to install the Puppeteer MCP server globally:

```bash
npm install -g @modelcontextprotocol/server-puppeteer
```

After the install completes:

Tell the user:

---
Puppeteer is installed. To activate it, I need to register it with Claude Desktop. Here's how:

1. Click the **gear icon** in the **bottom-left corner** of Claude Desktop to open Settings.
2. Click **MCP Servers** (or **Developer** → **MCP Servers** depending on your version).
3. Click **Add Server** or **Edit Config**.
4. Add the following entry to your MCP server configuration:

```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

If there are already other servers in the config, add `"puppeteer": {...}` alongside them — do not replace the existing entries.

5. Save the configuration and close Settings.

Let me know when that's done.

---

If the user has trouble finding the MCP server settings, offer: "Look for a section called MCP, Developer, or Advanced in the Settings sidebar. If you can't find it, check the application menu at the top of your screen under Preferences or Settings."

---

### S1-STEP 4 — INDEED CONNECTOR

If the Indeed connector is not active:

---
To search Indeed jobs, you need to connect your Indeed account:

1. Click the **gear icon** in the **bottom-left corner** of Claude Desktop to open Settings.
2. Click **Connectors** in the left sidebar.
3. Find **Indeed** in the connector list and click **Connect**.
4. A browser window will open. Log in to your Indeed account. If your account uses two-factor authentication, complete that step too — the browser window will wait for you.
5. Once logged in, return to Claude Desktop. Indeed should show a green "Connected" status.

Let me know when it shows as Connected, or if you run into any issues.

---

If the user reports the Settings icon or Connectors section looks different, tell them: "The Settings layout may vary by version. Look for a gear or cog icon anywhere on the left sidebar, or check the application menu at the top of the screen. Once in Settings, look for a Connectors or Integrations section."

---

### S1-STEP 5 — DICE CONNECTOR

If the Dice connector is not active:

---
To search Dice jobs (strong for tech and engineering roles):

1. Still in Settings → Connectors, find **Dice** in the list and click **Connect**.
2. Log in to your Dice account in the browser window that opens.
3. Return to Claude Desktop — Dice should show as Connected.

If you don't have a Dice account and don't plan to search Dice, say "skip" and I'll move on.

---

---

### S1-STEP 6 — CLAUDE IN CHROME (OPTIONAL)

Ask the user: "One optional step — do you plan to search LinkedIn or Wellfound? Both require a browser extension called Claude in Chrome. It takes about 5 minutes to set up. If you're not sure, you can skip this now and add it later."

If the user says yes:

---
To set up Claude in Chrome:

1. Open **Google Chrome** (not Edge, Firefox, or another browser).
2. Go to the Chrome Web Store and search for **"Claude in Chrome"**, or go to chrome.google.com/webstore and search by name.
3. Click **Add to Chrome** and confirm the installation.
4. Log in to your LinkedIn and/or Wellfound accounts in Chrome — you need to be already logged in before starting a search session.
5. When you run a LinkedIn or Wellfound search in Cowork, Claude will ask you to click **Connect** in the Chrome extension.

Let me know when the extension is installed, or say "skip" to do this later.

---

---

### S1-STEP 7 — RESTART HANDOFF

Tell the user the following. Do not abbreviate — the restart instruction must be clear:

---
Your workspace is set up and your tools are connected. **You need to restart Claude Desktop now** for the Puppeteer MCP server and any connectors to become fully active. This is a one-time step.

**To restart Claude Desktop:**
- **Windows:** Right-click the Claude icon in the **system tray** (bottom-right corner of the taskbar) → click **Quit**. If the icon is hidden, click the small ^ arrow to expand the tray. Then reopen Claude Desktop from your desktop or Start menu.
- **Mac:** Click the Claude icon in the **menu bar** (top-right of the screen) → click **Quit Claude**. Then reopen from Applications or your Dock.

**Once it has restarted:**
1. Open Cowork and select the same folder you used today.
2. Paste this prompt to run your profile setup:

> Set up FitFoundry — Session 2

Your workspace files are saved and will be there when you return.

---

---

## SESSION 2 — PROFILE SETUP

**Trigger phrase:** "Set up FitFoundry — Session 2"

---

### S2-STEP 0 — VERIFY

1. Read `.env` from the workspace folder to confirm `PROFILE_PATH`, `QUICK_REFERENCE_PATH`, and `OUTPUT_FOLDER` are set.
2. Check that the Puppeteer MCP (`puppeteer_navigate`), Indeed connector, and Dice connector are now active. Report status. If any are still missing, tell the user: "It looks like [item] is not active yet. You may need to reconnect it from Settings → Connectors, or check that Claude Desktop was fully restarted after Session 1."
3. Confirm that `PROFILEBUILDER.md` and `CAREER-PROFILE-INTERVIEW.md` are present in the workspace.

---

### S2-STEP 1 — RUN PROFILEBUILDER

Tell the user:

---
Everything looks good. The next step is to build your career profile — this is what FitFoundry uses to score every job against your background and preferences.

I'm going to run ProfileBuilder now. It will ask whether you have a resume to upload (recommended — it speeds things up significantly), then offer you a choice between a Quick Interview (~5 minutes) or a Full Interview (~20 minutes). The full interview produces better scoring results, especially for culture-fit and borderline roles.

You can say "skip" to any question and come back to it later.

---

Then read `PROFILEBUILDER.md` in full and execute it from STEP 0 through STEP 6 without re-summarizing the instructions to the user. Run it as written.

---

### S2-STEP 2 — BOARD-SPECIFIC SEARCH SETTINGS

After ProfileBuilder completes and `QUICK-REFERENCE.md` has been written, tell the user:

---
Your profile is saved. One more thing before your first search: open `QUICK-REFERENCE.md` in your workspace folder and fill in the **Board-Specific Search Settings** section near the bottom. Each board has a few fields — keywords, location, and in some cases industry filters.

These can't be auto-filled because the right keywords depend on how each board indexes jobs, and that varies. A few minutes spent here will sharpen every search you run.

For each board you plan to use, fill in the fields. Leave boards you won't use blank.

Let me know when you've filled those in, or say "skip" to do it before your first search instead.

---

---

### S2-STEP 3 — BASE DOCUMENTS FOR LAUNCHKIT

Tell the user:

---
One last optional setup step: if you plan to use LaunchKit (Stage 3) to generate tailored resumes and cover letters, add your base resume and a sample cover letter to your workspace folder now. Accepted formats are `.docx` and `.pdf`. You can add them any time before running LaunchKit — they don't need to be there today.

---

---

### S2-STEP 4 — CONFIRM COMPLETE

Tell the user:

---
FitFoundry setup is complete. Here is what's in your workspace:

| File | Purpose |
|------|---------|
| `MY-PROFILE.md` | Your career profile — source of truth for all job scoring |
| `QUICK-REFERENCE.md` | Search settings and scoring calibration |
| `FITFOUNDRY-WORKFLOW.md` | Paste into Cowork to run a job search (Stage 2) |
| `PROFILEBUILDER.md` | Re-run any time your situation changes (Stage 1) |
| `LAUNCHKIT.md` | Paste into Cowork to generate application materials (Stage 3) |
| `JOB-BOARD-SITE-NOTES.md` | Per-board scraping notes — loaded automatically |
| `sites/` | Individual board files — loaded automatically |
| `results/` | Where job search output is saved |
| `QUICKSTART.md` | Prompts and troubleshooting reference |

**To run your first job search (Stage 2):**
Open a new Cowork session, select this folder, and say:

> Run a FitFoundry job search

Your `QUICKSTART.md` has all the prompts you need going forward.

---
