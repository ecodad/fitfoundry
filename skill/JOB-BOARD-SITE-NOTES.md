# Job Board Site Notes

Reference for the agent running the job board workflow. Read this file at the start of each session, then load only the `sites/` file for the board you are running — do not load all site files into context at once.

---

## Known Boards

### Primary — MCP Connectors (Recommended Starting Point)

*Official API access. No Terms of Service concerns for personal job search use.*

| # | Label | Site File | Last Run | Auth | Method |
|---|-------|-----------|----------|------|--------|
| 14 | Indeed | `sites/Indeed.md` | Never | Indeed account (MCP connector) | Indeed MCP — `search_jobs` + `get_job_details` tools |
| 15 | Dice | `sites/Dice.md` | Never | Dice account (MCP connector) | Dice MCP (`search_jobs`) + Puppeteer for full descriptions |

### Secondary — Puppeteer Boards

*Mission-driven, niche, and direct company boards. Require Puppeteer MCP server.*

| # | Label | Site File | Last Run | Auth | Method |
|---|-------|-----------|----------|------|--------|
| 1 | ClimateBase (Remote) | `sites/Climatebase.md` | Never | None | Puppeteer JS |
| 2 | ClimateBase (In-Person) | `sites/Climatebase.md` | Never | None | Puppeteer JS |
| 3 | 80kHours | `sites/80kHours.md` | Never | None | Puppeteer (Algolia API) |
| 4 | Draper | `sites/Draper.md` | Never | None | Puppeteer (Workday API) |
| 5 | WorkOnClimate | `sites/WorkOnClimate.md` | N/A | Slack membership | Manual only |
| 6 | YCombinator | `sites/YCombinator.md` | Never | None | Puppeteer JS |
| 7 | BEV-Jobs | `sites/BreakthroughEnergy.md` | Never | None | Puppeteer JS (UI filters) |
| 8 | BEF-Jobs | `sites/BreakthroughEnergy.md` | Never | None | Puppeteer JS (no filter) |
| 9 | Formlabs | `sites/Formlabs.md` | Never | None | Puppeteer JS (native select filter; apply flow via Sensata Workday) |
| 10 | MIT | `sites/MIT.md` | Never | None | Puppeteer JS (AngularJS SPA, PeopleClick ATS) |
| 16 | Harvard | `sites/Harvard.md` | Never | None | Puppeteer API (SmartRecruiters public API — no auth) |
| 17 | Autodesk | `sites/Autodesk.md` | Never | None | Puppeteer JS (Workday CXS POST API) |
| 18 | Markforged | `sites/Markforged.md` | Never | None | Puppeteer JS (Greenhouse public API) |
| 19 | PTC | `sites/PTC.md` | Never | None | Puppeteer JS (Workday CXS POST API) |
| 20 | DesktopMetal | `sites/DesktopMetal.md` | N/A — domain parked | None | ⚠️ ON HOLD — company appears defunct as of 2026-03-17 |

### ⚠️ Caution — Terms of Service Concerns

*Automated scraping of these boards may violate their Terms of Service. Both require Claude in Chrome and an authenticated login session. Use at your own discretion.*

| # | Label | Site File | Last Run | Auth | Method |
|---|-------|-----------|----------|------|--------|
| 11 | LinkedIn | `sites/LinkedIn.md` | Never | LinkedIn login (required) | Claude in Chrome |
| 12 | Wellfound | `sites/Wellfound.md` | Never | Wellfound login (required) | Claude in Chrome |

---

## Session Start Checklist

1. Read this file (index only — do not load all site files)
2. Read `QUICK_REFERENCE_PATH` from `.env` and load that file for candidate profile, scoring calibration, and credentials
3. Ask the user which board to run
4. Load only `sites/[BoardName].md` for the selected board
5. Proceed with the workflow in `FITFOUNDRY-WORKFLOW.md`

---

## Site File Format

Every file in `sites/` follows this standard structure. When creating a new site file, include all sections — write "N/A" or "None" for sections that do not apply.

