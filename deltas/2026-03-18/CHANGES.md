# Delta: 2026-03-18

Apply this delta using the instructions in `DELTA-APPLY.md` at the repo root.

---

## New Site Files

Five new site files are included in `sites/`. These boards did not exist in the previous release.

| Board # | File | Method | Notes |
|---------|------|--------|-------|
| 16 | `sites/Harvard.md` | Puppeteer API (SmartRecruiters public API) | No auth required; UI broken, use API directly |
| 17 | `sites/Autodesk.md` | Puppeteer JS (Workday CXS POST API) | High volume — filter aggressively by location |
| 18 | `sites/Markforged.md` | Puppeteer JS (Greenhouse public API) | markforged.com unreachable via Puppeteer; use Greenhouse API URL |
| 19 | `sites/PTC.md` | Puppeteer JS (Workday CXS POST API) | Boston HQ — high-value local board |
| 20 | `sites/DesktopMetal.md` | N/A | ⚠️ ON HOLD — domain parked as of 2026-03-17; company may be defunct |

---

## Updated Site Files

Two existing site files have substantive changes. These files replace the previous versions; see merge rules in `DELTA-APPLY.md` for how to handle your personal `Last run` date.

### `sites/Climatebase.md`

- CSS container class history added: class changes on site rebuilds. Both the 2026-02-22 class (`sc-3d1ac256-2`) and 2026-03-17 class (`sc-977ff104-2`) are documented. Dynamic detection snippet added as a reliable fallback.
- Card selector updated: `[class*="Card_Information"]` added as a stable fallback alongside the hardcoded class.
- **New critical warning:** ClimateBase individual job detail pages (`climatebase.org/job/[id]`) are Cloudflare-blocked via Puppeteer. Full descriptions must be retrieved via Claude in Chrome after the initial card scrape and scoring pass.
- Run history section added for 2026-03-17 with notes on proxy mis-redirects, modal dismiss selector, and Boston In-Person pass returning 0 unique results.

### `sites/Indeed.md`

- **Critical workflow change:** `get_resume()` consistently hangs in Cowork sessions and should not be called. Resume context should be read from the `resumes/` folder in the workspace instead.
- Connector validation step updated: skip `get_resume()` and validate by running the first `search_jobs` call instead.

---

## Updated Index File

`JOB-BOARD-SITE-NOTES.md` has been updated with:

- New rows for boards 16–20 in the Secondary — Puppeteer Boards table.
- Formlabs method note updated: apply flow routes through Sensata Workday.
- New note at the bottom of General Browser Scraping Notes covering the ClimateBase Cloudflare limitation for detail pages.

All `Last run` values in the delta version are set to `Never`. When merging, preserve any dates you already have in your copy — see `DELTA-APPLY.md`.
