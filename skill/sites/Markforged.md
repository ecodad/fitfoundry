# Markforged

**Board #:** 18
**URL:** `https://job-boards.greenhouse.io/markforged`
**Last run:** Never
**Auth required:** No
**Scraping method:** Puppeteer JS — Greenhouse public API

---

## Overview

Markforged (NYSE: MKFG) is a Waltham, MA-based 3D printing hardware/software company. Relatively small board (typically 15–25 openings). Relevant teams for Jonathan's profile: Engineering, Product, Operations. Primary US hiring location is Waltham, MA; some Remote US roles. Also posts roles in Italy, Poland, and APAC — filter those out.

**Note:** The main markforged.com domain has been intermittently unreachable via Puppeteer (net::ERR_FAILED). Use the Greenhouse API directly instead of navigating to the company website.

---

## URL & Filter Parameters

All jobs fetched via Greenhouse public API — no auth required.

**API endpoint:** `https://boards-api.greenhouse.io/v1/boards/markforged/jobs`
**With full content:** append `?content=true`
**Single job detail:** `https://boards-api.greenhouse.io/v1/boards/markforged/jobs/{job_id}?content=true`

Navigate to the API URL directly in Puppeteer, then parse `document.body.innerText` as JSON.

---

## Extraction

**Step 1 — Fetch all job listings:**
Navigate to: `https://boards-api.greenhouse.io/v1/boards/markforged/jobs?content=false`

```javascript
const data = JSON.parse(document.body.innerText);
const jobs = data.jobs.map(j => ({
  id: j.id,
  title: j.title,
  updated: j.updated_at?.substring(0,10),
  location: j.location?.name,
  applyUrl: j.absolute_url
}));
JSON.stringify({ total: jobs.length, jobs })
```

**Step 2 — Fetch full job description** (for jobs passing pre-screen):
Navigate to: `https://boards-api.greenhouse.io/v1/boards/markforged/jobs/{id}?content=true`

```javascript
const d = JSON.parse(document.body.innerText);
JSON.stringify({ title: d.title, location: d.location?.name, content: d.content, departments: d.departments?.map(dep => dep.name) })
```

**Response fields:**
- `id` — numeric job ID
- `title` — job title
- `updated_at` — ISO timestamp of last update (proxy for recency; no `created_at` in v1 API)
- `location.name` — e.g. "Waltham, Massachusetts, United States" or "Remote, United States"
- `absolute_url` — full Greenhouse apply URL
- `content` (with `?content=true`) — full HTML job description

**Date filtering:** Use `updated_at`. Filter for jobs updated within the past 21 days. Note: `updated_at` reflects last modification, not original post date — some older postings may surface if they were recently edited.

---

## Known Issues & Quirks

- **markforged.com unreachable via Puppeteer:** net::ERR_FAILED when navigating directly to markforged.com or markforged.com/about/careers. Always use the Greenhouse API URL instead.
- **No `created_at` in Greenhouse v1 API:** Only `updated_at` is available. Use as a recency proxy — treat jobs updated >30 days ago as likely stale.
- **Greenhouse board URL changed:** The canonical Greenhouse board is now at `job-boards.greenhouse.io/markforged` (new UI), but the v1 API endpoint (`boards-api.greenhouse.io`) still works and is preferred for scraping.
- **Small board:** Only ~15–25 listings at any given time. Full board scan is fast — no need to keyword-filter the API call.
- **International postings:** Filter out Italy, Poland, and APAC listings in post-processing. Keep Waltham MA, Remote US.

---

## Recommended Cadence

Bi-weekly. Small board with slow posting velocity. Worth checking given Waltham HQ proximity and hardware/manufacturing focus.
