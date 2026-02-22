# Job Board Site Notes

Reference for the agent running the job board workflow. Read this file at the start of each session to select the right scraping approach for the chosen board.

---

## Known Boards

| # | Label | URL | Last Run | Run Type |
|---|-------|-----|----------|----------|
| 1 | ClimateBase-Remote | See workflow variables | 2026-02-21 | Remote only |
| 2 | ClimateBase-In-Person | See workflow variables | 2026-02-22 | Hybrid + In-person |
| 3 | GreentownLabs | https://greentownlabs.com/careers/?type=all&location=boston#job-type | 2026-02-22 | All (no reliable filter) |
| 4 | LinkedIn | https://www.linkedin.com/jobs/search/?keywords=...&location=Boston%2C+MA&f_TPR=r2592000&f_E=4%2C5&f_JT=F&sortBy=DD | Never | Requires login for full results |
| 5 | TheEngine | https://engine.xyz/careers?job_functions=Operations | 2026-02-22 | All (filter by MA in post-processing) |
| 6 | 80kHours | https://jobs.80000hours.org/?refinementList%5Btags_location_80k%5D%5B0%5D=Boston%20metro%20area&refinementList%5Btags_location_80k%5D%5B1%5D=Remote%2C%20USA | 2026-02-22 | Boston + Remote USA; full-time only |
| 7 | Draper | https://draper.wd5.myworkdayjobs.com/Draper_Careers | 2026-02-22 | Cambridge MA; all categories; clearance barrier — see notes |
| 8 | WorkOnClimate | https://workonclimate.org | Never | Newsletter/Slack only — not scrapable; see notes |
| 9 | Wellfound | https://wellfound.com/jobs?role=operations&role=product-manager&industry=Climatetech&industry=ArtificialIntelligence&jobType=fulltime | Never | Bot-protected — Claude in Chrome with login required |
| 10 | YCombinator | https://www.workatastartup.com/jobs?industry=Climatetech&industry=ArtificialIntelligence&jobType=fulltime&role=operations&role=product-manager | 2026-02-22 | Server-rendered; no login required; industry filter unreliable — see notes |
| 11 | BEV-Jobs | https://bevjobs.breakthroughenergy.org/jobs | Never | Getro (collection 1533); 767 jobs / 92 companies; UI filters required |
| 12 | BEF-Jobs | https://befjobs.breakthroughenergy.org/jobs | Never | Getro (collection 2567); 77 jobs; early-stage Fellows portfolio |

For ClimateBase URLs, reconstruct from the base URL with the appropriate filter parameters (see notes below). The exact filtered URL from a prior run is recorded in the corresponding master list file.

---

## Site-Specific Notes

---

### 1 & 2. Climatebase (`climatebase.org`)

**Site type:** React single-page app with lazy-loaded job cards.

**Standard link selectors do not work.** `a[href*="/jobs/"]` returns nothing. Use the card class approach below.

**Scrollable container:**
The job list renders inside a nested `div`, not `document.body`. Find it by detecting elements with `scrollHeight > clientHeight`. The stable class pattern is `sc-3d1ac256-2` (may change on site rebuild — re-detect if it stops working). Scroll this container, not the page:
```javascript
const container = document.querySelector('.sc-3d1ac256-2.cPkuCi');
container.scrollTo(0, container.scrollHeight);
```
Scroll in increments (e.g., 2000px steps with 800ms waits) to trigger lazy loading. Repeat until `scrollHeight` stabilizes across two consecutive reads.

**Job card class:** `.Card_Information__HkDL9`

**Card field extraction:**
```javascript
const card = /* one .Card_Information__HkDL9 element */;
const title = card.querySelector('h2 span')?.textContent?.trim();
const lis = Array.from(card.querySelectorAll('ul li'));
const company = lis[0]?.textContent?.trim();
const location = lis[1]?.textContent?.trim();
const workplace = lis[2]?.textContent?.trim();  // e.g. "In-person", "Hybrid", "Remote"
const jobType = lis[3]?.textContent?.trim();     // e.g. "Full time role"
const description = card.querySelector('.Card_Details__NXC2i, p')?.textContent?.trim();
// Walk up to find parent <a> for the link:
let el = card;
for (let i = 0; i < 6; i++) {
  el = el.parentElement;
  if (!el) break;
  if (el.tagName === 'A') { link = el.href; break; }
}
```

