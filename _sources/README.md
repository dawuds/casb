# _sources

Primary-source library. **One file per source.**

## Status

In development. At v0.0, this folder is scaffolded but not populated. Primary-source citation is referenced inline in policy / vendor / compliance files using the `[VERIFY]` marker until a source-file entry exists here.

## File-per-source rule

When populated, each source — vendor architecture doc, regulator publication, ISO clause, NIST SP, analyst report, public incident disclosure, independent test report, customer reference — gets its own file. File names: `kebab-case.md`, namespaced where useful (`netskope-architecture-overview.md`, `bnm-rmit-cloud-risk-clauses.md`, `iso-27017-controls.md`).

## Mandatory metadata (top of each file)

```
- Source type: vendor doc / regulator / standard / analyst / incident / independent test / customer reference
- Vendor-controlled? yes / no
- Title:
- Author / publisher:
- Publication date:
- Version:
- URL:
- Access date:
- Local copy: (path under `../references/` if mirrored)
- Cited from: (links to synthesis pages that cite this source)
```

## What does NOT belong here

- Vendor blog posts that re-summarise their own product pages — link to the product page instead
- Secondary analyst summaries written *by* vendors of analyst reports — these are marketing
- Anything behind an NDA

## Cross-references

- [`../CLAUDE.md`](../CLAUDE.md) — citation discipline (non-negotiable section)
- [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md) — outstanding items, including `_sources/` population
- [`../README.md`](../README.md) — repo-level navigation
