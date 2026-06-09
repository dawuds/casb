# API mode data-at-rest quarantine

> Status: v0.0 — MDA column from playbook v1; other vendors `[unverified]`.
> Required capabilities: [API-mode DLP for sanctioned SaaS](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector. No CAAC required.
> MDA playbook reference: [Policy 2](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Auto-label PCI as a related pattern; this policy is the quarantine variant).

## Purpose

Scan files already at rest in sanctioned SaaS storage, detect regulated content via DLP signatures, and quarantine matched files to an admin-owned location pending owner review. Catches what slipped past the inline upload gate, what arrived via sync clients, and what was uploaded before policies were enforcing. Counters MITRE ATT&CK `T1530` Detect leg.

## Action

- Primary: **quarantine** (move to admin-owned location) with owner notification
- Alternative: **alert + label-apply** without quarantine for less-sensitive classes

## Scope

- **Users:** all (file-owner attribution drives notification routing)
- **Apps:** sanctioned SaaS with API connectors (M365 OneDrive/SharePoint primary; Box, Dropbox, Google Drive secondary)
- **Device posture:** N/A — the file is in SaaS; device of upload is irrelevant at scan time
- **Network position:** N/A
- **Traffic:** at-rest scan (not transit)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Information protection → Create File policy | App filter = OneDrive + SharePoint (+ Box / Dropbox / GWS where API connector configured); Inspection method = Data Classification Service; SIT = regulated data class; Governance action = Put file in admin quarantine; Alert + notify owner with 14-day grace before second-stage quarantine | API-mode — post-upload scan, not real-time prevention. M365 latency near-real-time per Microsoft; Power BI / Dynamics 24-72hr per docs. File-policy ceiling = 200 per tenant (verify against current docs) | "Only the governance action of the first triggered policy is guaranteed to be applied" — policy ordering matters when multiple file policies could match. Externally-owned files in Box / Dropbox / Google Drive (where external user is the data owner under that platform's data model) are unscannable — only OneDrive reassigns ownership. Quarantine on a file under Microsoft Purview eDiscovery legal hold may be spoliation — Purview hold wins |
| Netskope | `[unverified]` — Netskope CASB API + Native DLP | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security API (formerly Aperture) | | | |
| Skyhigh | `[unverified]` — Skyhigh API mode with multi-mode DLP | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security API | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage prevention](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring of cloud services](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation)
- NIST CSF 2.0 subcategory(ies): `PR.DS-05` (data leak protection), `DE.CM-03` (personnel activity), `RS.MI-02` (incidents contained) `[VERIFY]`
- PCI DSS Req. 3 storage controls where PCI in scope

## False-positive risk

- DCS classifier hits on test fixtures, mock data, expired/cancelled cards, sample employee IDs
- Files legitimately containing regulated data in their named purpose (e.g. an HR template that references the SSN field structure)
- Format-shifting from cleaner formats to formats DCS extracts imperfectly

## Operational cost

- **Exception-handling load:** medium — owner-notification → reply → exception decision per quarantined file
- **Triage load:** medium — auto-quarantine reduces analyst load but exception-decisions remain
- **End-user friction:** medium — users get a quarantine notice; legitimate files retrieved via exception path

## Privacy / data-protection considerations

- Matching-content preview snippet stored in CASB metadata = regulated content (PCI / PII / etc.) — itself subject to storage controls under PCI DSS 3.4 / 3.5 and PDPA / GDPR
- API-mode scanning vs inline inspection — same content surface, same data-protection treatment
- DPIA scope includes both inline and API-mode DLP

## Coverage gaps

- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — exposure window between upload and scan completion
- [Encrypted-upload bypass](../../08-failure-modes/encrypted-upload-bypass.md) — encrypted-by-user content tagged `Password encrypted` / `Azure RMS encrypted` and skipped
- Externally-owned files in third-party SaaS (Box / Dropbox / GWS) where external user retains ownership — unscannable
- Files larger than DCS content-inspection limit — unscannable
- Pre-existing files in tenant before policy enabled — backfill scan required as separate operation

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