**Cookie banner:** Appears on first load. Dismiss with:
```javascript
Array.from(document.querySelectorAll('button')).find(b => b.textContent.trim() === 'Reject')?.click();
```

**Location filter (Boston):**
The location input is a react-select field with id `locationNew-desktop`. Fill it and select the dropdown option:
```javascript
// Fill the input
document.querySelector('#locationNew-desktop').value = 'Boston';
document.querySelector('#locationNew-desktop').dispatchEvent(new Event('input', {bubbles: true}));
// Then wait ~1s for dropdown, find the Boston option and click it:
Array.from(document.querySelectorAll('[class*="option"]')).find(o => o.textContent.includes('Boston'))?.click();
```
After applying the location filter, a "job alert" modal appears — dismiss it before proceeding.

**Workplace filter (In-Person/Hybrid run):**
The URL parameter `remote=false` does NOT reliably exclude remote jobs after a location filter is applied. After setting the location filter, manually check the Hybrid and In-person checkboxes in the Workplace Preference sidebar. Remote-only runs should be verified by checking that the Remote checkbox is unchecked.

**Puppeteer variable scope:** `const`/`let` declarations persist across `puppeteer_evaluate` calls in the same session. Use unique variable names per call (e.g., `allJobsRun2`, `containerEl`) or use anonymous expressions to avoid `Identifier already declared` errors.

**Output file naming:**
Use the label in the filename to distinguish run types on the same date:
- `2026-02-22-ClimateBase-Remote.md`
- `2026-02-22-ClimateBase-In-Person.md`

**Pre-screening:** With 100–200 filtered listings, visiting every posting is impractical. Apply a keyword pre-screen on job titles (Step 2.5 in workflow) to reduce candidates before visiting individual postings. Include terms like: Program Manager, Technical, Operations, Research, Strategy, Director, Head. Exclude terms like: Software Engineer, Data Scientist, Analyst, Sales, Marketing, Policy, Legal, Finance, Accountant.

---

### 3. Greentown Labs (`greentownlabs.com/careers`)

**Site type:** Standard WordPress/static page. No lazy loading, no infinite scroll. `document.body.scrollHeight` stabilizes immediately.

**Job card selector:** `a.card.x-between.flex.y-center` (or just `a.card`)

**Card field extraction:**
```javascript
const cards = Array.from(document.querySelectorAll('a.card'));
cards.map(card => ({
  title: card.querySelector('h3')?.textContent?.trim(),
  company: card.querySelector('h4')?.textContent?.trim(),
  link: card.href
}));
```

**Category association:** Categories are `<h3>` elements inside `.col-10.jobs` that are NOT inside an `<a>` tag. Walk backwards through siblings and up the DOM from each card to find the nearest preceding ungrouped `<h3>`:
```javascript
let node = card;
let cat = '';
outer: while (node) {
  let prev = node.previousElementSibling;
  while (prev) {
    const h3 = prev.tagName === 'H3' ? prev : prev.querySelector('h3');
    if (h3 && !h3.closest('a')) { cat = h3.textContent.trim(); break outer; }
    prev = prev.previousElementSibling;
  }
  node = node.parentElement;
  if (!node || node.tagName === 'BODY') break;
}
```

**Location filter:** The URL parameter `location=boston` has no effect. The dropdown offers only "All Locations." The board mixes Greentown Boston (Somerville, MA) and Greentown Houston (Houston, TX) members with no distinction on listing cards. Location must be confirmed by visiting each individual posting.

**Board characteristics:**
- ~60% IC engineering/software roles — lower PM/leadership density than Climatebase.
- No date filter — all listings appear as current; no way to filter for recency.
- Not all members are climate tech. NeuroBionics (neurotech medical devices) is one example.
- Most useful as a supplement to Climatebase for surfacing early-stage Boston climate startups not yet on larger boards.