```markdown
# [Board Name]

**Board #:** [N] (assign next available number and add a row to the Known Boards table above)
**URL:** `[starting URL including any default filter params]`
**Last run:** [YYYY-MM-DD or "Never"]
**Auth required:** [No / Yes — describe what kind]
**Scraping method:** [Puppeteer JS / Puppeteer API / Claude in Chrome / Manual]

---

## Overview

[2–4 sentences: what this board covers, who it targets, signal quality for ops/PM roles,
and any major structural limitation (clearance, CAPTCHA, no date filter, etc.).]

---

## URL & Filter Parameters

[List all supported URL parameters with their values and meaning.
Note which filters work reliably and which are broken or ignored.
Include the recommended starting URL.]

---

## Extraction

[Site type — React SPA / static / server-rendered / API-only.
Card selector(s).
Full field extraction code snippet.
Pagination approach.
Any lazy-loading or virtual DOM behavior that affects extraction.]

---

## Known Issues & Quirks

[Bullet list of gotchas discovered during actual runs:
- Broken filters
- Blocked ATS systems (Dover, etc.)
- Stale postings
- Class name instability
- Login or CAPTCHA behavior
- Overlapping results with other boards
- Any other session-specific finding worth preserving]

---

## Recommended Cadence

[How often to run this board and any notes on what changed since the last run.]
```

---

## Adding a New Board

1. Run the workflow and select "new board" at the site selection step.
2. Navigate to the board and identify:
   - Site type (React SPA / static / server-rendered / API-backed)
   - Authentication requirements
   - Card selector and field extraction pattern
   - Location and workplace filter approach
   - Pagination method
   - Any blocked ATS systems
3. Create a new file in `sites/` using the standard format above.
4. Add a row to the Known Boards table in this file.
5. Record lessons learned in the Improvement Log in `FITFOUNDRY-WORKFLOW.md`.
6. Update `README.md` with the new board in the Supported Job Boards table.

### ATS Identification Tip

If you don't know which ATS a company uses for the ghost job check, navigate to their careers page and check where the apply/listing links point. The subdomain almost always identifies the platform:

| URL pattern | ATS |
|-------------|-----|
| `*.wd5.myworkdayjobs.com` | Workday |
| `jobs.lever.co/*` | Lever |
| `boards.greenhouse.io/*` | Greenhouse |
| `jobs.ashbyhq.com/*` | Ashby |
| `app.dover.com/*` | Dover (⚠️ blocked by egress proxy) |
| `*.icims.com` | iCIMS |
| `*.smartrecruiters.com` | SmartRecruiters |
| `*.bamboohr.com/jobs` | BambooHR |

For Workday boards: use the undocumented CXS POST API — see `sites/Draper.md` for the pattern. Substitute the company's Workday tenant ID in the URL.

For Lever boards: navigate to `jobs.lever.co/[company-slug]` and search the page for the job title.

For Greenhouse boards: query `https://boards-api.greenhouse.io/v1/boards/[company-slug]/jobs?content=false` and filter by title.

**Egress proxy note:** Most career sites are blocked by the VM's network egress proxy. Use Puppeteer (not WebFetch, curl, or Python requests) for all career site access — Puppeteer bypasses the proxy. If Puppeteer is also blocked (e.g., Cloudflare-protected pages), use Claude in Chrome as a fallback if the extension is connected.

---

### General Browser Scraping Notes

**Puppeteer variable scope:** `puppeteer_evaluate` calls share a JavaScript context within a session. Use anonymous expressions or unique variable names per call to avoid `Identifier already declared` errors.

**LinkedIn short URLs:** Some boards surface listings that link to `lnkd.in/` shortened URLs, which redirect through a LinkedIn warning page. Navigate to the final destination URL directly rather than following through LinkedIn.

**Dover ATS:** Dover-hosted job postings (`app.dover.com`) are blocked by Cloudflare bot protection and the network egress proxy. Cannot be fetched via Puppeteer or WebFetch. Mark all Dover postings as ❓ Unverified.

**ClimateBase full descriptions:** ClimateBase individual job detail pages (`climatebase.org/job/[id]`) are Cloudflare-protected and cannot be fetched via Puppeteer. After the initial card scrape and scoring pass, use **Claude in Chrome** to open each shortlisted job's detail page and retrieve the full description before writing individual job files.
