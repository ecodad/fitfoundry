# Job Search Workflow

An AI-assisted job board scraping and evaluation system designed to run inside [Cowork](https://www.anthropic.com) (Anthropic's agentic desktop tool). The workflow automates the repetitive parts of a job search — scraping listings, filtering by location and role type, assessing each posting against a personal profile, and generating structured output files for follow-up.

---

## What It Does

1. **Scrapes job boards** using Puppeteer-based browser automation, handling lazy-loaded React apps, static pages, and direct API queries where available.
2. **Filters listings** by location, workplace type (remote / hybrid / in-person), and employment type (full-time only).
3. **Pre-screens candidates** using configurable title keyword include/exclude lists to reduce large result sets before visiting individual postings.
4. **Evaluates each job** against your personal profile, scoring interest and fit on a 1–10 scale with written rationale.
5. **Generates structured output** — a master list markdown table per run, and one individual markdown file per scored job containing the full job description, a fit summary, and a direct application link.
6. **Logs lessons learned** per board so scraping approaches improve over time without re-discovery.

---

## Repository Structure

```
job-search/
├── .env.example                  ← safe config template — copy to .env and fill in values
├── .gitignore                    ← excludes .env, MY-PROFILE.md, and results/
├── MY-PROFILE.md                 ← your personal profile template (not committed)
├── JOB-BOARD-WORKFLOW.md        ← the workflow prompt to paste into Cowork each session
├── JOB-BOARD-SITE-NOTES.md     ← per-site scraping reference, updated after each run
├── master_job_listing_sites.md  ← optional: notes on additional boards under evaluation
└── results/                     ← generated output files (not committed)
    ├── YYYY-MM-DD-BoardName.md  ← master list + session summary for each run
    └── Company_Role_YYYY-MM-DD.md  ← individual file for each scored job
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/ecodad/fit_foundry.git
cd fit_foundry
```

### 2. Configure your environment

```bash
cp .env.example .env
```

Open `.env` and set:

- `PROFILE_PATH` — path to your profile file (default: `./MY-PROFILE.md`)
- `OUTPUT_FOLDER` — where results are written (default: `./results/`)
- `ALGOLIA_APP_ID` / `ALGOLIA_API_KEY` / `ALGOLIA_INDEX` — credentials for the 80,000 Hours board (see note below)

### 3. Fill in your profile

Open `MY-PROFILE.md` and complete every section. The more specific you are, the more accurately the agent can assess job fit. Your profile is never committed to the repository.

### 4. Run the workflow

Open [Cowork](https://www.anthropic.com) and paste the full contents of `JOB-BOARD-WORKFLOW.md` into a new session. The agent will guide you through board selection and run the full workflow interactively.

---

## Supported Job Boards

| Board | URL | Notes |
|-------|-----|-------|
| Climatebase | climatebase.org | React SPA; requires scrollable container approach |
| Greentown Labs | greentownlabs.com/careers | Static WordPress; mixes Boston and Houston members |
| LinkedIn | linkedin.com/jobs | Login recommended for full results; capped at ~66 cards without |
| The Engine | engine.xyz/careers | MIT Tough Tech portfolio; static Getro platform |
| 80,000 Hours | jobs.80000hours.org | Algolia API; high curation, low volume |

Detailed scraping notes, selector patterns, filter quirks, and known ATS compatibility issues for each board are documented in `JOB-BOARD-SITE-NOTES.md`.

---

## Output Format

### Master list (`YYYY-MM-DD-BoardName.md`)

A markdown table of all listings collected in a run, followed by a session summary:

```
| # | Title | Company | Location | Workplace | Description | Link |
```

Appended at the end of each run:

```
## Session Summary
- Total listings reviewed: N
- Listings passing filter: N
- Jobs scored: N
- Jobs skipped: N

### Scored Jobs (sorted high to low)
| Score | Title | Company | File |
```

### Individual job file (`Company_Role_YYYY-MM-DD.md`)

Generated for each posting where both interest and fit are positive:

```markdown
# [Job Title] — [Company]

**Score:** X/10 — one sentence rationale

**Location:** ... | **Workplace:** ...

**Apply:** [link]

## Why This Role
...

## Potential Gaps
...

## Full Job Description
...
```

Scores follow this scale:

| Score | Meaning |
|-------|---------|
| 10 | Dream job — aligns perfectly with stated ideal role |
| 7–9 | Strong interest, good fit, worth pursuing |
| 4–6 | Acceptable role, reasonable fit, would consider |
| 1–3 | Would do it to get by, limited enthusiasm |
| Skip | Low interest OR poor fit — no file created |

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PROFILE_PATH` | Path to your personal profile file | `./MY-PROFILE.md` |
| `OUTPUT_FOLDER` | Directory for generated result files | `./results/` |
| `ALGOLIA_APP_ID` | 80k Hours Algolia application ID | *(required for board 6)* |
| `ALGOLIA_API_KEY` | 80k Hours Algolia read-only API key | *(required for board 6)* |
| `ALGOLIA_INDEX` | Algolia index name | `jobs_prod` |

Algolia credentials can be retrieved from browser devtools (Network tab) while on the 80,000 Hours job board. These are read-only public search keys. Rotate your `.env` values if the board's credentials change.

---

## What Is and Is Not Committed

| File / Folder | Committed | Reason |
|---------------|-----------|--------|
| `.env.example` | ✅ Yes | Safe template with no real values |
| `.env` | ❌ No | Contains API keys and personal paths |
| `MY-PROFILE.md` | ❌ No | Contains personal contact info and career details |
| `results/` | ❌ No | Contains scraped job descriptions and personal assessments |
| `JOB-BOARD-WORKFLOW.md` | ✅ Yes | Generic workflow prompt |
| `JOB-BOARD-SITE-NOTES.md` | ✅ Yes | Technical scraping reference |

---

## Adding a New Board

1. Run the workflow and select "new board" at the site selection step.
2. Document the scraping approach, card selectors, filter quirks, and any blocked ATS systems.
3. After the run, add a new section to `JOB-BOARD-SITE-NOTES.md` and a new row to the Known Boards table.
4. Record lessons learned in the Improvement Log section of `JOB-BOARD-WORKFLOW.md`.

---

## Requirements

- [Cowork](https://www.anthropic.com) — Anthropic's agentic desktop tool (used to run the workflow)
- A personal profile in `MY-PROFILE.md` (see template)
- `.env` configured with your paths and any required API keys

No additional software installation is required. All browser automation runs inside Cowork's built-in Puppeteer environment.

---

## License

MIT
