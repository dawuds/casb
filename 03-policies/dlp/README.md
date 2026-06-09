# 03-policies/dlp

Policies that **detect** and **act on** regulated data in motion (upload / download / paste / share) or at rest (existing files in SaaS storage).

## Contents

| File | What it does |
|---|---|
| [`inline-upload-block-regulated-data.md`](inline-upload-block-regulated-data.md) | Pre-upload block of files containing PCI / PII / source / secrets — proxy-mode prevention |
| [`api-data-at-rest-quarantine.md`](api-data-at-rest-quarantine.md) | API-mode scan + quarantine of regulated content already in sanctioned SaaS storage |
| [`external-share-link-quarantine.md`](external-share-link-quarantine.md) | Two-stage notify-then-quarantine for externally-shared files with no recent activity |
| [`auto-label-pci-data.md`](auto-label-pci-data.md) | Auto-apply Purview sensitivity label on files containing PCI cardholder data |

## The prevent-vs-detect split

- **Inline / proxy mode** (`inline-upload-block-regulated-data`) prevents the data from landing
- **API mode** (`api-data-at-rest-quarantine`, `external-share-link-quarantine`, `auto-label-pci-data`) detects and remediates after the fact

See [`../../08-failure-modes/api-mode-is-not-prevention.md`](../../08-failure-modes/api-mode-is-not-prevention.md) for the structural limitation of detect-and-remediate.

## Typical deployment cadence

- **Day 30:** `inline-upload-block-regulated-data`, `api-data-at-rest-quarantine`, `external-share-link-quarantine`, `auto-label-pci-data` — all CAAC- or API-dependent

## Cross-references

- [`../README.md`](../README.md) — policy-library overview
- [`../../02-capabilities/capability-matrix.md`](../../02-capabilities/capability-matrix.md) — DLP capabilities
- [`../../05-architecture/proxy-only-pattern.md`](../../05-architecture/proxy-only-pattern.md), [`../../05-architecture/api-only-pattern.md`](../../05-architecture/api-only-pattern.md), [`../../05-architecture/hybrid-pattern.md`](../../05-architecture/hybrid-pattern.md)
- [`../../08-failure-modes/encrypted-upload-bypass.md`](../../08-failure-modes/encrypted-upload-bypass.md) — defeated-by class
- [`../../08-failure-modes/api-mode-is-not-prevention.md`](../../08-failure-modes/api-mode-is-not-prevention.md)
- [`../../06-compliance/control-mapping-framework.md`](../../06-compliance/control-mapping-framework.md) — regulatory mappings (PCI DSS Req. 3 + 4; BNM RMiT data leakage; ISO 27017 CLD.12.4.5)
