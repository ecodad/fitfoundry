# Wellfound (AngelList Talent)

**Board #:** 9
**URL:** `https://wellfound.com/jobs?role=operations&role=product-manager&industry=Climatetech&industry=ArtificialIntelligence&jobType=fulltime`
**Last run:** Never
**Auth required:** Yes — Wellfound account login via Claude in Chrome
**Scraping method:** Claude in Chrome — accessibility tree (`read_page`). Puppeteer is blocked by Cloudflare CAPTCHA.

---

## Overview

Startup-focused job board with strong climate tech and AI filters. Backed by AngelList. Cannot be scraped with Puppeteer from the VM — direct navigation triggers a Cloudflare human verification slider that cannot be bypassed programmatically. Use Claude in Chrome with an active Wellfound session instead.

---

## URL & Filter Parameters

URL params apply server-side and combine freely.

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

---

## Extraction (Claude in Chrome)

**Site type:** React SPA — bot-protected (Cloudflare CAPTCHA on direct navigation). Authenticated browser sessions bypass the CAPTCHA.

Use `read_page` (accessibility tree) to extract job cards once the page loads. Card class names are hashed and may change — identify cards by structure (each has a heading for title, company name, location, and optionally salary).

```javascript
// Reference only — JS injection may also be blocked; use read_page if so
const cards = Array.from(document.querySelectorAll('[class*="styles_component"]'))
  .filter(el => el.querySelector('h2'));
```

---

## Known Issues & Quirks

- **Cloudflare CAPTCHA blocks all Puppeteer access.** Must use Claude in Chrome with an authenticated Wellfound session — the CAPTCHA does not appear for logged-in users.
- **Class names are hashed** and may change on site updates. Re-detect card selectors if extraction stops working.
- **Login requirement.** Full results require a Wellfound account. Anonymous access shows partial listings.

---

## Recommended Cadence

Monthly, via Claude in Chrome.
