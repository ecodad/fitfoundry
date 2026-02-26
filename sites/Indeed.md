# Indeed

**Board #:** 14
**URL:** `https://www.indeed.com`
**Last run:** Never
**Auth required:** Yes — Indeed account connected via MCP connector
**Scraping method:** Indeed MCP — `search_jobs` and `get_job_details` tools (official API access)

---

## Overview

Indeed via the official MCP connector. Unlike the Puppeteer-based boards, this uses Indeed's sanctioned API surface — no ToS exposure for personal use. Access requires the Indeed connector to be active at Cowork session start; it cannot be added mid-session. Broad job board with high volume; pre-screening by title before fetching details is essential. Run multiple targeted keyword searches and deduplicate before pulling any descriptions.

---

## Per-Job Fields (MCP tool responses)

### `search_jobs` — returned per job

| Field | Present | Notes |
|-------|---------|-------|
| `Job Title` | ✅ | Job title |
| `Job Id` | ✅ | Indeed internal ID — used as key for `get_job_details` and deduplication |
| `Company` | ✅ | Company name |
| `Location` | ✅ | City, state |
| `Posted on` | ✅ | Human-readable date (e.g. `February 24, 2026`) |
| `Job Type` | ✅ | `Fulltime`, `Parttime`, etc. |
| `Compensation` | ✅ | Salary range where listed — more reliable than Dice |
| `View Job URL` | ✅ | indeed.com redirect URL |
| Full job description | ❌ | Not in search results — requires `get_job_details` |

### `get_job_details(job_id)` — additional fields

| Field | Present | Notes |
|-------|---------|-------|
| Full description | ✅ | Complete job description text |
| Requirements / qualifications | ✅ | Included in description body |
| Benefits | ✅ | Where listed by employer |
| Application questions | ✅ | Screening questions shown where present |
| Work location type | ✅ | Hybrid / remote / on-site noted in description |

### `get_company_data(company)` — company-level fields

| Field | Present | Notes |
|-------|---------|-------|
| Overall rating | ✅ | 1–5 scale |
| Recommend to friend | ✅ | Yes/No counts |
| CEO approval | ✅ | Approval percentage |
| Salary satisfaction | ✅ | Yes/No counts |
| Culture / work-life balance ratings | ✅ | Sub-ratings |
| Interview difficulty / experience | ✅ | Aggregate data |
| Company description / CEO name | ✅ | Employer-provided metadata |
| Salary for specific job title | ✅ | Pass `jobTitle` param — based on employee reports |

---

## URL & Filter Parameters

No URL manipulation required. All parameters passed directly to MCP tools.

**`search_jobs` parameters:**

| Parameter | Value | Notes |
|-----------|-------|-------|
| `query` | *(from QUICK_REFERENCE_PATH → Indeed → Keywords)* | Run as separate searches per keyword |
| `location` | *(from QUICK_REFERENCE_PATH → Indeed → Location, or Default search location)* | Use city + state |
| `employment_type` | `fulltime` | Excludes contract, part-time, intern |

Run each keyword as a separate search call, collect all results, then deduplicate by job ID before proceeding.

---

## Extraction

**Step 1 — Search (run all keywords, deduplicate by job ID):**
```
search_jobs(query="[keyword]", location="[location]", employment_type="fulltime")
```
Returns: job titles, companies, locations, salaries where listed, application URLs, job IDs.

**Step 2 — Detail retrieval for pre-screened candidates:**
```
get_job_details(job_id="[ID from search results]")
```
Returns: full job description, requirements, qualifications, benefits, company information.
Only call `get_job_details` for jobs that passed the title pre-screen. Do not call it for all results.

**Step 3 — Optional resume context (once per session):**
```
get_resume()
```
Retrieves experience from the authenticated Indeed profile. Run once at session start to add supplementary context for scoring. Also serves as the connector validation check.

**Step 4 — Optional company research (for top scores):**
```
get_company_data(company="[Company Name]")
```
Returns Indeed data on employee satisfaction, compensation, culture, management, and reviews. Call for any job scoring 7 or higher, before creating its individual file.

---

## MCP Workflow Notes

These notes describe how the Indeed connector flow differs from the standard Puppeteer flow. The FITFOUNDRY-WORKFLOW.md MCP branch references this section.

**Connector validation:** Call `get_resume()` at session start. If it fails, the connector is not active — stop and restart the session with the connector connected. If it succeeds, hold any returned experience as supplementary scoring context.

**Search and deduplication:** Run all keywords from QUICK_REFERENCE_PATH (Indeed → Keywords) sequentially. Collect results. Deduplicate by Job Id before proceeding to title pre-screen. A single job may match multiple keywords; keep one entry and note which queries matched it.

**Title pre-screen before detail fetch:** Unlike Puppeteer boards where you visit each posting inline, screen the deduplicated title list first using the target roles and "Not a fit" constraints from your profile. Only call `get_job_details` for titles that survive the pre-screen. This avoids making unnecessary API calls on clearly irrelevant postings.

**Company research:** For any job scoring 7 or higher, call `get_company_data(company)` before creating the individual file. Include the result as a **Company Notes** section in the file (2–3 sentences on culture, satisfaction, and compensation). Omit this section for scores below 7.

**Individual file — additional fields for Indeed runs:**
```
**Source:** Indeed (MCP)
**Type:** Full-time
**Found:** [YYYY-MM-DD]
```
And for scores ≥7, add after Potential Gaps:
```
## Company Notes
[2–3 sentences from get_company_data on culture, satisfaction, and compensation.]
```

**Master list column format:**
```
| # | Title | Company | Location | Salary | Score | Ghost Check | File |
```

---

## Known Issues & Quirks

- **Session start required:** The Indeed MCP connector must be active when the Cowork session starts. Connecting mid-session does not inject the tools — start a new session after connecting.
- **Volume:** Indeed returns high volumes; aggressive pre-screening by title is essential before pulling full descriptions with `get_job_details`.
- **Deduplication:** Run all keyword searches first, collect all job IDs, deduplicate, then pull details in one pass.
- **ToS note:** This board uses Indeed's official MCP connector — no ToS exposure for personal job search use. See `ToS/Indeed.md` for context on why the Puppeteer approach was not pursued.

---

## Recommended Cadence

Weekly. High volume board; MCP access makes it fast and clean. Pair with LinkedIn weekly run for broad coverage.
