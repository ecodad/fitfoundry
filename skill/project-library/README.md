# Project Library

A modular bank of your technical projects and accomplishments, stored **one card per
project**, that LaunchKit draws from when drafting a tailored résumé and cover letter.

## Why this exists

Without a project library, every accomplishment that lands in a draft is trapped inside
whichever base résumé happened to be paired to the job. If you apply across several role
types (e.g., hardware engineering, technical product, research), that forces you to
maintain a separate résumé per niche.

The project library decouples **content** (what you did) from **format** (how a given
base résumé frames it). Each project is tagged, so LaunchKit can *select* the best-fitting
projects for a specific posting — and reframe the same project differently depending on
whether the role leans hardware, product, or research. One project, many angles, no
duplicate résumés.

It also makes drafts **truthful by construction**: LaunchKit is instructed to ground every
résumé bullet and cover-letter claim in a real card. If it isn't in a card or your profile,
it doesn't go in the draft.

## How to use it

1. Copy `_TEMPLATE.md` once per project into your **personal** library folder (the
   gitignored `my-projects/` folder — see Privacy below), renaming it after the project.
2. Fill in the front-matter tags and the bullets. Aim for **6–9 projects total** — enough
   coverage across your role types without noise.
3. Point LaunchKit at the folder (via `.env`, once the LaunchKit wiring lands) and run it
   normally. It reads the cards, scores each against the job, and drafts from the top matches.

## Card format

Each card has YAML front-matter (used for machine selection) and a short body (used for
drafting). See `_TEMPLATE.md` for the blank form and `EXAMPLE-modular-test-fixture.md` for
a filled one.

Key fields:

- **`domains`** — the sectors/problem-spaces the project belongs to. Drives coarse matching.
- **`skills`** — concrete skills/tools demonstrated. Drives fine matching against a posting's
  requirements.
- **Bullets at three lengths** — `short` (one clause), `medium` (one line with a metric),
  `long` (full STAR-style). LaunchKit picks the length that fits the target résumé's density.
- **`one_liner`** — the cover-letter-ready framing of the project's headline result.

## Tagging vocabulary (keep it consistent)

Consistent tags are what make selection work. Reuse these where they fit and extend the
list as needed — but avoid one-off synonyms (`ml` vs `machine-learning` vs `AI`; pick one).

**Suggested `domains`:** `hardware`, `manufacturing`, `product`, `research`, `climate-tech`,
`ai-ml`, `robotics`, `advanced-materials`, `systems-engineering`, `operations`,
`cross-functional-leadership`

**Suggested `skills`:** `DFM`, `GD&T`, `design-review`, `prototyping`, `test-automation`,
`data-analysis`, `python`, `roadmapping`, `stakeholder-management`, `vendor-management`,
`cost-reduction`, `reliability`, `simulation`, `technical-writing`

Every bullet should carry an **action + scope + quantified result** wherever a real number
exists. Never invent a metric — if you don't have one, describe the outcome qualitatively.

## Privacy

Project cards describe real work and may include employer names, program details, and
metrics. **Your filled cards are personal data and must not be committed to the FitFoundry
skill repo.** They live in the gitignored `my-projects/` folder.

Only the boilerplate in this `skill/project-library/` folder (this README, `_TEMPLATE.md`,
and the clearly-marked fictional `EXAMPLE-*.md`) is committed — it ships with the skill for
everyone. Nothing in it describes your real work.
