# B2B / partner-tenant exfiltration alert

> Status: v0.0 — MDA column from playbook v1 (Policy 6b — Day 30 addition); other vendors `[unverified]`.
> Required capabilities: [Anomalous-activity detection + B2B guest-user attribution](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 6b](../../04-vendors/microsoft-defender-for-cloud-apps.md) (B2B / partner-tenant exfiltration alert).

## Purpose

Detect external-domain guest-user file operations (download / share) against the tenant at volumes exceeding a threshold. Counters MITRE ATT&CK `T1199 Trusted Relationship` (Detect) and `T1213.002 SharePoint` (Detect). Catches Storm-0539 / Storm-2372-style consent-phishing-into-B2B attack chains.

## Action

- Primary: **alert + email** to identity admin and SOC

## Scope

- **Users:** external / guest users (UserType = Guest in Entra; or IsExternalUser = true in CASB telemetry)
- **Apps:** sanctioned SaaS where B2B sharing is enabled (SharePoint, OneDrive primary)
- **Device posture:** N/A (the user is external; their device is out of scope)
- **Network position:** N/A
- **Traffic:** download + share events by external users

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Create Activity policy | Activity type = Download file OR Share file; User type = External user; Threshold = baseline-derived; starting point 1,000 in 24h; Action = Alert + email to identity admin and SOC | API-mode; post-event; classification of user as External depends on the user's home-tenant claim | "External user" classification depends on the user's home tenant claim — mis-classification at federation time means the filter misses them. A compromised partner-tenant account behaving within threshold per session = slow-and-low B2B exfil (sub-threshold). Pair with Cross-Tenant Access Settings restricting partner scope |
| Netskope | `[unverified]` — Netskope external-user attribution + activity policy | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security external-user policies | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB external-user detection | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT third-party risk + DLP](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.6.3.1 shared responsibility, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation)
- NIST CSF 2.0 subcategory(ies): `DE.CM-06` (external service-provider activity monitored), `PR.AA-05` `[VERIFY]`

## False-positive risk

- Legitimate B2B partner collaboration projects with high-volume file exchange — name an exception group
- Misclassified guest users (set up as guest but functioning as member)

## Operational cost

- **Exception-handling load:** medium — partner-tenant projects need scoping
- **Triage load:** medium — alerts correlate with partner-tenant relationship inventory
- **End-user friction:** N/A (the user is external)

## Privacy / data-protection considerations

- External-user activity audit-log = third-party processing record
- Partner-tenant data flow tracked; document under PDPA / GDPR processor-controller treatment

## Coverage gaps

- Slow-and-low B2B exfil within threshold — bypass class
- Cross-Tenant Access Settings drift — partner-tenant scope changes without policy update
- B2B guest device claims come from home tenant — device-posture checks unreliable for external users
- Personal-account-as-guest scenarios

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
