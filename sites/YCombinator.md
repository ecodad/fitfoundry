# Y Combinator — Work at a Startup

**Board #:** 10
**URL:** `https://www.workatastartup.com/jobs?industry=Climatetech&industry=ArtificialIntelligence&jobType=fulltime&role=operations&role=product-manager`
**Last run:** 2026-02-22
**Auth required:** No
**Scraping method:** Puppeteer JS — server-rendered Next.js, direct page extraction

---

## Overview

Job board for Y Combinator-backed startups. All companies are YC-vetted and funded. Skews heavily toward SF and AI/fintech; limited climate and Boston roles. Useful as an AI startup discovery tool rather than a climate board — the `industry=Climatetech` filter is broken and does not surface true climate companies.

---

## URL & Filter Parameters

```
https://www.workatastartup.com/jobs
  ?industry=Climatetech
  &industry=ArtificialIntelligence
  &jobType=fulltime
  &role=operations
  &role=product-manager
```

**Known valid `industry` values:** `Climatetech`, `ArtificialIntelligence`, `Robotics`, `Fintech`, `Healthcare`

**Known valid `role` values:** `operations`, `product-manager`, `software-engineer`, `science`, `recruiting`

**Known valid `jobType` values:** `fulltime`, `parttime`, `internship`

**⚠️ `location` parameter has no effect** — does not filter results. Post-filter by location text in cards.

**⚠️ `industry` filter is broken** — `industry=Climatetech` returns identical results with or without the filter. Do not rely on this for climate-specific results.

---

## Extraction

**Site type:** Server-rendered Next.js. No bot detection, no login required. URL params apply server-side.

**Card selector:** `.jobs-list > div` (each `div` is one company+job block)

**Card field extraction:**
```javascript
const items = Array.from(document.querySelectorAll('.jobs-list > div'));
const jobs = items.map(item => ({
  company:  item.querySelector('.font-bold')?.textContent?.trim(),
  batch:    item.querySelector('.text-gray-600')?.textContent?.trim(),
  jobName:  item.querySelector('[class*="job-name"]')?.textContent?.trim(),
  jobType:  item.querySelector('p.job-details span:first-child')?.textContent?.trim(),
  location: item.querySelector('p.job-details span:last-child')?.textContent?.trim(),
  jobHref:  item.querySelector('a[href*="/jobs/"]')?.href,
  jobId:    item.querySelector('a[href*="/jobs/"]')?.getAttribute('data-jobid')
}));
```

**Location post-filtering:** The `location` field is free-text (e.g., "San Francisco", "Remote", "Boston, MA"). Filter for `Remote` or target city in post-processing.

**Individual job pages:** `https://www.ycombinator.com/companies/{slug}/jobs/{jobid}-{title-slug}`. Full JD available without login.

**Total results:** Typically 20–40 for Operations + PM filters combined. No pagination needed.

---

## Board Characteristics

- All companies are YC-backed (vetted, funded). Batch tag (W25, S24, etc.) indicates recency.
- Many listings are "Founding" roles — small team, broad scope, high equity.
- Skews heavily toward SF in-person. Few remote or Boston roles.
- Low climate signal for Ops/PM roles — use Climatebase, The Engine, or BEV/BEF for climate.

---

## Known Issues & Quirks

- **`industry` filter is broken.** Climatetech and ArtificialIntelligence filters return identical results to no filter. Zero real climate companies appeared in the 2026-02-22 run for Operations/PM roles.
- **`location` filter has no effect.** Always post-filter by location text.

---

## Recommended Cadence

Monthly. Useful for AI-adjacent startup exposure; low value for climate roles.
