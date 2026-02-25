# 80,000 Hours

**Board #:** 6
**URL:** `https://jobs.80000hours.org/?refinementList%5Btags_location_80k%5D%5B0%5D=Boston%20metro%20area&refinementList%5Btags_location_80k%5D%5B1%5D=Remote%2C%20USA`
**Last run:** 2026-02-22
**Auth required:** No
**Scraping method:** Puppeteer JS — Algolia API query (faster and more complete than DOM scraping)

---

## Overview

Curated job board from the 80,000 Hours organization, focused on high-impact careers (AI safety, biosecurity, climate, global health). All roles are handpicked. Low volume (~89 Boston + Remote USA roles as of 2026-02-22), high curation, minimal noise. Heavily weighted toward AI safety & policy and biosecurity; climate has only ~8 roles but they are high quality.

Individual job pages link directly to the employer's ATS — no side panel navigation needed.

---

## URL & Filter Parameters

URL-based filters use Algolia facet syntax. These set the initial state of the UI but the Algolia API query below is faster and more complete.

```
refinementList[tags_location_80k][0]=Boston metro area
refinementList[tags_location_80k][1]=Remote, USA
```

**Algolia filter syntax (for API queries):**
- Location: `tags_location_80k:"Boston metro area"`, `"Remote, USA"`, `"Remote, Global"`
- Area: `tags_area:"AI safety & policy"`, `"Climate change"`, `"Biosecurity"`, etc.
- Role type: `tags_role_type:"Full-time"`, `"Internship"`, `"Fellowship"`
- Combine with ` OR ` / ` AND `

---

## Extraction

**Site type:** Nuxt.js SPA with Algolia InstantSearch backend. Job cards are `button.job-card` elements — they open a side panel but do NOT update the URL. Use the Algolia API directly rather than DOM scraping.

**Algolia credentials** — load from `.env` or `QUICK_REFERENCE_PATH` file before running:
```
ALGOLIA_APP_ID    (from .env)
ALGOLIA_API_KEY   (from .env)
ALGOLIA_INDEX     (from .env, default: jobs_prod)
```

If credentials have rotated, retrieve current values from browser DevTools (Network tab) while on the 80k Hours board — look for requests to `algolia.net`.

**API query via Puppeteer fetch** (not Python/bash — egress proxy blocks Algolia from the VM):
```javascript
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
    attributesToRetrieve: [
      'title', 'company_name', 'url_external', 'tags_location_80k',
      'tags_area', 'tags_role_type', 'tags_experience', 'posted_at', 'description_short'
    ]
  })
}).then(r => r.json()).then(d => { window._80kData = d; });
// Then read: window._80kData.hits
```

**Key fields per hit:** `title`, `company_name`, `url_external` (direct ATS link), `tags_area`, `tags_location_80k`, `tags_role_type`, `posted_at` (Unix timestamp), `description_short` (HTML snippet).

---

## Pre-screening Filters

With ~70 full-time roles for Boston + Remote USA, pre-screening is quick but worthwhile.

**Exclude role types:** Internship, Fellowship, Part-time, Other

**Exclude title keywords:** software engineer, data engineer, ml engineer, research scientist, research associate, research assistant, postdoctoral, analyst, recruiter, video producer, undercover investigator, data scientist

**Include title keywords:** program manager, program director, program lead, head of operations, operations manager, director, senior manager, principal, technical program, policy lead, ai policy, research lead, senior director

---

## Known Issues & Quirks

- **Algolia credentials are public read-only keys** but may rotate. Always check if they still work before querying; retrieve fresh values from DevTools if not.
- **Egress proxy blocks Algolia from bash/Python.** Use `puppeteer_evaluate` with `fetch()` instead.
- **Duplicate listings.** Active Site (panoplialabs) has posted the same "Head of Operations" role twice with different object IDs. Deduplicate by title + company.
- **Anthropic has 45 listings** in the full board — mostly SF-based. Filter by location facet to avoid noise.
- **Board heavily skews AI safety & policy (426 roles), biosecurity (64), global health (75).** Climate has ~8 roles — small but worth reviewing every run.

---

## Recommended Cadence

Monthly. Small board, high curation, minimal overlap with other boards.
