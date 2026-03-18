# FitFoundry Setup — Stage 2: Workspace, Profile, and First Search

Stage 2 runs inside Cowork. It sets up your personal job search folder, builds your career profile, and prepares your search settings. Complete Stage 1 and restart Claude Desktop before starting here.

---

## Step 1 — Create Your Job Search Folder

Create a folder on your computer to hold your personal FitFoundry files — your profile, search settings, and job results. This is separate from the FitFoundry skill folder and stays on your machine.

Name it whatever you like. For example: `FitFoundry` or `Job Search 2026`.

---

## Step 2 — Switch to Cowork and Select Your Folder

1. Open Claude Desktop.
2. Click **Cowork** in the left sidebar (or from the mode switcher at the top of the window).
3. Click the **folder icon** in the Cowork sidebar to set your working directory.
4. Navigate to and select the folder you created in Step 1.

The folder name will appear in the sidebar when selected. You will need to select this same folder at the start of every FitFoundry session.

---

## Step 3 — Run Session 1 (Workspace Bootstrap)

With your folder selected, paste this prompt in the Cowork chat:

> Set up FitFoundry — Session 1

Claude will:

- Copy the FitFoundry workflow files from the installed skill into your folder
- Verify that Node.js and Puppeteer are available
- Confirm that Indeed and Dice connectors are active
- Offer to walk through Claude in Chrome setup if you skipped it in Stage 1
- Create a `.env` configuration file with your folder paths set automatically

This takes around 5–10 minutes. At the end, Claude will tell you to restart Claude Desktop.

**Restart Claude Desktop before continuing to Session 2.** The restart is required for all tools to be fully active.

How to restart:
- **Windows:** Right-click the Claude icon in the system tray (bottom-right taskbar — expand ^ if the icon is hidden) → **Quit**. Reopen from the Start menu or desktop shortcut.
- **Mac:** Click the Claude icon in the menu bar (top-right) → **Quit Claude**. Reopen from Applications or Dock.

---

## Step 4 — Run Session 2 (Profile Setup)

After restarting, open Cowork, select the same folder, and paste:

> Set up FitFoundry — Session 2

Claude will verify your tools are active, then run **ProfileBuilder** — a structured interview that creates two files in your folder:

- `MY-PROFILE.md` — your career profile, used to score every job
- `QUICK-REFERENCE.md` — your search settings and scoring calibration

**Resume tip:** If you have a resume as a `.docx` or `.pdf`, add it to your folder before starting this session. ProfileBuilder will pre-fill what it can infer from the resume and only ask about the rest, cutting the interview time significantly.

Choose your interview depth when prompted:

- **Quick (~5 minutes):** Covers role targets, skills, location, and hard constraints. Sufficient for basic matching.
- **Full (~20 minutes):** Adds culture, motivations, and work style. Improves scoring quality for borderline roles.

---

## Step 5 — Fill In Board-Specific Search Settings

After the profile interview completes, open `QUICK-REFERENCE.md` in your folder and fill in the **Board-Specific Search Settings** section near the bottom. Each board you plan to use has a few fields — keywords, location, and sometimes an industry filter. Leave boards you won't use blank.

This step cannot be automated because the right keywords depend on how each board indexes jobs, and that varies by board. A few minutes here will sharpen every search you run.

---

## Step 6 — Add Base Documents for LaunchKit (Optional)

If you plan to use LaunchKit (Stage 3) to generate tailored resumes and cover letters, copy your current resume and a sample cover letter into your job search folder. Accepted formats: `.docx` and `.pdf`. Multiple files are fine — LaunchKit will synthesize from all of them.

You can add these at any time before running LaunchKit. They are not needed for job searching.

---

## You Are Ready — Run Your First Search

Open a new Cowork session, select your folder, and paste:

> Run a FitFoundry job search

Claude will ask which board to search and what workplace type (Remote, Hybrid, In-Person, or All), then run the full workflow: scrape → filter → score → ghost job check → save results to your `results/` folder.

See `QUICKSTART.md` for all prompts and a troubleshooting reference.

---

## What Gets Saved Where

| File | Location | Contains personal data? |
|------|----------|------------------------|
| `MY-PROFILE.md` | Your job search folder | Yes — do not share |
| `QUICK-REFERENCE.md` | Your job search folder | Yes — do not share |
| `.env` | Your job search folder | Yes (file paths, API keys) — do not share |
| `results/` | Your job search folder | Yes — do not share |
| All workflow `.md` files | Your job search folder | No — generic, safe to share |

Your personal files stay on your computer and are never uploaded anywhere.
