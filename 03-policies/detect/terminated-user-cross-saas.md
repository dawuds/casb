# Terminated user — cross-SaaS activity

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Required capabilities: [Compromised-account detection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector. Entra-as-IdP for the deletion signal.
> MDA playbook reference: [Policy 10](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Terminated-user activity in connected SaaS).

## Purpose

Detect cross-SaaS account activity after a user's Entra account is deleted. Catches the offboarding-gap class — terminated user retains a local account in a connected SaaS under a different email format. Counters MITRE ATT&CK `T1078.004 Cloud Accounts` (persistence), `T1136 Create Account` (where the user created a SaaS-local account before leaving), `T1098.001 Additional Cloud Credentials` (pre-termination credential additions), `T1556` (MFA-method additions pre-termination).

## Action

- Primary: **alert + manual revoke** per app (no auto-suspend for non-Entra accounts — Entra suspend cannot disable a non-Entra local account)
- Notify identity admin + SOC

## Scope

- **Users:** all terminated employees (Entra-deletion event is the trigger)
- **Apps:** all connected SaaS where activity logs surface user identifiers (SharePoint, OneDrive, Salesforce, AWS, GitHub, ServiceNow, Workday, etc.)
- **Device posture:** N/A
- **Network position:** any
- **Traffic:** activity-log events with user identifier

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Anomaly detection policy → Activity performed by terminated user (built-in; edit, do not create) | Scope = all users; Per-app governance action = Notify admin + manual revoke; Severity = High; Alert to identity admin + SOC | API-mode; built-in anomaly; requires Entra-as-IdP for the deletion signal; correlation depends on shared identifier (usually email) between Entra deleted-user and SaaS-app account | **Correlation depends on exact-match email between Entra deleted-user and SaaS-app account.** If Entra UPN = `user.surname@corp.com` but Salesforce account = `surname.user@corp.com`, no match, no alert. Three additional offboarding sub-cases NOT covered by this built-in: (a) service-principal credentials / PATs / API keys added pre-termination — check Entra Audit `AppRoleAssignment` and `Add service principal credentials` events; (b) MFA-method additions / backup-email changes (T1556) — Entra Audit `User registered alternative authentication device`; (c) accounts created by terminated user in connected SaaS (T1136) — per-SaaS admin audit, not in MDA |
| Netskope | `[unverified]` — Netskope terminated-user correlation | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security identity correlation | | | |
| Skyhigh | `[unverified]` — Skyhigh terminated-user detection | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + offboarding](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.9.x access control overlay; CLD.8.1.5 removal of assets](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation)
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions), `PR.AA-04` (identities removed when no longer needed), `DE.CM-09` `[VERIFY]`

## False-positive risk

- Legitimate guest-account scenarios (user retains a vendor-side account legitimately due to ongoing engagement)
- Service-account UPN that resembles a former user

## Operational cost

- **Exception-handling load:** low — terminated-user events are clearly defined
- **Triage load:** low to medium — manual revoke per SaaS adds operational time
- **End-user friction:** N/A (user is terminated)

## Privacy / data-protection considerations

- Cross-SaaS activity correlation involves processing of former-employee data
- Lawful basis: post-employment data handling under PDPA / GDPR — document retention and deletion treatment

## Coverage gaps

- Non-email-matched accounts (different identifier format between Entra and SaaS local)
- Service-principal abuse — pre-departure credential additions; T1098.001 not covered by this built-in
- MFA-method additions before departure (T1556) — separate Entra Audit-log query required
- Personal-account exfil before departure — invisible to this control
- See [OAuth blind spot](../../08-failure-modes/oauth-blind-spot.md) for the service-principal class

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
