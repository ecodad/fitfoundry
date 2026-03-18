# Climatebase

**Board #:** 1 & 2
**URL:** `https://climatebase.org/jobs`
**Last run:** Never
**Auth required:** No
**Scraping method:** Puppeteer JS — scroll-based lazy load + CSS class extraction

---

## Overview

The primary climate-focused job board. Aggregates roles from climate companies worldwide. High signal-to-noise for climate ops/PM roles. Run as two separate passes: Remote only, and Hybrid + In-person. The exact filtered URL for each prior run is recorded in the corresponding master list file.

---

## URL & Filter Parameters

Location filter is applied via UI (react-select input), not URL parameter. After setting location, apply workplace type via the sidebar checkboxes.

Relevant URL parameters (partial — location and workplace must be set via UI). Example using a city/state location:
```
https://climatebase.org/jobs?location=[City%2C+State%2C+Country]&remote=false
```

**⚠️ `remote=false` is unreliable** after applying a location filter via UI. Always manually verify the Hybrid and In-person checkboxes are set in the Workplace Preference sidebar.

---

## Extraction

**Site type:** React SPA with lazy-loaded job cards. Standard link selectors (`a[href*="/jobs/"]`) return nothing — use the class-based approach below.

**Scrollable container:** The job list renders in a nested `div`, not `document.body`. Detect it by finding elements where `scrollHeight > clientHeight`. The stable class pattern changes on site rebuilds — re-detect each run using the scrollHeight approach.

- 2026-02-22 class: `sc-3d1ac256-2`
- 2026-03-17 class: `sc-977ff104-2`

```javascript
const container = document.querySelector('.sc-977ff104-2');
// Or detect dynamically:
// Array.from(document.querySelectorAll('div')).find(el => { const s = window.getComputedStyle(el); return (s.overflowY==='auto'||s.overflowY==='scroll') && el.scrollHeight > 1000; });
container.scrollTo(0, container.scrollHeight);
```

Scroll in 2000px increments with 800ms waits. Repeat until `scrollHeight` stabilizes across two consecutive reads.

**Job card selector:** `.Card_Information__HkDL9` (use `[class*="Card_Information"]` as a stable fallback — class suffix changes on site rebuilds)

**Card field extraction:**
```javascript
const cards = Array.from(document.querySelectorAll('.Card_Information__HkDL9'));
cards.map(card => {
  const title    = card.querySelector('h2 span')?.textContent?.trim();
  const lis      = Array.from(card.querySelectorAll('ul li'));
  const company  = lis[0]?.textContent?.trim();
  const location = lis[1]?.textContent?.trim();
  const workplace= lis[2]?.textContent?.trim(); // "In-person", "Hybrid", "Remote"
  const jobType  = lis[3]?.textContent?.trim(); // "Full time role"
  const desc     = card.querySelector('.Card_Details__NXC2i, p')?.textContent?.trim();
  // Walk up DOM to find parent <a> for link:
  let el = card, link = '';
  for (let i = 0; i < 6; i++) {
    el = el.parentElement;
    if (!el) break;
    if (el.tagName === 'A') { link = el.href; break; }
  }
  return { title, company, location, workplace, jobType, desc, link };
});
```

---

## Setup Steps

**1. Cookie banner** — dismiss on first load:
```javascript
Array.from(document.querySelectorAll('button'))
  .find(b => b.textContent.trim() === 'Reject')?.click();
```

**2. Location filter** — the input is a react-select field with id `locationNew-desktop`. Replace `[YOUR_CITY]` with the city from `QUICK_REFERENCE_PATH → Default search location`:
```javascript
// Replace [YOUR_CITY] with the city from QUICK_REFERENCE_PATH → Default search location
document.querySelector('#locationNew-desktop').value = '[YOUR_CITY]';
document.querySelector('#locationNew-desktop')
  .dispatchEvent(new Event('input', { bubbles: true }));
// Wait ~1s, then click the matching city in the dropdown:
Array.from(document.querySelectorAll('[class*="option"]'))
  .find(o => o.textContent.includes('[YOUR_CITY]'))?.click();
```

**3. Job alert modal** — appears after location filter is applied. Dismiss before proceeding.

**4. Workplace filter** — check Hybrid and In-person checkboxes in the sidebar (for the In-person run). For the Remote run, confirm the Remote checkbox is checked and others are not.

---

## Known Issues & Quirks

- **⚠️ Individual job detail pages are Cloudflare-blocked via Puppeteer.** `climatebase.org/job/[id]` pages cannot be fetched via Puppeteer or WebFetch. Use **Claude in Chrome** to open each job detail page and retrieve the full description. Card extraction via Puppeteer gives only the truncated summary — full descriptions (salary, requirements, responsibilities) require a separate Claude in Chrome pass after scoring.
- `remote=false` URL param does not reliably exclude remote after a location filter is applied via UI. Always set via sidebar checkboxes.
- Class names (`sc-3d1ac256-2`, `Card_Information__HkDL9`) may change on site rebuilds. Re-detect if extraction stops working.
- A "job alert" modal fires after setting the location filter — must be dismissed before scrolling.
- Puppeteer variable scope: `const`/`let` declarations persist across `puppeteer_evaluate` calls. Use unique variable names per call or anonymous expressions to avoid `Identifier already declared` errors.

---

## Output File Naming

Use the run type as a label suffix to distinguish passes on the same date:
- `YYYY-MM-DD-ClimateBase-Remote.md`
- `YYYY-MM-DD-ClimateBase-In-Person.md`

---

## Recommended Cadence

Monthly. High signal, active board. Run both Remote and In-person passes.

---

## Run History

### 2026-03-17 — Remote + Boston/In-Person Run

Filters: Remote pass (Past Month, Full time, Remote workplace) + Boston In-Person/Hybrid pass. 48 unique listings extracted; 31 US-eligible after removing international. 15 individual files created (score ≥ 6). Key findings:

- Container class changed: `sc-3d1ac256-2` → `sc-977ff104-2`. Use `[class*="Card_Information"]` for cards and dynamic scrollHeight detection for container.
- ClimateBase individual job detail pages (climatebase.org/job/[id]) are Cloudflare-blocked via Puppeteer. Full descriptions must be retrieved via browser or Claude in Chrome.
- "Past Month" date filter used; timestamp text on cards used to filter to last 10 days during JS extraction.
- The Job Alert modal fires after location filter — dismiss via `JobAlertModal_CloseButton__akJvI` class button.
- Boston In-Person/Hybrid pass returned 0 unique results (Ekotrope was already in Remote pass as Hybrid, Remote).
- VM egress proxy caused mis-redirects for some company career pages (e.g., ekotrope.com → formlabs.com, dynapower.com → sensata.com). Dynapower confirmed acquired by Sensata; use Sensata Workday.
- Hayden AI (haydenai.com) redirects to PTC; their Staff TPM posting may be a ghost from before acquisition.

### 2026-02-22 — In-Person Run

First successful run. Confirmed the scroll-based extraction approach documented above. Key findings incorporated into the Extraction and Known Issues sections:

- Standard link selectors (`a[href*="/jobs/"]`) return nothing — class-based card extraction is required.
- `remote=false` URL param is unreliable after a location filter is applied via UI — must use sidebar checkboxes.
- Job alert modal fires after location filter — must dismiss before scrolling.
- Pre-screening by title is essential at scale: 143 filtered listings reduced to 14 candidates before visiting individual postings.