**LinkedIn short URLs (`lnkd.in`):** Some Greentown listings link to `lnkd.in/` shortened URLs. These redirect through a LinkedIn warning page that shows the actual destination URL. Copy and navigate to the destination directly (do not follow through LinkedIn's redirect).

**Dover ATS (`app.dover.com`):** Blocked by Cloudflare bot protection and the network egress proxy. Cannot be fetched via Puppeteer or WebFetch. If a posting is on Dover, skip the detailed review and note it as unverifiable in the master list.

**Expired postings:** Some listings redirect to a 404 or "job not found" page. Note as expired/removed and skip.

---

### 4. LinkedIn (`linkedin.com/jobs`)

**Site type:** React SPA. No Cloudflare blocking — URL parameters work directly, no need to navigate from the homepage first.

**⚠️ Login required for full results.** Anonymous access caps at ~66 cards per search query (190 exist for "technical program manager" + Boston + past 30 days). Login removes this cap. Recommend using Claude in Chrome with your existing LinkedIn session for full coverage.

**Without login — workaround:** Run multiple narrow keyword searches to maximize coverage, then deduplicate by job ID:
- `technical program manager`
- `R&D program manager`
- `hardware program manager`
- `operations program manager`
- `research program manager`

**URL filter parameters (all work via direct navigation):**
```
keywords=technical+program+manager
location=Boston%2C+MA
f_TPR=r2592000     # Past 30 days (r604800 = 7 days)
f_E=4%2C5          # Experience: 4=Mid-Senior, 5=Director
f_JT=F             # Job type: F=Full-time
f_WT=2             # Workplace: 1=Onsite, 2=Remote, 3=Hybrid (omit for all)
sortBy=DD          # Sort by date descending
```

Example full URL:
`https://www.linkedin.com/jobs/search/?keywords=technical+program+manager&location=Boston%2C+MA&f_TPR=r2592000&f_E=4%2C5&f_JT=F&sortBy=DD`

**Login modal dismissal:**
```javascript
document.querySelectorAll('[class*="modal"], [role="dialog"]').forEach(m => m.style.display = 'none');
```
The `button.modal__dismiss` selector also works but may need retry. Hiding via CSS is more reliable.

**Card selector:** `.job-search-card` (also `.base-card`)

**Card field extraction:**
```javascript
const cards = Array.from(document.querySelectorAll('.job-search-card'));
cards.map(card => ({
  title: card.querySelector('h3')?.textContent?.trim(),
  company: card.querySelector('h4')?.textContent?.trim(),
  location: card.querySelector('.job-search-card__location')?.textContent?.trim(),
  posted: card.querySelector('time')?.getAttribute('datetime'),  // ISO format
  postedLabel: card.querySelector('time')?.textContent?.trim(),  // e.g. "2 weeks ago"
  link: card.querySelector('a.base-card__full-link')?.href,
  jobId: card.querySelector('a')?.href?.match(/view\/[^?]+?-(\d+)\?/)?.[1]
}));
```

**Individual job pages:** Accessible without login at `/jobs/view/[slug]-[id]`. Strip tracking params from links. Full JD requires clicking "Show more":
```javascript
document.querySelector('.show-more-less-html__button--more')?.click();
const jd = document.querySelector('.show-more-less-html__markup')?.innerText?.trim();
```

**Structured fields available on job pages (no login required):**
- Seniority level (e.g., "Mid-Senior level")
- Employment type (e.g., "Full-time")
- Job function (e.g., "Engineering")
- Industries (e.g., "IT Services and IT Consulting") — useful for filtering non-target sectors

**No sector/industry filter on search results.** Unlike Climatebase, LinkedIn has no climate/sustainability filter. All industries appear. Use title-based keyword pre-screening aggressively and check the "Industries" field on individual postings to skip obviously irrelevant sectors (healthcare staffing, retail, finance, etc.).

**Pagination/lazy loading:** Cards load as you scroll. Without login, scrolling stops loading after ~66 cards. With login, the full result set loads on scroll.

**Notification popups:** LinkedIn shows "Get the app" and "Know when new jobs open up" banners — dismiss by hiding `[class*="cta-modal"]` elements.

---

### 5. The Engine (`engine.xyz/careers`)

**Site type:** Static/server-rendered (Getro platform). No JavaScript lazy loading — all cards on a page are in the DOM on initial load. No bot detection.

**Total jobs:** ~442 across all companies in the Engine portfolio (MIT Tough Tech accelerator). Covers climate, energy, materials, advanced manufacturing, robotics, and AI. High signal for Jonathan's target sector.

**URL structure — filters and pagination combine cleanly:**
```
Base:          https://engine.xyz/careers
With filter:   https://engine.xyz/careers?job_functions=Operations&topics=Energy
Page 2+:       https://engine.xyz/careers/p2?job_functions=Operations
```
Navigate each page directly — no JavaScript interaction needed.

**Filter parameters:**

`job_functions` (single value per request — run separately and deduplicate):
- `Operations` — 53 jobs (2 pages); most relevant for Jonathan
- `Product` — covers PM roles
- `IT` — covers IS&T-adjacent roles
- `Other Engineering` — catches TPM/engineering program roles

`locations` (single MA city per request — see location strategy below):
- `Boston, MA, USA` / `Cambridge, MA, USA` / `Somerville, MA, USA`
- `Devens, MA, USA` / `Waltham, MA, USA` / `Woburn, MA, USA`
- `Stow, MA, USA` / `Chelsea, MA, USA` / `Northampton, MA, USA`
- `Multiple locations` — includes some MA-based roles

`topics` (optional additional filter):
- `Energy` / `AI & ML` / `Applied AI & ML` / `Advanced Manufacturing`
- `Autonomous Systems & Robotics` / `Materials` / `Food & Agriculture`

**Recommended workflow — grab all, filter in Python:**
Rather than running per-city location queries, omit the location filter, run all 4 pages for each target `job_functions` value, extract all cards, then post-filter by MA location. ~442 total jobs, ~50 per page.

```python
ma_cities = ['Boston, MA', 'Cambridge, MA', 'Somerville, MA', 'Devens, MA',
             'Waltham, MA', 'Woburn, MA', 'Stow, MA', 'Chelsea, MA',
             'Northampton, MA', 'Multiple locations']

def is_ma(location):
    return any(city.lower() in location.lower() for city in ma_cities)
```

**Card selector:** `.careers-jobs__item`

**Card field extraction:**
```javascript
const cards = Array.from(document.querySelectorAll('.careers-jobs__item'));
cards.map(li => ({
  title: li.querySelector('.careers-jobs__title a')?.textContent?.trim(),
  link: li.querySelector('.careers-jobs__title a')?.href,
  company: li.querySelector('.careers-jobs__meta p:nth-child(1)')?.textContent?.trim(),
  location: li.querySelector('.careers-jobs__meta p:nth-child(2)')?.textContent?.trim(),
  posted: li.querySelector('.careers-jobs__meta p:last-child')?.textContent?.trim(),
  topics: li.querySelector('[class*="topic"], [class*="tag"]')?.textContent?.trim().replace(/\s+/g, ' ')
}));
```

**Posted date format:** Relative label only ("Posted today", "Posted yesterday", "Posted 3 days ago"). No ISO date available from the card — note as-is in the master list.

**Pagination:** 4 pages for full 442-job set (~50 per page, ~110 per page with no filter). Navigate `/careers`, `/careers/p2`, `/careers/p3`, `/careers/p4`. Confirm actual page count from the pagination nav after loading page 1 — it may vary if running with a filter applied.

**ATS systems used by Engine portfolio companies:**
- Lever (`jobs.lever.co`) — accessible ✓
- LinkedIn (`linkedin.com/jobs/view/`) — accessible ✓
- Ashby (`jobs.ashbyhq.com`) — accessible ✓ (but see Form Energy note below)
- Gusto (`jobs.gusto.com`) — accessible ✓
- Dover (`app.dover.com`) — **blocked** ✗ (Cloudflare + egress proxy)

**Form Energy ATS (2026-02-22):** Form Energy's Ashby board (`jobs.ashbyhq.com/formenergy`) returns 404 — they have removed or migrated their ATS. All Form Energy job links from engine.xyz are stale. Skip Form Energy postings until the board is updated with a working ATS URL.

**No date filter available.** Jobs are not filterable by recency. Run periodically and deduplicate against prior master lists by job link URL.

**Applied AI & ML topic filter — do not repeat (2026-02-22 finding):** This filter returns 56 jobs dominated by Osmo (fragrance AI, Elizabeth NJ) and ISEE (autonomous vehicles, TX/AL). ISEE's ~33 listings are largely 1,000–2,200 days stale. Only 4 MA-area jobs found, none new or relevant. For AI-adjacent roles in the Engine portfolio, use the Operations job_function filter and post-screen by title keyword. 80,000 Hours (board #6) is a higher-signal source for AI-focused opportunities.

**"Multiple locations" jobs:** Some cards show "Multiple locations" — check the individual posting to confirm if MA is one of them.

---

### 6. 80,000 Hours (`jobs.80000hours.org`)

**Site type:** Nuxt.js SPA with Algolia InstantSearch backend. Job cards are `button.job-card` elements — they open a side panel on click but do NOT update the URL. **Use the Algolia API directly instead of scraping card DOM.**

**Algolia credentials:**
Load from `.env` before running (see `.env.example` for setup). Do not hardcode these values.
```
App ID:    ALGOLIA_APP_ID     (from .env)
API Key:   ALGOLIA_API_KEY    (from .env)
Index:     ALGOLIA_INDEX      (from .env, default: jobs_prod)
```
Retrieve current values from browser devtools (Network tab) while on the 80k Hours job board if credentials have rotated.

Query via Puppeteer `fetch()` (not Python — egress proxy blocks Algolia from bash):
```javascript
// Load these from your .env before starting the session:
// ALGOLIA_APP_ID, ALGOLIA_API_KEY, ALGOLIA_INDEX
fetch(`https://${ALGOLIA_APP_ID}-dsn.algolia.net/1/indexes/${ALGOLIA_INDEX}/query`, {
  method: 'POST',
  headers: {
    'X-Algolia-Application-Id': ALGOLIA_APP_ID,
    'X-Algolia-API-Key': ALGOLIA_API_KEY,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    filters: 'tags_location_80k:"Boston metro area" OR tags_location_80k:"Remote, USA"',
    hitsPerPage: 200,
    page: 0,
    attributesToRetrieve: ['title','company_name','url_external','tags_location_80k',
                           'tags_area','tags_role_type','tags_experience','posted_at','description_short']
  })
}).then(r => r.json()).then(d => { window._80kData = d; });
// Then read: window._80kData.hits
```

**Key fields returned per hit:**
- `title`, `company_name`, `url_external` (the actual job URL), `tags_area` (e.g. "AI safety & policy"), `tags_location_80k`, `tags_role_type` (Full-time / Internship / Fellowship / Part-time), `posted_at` (Unix timestamp), `description_short` (HTML snippet)

**Filter parameters (URL-based, uses Algolia facet syntax):**
- Location: `tags_location_80k:"Boston metro area"`, `tags_location_80k:"Remote, USA"`, `tags_location_80k:"Remote, Global"`
- Area: `tags_area:"AI safety & policy"`, `tags_area:"Climate change"`, etc.
- Role type: `tags_role_type:"Full-time"`
- Combine with ` OR ` / ` AND `

**URL filter format (for browser navigation):**
`https://jobs.80000hours.org/?refinementList%5Btags_location_80k%5D%5B0%5D=Boston%20metro%20area&refinementList%5Btags_location_80k%5D%5B1%5D=Remote%2C%20USA`

