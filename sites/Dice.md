# Dice

**Board #:** 15
**URL:** `https://www.dice.com/jobs`
**Last run:** Never
**Auth required:** Yes — Dice account connected via MCP connector
**Scraping method:** Dice MCP (`search_jobs`) + Puppeteer for full job descriptions

---

## Overview

Tech-focused job board. Higher signal density for TPM and technical ops roles than general boards — the candidate pool and job mix skew engineering and technical. Accessed via the official Dice MCP connector for job discovery, with Puppeteer used to retrieve full descriptions for candidates identified in Step 1. MCP access sidesteps the ToS exposure that would apply to direct scraping (see `ToS/Dice.md`).

**Important difference from Indeed:** Dice MCP only provides `search_jobs` — there is no `get_job_details` equivalent. Full descriptions require a second pass via Puppeteer on individual job URLs returned by the search.

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
| `employerType` | ✅ | `Direct Hire` vs `Recruiter` — useful for filtering aggregator noise |
| `workplaceTypes` | ✅ | Array: `On-Site`, `Remote`, `Hybrid` |
| `isRemote` | ✅ | Boolean — note: **inconsistent with `workplaceTypes`** in practice |
| `workFromHomeAvailability` | ✅ | `TRUE`/`FALSE` — also inconsistent with `workplaceTypes` |
| `easyApply` | ✅ | Boolean |
| `willingToSponsor` | ✅ | Boolean |
| `score` | ✅ | Dice internal relevance score |
| `id` / `guid` | ✅ | Internal Dice IDs — use for deduplication |
| Full job description | ❌ | Not available — requires Puppeteer on `detailsPageUrl` |
| Employer ATS job ID | ❌ | Not exposed |

**Filtering tip:** `employerType: "Direct Hire"` removes staffing firm reposts. Many postings labeled `Recruiter` are third-party reposts of direct employer listings (e.g., the same Amazon Ring role appears multiple times via different aggregators).

---

## URL & Filter Parameters

**MCP tool — `search_jobs`:**

| Parameter | Value | Notes |
|-----------|-------|-------|
| `query` | `technical program manager` | Run separate searches per keyword |
| `location` | `Boston, MA` | City + state |
| `employment_type` | `FULLTIME` | All-caps per Dice convention — confirm on first run |

**Recommended query sequence:**
1. `technical program manager`
2. `operations program manager`
3. `senior program manager`

**Dice job URL structure** (for Puppeteer detail fetch):
```
https://www.dice.com/job-detail/[job-id]
```
Job IDs are returned in the MCP search results. Navigate to this URL via Puppeteer to retrieve the full description.

**Dice job page — description selector:**
```javascript
document.querySelector('[data-cy="jobDescription"], .job-description, [class*="description"]')?.innerText
```
Note: Dice uses a React app — confirm selectors on first run and update this file if they have changed.

---

## Extraction

**Step 1 — MCP search (run all queries, deduplicate):**
```
search_jobs(query="technical program manager", location="Boston, MA", employment_type="FULLTIME")
search_jobs(query="operations program manager", location="Boston, MA", employment_type="FULLTIME")
search_jobs(query="senior program manager", location="Boston, MA", employment_type="FULLTIME")
```
Returns: titles, companies, locations, salaries (where listed), job URLs, job IDs.
Deduplicate by job ID before proceeding.

**Step 2 — Puppeteer detail fetch for pre-screened candidates:**
```javascript
// Navigate to each job URL returned by the MCP search
// Example:
await page.goto('https://www.dice.com/job-detail/[job-id]');
const description = document.querySelector('[data-cy="jobDescription"]')?.innerText;
```
Only fetch details for jobs that pass the title pre-screen. Do not fetch all — Dice has high volume.

---

## Known Issues & Quirks

- **No `get_job_details` in MCP:** Unlike Indeed, Dice MCP only offers `search_jobs`. Full descriptions require Puppeteer navigation to individual job URLs.
- **Session start required:** Dice MCP connector must be active at Cowork session start. Connecting mid-session does not inject the tools.
- **Tech-skewed results:** Dice surfaces more IC engineering roles than management-level ops/PM. Title pre-screening is especially important — filter aggressively before fetching descriptions.
- **Selector stability:** Dice uses a React app. The `[data-cy="jobDescription"]` selector is based on observed patterns; confirm on first run and update if needed.
- **Duplicate postings:** Dice aggregates from company ATS postings and staffing firms. Staffing firm postings (where company name is a recruiter, not the employer) are lower value — note them in the master list.
- **ToS note:** MCP connector = official access, no ToS exposure. Direct Puppeteer scraping of Dice is High risk — see `ToS/Dice.md`.

---

## Recommended Cadence

Weekly, paired with Indeed and LinkedIn. Dice's tech focus gives it different coverage than Indeed — worth running both.
