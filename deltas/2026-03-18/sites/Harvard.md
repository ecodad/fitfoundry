# Harvard University (Careers@Harvard)

**Board #:** 16
**URL:** `https://careers.harvard.edu/jobs`
**API:** `https://api.smartrecruiters.com/v1/companies/HarvardUniversity/postings?limit=100&offset=0&sort=newest`
**Last run:** Never
**Auth required:** No (public SmartRecruiters API; external candidate portal is public)
**Scraping method:** Puppeteer API (SmartRecruiters public API — no authentication required)

---

## Overview

Harvard's external job board runs on SmartRecruiters at `careers.harvard.edu`. The SmartRecruiters public REST API is accessible without authentication and returns structured JSON including `releasedDate` (ISO timestamp), location, work format, employment type, department, school/brand, and salary grade. Signal quality for technical PM, AI/data, and policy-adjacent roles is moderate — Harvard posts across all functions and schools in a single feed, so most volume is healthcare, admin, finance, and lab research. Technical and AI-focused roles are worth filtering for, especially at HBS, SEAS, and HKS.

---

## URL & Filter Parameters

### UI (unreliable — use API instead)
The `careers.harvard.edu/jobs` UI returned 0 results in initial testing (likely a filter state or rendering issue). Use the SmartRecruiters API directly.

### API (recommended)
```
GET https://api.smartrecruiters.com/v1/companies/HarvardUniversity/postings?limit=100&offset=0&sort=newest
```

Key response fields per posting:
- `id` — SmartRecruiters job ID
- `name` — job title
- `releasedDate` — ISO timestamp; use for date filtering
- `location.fullLocation`, `location.hybrid`, `location.remote`
- `typeOfEmployment.label` — Full-time / Part-time / Contract
- `function.label` — functional category
- `department.label` — specific department
- `customField` array — includes Work Format (Hybrid / Fully On-Site / Fully Remote), Salary Grade, FLSA Status, Term Appointment (Yes/No), and Brands (school/unit)

### Pagination
Total postings typically 200–250. With `limit=100`, two pages are needed:
- Page 1: `?limit=100&offset=0&sort=newest`
- Page 2: `?limit=100&offset=100&sort=newest`

### Date filtering
Filter in JavaScript: `new Date(job.releasedDate) >= cutoff`
No native date filter parameter in the API; filter client-side.

### Full posting detail
```
GET https://api.smartrecruiters.com/v1/companies/HarvardUniversity/postings/{id}
```
Returns `jobAd.sections` array with full description HTML. Strip tags with `.replace(/<[^>]*>/g,' ')`.

Apply link format: `https://www.smartrecruiters.com/HarvardUniversity/{id}`

---

## Extraction

**Site type:** SmartRecruiters (cloud ATS) — JSON API, no DOM scraping needed.

**Workflow:**
1. Navigate Puppeteer to the API URL (no auth headers required).
2. `document.querySelector('pre').innerText` returns raw JSON.
3. `JSON.parse()` → filter by `releasedDate` → extract fields.
4. For descriptions, navigate to each `postings/{id}` endpoint.

**School/brand mapping** (from `customField` where `fieldLabel === 'Brands'`):
- Harvard Business School
- Harvard Kennedy School
- Harvard John A. Paulson School of Engineering and Applied Sciences (SEAS)
- Harvard Faculty of Arts and Sciences
- Harvard University Central Administration
- Harvard Graduate School of Education
- Harvard T.H. Chan School of Public Health

---

## Known Issues & Quirks

- **UI broken on initial load:** `careers.harvard.edu/jobs` showed 0 results without any filters set. Bypass entirely with the API.
- **SmartRecruiters IS the ATS:** All active postings on the API are live. Ghost job check is automatic — mark all as ✅ Confirmed.
- **Part-time and term roles:** The feed includes many part-time healthcare roles (RN, MD) and short-term appointments. Pre-screen these out by `typeOfEmployment` and by checking `Term Appointment` in the `customField` array.
- **Pagination required:** ~233 total postings as of Mar 2026; `limit=100` captures the 100 most recent. Run two pages to get full last-month coverage.
- **No salary range in API:** Harvard does not publish pay ranges in the SmartRecruiters API feed. Salary grade is included as a numeric code (e.g., "056", "059") but does not map to dollar amounts in the API. Check the individual posting description for compensation language.

---

## Recommended Cadence

Run every 2 weeks. Harvard posts continuously across all schools. Most relevant roles (AI, policy, engineering, research management) appear at HBS, SEAS, and HKS. Filter by school/unit using the Brands custom field to reduce noise.