**Board characteristics:**
- 831 total roles as of 2026-02-22; Boston + Remote USA = 89; after full-time filter = 70
- Heavily weighted toward AI safety & policy (426 roles), biosecurity (64), global health (75)
- Climate change has only 8 roles — very small but high signal
- Anthropic has 45 listings (mostly SF-based)
- Skews policy/research; limited hardware TPM or climate tech roles
- High curation — all roles are "handpicked." Low noise, low volume.
- **Recommended cadence: monthly** (small board, high curation, minimal overlap with other boards)

**Pre-screening filters (for Jonathan):**
- EXCLUDE role types: Internship, Fellowship, Part-time, Other
- INCLUDE title keywords: program manager, program director, program lead, head of operations, operations manager, director, senior manager, principal, technical program, policy lead, ai policy, research lead, senior director
- EXCLUDE title keywords: software engineer, data engineer, ml engineer, research scientist, research associate, research assistant, postdoctoral, analyst, recruiter, video producer, undercover investigator, data scientist

**Individual job pages:** Each hit has `url_external` pointing directly to the employer's ATS (Greenhouse, Ashby, Workday, Bamboo, etc.). Navigate directly — no side panel needed.

**Note on duplicate Active Site listings:** Active Site (panoplialabs) posted the same "Head of Operations" role twice with different objectIDs but different URLs (one Ashby, one Google Doc). Deduplicate by title + company.

