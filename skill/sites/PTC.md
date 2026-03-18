# PTC

**Board #:** 19
**URL:** `https://ptc.wd1.myworkdayjobs.com/PTC`
**Last run:** Never
**Auth required:** No
**Scraping method:** Puppeteer JS — Workday undocumented CXS POST API

---

## Overview

PTC is a Boston-area enterprise software company (CAD/PLM/IoT/AR) headquartered in Boston, MA. Known products include Creo, Windchill, Vuforia, and ThingWorx. Strong local presence makes this a high-priority board. Career board typically carries 100–300 active postings globally. Relevant teams for Jonathan: Product R&D, Strategy, Customer Success, Technical Operations/Program Management.

The ptc.com/en/careers page does not link directly to the ATS — navigate directly to the Workday board.

---

## URL & Filter Parameters

All jobs fetched via Workday CXS API — no URL filters needed. Filter in post-processing.

**Workday tenant:** `ptc`
**Board slug:** `PTC`
**CXS base URL:** `https://ptc.wd1.myworkdayjobs.com/wday/cxs/ptc/PTC/jobs`

---

## Extraction

**Site type:** Workday standard board. No bot detection observed.

**Workday CXS POST API:**
```javascript
async function fetchPTCJobs(keyword) {
  const url = 'https://ptc.wd1.myworkdayjobs.com/wday/cxs/ptc/PTC/jobs';
  const headers = { 'Content-Type': 'application/json', 'Accept': 'application/json' };
  const limit = 20;
  const first = await fetch(url, {
    method: 'POST', headers,
    body: JSON.stringify({ limit, offset: 0, searchText: keyword, appliedFacets: {}, returnFacets: true })
  }).then(r => r.json());
  const total = first.total;
  let all = first.jobPostings || [];
  for (let offset = limit; offset < total; offset += limit) {
    const d = await fetch(url, {
      method: 'POST', headers,
      body: JSON.stringify({ limit, offset, searchText: keyword, appliedFacets: {}, returnFacets: true })
    }).then(r => r.json());
    all = all.concat(d.jobPostings || []);
  }
  return { keyword, total, fetched: all.length, jobs: all };
}
window._ptcDone = false;
fetchPTCJobs('program manager').then(r => { window._ptcResult = r; window._ptcDone = true; });
```

**Response fields per posting:**
- `title` — job title
- `externalPath` — relative path, e.g. `/job/Boston-MA/Technical-Program-Manager_R003456`
- `locationsText` — e.g. "Boston, MA" or "Remote, United States"
- `postedOn` — relative string, e.g. "Posted 3 Days Ago" or "Posted 30+ Days Ago"
- `bulletFields[0]` — req ID

**Full job URL pattern:** `https://ptc.wd1.myworkdayjobs.com/en-US/PTC{externalPath}`

**Date filtering:** Use `postedOn`. Filter for ≤ 21 days. Discard "Posted 30+ Days Ago".

---

## Post-Processing Filters

1. **Location:** Keep listings where `locationsText` contains "Boston", "Massachusetts", "MA", "Remote" (US), or "Hybrid"
2. **Title pre-screen:** Match against Tier 1–3 keywords in QUICK-REFERENCE.md; also check for "program manager", "technical operations", "innovation"
3. **Date:** Discard "Posted 30+ Days Ago"

---

## Known Issues & Quirks

- **ptc.com careers pages have no ATS link:** The marketing careers pages (ptc.com/en/careers and sub-pages) contain no links to the Workday board. Navigate to `ptc.wd1.myworkdayjobs.com/PTC` directly.
- **"Posted 30+ Days Ago":** Workday caps relative dates at this string — treat as stale for recency-filtered searches.
- **Global postings:** PTC posts heavily in India, Israel, and Europe. Boston/Remote filter is essential.
- **PTC product team roles:** "Creo", "Windchill", "Vuforia", "ThingWorx" product teams may use different title conventions — also search for "product manager" and "solutions architect" if volume is low.

---

## Recommended Cadence

Bi-weekly. PTC headquarters in Boston makes this a high-value local board. Good signal for senior PM and technical operations roles.
