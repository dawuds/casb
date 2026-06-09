# Geo-residency block

> Status: v0.0 — cross-vendor pattern; MDA implementation via location-filter + sensitivity-label filter; other vendors `[unverified]`.
> Required capabilities: [Data-residency / geo-policy enforcement + Inline DLP](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Hybrid (inline for block; API for residency-aware classification).
> MDA playbook reference: implicit via Location filter on session/access/activity policies; not a numbered Day 1/30/90 policy in MDA v1.

## Purpose

Block upload to (or download from) data regions outside the firm's permitted geographic perimeter for tagged regulated data. Enforces data-residency requirements imposed by regulator or contract — e.g. Malaysian-tagged data not allowed to leave MY-resident regions; EU-tagged data subject to GDPR Chapter V transfer rules. Reduces cross-border-transfer compliance exposure.

## Action

- Primary: **block** upload/download to/from non-permitted region
- Alternative: **monitor** + alert (audit mode while baselining)

## Scope

- **Users:** all (or scoped to teams handling residency-tagged data — Finance, HR, KYC, Legal)
- **Apps:** sanctioned SaaS with region-aware storage (M365 multi-geo; AWS S3 with bucket-region tag; Google Workspace with data-location settings)
- **Device posture:** managed and BYOD (residency is data-attribute-driven, not device-driven)
- **Network position:** any
- **Traffic:** upload + download involving residency-tagged data

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Session/Activity policy with Location filter (country-level). Plus Purview sensitivity labels carrying residency metadata | Location filter = country whitelist; File filter = Sensitivity label ∈ {residency-tagged}; Action = Block | Country-level only — no sub-country granularity. M365 multi-geo "supported only for OneDrive" per docs; SharePoint multi-geo events not fully captured | Location filter on session policy fires for the user's *current location*, not the file's *origin region*. To enforce file-region-residency you need Purview Information Protection rules at the data layer in parallel — MDA enforces user-action-by-location, not data-by-region |
| Netskope | `[unverified]` — Netskope geo-fencing + DLP integration | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access geo-policy + SaaS Security | | | |
| Skyhigh | `[unverified]` — Skyhigh multi-mode geo control | | | |
| Zscaler | `[unverified]` — ZIA geo filtering + CASB | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT cloud services + cross-border data transfer](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- PDPA MY 2024 (Act A1709): cross-border-transfer regime `[VERIFY against gazetted text]`
- ISO 27017 control(s): [CLD.6.3.1 shared responsibility, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 A.10, A.11.1: use limitation, sub-processor disclosure relevant
- GDPR Chapter V (international transfers) for EU-tagged data
- NIST CSF 2.0 subcategory(ies): `PR.DS-01` (data-at-rest), `PR.DS-02` (data-in-transit) `[VERIFY]`

## False-positive risk

- "Absence of clearly defined location" may identify risky activities — null-location events caught by Location filter without an `is set` companion filter. Mitigate with explicit `Location is set` clause
- Roaming employees legitimately accessing data from outside permitted region — name an exception group with documented justification
- VPN egress points geographically distant from user — may register as wrong country

## Operational cost

- **Exception-handling load:** high during business-travel periods; lower steady-state
- **Triage load:** medium — region anomalies often correlate with legitimate travel
- **End-user friction:** medium for roaming users; high for cross-border collaboration

## Privacy / data-protection considerations

- Location enrichment uses IP + geo-lookup — workforce-location data; document under PDPA / GDPR Art. 88 workforce-monitoring posture
- Country-level location accuracy is Microsoft's claim; VPN endpoints, cloud-provider IPs, residential proxies all routinely produce wrong country

## Coverage gaps

- Tenant primary-data region (where the CASB itself stores activity logs / DLP snippets) is separate from this policy's geo-block — the CASB log data residency is determined at tenant provisioning. For MDA: EU / US / UK / AU / Japan East (verify Trust Center on day of attestation)
- API-mode delays (Power BI / Dynamics 24-72hr) — region-block is post-event for non-inline traffic
- File-region-residency requires data-layer enforcement (Purview / GCP / AWS bucket-region rules) — MDA alone is user-action-by-location
- [Encrypted-upload bypass](../../08-failure-modes/encrypted-upload-bypass.md) — encrypted-by-user content cannot be tagged by content scan

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