---

### 7. Draper Laboratory (`draper.wd5.myworkdayjobs.com`)

**Site type:** Workday. Standard Workday Careers board. No bot detection.

**Total jobs:** 150 (as of 2026-02-22). Predominantly Cambridge, MA with outliers (Odon IN, Clearfield UT, St. Petersburg FL, Huntsville AL, Reston VA, Houston TX).

**Job categories (2026-02-22):** Engineering (110), Miscellaneous (20), Security & IA (7), Finance (5), Facilities (2), R&D & Technicians (2), Property & Supply Chain (1), HR (1), Operations (1), Contracts (1).

**Workday CXS API (undocumented but reliable):**
```javascript
// Fetches all jobs — paginate in steps of 20 (limit:200 returns HTTP 400)
// Important: returnFacets:true is required — omitting it returns HTTP 400
// Important: only page 1 returns total; subsequent pages return total:0 — cache total from first call

async function fetchAllDraper() {
  const url = 'https://draper.wd5.myworkdayjobs.com/wday/cxs/draper/Draper_Careers/jobs';
  const headers = { 'Content-Type': 'application/json', 'Accept': 'application/json' };
  const limit = 20;
  const first = await fetch(url, { method: 'POST', headers,
    body: JSON.stringify({ limit, offset: 0, searchText: '', appliedFacets: {}, returnFacets: true })
  }).then(r => r.json());
  const total = first.total;
  let all = first.jobPostings || [];
  for (let offset = limit; offset < total; offset += limit) {
    const d = await fetch(url, { method: 'POST', headers,
      body: JSON.stringify({ limit, offset, searchText: '', appliedFacets: {}, returnFacets: true })
    }).then(r => r.json());
    all = all.concat(d.jobPostings || []);
  }
  return { total, fetched: all.length, jobs: all };
}
window._draperDone = false;
fetchAllDraper().then(r => { window._draperResult = r; window._draperDone = true; });
```

