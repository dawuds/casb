# Tenant restriction — corporate-only

> Status: v0.0 — partial (cross-vendor pattern; MDA implementation via CAAC + tenant restrictions); other vendors `[unverified]`.
> Required capabilities: [Risk-based session policy + Identity integration](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (IdP-mediated) + browser headers via SWG.
> MDA playbook reference: implicit (CAAC + Entra tenant restriction); not numbered separately in MDA Day 1/30/90.

## Purpose

Block sign-in to M365 / Google Workspace / Salesforce / similar SaaS using personal accounts or non-corporate tenants from corporate networks and managed devices. Stops the dominant 2024-2026 data-leak pattern of "user signs into ChatGPT (or any SaaS) with personal Gmail on a corporate device → exfiltrates corporate data to a personal account". Counters MITRE ATT&CK `T1078.004 Cloud Accounts` (personal-account variant) and the personal-tenant variant of `T1567`.

## Action

- Primary: **block** sign-in from non-corporate tenant
- Secondary: **monitor** + alert on attempted personal-tenant sign-in from corporate device

## Scope

- **Users:** all employees + contractors (during work hours / on corporate network / on managed device — scope decision)
- **Apps:** M365, Google Workspace, ChatGPT Enterprise, Slack, Salesforce, any SaaS supporting tenant-restriction headers
- **Device posture:** managed primarily; BYOD-mobile via MAM where supported
- **Network position:** corporate network + roaming via endpoint agent
- **Traffic:** sign-in flow

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Entra Conditional Access policy + SWG injection of tenant-restriction headers (`Restrict-Access-To-Tenants`, `Restrict-Access-Context`). MDA itself does not inject these headers — the SWG / proxy layer does | Header values = comma-separated allowed tenant IDs; CA policy = block sign-in from non-listed tenants | Tenant-restriction headers must be injected by the SWG / proxy in the user's outbound traffic — pure-MDA does not do this. Requires Global Secure Access (GSA) or a third-party SWG | Microsoft tenant-restriction-v2 supports per-tenant context; v1 was app-class-only. Verify which version your SWG injects. Personal account on personal device on personal network = entirely uncovered |
| Netskope | `[unverified]` — Netskope SWG can inject tenant-restriction headers; native CASB tagging supports policy | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access SWG injection supported | | | |
| Skyhigh | `[unverified]` — multi-mode injection | | | |
| Zscaler | `[unverified]` — ZIA URL filtering + header injection | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT identity + access management](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.9.x (Access Control overlay)](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions/authorisations) `[VERIFY]`

## False-positive risk

- Legitimate B2B guest access to partner-tenant apps — exempt by named partner tenant ID
- User accessing legitimate personal services that share the SaaS family (e.g. personal LinkedIn alongside LinkedIn for Business) — header may block all variants

## Operational cost

- **Exception-handling load:** low — exceptions are rare and well-defined (B2B partner integrations)
- **Triage load:** low — most blocks are silent at the IdP layer
- **End-user friction:** medium — users who routinely sign into personal accounts on work devices will see blocks immediately; pre-rollout communication critical

## Privacy / data-protection considerations

- The block does not inspect personal traffic content; it inspects only sign-in tenant claim — lower privacy footprint than SSL inspection
- Workforce-notice posture still applies: users must understand which sign-in patterns are blocked

## Coverage gaps

- Off-network personal-device personal-account = entirely uncovered
- SaaS apps that do not support tenant-restriction headers (most consumer apps) — only blockable via SWG URL filtering, not at the sign-in layer
- [BYOD and unmanaged endpoint coverage gap](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md)

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
