# Draper Laboratory

**Board #:** 7
**URL:** `https://draper.wd5.myworkdayjobs.com/Draper_Careers`
**Last run:** 2026-02-22
**Auth required:** No
**Scraping method:** Puppeteer JS — Workday undocumented CXS POST API

---

## Overview

Draper Laboratory is an independent, nonprofit research and development organization based in Cambridge, MA. Focus areas include defense, space, biomedical, and autonomous systems. ~150 open roles as of 2026-02-22, predominantly Cambridge MA.

⚠️ **Most management-tier roles require an active US Government security clearance (Secret or higher).** This is a structural pattern across the entire board — not limited to specific roles. Low-yield board unless the candidate has or is pursuing a clearance.

---

## URL & Filter Parameters

All jobs fetched via Workday CXS API — no URL filters needed. Filter in post-processing.

---

## Extraction

**Site type:** Workday. Standard Workday Careers board. No bot detection.

**Workday CXS POST API** (undocumented but reliable):
```javascript
// Important: returnFacets:true is required — omitting it returns HTTP 400
// Important: limit max is 20 per call (limit:200 returns HTTP 400)
// Important: only page 1 returns a non-zero total — cache it from the first call

async function fetchAllDraper() {
  const url = 'https://draper.wd5.myworkdayjobs.com/wday/cxs/draper/Draper_Careers/jobs';
  const headers = { 'Content-Type': 'application/json', 'Accept': 'application/json' };
  const limit = 20;
  const first = await fetch(url, {
    method: 'POST', headers,
    body: JSON.stringify({ limit, offset: 0, searchText: '', appliedFacets: {}, returnFacets: true })
  }).then(r => r.json());
  const total = first.total;
  let all = first.jobPostings || [];
  for (let offset = limit; offset < total; offset += limit) {
    const d = await fetch(url, {
      method: 'POST', headers,
      body: JSON.stringify({ limit, offset, searchText: '', appliedFacets: {}, returnFacets: true })
    }).then(r => r.json());
    all = all.concat(d.jobPostings || []);
  }
  return { total, fetched: all.length, jobs: all };
}
window._draperDone = false;
fetchAllDraper().then(r => { window._draperResult = r; window._draperDone = true; });
```

**Response fields per posting:**
- `title` — job title
- `externalPath` — relative path, e.g. `/job/Cambridge-MA/Hardware-Analog-Electronics-Design-Engineer_JR001146`
- `locationsText` — e.g. "Cambridge, MA" or "2 Locations"
- `postedOn` — relative string, e.g. "Posted 3 Days Ago"
- `bulletFields[0]` — req ID, e.g. "JR001146"

**Full job URL pattern:** `https://draper.wd5.myworkdayjobs.com/en-US/Draper_Careers{externalPath}`

---

## Job Categories (2026-02-22)

Engineering (110), Miscellaneous (20), Security & IA (7), Finance (5), Facilities (2), R&D & Technicians (2), Property & Supply Chain (1), HR (1), Operations (1), Contracts (1).

No PM-titled roles were present in the 2026-02-22 run.

---

## Known Issues & Quirks

- **Clearance barrier.** All Group Leader, Supervisor, Manager, and Director roles require Active US Government Security Clearance (Secret or higher). Skip all management roles unless the candidate has or is actively pursuing a clearance.
- **Outlier locations.** Some roles are in Odon IN, Clearfield UT, St. Petersburg FL, Huntsville AL, Reston VA, Houston TX. Filter for Cambridge MA in post-processing.
- **Req ID changes.** Job req IDs (e.g., JR001146) change if a posting closes and reopens. Do not use req ID alone for deduplication — use `externalPath`.

---

## Recommended Cadence

Quarterly, or when the candidate's clearance status changes. Low yield for most PM/ops candidates without a clearance.