**Response fields per job:**
- `title` — job title
- `externalPath` — relative path, e.g. `/job/Cambridge-MA/Hardware-Analog-Electronics-Design-Engineer_JR001146`
- `locationsText` — e.g. "Cambridge, MA" or "2 Locations"
- `postedOn` — relative string, e.g. "Posted 3 Days Ago"
- `bulletFields[0]` — req ID, e.g. "JR001146"

**Full job URL pattern:** `https://draper.wd5.myworkdayjobs.com/en-US/Draper_Careers{externalPath}`

**⚠️ Clearance barrier:** All management-tier roles (Group Leader, Supervisor, Manager, Director) require an Active US Government Security Clearance (Secret or higher). This is a structural pattern across the entire Draper board — not role-specific. IC engineering roles may not all require clearance, but none of those are relevant to Jonathan. **Do not repeat this run unless Jonathan has obtained or is actively pursuing a clearance.**

**Jonathan's existing application:** Sensor Development Program Manager (not in current 150-job listing — likely closed or filled). Draper is already tracked in his active applications list.

**Pre-screening for Jonathan:** With the clearance barrier, management roles (Group Leader, Supervisor, Director, Manager) should be skipped unless Jonathan's clearance status changes. No PM-titled roles were present in the 2026-02-22 run.

**Recommended cadence:** Quarterly check for new PM-titled openings, or when Jonathan has a clearance.

---

### 8. Work on Climate (`workonclimate.org`)

**Site type:** Community platform — no public job board.

Work on Climate is a Slack community (~35,000 members) with jobs shared via:
1. The `#job-board` Slack channel (members post directly, high volume)
2. A monthly newsletter (`mailchi.mp/workonclimate.org/website-newsletter-signup`)

**There is no scrapable job listing page.** The `/jobs` URL returns a 404.

**Access method:** Jonathan should join the Slack workspace (`bit.ly/joinWoCl`) and subscribe to the newsletter. Jobs are not automatable — monitor manually.

