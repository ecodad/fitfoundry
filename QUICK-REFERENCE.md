# Quick Reference — [Your Name]

Personal settings file for FitFoundry. Copy this file, rename it (e.g., `MyName-QUICK-REFERENCE.md`), and set `QUICK_REFERENCE_PATH` in your `.env` to point to it. Do not commit your personal copy to the repository.

The agent reads this file at the start of every session to calibrate scoring and apply your search preferences.

---

## Candidate Profile

- **Name:** [Your name]
- **Location:** [Your city / region; note if open to remote and what scope — US only, global, etc.]
- **Default search location:** [City, State used as the location parameter in job board searches — e.g., `Boston, MA`]
- **Background:** [2–3 sentences on your experience — function, industry, years, notable employers or projects]
- **Target sectors:** [e.g., Climate tech, AI/ML, Advanced manufacturing, Healthcare]
- **Goal:** [One sentence on what you are optimizing for in your next role]
- **Seniority:** [e.g., Senior IC, Manager, Director, VP]
- **Not a fit:** [Hard constraints — e.g., clearance-required roles, pure software engineering, roles below a certain salary threshold, specific geographies]
- **Full profile:** [`MY-PROFILE.md` or your custom profile filename]

---

## Scoring Calibration

Adjust these thresholds to match your current situation. If you are actively job hunting and want more options, lower the individual file threshold to 5. If you are passive and highly selective, raise it to 7.

| Score | Meaning | Action |
|-------|---------|--------|
| 9–10 | Dream job — near-perfect alignment | Individual file |
| 7–8 | Strong fit — worth pursuing | Individual file |
| 6 | Partial fit — one significant gap | Individual file |
| 5 | Partial fit — multiple gaps | Master list only |
| 4 | Marginal | Master list only |
| ≤ 3 | Poor fit | Skip |

---

## Credentials

```
ALGOLIA_APP_ID=[your value — retrieve from DevTools on jobs.80000hours.org]
ALGOLIA_API_KEY=[your value]
ALGOLIA_INDEX=jobs_prod
```

---

## Board-Specific Search Settings

### LinkedIn
- **Keyword(s):** [e.g., `technical program manager` — keep to one keyword; LinkedIn matches on full body text]
- **Pages:** [Recommended: 1–5 for freshest results]
- **Filters:** Past 30 days, [your seniority levels], Full-time, [your target location]

### Wellfound
- **Industries:** [e.g., `Climatetech`, `ArtificialIntelligence`, `Robotics`]
- **Roles:** [e.g., `operations`, `product-manager`]

### 80,000 Hours
- **Locations:** [e.g., Boston metro area, Remote (USA)]
- **Exclude role types:** [e.g., Internship, Fellowship, Part-time]

### Indeed
- **Keywords:** [e.g., `technical program manager`, `operations program manager`, `senior program manager` — run as separate searches, results are deduplicated]
- **Location:** [City, State — e.g., `Boston, MA`; defaults to Default search location above if blank]
- **Employment type:** `fulltime`

### Dice
- **Keywords:** [e.g., `technical program manager`, `operations program manager`, `senior program manager` — run as separate searches, results are deduplicated]
- **Location:** [City, State — e.g., `Boston, MA`; defaults to Default search location above if blank]
- **Employment type:** `FULLTIME`
- **Employer type filter:** `Direct Hire` (set to blank to include staffing firm postings — they will be flagged either way)

---

## Notes

[Use this section for anything else the agent should know — boards to skip this cycle, companies you've already applied to, active applications to avoid duplicating, etc.]
