# Breakthrough Energy — BEV Jobs & BEF Jobs

**Board #:** 11 (BEV) & 12 (BEF)
**URLs:**
- BEV: `https://bevjobs.breakthroughenergy.org/jobs`
- BEF: `https://befjobs.breakthroughenergy.org/jobs`
**Last run:** 2026-02-23
**Auth required:** No
**Scraping method:** Puppeteer JS — Getro Next.js SPA; UI filter clicks required

---

## Overview

Two Getro-powered job boards for Breakthrough Energy's portfolio companies.

| Board | Description | Total Jobs | Companies |
|-------|-------------|------------|-----------|
| BEV Jobs (collection 1533) | Breakthrough Energy Ventures VC portfolio — CFS, Electric Hydrogen, Fervo, Antora, etc. | ~767 | ~92 |
| BEF Jobs (collection 2567) | Breakthrough Energy Fellows — early-stage deep-tech startups by BEF alumni | ~77 | ~30 |

Both boards are heavy on deep-tech hardware and clean energy. Operations and PM roles are a minority but high quality. BEF is small enough to review all titles without filtering.

⚠️ **URL query parameters cause HTTP 500 errors** on these boards. Filters must be applied via UI clicks — not query strings. This distinguishes them from The Engine (also Getro), where URL params work.

---

## Extraction

**Site type:** Getro Next.js SPA — same platform as The Engine but different authentication model. API at `api.getro.com/api/v2/collections/{id}/jobs` requires server-side auth not available client-side.

**SSR embedded data** (first 20 jobs, unfiltered — useful to confirm total count):
```javascript
const nd = JSON.parse(document.getElementById('__NEXT_DATA__').textContent);
const jobs = nd.props.pageProps.initialState.jobs;
// jobs.total = total job count; jobs.found = array of first 20 jobs
// Fields: id, title, organization.name, locations[], workMode, url, seniority, createdAt, slug
```

**Card selector (after UI filter is applied):** `[data-testid="job-list-item"]`

**Card field extraction:**
```javascript
const cards = Array.from(document.querySelectorAll('[data-testid="job-list-item"]'));
const jobs = cards.map(c => ({
  title: c.querySelector('[data-testid="job-title-link"]')?.textContent?.trim(),
  url:   c.querySelector('[data-testid="job-title-link"]')?.href,
  text:  c.innerText  // parse company, location, seniority from text
}));
```

---

## Applying UI Filters (BEV — required)

URL params cause HTTP 500. Use UI interaction:

```javascript
// 1. Click the "Job function" filter button
document.querySelector('[data-testid="filter-option-item-0"]').click();
// 2. Wait for dropdown, then find and click "Operations" checkbox
// 3. Also click "Product" checkbox
// 4. Wait for results to reload, then extract cards
```

**Available job function filter values (BEV):**
Operations, Software Engineering, IT, Other Engineering, Sales & Business Development, Design, Accounting & Finance, People & HR, Quality Assurance, Product, Marketing & Communications, Customer Service, Data Science, Legal, Administration

**BEF approach:** Omit filters entirely — 77 total jobs is small enough to review all titles in one pass.

---

## Known Portfolio Companies

**BEV:** CFS (Commonwealth Fusion Systems), Form Energy, Electric Hydrogen, Fervo Energy, Antora Energy, Stoke Space, Quantum SI, and ~85 others.

**BEF:** Smaller, earlier-stage companies by Breakthrough Energy Fellows alumni. Very high climate-tech signal.

---

## Known Issues & Quirks

- **URL params → HTTP 500.** Do not use query strings for filtering on these boards. UI clicks only.
- **Pagination via scroll.** After applying filters, scroll down to load additional job cards — the board lazy-loads.
- **Form Energy ATS (as of 2026-02-22):** `jobs.ashbyhq.com/formenergy` returns 404. Form Energy links from BEV are stale until updated.

---

## Recommended Cadence

Monthly. Run BEV with Operations + Product UI filters; run BEF unfiltered.