**Signal quality:** High for climate ops/PM roles. Community skews senior, mission-driven. Lower engineering-to-ops ratio than Climatebase.

**Recommended cadence:** Monitor Slack `#job-board` weekly; newsletter arrives monthly.

---

### 9. Wellfound / AngelList Talent (`wellfound.com`)

**Site type:** React SPA — **bot-protected** (Cloudflare slider CAPTCHA on direct navigation).

**Cannot be scraped with Puppeteer from the VM.** Attempting to navigate directly triggers a human verification slider that cannot be bypassed.

**Access method:** Use Claude in Chrome (browser extension) with Jonathan's existing Wellfound login session. The CAPTCHA does not appear for authenticated browser sessions.

**URL filter parameters (all apply server-side, combine freely):**
```
https://wellfound.com/jobs
  ?role=operations
  &role=product-manager
  &industry=Climatetech
  &industry=ArtificialIntelligence
  &jobType=fulltime
  &remote=only          # omit for all locations
```

**Known valid `industry` values:** `Climatetech`, `ArtificialIntelligence`, `Robotics`, `CleanEnergy`, `B2B`, `SaaS`

**Known valid `role` values:** `operations`, `product-manager`, `software-engineer`, `recruiting`, `marketing`

**Card selector (once logged in and page loads):**
```javascript
const cards = Array.from(document.querySelectorAll('[class*="styles_component"]')).filter(el => el.querySelector('h2'));
```
(Class names are hashed and may change — identify by structure: each card has h2 for title, company name, location, and salary.)

**Login requirement:** Full results require login. Anonymous access shows partial listings.

**Recommended cadence:** Monthly, via Claude in Chrome.

---

### 10. Y Combinator — Work at a Startup (`workatastartup.com`)

**Site type:** Server-rendered (Next.js). No bot detection, no login required. URL params apply server-side.

**Recommended URL for Jonathan:**
```
https://www.workatastartup.com/jobs?industry=Climatetech&industry=ArtificialIntelligence&jobType=fulltime&role=operations&role=product-manager
```

**URL filter parameters:**
- `industry` (multi-value): `Climatetech`, `ArtificialIntelligence`, `Robotics`, `Fintech`, `Healthcare`
- `role` (multi-value): `operations`, `product-manager`, `software-engineer`, `science`, `recruiting`
- `jobType`: `fulltime`, `parttime`, `internship`
- `remote`: `only` (remote-only), omit for all
- `location`: **no effect** — does not filter results. Post-process by location text in cards.

**Card selector:** `.jobs-list > div` (each `div` is one company+job block)

**Field extraction:**
```javascript
const items = Array.from(document.querySelectorAll('.jobs-list > div'));
const jobs = items.map(item => ({
  company:  item.querySelector('.font-bold')?.textContent?.trim(),
  batch:    item.querySelector('.text-gray-600')?.textContent?.trim(),  // e.g. "AI-native insurance brokerage"
  jobName:  item.querySelector('[class*="job-name"]')?.textContent?.trim(),
  jobType:  item.querySelector('p.job-details span:first-child')?.textContent?.trim(),
  location: item.querySelector('p.job-details span:last-child')?.textContent?.trim(),
  jobHref:  item.querySelector('a[href*="/jobs/"]')?.href,
  jobId:    item.querySelector('a[href*="/jobs/"]')?.getAttribute('data-jobid')
}));
```

**Location post-filtering:** The `location` field extracted above is free-text (e.g., "San Francisco", "Remote", "Boston, MA"). Filter for `Remote` or `Boston` in post-processing.

**Individual job pages:** Navigate to `https://www.ycombinator.com/companies/{slug}/jobs/{jobid}-{title-slug}`. Full JD available without login.

**Total results:** Typically 20–40 for Operations + Climatetech + AI filters combined. No pagination needed.

**⚠️ Industry filter finding (2026-02-22):** The `industry=Climatetech` and `industry=ArtificialIntelligence` URL params do not reliably filter results. The 23 Operations+PM results were identical with or without either industry filter — all results were AI/fintech/SaaS companies with no climate relevance. Zero climate-specific Operations or PM roles were open across the YC portfolio on this date. The board is useful as an AI startup discovery tool, not a climate board. **Do not expect Climatetech filter to surface true climate companies.**

