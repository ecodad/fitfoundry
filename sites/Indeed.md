# Indeed

**Board #:** 14
**URL:** `https://www.indeed.com`
**Last run:** Never
**Auth required:** Yes — Indeed account connected via MCP connector
**Scraping method:** Indeed MCP — `search_jobs` and `get_job_details` tools (official API access)

---

## Overview

Indeed via the official MCP connector. Unlike the Puppeteer-based boards, this uses Indeed's sanctioned API surface — no ToS exposure for personal use. Access requires the Indeed connector to be active at Cowork session start; it cannot be added mid-session. Broad job board with high volume; pre-screening by title is essential. Run multiple targeted keyword searches rather than one broad search.

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
| `query` | `technical program manager` | Run as separate searches per keyword |
| `location` | `Boston, MA` | Use city + state; broader than zip |
| `employment_type` | `fulltime` | Excludes contract, part-time, intern |

**Recommended query sequence (run in order, deduplicate by job ID):**
1. `technical program manager`
2. `operations program manager`
3. `senior program manager`

---

## Extraction

**Step 1 — Search:**
```
search_jobs(query="technical program manager", location="Boston, MA", employment_type="fulltime")
```
Returns: job titles, companies, locations, salaries (where listed), application URLs, and job IDs.

**Step 2 — Detail retrieval for candidates:**
```
get_job_details(job_id="[ID from search results]")
```
Returns: full job description, requirements, qualifications, benefits, company information.

**Step 3 — Optional resume personalization:**
```
get_resume()
```
Retrieves experience from the authenticated Indeed profile. Run once per session to add context for scoring.

**Step 4 — Optional company research:**
```
get_company_data(company="[Company Name]")
```
Returns Indeed data on employee satisfaction, compensation, culture, management, and reviews. Run on any company scoring 7+ before creating the individual file.

---

## Known Issues & Quirks

- **Session start required:** The Indeed MCP connector must be active when the Cowork session starts. Connecting mid-session does not inject the tools — start a new session after connecting.
- **Volume:** Indeed returns high volumes; aggressive pre-screening by title is essential before pulling full descriptions with `get_job_details`.
- **Deduplication:** Run all keyword searches first, collect all job IDs, deduplicate, then pull details in one pass.
- **ToS note:** This board uses Indeed's official MCP connector — no ToS exposure for personal job search use. See `ToS/Indeed.md` for context on why the Puppeteer approach was not pursued.

---

## Recommended Cadence

Weekly. High volume board; MCP access makes it fast and clean. Pair with LinkedIn weekly run for broad coverage.
