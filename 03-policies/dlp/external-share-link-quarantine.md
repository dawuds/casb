# External share-link quarantine

> Status: v0.0 — MDA column from playbook v1; other vendors `[unverified]`.
> Required capabilities: [Share-link governance + API-mode DLP](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 7](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Quarantine externally-shared OneDrive files).

## Purpose

Identify files in sanctioned cloud storage that are shared externally and have not been accessed or modified within a defined window; two-stage notify-then-quarantine flow. Catches the sharing-drift class — old external shares accumulate over years, each a small leak. Counters MITRE ATT&CK `T1213 Data from Information Repositories` (long-tail cleanup).

## Action

- Primary (stage 1): **notify** the file owner with a 14-day grace period
- Primary (stage 2, 14 days later): **quarantine** (move to admin-owned location) on no-response

## Scope

- **Users:** all file owners
- **Apps:** OneDrive + SharePoint primary; Google Drive, Box, Dropbox where API connectors and quarantine actions are supported
- **Device posture:** N/A (at-rest scan)
- **Network position:** N/A
- **Traffic:** at-rest

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Information protection → Create File policy (two policies, 14 days apart) | **Stage 1:** App = OneDrive for Business; Access level = External OR Public-with-a-link OR Public; Last modified = >180 days; Governance action = Notify owner; severity = Medium. **Stage 2:** same filters, Governance action = Put file in admin quarantine; severity = High. Exclude folder allowlist (`/Shared Templates`, etc.) | API-mode; SharePoint multi-geo "supported only for OneDrive" per docs; SharePoint events not fully captured in multi-geo tenants | Single-step quarantine breaks links for internal users still referencing the file; owner-notification email often lands in junk. Always run notify-then-quarantine. **A quarantined file under Purview eDiscovery legal hold may be spoliation** — Purview hold wins, but operator may not know about hold at policy-design time. Consult Legal before policy design |
| Netskope | `[unverified]` — Netskope CASB API with share-link governance | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security API | | | |
| Skyhigh | `[unverified]` — multi-mode external-sharing controls | | | |
| Zscaler | `[unverified]` — SaaS Security API external-sharing | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage prevention](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring, CLD.8.1.5 removal of assets](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.5 (sub-processor disclosure where shared externally)
- NIST CSF 2.0 subcategory(ies): `PR.DS-05`, `RS.MI-02` `[VERIFY]`

## False-positive risk

- Stale-but-intentional shares (contract templates, public reference docs) — exclude via folder allowlist
- Files whose `Last modified` is touched by automated processes (BCM rehearsals, schema-update jobs) — may bypass the 180-day filter inadvertently
- Shares to known-trusted external domains that ought to persist (long-term vendor collaboration spaces) — name-based exception

## Operational cost

- **Exception-handling load:** medium — owner-replies → exception decisions during stage 1
- **Triage load:** low — fully automated through to quarantine
- **End-user friction:** medium — quarantined files surface as broken links; recovery via exception ticket

## Privacy / data-protection considerations

- Owner-attribution required for notification — workforce-monitoring-adjacent
- Files containing personal data of external recipients (partners, customers) — quarantine event may itself be a regulated data-handling action

## Coverage gaps

- [BYOD and unmanaged endpoint coverage gap](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — files shared via personal accounts on personal devices = invisible
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — share was active for the period before detection
- Files publicly shared with 'anyone' may "incorrectly show up as private" per OneDrive/SharePoint connector documentation — known fidelity gap
- External recipients who already downloaded the file before quarantine — no recall mechanism

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
