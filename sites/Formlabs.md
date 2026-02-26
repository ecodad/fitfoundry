# Formlabs

**Board #:** 13
**URL:** `https://careers.formlabs.com/`
**Last run:** 2026-02-25
**Auth required:** No
**Scraping method:** Puppeteer JS — native `<select>` filter + React Table extraction

---

## Overview

Direct company career site for Formlabs (3D printing hardware/software, HQ in Somerville, MA). Not an aggregator — all listings are internal roles. High signal for hardware TPM, operations, and product management. Jobs load fully on initial page render with no lazy loading or pagination. Relevant teams for ops/PM targeting: Hardware Engineering, Manufacturing, Product, Customer Strategy & Operations, Operations.

**Duplicate posting quirk:** Formlabs lists most roles twice — once as "Somerville, MA" and once as "Boston, MA" with distinct job IDs. Both link to the same physical HQ. Filter to Somerville to get the canonical set; skip Boston duplicates in results.

---

## URL & Filter Parameters

No URL parameters control filtering — all filters operate via `<select>` DOM elements with a `change` event dispatch.

Starting URL: `https://careers.formlabs.com/`

**Location select value for Somerville:** `3986`

Team filter options (select index 0):
- Customer Strategy & Operations, Global Marketing, Global Sales, Global Services, Hardware Engineering, Legal, Manufacturing, Materials Engineering, Operations, People & Culture, Product, Software Engineering, Spectra, Systems

Type filter options (select index 2): Full-time, Intern

---

## Extraction

**Site type:** Custom React app using React Table (`.rt-*` classes). All rows render immediately — no scrolling or pagination required.

**Step 1 — Apply location filter:**
```javascript
(() => {
  const selects = document.querySelectorAll('select');
  const locationSelect = selects[1]; // index 1 = Location
  const option = Array.from(locationSelect.options)
    .find(o => o.text.includes('Somerville'));
  locationSelect.value = option.value; // '3986'
  locationSelect.dispatchEvent(new Event('change', { bubbles: true }));
  return `Selected: ${option.text}`;
})()
```

**Step 2 — Optionally filter by type** (skip interns):
```javascript
(() => {
  const selects = document.querySelectorAll('select');
  const typeSelect = selects[2]; // index 2 = Type
  const option = Array.from(typeSelect.options)
    .find(o => o.text === 'Full-time');
  typeSelect.value = option.value;
  typeSelect.dispatchEvent(new Event('change', { bubbles: true }));
  return `Type filter: Full-time`;
})()
```

**Step 3 — Extract all visible rows:**
```javascript
(() => {
  const rows = Array.from(document.querySelectorAll('.rt-tr-group'));
  return rows.map(row => {
    const cells = Array.from(row.querySelectorAll('.rt-td'));
    const link = row.querySelector('a[href*="/job/"]');
    return {
      title:    cells[0]?.textContent?.trim(),
      team:     cells[1]?.textContent?.trim(),
      location: cells[2]?.textContent?.trim(),
      type:     cells[3]?.textContent?.trim(),
      href:     link?.href
    };
  }).filter(r => r.title && r.title.length > 0);
})()
```

**Job detail pages:** Accessible at `/job/[ID]/apply/` — NOT at `/job/[ID]/` (returns 404). Full description is in `document.body.innerText` starting after the Department/Location header lines.

---

## Known Issues & Quirks

- **Duplicate postings:** Every role typically appears twice — "Somerville, MA" and "Boston, MA" — with different job IDs. Both are the same physical role at HQ. Filter or deduplicate by keeping only the Somerville entry.
- **Job detail URL pattern:** The listing links go to `/job/[ID]/apply/` with `/apply/` suffix. Navigating to `/job/[ID]/` (without suffix) returns a 404.
- **No URL-based filtering:** All three filters (Team, Location, Type) are native `<select>` elements. Filter state is not reflected in the URL and resets on page reload.
- **React Table classes (`rt-tr-group`, `rt-td`) are stable** across observed runs — less likely to change than styled-components class names on SPAs.
- **No login, CAPTCHA, or cookie banner** observed on initial load.
- **No pagination** — all matching rows render in one pass after filter dispatch.

---

## Recommended Cadence

Bi-weekly. Formlabs posts in batches; the Manufacturing and Hardware Engineering teams in particular have rolling TPM/ops openings. Low effort run — single filter dispatch, immediate full render.
