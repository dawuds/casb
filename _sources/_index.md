# _sources

Primary-source library. **One file per source.** Mirror the `cyber-business/_sources/` and `cyberkpis/02-standards/` pattern.

## File-per-source rule

- Each source — vendor architecture doc, regulator publication, ISO clause, NIST SP, analyst report, public incident disclosure, independent test report, customer reference write-up — gets its own file.
- File name: `kebab-case.md`, namespaced where useful (`netskope-architecture-overview.md`, `bnm-rmit-cloud-risk-clauses.md`, `iso-27017-controls.md`).

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

## What does *not* belong here

- Vendor blog posts that re-summarise their own product pages — link to the product page instead.
- Secondary analyst summaries written *by* vendors of analyst reports — these are marketing.
- Anything behind an NDA.
