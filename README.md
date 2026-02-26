# FitFoundry — AI-Assisted Job Search Workflow

An AI-assisted job board scraping and evaluation system designed to run inside [Cowork](https://www.anthropic.com) (Anthropic's agentic desktop tool). The workflow automates the repetitive parts of a job search — scraping listings, filtering by location and role type, assessing each posting against a personal profile, verifying postings are real, and generating structured output files for follow-up.

For setup instructions, see [SETUP.md](SETUP.md).

---

## What It Does

1. **Scrapes job boards** using Puppeteer-based browser automation and, for login-gated boards, the Claude in Chrome extension. Handles lazy-loaded React apps, static pages, and direct API queries where available.
2. **Filters listings** by location, workplace type (remote / hybrid / in-person), and employment type (full-time only).
3. **Pre-screens candidates** using title keyword include/exclude lists to reduce large result sets before visiting individual postings.
4. **Evaluates each job** against your personal profile, scoring interest and fit on a 1–10 scale with written rationale.
5. **Verifies postings** against the company's own career site to flag potential ghost jobs before you invest time applying.
6. **Generates structured output** — a master list markdown table per run, and one individual markdown file per scored job containing the full job description, fit summary, ghost job status, and direct application links.
7. **Logs lessons learned** per board so scraping approaches improve over time without re-discovery.

---

## Supported Job Boards

| # | Board | URL | Notes |
|---|-------|-----|-------|
| 1 | Climatebase (Remote) | climatebase.org | React SPA; Algolia-backed; scrollable container approach |
| 2 | Climatebase (In-Person) | climatebase.org | Same board, Hybrid + In-person filter via UI |
| 3 | Greentown Labs | greentownlabs.com/careers | Static; mixes Boston and Houston members; no date filter |
| 4 | LinkedIn | linkedin.com/jobs | **Requires Claude in Chrome + login.** JS injection blocked — uses accessibility tree. Pages 1–5 of TPM search adequate. |
| 5 | The Engine | engine.xyz/careers | MIT Tough Tech portfolio; Getro platform; URL params work |
| 6 | 80,000 Hours | jobs.80000hours.org | Algolia API; high curation, low volume; creds in `.env` |
| 7 | Draper Laboratory | draper.wd5.myworkdayjobs.com | Workday CXS API; most roles require security clearance |
| 8 | Work on Climate | workonclimate.org | Slack community + newsletter only — not scrapable |
| 9 | Wellfound | wellfound.com/jobs | **Requires Claude in Chrome + login.** Cloudflare CAPTCHA blocks Puppeteer. |
| 10 | Y Combinator | workatastartup.com | Server-rendered; no login required; industry filter unreliable |
| 11 | BEV Jobs | bevjobs.breakthroughenergy.org | Getro; ~767 jobs; UI filters required (URL params → HTTP 500) |
| 12 | BEF Jobs | befjobs.breakthroughenergy.org | Getro; ~77 jobs; small enough to review unfiltered |
| 13 | Formlabs | careers.formlabs.com | Direct company site; React Table; native select filter; no auth; all roles load immediately |
| 14 | Indeed | indeed.com | MCP connector — `search_jobs` + `get_job_details` + `get_company_data`. Requires Indeed connector active at session start. |
| 15 | Dice | dice.com | MCP connector for discovery + Puppeteer for full descriptions. Tech-focused; higher IC engineering density. |

Detailed scraping notes, selector patterns, filter quirks, and known ATS compatibility issues for each board are in `JOB-BOARD-SITE-NOTES.md`.

---

## Ghost Job Check

After scoring jobs and creating individual files, the agent attempts to verify each posting on the company's own career site — not the aggregator board it was sourced from. This catches ghost jobs: postings left up on job boards after a role has been filled or quietly cancelled.

**How it works:** The agent navigates to the company's ATS (Workday, Lever, Greenhouse, Ashby, etc.) and searches for the exact job title. Most companies' career sites point directly to their ATS, and many ATS platforms expose a public search API that can be queried without authentication.

**Each individual job file is updated with one of:**

| Status | Meaning |
|--------|---------|
| ✅ Confirmed live on [ATS] (date) | Posting found on company career site — real job |
| ⚠️ Possible Ghost Job — posting not found on company career site (date) | Not found; may be filled, cancelled, or never posted directly |
| ❓ Unverified — career site inaccessible (date) | Site blocked or CAPTCHA prevented verification |

Jobs that cannot be confirmed are kept — the posting may be legitimate but listed only on the aggregator, or the career site may have been temporarily inaccessible. The status is advisory.

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

**Apply:** [link to job board listing]
**Company career site:** [direct link to posting on company ATS]
**Ghost job check:** ✅ / ⚠️ / ❓ [status and date]

## Why This Role
...

## Potential Gaps
...

## Full Job Description
...
```

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

1. Run the workflow and select "new board" at the site selection step.
2. Document the scraping approach, card selectors, filter quirks, and any blocked ATS systems in `JOB-BOARD-SITE-NOTES.md`.
3. Add a new row to the Known Boards table in `JOB-BOARD-SITE-NOTES.md`.
4. Record lessons learned in the Improvement Log section of `FITFOUNDRY-WORKFLOW.md`.

---

## Requirements

- [Cowork](https://www.anthropic.com) — Anthropic's agentic desktop tool
- Claude in Chrome browser extension (for LinkedIn and Wellfound)
- A completed `MY-PROFILE.md` and configured `.env`

See [SETUP.md](SETUP.md) for full configuration instructions.
