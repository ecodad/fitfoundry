# FitFoundry — AI-Assisted Job Search Workflow

An AI-assisted job search system designed to run inside [Cowork](https://www.anthropic.com) (Anthropic's agentic desktop tool). FitFoundry automates the repetitive parts of a job search across three stages: building a career profile, scraping and scoring job boards, and generating tailored application materials.

For setup instructions, see [SETUP.md](SETUP.md). For a full list of tools and connectors required, see [REQUIREMENTS.md](REQUIREMENTS.md).

---

## Three-Stage Workflow

| Stage | File | Purpose | When to Run |
|---|---|---|---|
| 1 — ProfileBuilder | `PROFILEBUILDER.md` | Conducts an interview to generate your career profile and scoring quick reference | Once; re-run when your situation changes |
| 2 — FitFoundry | `FITFOUNDRY-WORKFLOW.md` | Scrapes a job board, scores listings against your profile, verifies postings, and generates output files | Repeatedly — different boards and periodic re-runs to catch new listings |
| 3 — LaunchKit | `LAUNCHKIT.md` | Generates tailored resumes and cover letters for jobs you select from your results | After reviewing Stage 2 output and deciding which roles to pursue |

---

## What Each Stage Does

### Stage 1 — ProfileBuilder

Conducts a structured interview (quick ~5 min or full ~20 min) and generates two files that drive all subsequent stages:

- `MY-PROFILE.md` — your career profile, used by FitFoundry as the scoring source of truth
- `QUICK-REFERENCE.md` — scoring calibration, hard filters, and board-specific search settings

If you provide a resume, ProfileBuilder pre-fills answers it can infer and only asks for what's missing.

### Stage 2 — FitFoundry

1. **Scrapes job boards** using the Puppeteer MCP server and, for login-gated boards, the Claude in Chrome extension. Handles lazy-loaded React apps, static pages, and direct API queries where available.
2. **Filters listings** by location, workplace type (remote / hybrid / in-person), and employment type (full-time only).
3. **Pre-screens by title** using include/exclude keywords to reduce large result sets before visiting individual postings.
4. **Scores each job** against your profile — interest and fit on a 1–10 scale with written rationale.
5. **Verifies postings** against the company's own career site to flag potential ghost jobs.
6. **Generates structured output** — a master list per run, and one individual markdown file per scored job.
7. **Logs lessons learned** per board so scraping approaches improve over time without re-discovery.

### Stage 3 — LaunchKit

1. Scans your results folder and presents scored jobs as a numbered list.
2. You select which jobs to pursue, with optional per-job notes to Claude.
3. LaunchKit discovers your base resume(s) and cover letter(s) in the workspace and synthesizes tailored versions for each job.
4. Creates a folder per job containing the original report, a tailored resume, and a tailored cover letter.
5. Batches any ambiguous document-pairing questions to the end rather than interrupting the run.

---

## Supported Job Boards

| # | Board | URL | Notes |
|---|-------|-----|-------|
| 1 | Climatebase (Remote) | climatebase.org | React SPA; Algolia-backed; scrollable container approach |
| 2 | Climatebase (In-Person) | climatebase.org | Same board, Hybrid + In-person filter via UI |
| 3 | Greentown Labs | greentownlabs.com/careers | Static; mixes Boston and Houston members; no date filter |
| 4 | LinkedIn | linkedin.com/jobs | **Requires Claude in Chrome + login.** JS injection blocked — uses accessibility tree. |
| 5 | The Engine | engine.xyz/careers | MIT Tough Tech portfolio; Getro platform; URL params work |
| 6 | 80,000 Hours | jobs.80000hours.org | Algolia API; high curation, low volume; creds in `.env` |
| 7 | Draper Laboratory | draper.wd5.myworkdayjobs.com | Workday CXS API; most roles require security clearance |
| 8 | Work on Climate | workonclimate.org | Slack community + newsletter only — not scrapable |
| 9 | Wellfound | wellfound.com/jobs | **Requires Claude in Chrome + login.** Cloudflare CAPTCHA blocks Puppeteer. |
| 10 | Y Combinator | workatastartup.com | Server-rendered; no login required; industry filter unreliable |
| 11 | BEV Jobs | bevjobs.breakthroughenergy.org | Getro; ~767 jobs; UI filters required (URL params → HTTP 500) |
| 12 | BEF Jobs | befjobs.breakthroughenergy.org | Getro; ~77 jobs; small enough to review unfiltered |
| 13 | Formlabs | careers.formlabs.com | Direct company site; React Table; native select filter; no auth |
| 14 | Indeed | indeed.com | MCP connector — `search_jobs` + `get_job_details` + `get_company_data` |
| 15 | Dice | dice.com | MCP connector for discovery + Puppeteer for full descriptions |

Detailed scraping notes, selector patterns, filter quirks, and known ATS compatibility issues for each board are in `JOB-BOARD-SITE-NOTES.md`.

---

## Ghost Job Check

After scoring jobs, FitFoundry attempts to verify each posting on the company's own career site to catch ghost jobs — listings left up after a role has been filled or cancelled.

| Status | Meaning |
|--------|---------|
| ✅ Confirmed live on [ATS] (date) | Posting found on company career site |
| ⚠️ Possible Ghost Job — not found on company career site (date) | May be filled, cancelled, or never posted directly |
| ❓ Unverified — career site inaccessible (date) | Site blocked or CAPTCHA prevented verification |

---

## Output Format

### Master list (`results/YYYY-MM-DD-BoardName.md`)

A markdown table of all listings collected in a run, followed by a session summary.

### Individual job file (`results/Company_Role_YYYY-MM-DD.md`)

Generated by FitFoundry for each posting scoring 6 or above:

```markdown
# [Job Title] — [Company]

**Score:** X/10 — one sentence rationale
**Location:** ... | **Workplace:** ...
**Apply:** [link]
**Company career site:** [link]
**Ghost job check:** ✅ / ⚠️ / ❓ [status and date]

## Why This Role
## Potential Gaps
## Full Job Description
```

When LaunchKit processes a job, it adds a status block after the Ghost job check line and moves the file into the job's application folder:

```markdown
**Application Status:** Pending
**Applied:** —
**Decision Notes:** —
```

Status values: `Pending` (assets generated), `Applied` (with submission date), `Declined` (with reason).

### Application folder (`results/Company_JobTitle/`)

Created by LaunchKit for each job selected for pursuit:

```
results/
└── Company_JobTitle/
    ├── Company_JobTitle_YYYY-MM-DD.md     ← job report (moved from results/)
    ├── Resume_Company_JobTitle.docx        ← tailored resume
    └── CoverLetter_Company_JobTitle.docx   ← tailored cover letter
```

Prefix a folder with `_` to flag it as an active pursuit at a glance in your file system.

### Scoring scale

| Score | Meaning | Action |
|-------|---------|--------|
| 9–10 | Dream job — near-perfect alignment | Individual file |
| 7–8 | Strong fit — worth pursuing | Individual file |
| 6 | Partial fit — one significant gap | Individual file |
| 5 | Partial fit — multiple gaps | Master list only |
| 4 | Marginal | Master list only |
| ≤ 3 | Poor fit | Skip |

---

## Adding a New Board

1. Run FitFoundry and select "new board" at Step 0.
2. Document the scraping approach, card selectors, and filter quirks in a new `sites/[BoardName].md` file.
3. Add a row to the Known Boards table in `JOB-BOARD-SITE-NOTES.md`.
4. Record lessons learned in the Improvement Log in `FITFOUNDRY-WORKFLOW.md`.

---

## Requirements

- [Cowork](https://www.anthropic.com) — Anthropic's agentic desktop tool
- Puppeteer MCP server — required for Stage 2 on most boards; not bundled with Cowork
- Claude in Chrome browser extension — required for LinkedIn and Wellfound
- Node.js — required for Puppeteer

See [REQUIREMENTS.md](REQUIREMENTS.md) for the full breakdown by stage and board.
See [SETUP.md](SETUP.md) for step-by-step configuration instructions.
