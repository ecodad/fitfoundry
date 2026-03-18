# MIT Human Resources

**Board #:** 16
**URL:** `https://careers.peopleclick.com/careerscp/client_mit/external/search/search.html`
**Entry point:** `https://hr.mit.edu/jobs/search` → redirects to external candidate portal above
**Last run:** Never
**Auth required:** No (external candidate search is public)
**Scraping method:** Puppeteer JS

---

## Overview

MIT's external job board uses PeopleClick (iCIMS legacy platform) at `careers.peopleclick.com`. The portal lists all open positions across all MIT departments and schools. Signal quality for technical PM, R&D, and engineering roles is good when MIT is in a normal hiring cycle — during the current hiring pause (announced early 2026), postings skew heavily toward research/lab, healthcare, finance, and administration.

**Important context (as of Mar 2026):** MIT has paused hiring for most non-essential positions. Currently posted roles are ones MIT is actively seeking to fill. Re-run after the pause lifts for better coverage of technical and program management roles.

---

## URL & Filter Parameters

The search form at `/search/search.html` supports these filters (all via `<select>` dropdowns):

| Field | Select ID | Key values |
|-------|-----------|------------|
| Functional Area | `com.peopleclick.cp.formdata.FLD_JP_POSITION_CATEGORY` | All (-1), Information Technology (74), Research-Engineering (79), Administration (58), Business Development (90) |
| Department | `com.peopleclick.cp.formdata.FLD_JP_DEPARTMENT` | Hundreds of departments — use All unless targeting specific labs |
| Division | `com.peopleclick.cp.formdata.FLD_JPM_DIVISION` | Engineering (15), Schwarzman College of Computing (36), VP Research (31), Office of Provost (22) |
| Duration | `com.peopleclick.cp.formdata.JPM_DURATION` | All (-1), Full-Time (2), Full-time Hybrid (9), Full-time Remote Only (10) |
| Date posted | `com.peopleclick.cp.formdata.SEARCHCRITERIA_JOBPOSTAGE` | All (-1), Last 24h (HOUR_24), Last 2 days (DAY_2), Last week (WEEK_1), Last month (MONTH_1) |
| Results per page | `com.peopleclick.cp.formdata.hitsPerPage` | 10, 20, 30, 50 |

**Recommended starting URL:** Navigate to `/search/search.html`, set Date to `MONTH_1`, hits per page to `50`, then click `#sp-searchButton`.

Results page URL: `https://careers.peopleclick.com/careerscp/client_mit/external/results/searchResult.html`

---

## Extraction

**Site type:** AngularJS SPA (results rendered client-side after form submit).

**Job card links:** `a[href*="jobDetail"]` — each links to the full posting detail page.

**Pagination:** AngularJS `ng-click="selectPage(page.number)"` — URLs do not change between pages. Navigate pages by clicking the page number link in JS:
```javascript
Array.from(document.querySelectorAll('a')).find(a => a.innerText.trim() === '2').click()
```
Wait ~1 second for content to re-render before extracting.

**Per-job fields from detail page** (`/jobDetails/jobDetail.html?jobPostId=XXXXX`):
- Title: page `<title>` or first `<h1>`
- Job Number, Functional Area, Department, School Area, Pay Range Min/Max, Employment Type, Pay Grade: structured text block after "Back to Search Results"
- Description and requirements: free text below "Posting Description" heading
- Posted date: appears at end of requirements block (format: M/D/YYYY)

**Pay range:** MIT publishes explicit min/max ranges on every posting. Use these for salary pre-screening before reading full descriptions — roles where the posted max falls below the candidate's salary floor (from their profile) can be eliminated without reading the full description.

---

## Known Issues & Quirks

- **Hiring pause (Feb–Mar 2026):** MIT announced a pause on most non-essential positions. Active postings are genuinely open but the mix skews toward research/lab, healthcare, and administration. Check `hr.mit.edu/jobs/search` for any updated announcements before running.
- **AngularJS pagination:** Page number links use `ng-click` (no href target). Must trigger via `.click()` in Puppeteer JS — direct URL navigation to page 2 does not work.
- **Variable scope:** Puppeteer evaluate calls share JS context. Use IIFE (`(function() { ... })()`) to avoid `Identifier already declared` errors across successive calls.
- **No "last 2 weeks" filter:** The closest option is "last month" (MONTH_1). Filter results manually by posted date if tighter date range is needed.
- **Ghost job check not needed:** MIT's PeopleClick portal IS the ATS. Any active listing here is confirmed live. Mark all scored jobs as ✅ Confirmed automatically.
- **Functional area mislabeling:** Some roles appear under unexpected categories (e.g., Project Manager at PSFC listed under Finance/Accounting). Do not rely on functional area for pre-screening — read titles directly.

---

## Recommended Cadence

Run monthly. Given the current hiring pause, re-run after an announcement that normal hiring has resumed — this will likely surface a stronger batch of technical PM, engineering, and solutions roles. Check `hr.mit.edu/jobs/search` for hiring pause status before each run.
