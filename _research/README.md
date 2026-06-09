# _research

Working notes, reading list, scratch. **Not citation-gated.** Multi-agent workflow output also lands here first.

> Nothing in `_research/` is considered authoritative. Promotion to `01-08` requires citation gate + three-lens review + schema compliance for the target folder.

## Contents

| Subfolder | What's there |
|---|---|
| [`2026-06-10-vendor-practitioner-reference/`](2026-06-10-vendor-practitioner-reference/) | First major workflow run — 5 vendor drafts, 15 three-lens reviews, 3 cross-vendor synthesis files, 5-specialist review of MDA v0, consolidated changes, MDA v1 playbook, changelog, repo-wide recommendations |
| [`2026-06-10-navigation-restructure/`](2026-06-10-navigation-restructure/) | Second workflow run — 4-persona structural review of repo navigation; consolidated restructure spec that drove the 03-policies refactor |

## What gets promoted from here

Per the [`promotion-log.md`](../00-meta/promotion-log.md) under `00-meta/`:

- **2026-06-10-vendor-practitioner-reference/** promoted:
  - `synthesis/capability-matrix.md` → `02-capabilities/capability-matrix.md`
  - `synthesis/policy-archetypes.md` → `03-policies/_archetype-index.md` (legacy navigation aid)
  - `synthesis/failure-modes.md` → `08-failure-modes/_failure-modes.md`
  - `microsoft-defender-for-cloud-apps-practitioner-playbook-v1.md` → `04-vendors/microsoft-defender-for-cloud-apps.md`
  - `raw/{netskope,zscaler,palo-alto-prisma-access,skyhigh-security}.md` → `04-vendors/<vendor>.md` (as drafts)

- **2026-06-10-navigation-restructure/** drove:
  - The README rewrite
  - The 03-policies/ control-family layout
  - The 19 new per-policy files

## Promotion criteria

For a file in `_research/` to be promoted to `01-08`:

1. Citation gate — vendor claims corroborated, regulator claims clause-cited, analyst material labelled as opinion
2. Three-lens review (Architect / Product / Compliance) completed
3. Schema compliance for the target folder
4. Three-lens sign-off block recorded on the promoted page

See [`../CLAUDE.md`](../CLAUDE.md) for the workflow and editorial floor.

## What does NOT belong in `_research/`

- Vendor-confidential or NDA material
- Client engagement data
- Anything with sensitive credentials / API tokens

Even though `_research/` is not citation-gated, the [`../CLAUDE.md`](../CLAUDE.md) sanitisation rules still apply.

## Cross-references

- [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md) — what was promoted and what is outstanding
- [`../00-meta/three-lens-review.md`](../00-meta/three-lens-review.md) — the review pattern that gates promotion
- [`../README.md`](../README.md) — repo-level navigation
