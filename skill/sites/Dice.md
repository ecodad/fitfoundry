# Dice

**Board #:** 15
**URL:** `https://www.dice.com/jobs`
**Last run:** Never
**Auth required:** Yes — Dice account connected via MCP connector
**Scraping method:** Dice MCP (`search_jobs`) + Puppeteer for full job descriptions

---

## Overview

Tech-focused job board. Higher signal density for technical and ops roles than general boards — the candidate pool and job mix skew engineering and technical. Accessed via the official Dice MCP connector for job discovery, with Puppeteer used to retrieve full descriptions for candidates identified in the search. MCP access sidesteps the ToS exposure that would apply to direct scraping (see `ToS/Dice.md`).

**Key difference from Indeed:** Dice MCP only provides `search_jobs` — there is no `get_job_details` equivalent. Full descriptions require a second pass via Puppeteer on individual job URLs returned by the search.

---

## Per-Job Fields (MCP `search_jobs` response)

| Field | Present | Notes |
|-------|---------|-------|
| `title` | ✅ | Job title |
| `summary` | ✅ | Short excerpt only — not the full description |
| `postedDate` | ✅ | ISO timestamp (e.g. `2026-02-12T01:13:43Z`) |
| `modifiedDate` | ✅ | Last modified ISO timestamp |
| `jobLocation.displayName` | ✅ | Location string |
| `companyName` | ✅ | Company name |
| `employmentType` | ✅ | `Full-time`, `Contract`, etc. |
| `salary` | ⚠️ | Field present but **null in practice** — unreliable |
| `detailsPageUrl` | ✅ | Dice redirect URL (routes through appljack or similar aggregator) |
| `companyPageUrl` | ✅ | Dice company profile page |
| `employerType` | ✅ | `Direct Hire` vs `Recruiter` — use for staffing firm flagging |
| `workplaceTypes` | ✅ | Array: `On-Site`, `Remote`, `Hybrid` |
| `isRemote` | ✅ | Boolean — note: **inconsistent with `workplaceTypes`** in practice |
| `workFromHomeAvailability` | ✅ | `TRUE`/`FALSE` — also inconsistent with `workplaceTypes` |
| `easyApply` | ✅ | Boolean |
| `willingToSponsor` | ✅ | Boolean |
| `score` | ✅ | Dice internal relevance score |
| `id` / `guid` | ✅ | Internal Dice IDs — use for deduplication |
| Full job description | ❌ | Not available via MCP — requires Puppeteer on `detailsPageUrl` |
| Employer ATS job ID | ❌ | Not exposed |

---

## URL & Filter Parameters

**MCP tool — `search_jobs`:**

| Parameter | Value | Notes |
|-----------|-------|-------|
| `query` | *(from QUICK_REFERENCE_PATH → Dice → Keywords)* | Run as separate searches per keyword |
| `location` | *(from QUICK_REFERENCE_PATH → Dice → Location, or Default search location)* | City + state |
| `employment_type` | `FULLTIME` | All-caps per Dice convention |

Run each keyword as a separate search call, collect all results, then deduplicate by job ID.

**Dice job URL structure** (for Puppeteer detail fetch):
```
https://www.dice.com/job-detail/[job-id]
```
Job IDs are returned in the MCP search results.

**Dice job page — description selector:**
```javascript
document.querySelector('[data-cy="jobDescription"], .job-description, [class*="description"]')?.innerText
```
Note: Dice is a React app — confirm selectors on first run and update this file if they have changed.

---

## Extraction

**Step 1 — MCP search (run all keywords, deduplicate by job ID):**
```
search_jobs(query="[keyword]", location="[location]", employment_type="FULLTIME")
```
Returns: titles, companies, locations, salaries (where listed), job URLs, job IDs, employerType.
Deduplicate by job ID before proceeding.

**Step 2 — Puppeteer detail fetch for pre-screened candidates:**
```javascript
// Navigate to each job URL returned by the MCP search
await page.goto('https://www.dice.com/job-detail/[job-id]');
const description = document.querySelector('[data-cy="jobDescription"]')?.innerText;
```
Only fetch details for jobs that pass the title pre-screen. Dice has high volume — do not fetch all.

---

## MCP Workflow Notes

These notes describe how the Dice connector flow differs from the standard Puppeteer flow. The FITFOUNDRY-WORKFLOW.md MCP branch references this section.

**Connector validation:** At session start, attempt `search_jobs` with a minimal test query (e.g., one keyword, limit 1). If it fails, the connector is not active — stop and restart the session with the connector connected.

**Search and deduplication:** Run all keywords from QUICK_REFERENCE_PATH (Dice → Keywords) sequentially. Collect results. Deduplicate by `id` or `guid` before proceeding to title pre-screen. A single job may match multiple keywords; keep one entry.

**Staffing firm flagging:** Check the `employerType` field on each result. Where `employerType` is `Recruiter`, the posting is from a staffing firm or aggregator, not the direct employer. Flag these in the master list (Staffing Firm? column) and keep them — they may still be worth reviewing, but treat with lower priority than direct employer postings.

**Title pre-screen before detail fetch:** Screen the deduplicated title list using the target roles and "Not a fit" constraints from your profile before making any Puppeteer calls. Dice has high volume and a higher proportion of IC engineering roles; screen aggressively.

**Full description fetch via Puppeteer:** Unlike Indeed (which has `get_job_details`), Dice MCP does not return full descriptions. After the title pre-screen, navigate to each surviving job's `detailsPageUrl` via Puppeteer to retrieve the full description. Use the selector `[data-cy="jobDescription"]` with fallbacks to `.job-description` and `[class*="description"]`. Dice is a React app — if the page returns empty on first load, wait 1–2 seconds and re-query.

**Individual file — additional fields for Dice runs:**
```
**Source:** Dice (MCP)
**Type:** Full-time
**Found:** [YYYY-MM-DD]
**Staffing firm posting:** [Yes / No / Unknown]
```

**Master list column format:**
```
| # | Title | Company | Location | Salary | Staffing Firm? | Score | Ghost Check | File |
```

---

## Known Issues & Quirks

- **No `get_job_details` in MCP:** Unlike Indeed, Dice MCP only offers `search_jobs`. Full descriptions require Puppeteer navigation to individual job URLs.
- **Session start required:** Dice MCP connector must be active at Cowork session start. Connecting mid-session does not inject the tools.
- **Tech-skewed results:** Dice surfaces more IC engineering roles than management-level ops/PM. Title pre-screening is especially important.
- **Selector stability:** Dice uses a React app. The `[data-cy="jobDescription"]` selector is based on observed patterns; confirm on first run and update if needed.
- **Duplicate postings:** Dice aggregates from company ATS postings and staffing firms. Use `employerType` to distinguish; note staffing firm postings in the master list.
- **ToS note:** MCP connector = official access, no ToS exposure. Direct Puppeteer scraping of Dice is High risk — see `ToS/Dice.md`.

---

## Recommended Cadence

Weekly, paired with Indeed and LinkedIn. Dice's tech focus gives it different coverage than Indeed — worth running both.
