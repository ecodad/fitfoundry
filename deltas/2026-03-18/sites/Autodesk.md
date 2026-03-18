# Autodesk

**Board #:** 17
**URL:** `https://autodesk.wd1.myworkdayjobs.com/Ext`
**Last run:** Never
**Auth required:** No
**Scraping method:** Puppeteer JS — Workday undocumented CXS POST API

---

## Overview

Autodesk is a large software/hardware company (CAD, PLM, manufacturing tools) headquartered in San Francisco with a significant Boston/MA presence. Thousands of global employees; career board typically carries 300–600 active postings. Relevant teams for Jonathan's profile: Program Management, Technical Operations, Product Management, Sustainability, Manufacturing/Make. High signal for TPM, EPM, and innovation PM roles.

**Note on volume:** Autodesk posts globally — filter aggressively by location (Boston, MA / Remote US) in post-processing to keep results manageable.

---

## URL & Filter Parameters

All jobs fetched via Workday CXS API — no URL filters needed. Filter in post-processing.

**Workday tenant:** `autodesk`
**Board slug:** `Ext`
**CXS base URL:** `https://autodesk.wd1.myworkdayjobs.com/wday/cxs/autodesk/Ext/jobs`

To filter by keyword via the API, use `searchText` in the POST body.

---

## Extraction

**Site type:** Workday standard board. No bot detection observed.

**Workday CXS POST API:**
```javascript
// Fetch all jobs matching a keyword — paginate in batches of 20
async function fetchAutodeskJobs(keyword) {
  const url = 'https://autodesk.wd1.myworkdayjobs.com/wday/cxs/autodesk/Ext/jobs';
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
window._autodeskDone = false;
fetchAutodeskJobs('program manager').then(r => { window._autodeskResult = r; window._autodeskDone = true; });
```

**Response fields per posting:**
- `title` — job title
- `externalPath` — relative path, e.g. `/job/Boston-MA/Program-Manager_25WD84123`
- `locationsText` — e.g. "Boston, MA" or "Remote, United States"
- `postedOn` — relative string, e.g. "Posted 5 Days Ago" or "Posted 30+ Days Ago"
- `bulletFields[0]` — req ID

**Full job URL pattern:** `https://autodesk.wd1.myworkdayjobs.com/en-US/Ext{externalPath}`

**Date filtering:** Use `postedOn` field. Filter for "Posted X Days Ago" where X ≤ 21 (3 weeks). Discard "Posted 30+ Days Ago".

---

## Post-Processing Filters

1. **Location:** Keep only listings where `locationsText` contains "Boston", "Massachusetts", "MA", "Remote" (US only), or "Hybrid"
2. **Title pre-screen:** Match against Tier 1–3 keywords in QUICK-REFERENCE.md
3. **Date:** Keep only postings with `postedOn` ≤ 21 days ago (discard "30+ Days Ago")

---

## Known Issues & Quirks

- **High volume:** Autodesk posts globally. Always apply keyword + location filters — scraping all jobs without a keyword filter will return hundreds of irrelevant postings.
- **"Posted 30+ Days Ago":** Workday caps relative dates at "30+ Days Ago" — exact post date is not available. Treat these as stale and skip when running a recency-filtered search.
- **Multiple locations:** Some postings list "Multiple Locations" — navigate to the full posting to confirm if any location is in the Boston area or Remote.
- **Req ID in externalPath:** The Workday req ID (e.g., `25WD84123`) is embedded in the externalPath slug.

---

## Recommended Cadence

Bi-weekly. Large board with frequent new postings. Use Tier 1 search keywords to keep volume manageable. Boston/Remote filter is essential.