**Board characteristics:**
- All companies are YC-backed (vetted, funded). High signal for early-stage AI/tech startups.
- Batch tag (e.g., W25, S24) indicates recency — F/W/S + year.
- Many listings are "Founding" roles — small team, broad scope, high equity.
- Skews heavily SF; most roles are SF in-person. Few remote or Boston roles.
- Low climate signal for Ops/PM roles — use Climatebase, The Engine, BEV/BEF for climate.

**Recommended cadence:** Monthly. Useful for AI-adjacent startup roles; low value for climate.

---

### 11 & 12. Breakthrough Energy — BEV Jobs & BEF Jobs

Two Getro-powered job boards for Breakthrough Energy's portfolio:

| # | Label | URL | Collection ID | Total Jobs | Companies |
|---|-------|-----|---------------|------------|-----------|
| 11 | BEV-Jobs | https://bevjobs.breakthroughenergy.org/jobs | 1533 | ~767 | ~92 |
| 12 | BEF-Jobs | https://befjobs.breakthroughenergy.org/jobs | 2567 | ~77 | ~30 |

**BEV** = Breakthrough Energy Ventures (VC portfolio: CFS, Electric Hydrogen, Fervo, Antora, etc.)
**BEF** = Breakthrough Energy Fellows (early-stage deep-tech startups by BEF alumni). Smaller, higher signal for very early-stage roles.

**Site type:** Getro Next.js SPA — same platform as The Engine but different API authentication model.

**Key difference from The Engine:** URL query params (`job_functions=Operations`, etc.) cause **HTTP 500 errors** on these boards. Filters must be applied via UI interaction. The Getro API (`api.getro.com/api/v2/collections/{id}/jobs`) requires server-side authentication not available client-side.

**SSR embedded data (first 20 jobs, unfiltered):**
```javascript
const nd = JSON.parse(document.getElementById('__NEXT_DATA__').textContent);
const jobs = nd.props.pageProps.initialState.jobs;
// jobs.total = total job count; jobs.found = array of first 20 jobs
const firstJob = jobs.found[0];
// Fields: id, title, organization.name, locations[], workMode, url, seniority, createdAt, slug
```

**Card selector (after UI filter applied):** `[data-testid="job-list-item"]`

**Field extraction from DOM:**
```javascript
const cards = Array.from(document.querySelectorAll('[data-testid="job-list-item"]'));
const jobs = cards.map(c => ({
  title:    c.querySelector('[data-testid="job-title-link"]')?.textContent?.trim(),
  url:      c.querySelector('[data-testid="job-title-link"]')?.href,
  text:     c.innerText  // parse company name, location, seniority from text
}));
```

**Applying UI filters:**
```javascript
// 1. Click "Job function" filter (first filter option)
document.querySelector('[data-testid="filter-option-item-0"]').click();
// 2. Wait for dropdown, then find and click "Operations" checkbox
// 3. Repeat for "Product"
// 4. Wait for results to reload, then extract cards
```

**Available job function filter values (BEV):** Operations, Software Engineering, IT, Other Engineering, Sales & Business Development, Design, Accounting & Finance, People & HR, Quality Assurance, Product, Marketing & Communications, Customer Service, Data Science, Legal, Administration

**Board characteristics:**
- BEV: largest climate VC portfolio in the world. Companies include CFS, Form Energy, Electric Hydrogen, Fervo Energy, Antora, Stoke Space, Quantum SI. Heavy on deep-tech hardware.
- BEF: smaller, earlier-stage. Very high climate-tech signal. 77 jobs total makes it manageable without filtering.
- Both boards skew toward engineering and science roles. Operations/PM roles are a minority but high quality.
- For BEF: omit filters entirely — 77 jobs is small enough to review all titles in one pass.

**Recommended cadence:** Monthly. Run BEF unfiltered; run BEV with Operations + Product filter via UI.

---

## Adding a New Board

When running a new board for the first time:
1. Note the site type (React/SPA vs. static).
2. Identify the job card selector and field extraction pattern.
3. Document any location/workplace filter quirks.
4. Note any ATS systems that are blocked (Dover, etc.).
5. After the run, append a new section here and a new entry in the table above.
6. Update the improvement log in `JOB-BOARD-WORKFLOW.md` with lessons learned.
