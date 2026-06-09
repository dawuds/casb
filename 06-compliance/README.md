# 06-compliance

Regulator and standards mapping. **Capability → policy → control → evidence** traceability.

> **Material on regulatory clauses is illustrative; not legal/regulatory advice.** Confirm against gazetted text of each instrument and your assurance lead.

## Contents (framework pages — v0.0)

| File | What it contains |
|---|---|
| [`control-mapping-framework.md`](control-mapping-framework.md) | The mapping methodology — three-layer Control → Policy → Capability model; schema for control entries; what counts as evidence; common mapping patterns; audit-evidence package |
| [`iso-27017.md`](iso-27017.md) | ISO/IEC 27017:2015 framework page — key CLD.* clauses with CASB relevance, citation discipline, mapping pattern |
| [`iso-27018.md`](iso-27018.md) | ISO/IEC 27018:2019 framework page — A.* clauses for PII processor, what to require from the CASB vendor, sub-processor and cross-border transfer treatment |

## Sub-folders

| Folder | What's there |
|---|---|
| [`malaysia/`](malaysia/) | BNM RMiT framework page; PDPA 2010 + 2024 amendments + Cybersecurity Act 854 (2024) as material is built out |

## Status

This folder is at **framework-level only** in v0.0. The **full clause-by-clause mapping** per regulator is a separate deliverable per [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md). The framework pages document the approach — they do not constitute completed audit evidence.

## Planned scope (not yet built)

| Instrument | Jurisdiction | Status |
|---|---|---|
| BNM RMiT | MY | Framework page in [`malaysia/bnm-rmit.md`](malaysia/bnm-rmit.md); clause-by-clause map deferred |
| PDPA Malaysia (Act 709, 2010 + 2024 amendments — Act A1709) | MY | Not yet started |
| Cybersecurity Act 854 (2024) + NACSA sectoral | MY | Not yet started |
| MAS TRM + Notice on TRM | SG | Not yet started |
| HKMA SA-2 + General Principles | HK | Not yet started |
| PDPA Singapore | SG | Not yet started |
| GDPR + DORA | EU | Not yet started |
| ISO/IEC 27001:2022 | International | Not yet started (Annex A controls) |
| ISO/IEC 27017:2015 | International | Framework page; clause map deferred |
| ISO/IEC 27018:2019 | International | Framework page; clause map deferred |
| NIST CSF 2.0 | International | Not yet started (subcategory mapping) |
| NIST SP 800-53 Rev 5 | International | Not yet started |
| PCI DSS v4.0 | International | Not yet started |
| SOC 2 | International | Not yet started (attestation reference) |

## How to use the framework pages

Each policy in [`../03-policies/`](../03-policies/) has a "Control mappings" section pointing at the framework pages. Until clause-by-clause mappings are built out, treat the framework page references as `[VERIFY]` markers — the policy depends on a control in this standard, but the specific clause is not yet documented.

## Cross-references

- [`control-mapping-framework.md`](control-mapping-framework.md) — start here for the methodology
- [`../03-policies/`](../03-policies/) — the policy library, each policy references this folder for control mappings
- [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md) — outstanding items
- [`../../cyberkpis/`](../../cyberkpis/) (sibling repo) — KPI / KRI / KCI methodology
- [`../README.md`](../README.md) — repo-level navigation
