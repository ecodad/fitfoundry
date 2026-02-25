# Climatebase

**Board #:** 1 & 2
**URL:** `https://climatebase.org/jobs`
**Last run:** 2026-02-22
**Auth required:** No
**Scraping method:** Puppeteer JS — scroll-based lazy load + CSS class extraction

---

## Overview

The primary climate-focused job board. Aggregates roles from climate companies worldwide. High signal-to-noise for climate ops/PM roles. Run as two separate passes: Remote only, and Hybrid + In-person. The exact filtered URL for each prior run is recorded in the corresponding master list file.

---

## URL & Filter Parameters

Location filter is applied via UI (react-select input), not URL parameter. After setting location, apply workplace type via the sidebar checkboxes.

Relevant URL parameters (partial — location and workplace must be set via UI):
```
https://climatebase.org/jobs?location=Boston%2C+Massachusetts%2C+USA&remote=false
```

**⚠️ `remote=false` is unreliable** after applying a location filter via UI. Always manually verify the Hybrid and In-person checkboxes are set in the Workplace Preference sidebar.

---

## Extraction

**Site type:** React SPA with lazy-loaded job cards. Standard link selectors (`a[href*="/jobs/"]`) return nothing — use the class-based approach below.

**Scrollable container:** The job list renders in a nested `div`, not `document.body`. Detect it by finding elements where `scrollHeight > clientHeight`. The stable class pattern is `sc-3d1ac256-2` (may change on site rebuild — re-detect if it stops working).

```javascript
const container = document.querySelector('.sc-3d1ac256-2.cPkuCi');
container.scrollTo(0, container.scrollHeight);
```

Scroll in 2000px increments with 800ms waits. Repeat until `scrollHeight` stabilizes across two consecutive reads.

**Job card selector:** `.Card_Information__HkDL9`

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

**2. Location filter** — the input is a react-select field with id `locationNew-desktop`:
```javascript
document.querySelector('#locationNew-desktop').value = 'Boston';
document.querySelector('#locationNew-desktop')
  .dispatchEvent(new Event('input', { bubbles: true }));
// Wait ~1s, then click the Boston dropdown option:
Array.from(document.querySelectorAll('[class*="option"]'))
  .find(o => o.textContent.includes('Boston'))?.click();
```

**3. Job alert modal** — appears after location filter is applied. Dismiss before proceeding.

**4. Workplace filter** — check Hybrid and In-person checkboxes in the sidebar (for the In-person run). For the Remote run, confirm the Remote checkbox is checked and others are not.

---

## Known Issues & Quirks

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
