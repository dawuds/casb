# SSPM — tenant misconfiguration drift

> Status: v0.0 — adjacent to core CASB capability; CASB-light SSPM coverage; specialist SSPM tools have deeper coverage.
> Required capabilities: [API-mode + configuration-assessment integration](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: relates to App Governance posture features; not a numbered MDA Day 1/30/90 policy.

## Purpose

Continuously assess the security configuration posture of sanctioned SaaS tenants (Salesforce sharing settings, M365 attack-surface configuration, ServiceNow admin settings, etc.) against a documented posture-rule library. Detect drift from baseline. **Important: CASB does this lightly — for deep coverage, dedicated SSPM tools (Adaptive Shield, AppOmni, Obsidian, Wing) are the right tier.**

## Action

- Primary: **alert** on configuration drift + propose remediation
- Compensating: tracked in posture-management runbook with named owners per tenant

## Scope

- **Users:** N/A (configuration-attribute scan)
- **Apps:** sanctioned SaaS with configuration-assessment connectors — typical CASB coverage: M365, Salesforce, ServiceNow, GitHub, Okta, Box, Dropbox, Zendesk (vendor-dependent)
- **Device posture:** N/A
- **Network position:** N/A
- **Traffic:** at-rest configuration scan

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → App governance → Security configuration assessment (subset of connected apps) | Apps covered = M365, Salesforce, ServiceNow, GitHub, Okta, Box, Dropbox, Zendesk (per current docs); Posture rules = Microsoft Secure Score-derived + custom | Depth below SSPM specialists; coverage breadth limited; remediation actions limited | CASB SSPM is **shallow** — for tenant-deep posture (Salesforce profile/permission set configuration, GitHub branch protection, M365 attack-surface), specialist SSPM is required. Drift-detection cadence may be daily at best, not real-time |
| Netskope | `[unverified]` — Netskope SSPM module (separate licence) | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security posture | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB + posture | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security posture | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT cloud services + third-party risk](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.6.3.1 shared responsibility, CLD.12.1.5 administrator's operational security](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `ID.AM-04`, `PR.IP-01` (baseline configuration), `DE.CM-08` (vulnerability scans) `[VERIFY]`

## False-positive risk

- Legitimate config differences across tenants in multi-tenant federation
- Recently-applied admin changes pending baseline update
- Vendor-side default changes propagating before posture rule update

## Operational cost

- **Exception-handling load:** medium — every config drift is either a baseline update or a remediation ticket
- **Triage load:** low — posture alerts are not real-time SOC events; admin-team review cadence
- **End-user friction:** N/A

## Privacy / data-protection considerations

- Configuration metadata typically not personal data; access control on the SSPM scan results still required
- For PII-handling SaaS, drift in sharing-controls or external-access settings is a privacy-grade event

## Coverage gaps

- Apps not in the CASB's SSPM coverage list — many smaller SaaS unsupported
- Configuration-detection lag — drift may exist for hours-to-days before scan
- Remediation depth — most CASB SSPM is alert-only, not auto-remediate. Specialist SSPM tools (Adaptive Shield, AppOmni, Obsidian, Wing) have deeper remediation; verify before procuring on CASB-only coverage assumption

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
