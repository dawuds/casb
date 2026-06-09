# OAuth post-consent cleanup and ban

> Status: v0.0 — MDA column lens-reviewed from playbook v1 (Policy 2a); other vendors `[unverified]`.
> Required capabilities: [OAuth-app discovery and governance](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 2a](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Classic OAuth apps page anomalies).

## Purpose

Catalogue delegated-permission OAuth grants made by users to third-party apps in M365 / Workspace / Salesforce; ban known-malicious and per-user-revoke confirmed-bad. Counters MITRE ATT&CK `T1528 Steal Application Access Token` (Detect + Respond, post-consent), `T1550.001 Application Access Token` (Detect on subsequent token use), `T1567` / `T1071.001` (Detect, post-event exfil).

**Reframing:** this is a **cleanup control, not a primary defence.** Primary control for T1528 is the Entra admin-consent workflow + verified-publisher restriction, configured upstream in Entra (not MDA).

## Action

- Primary: **manual ban** (tenant-wide kill) for known-malicious apps; **per-user revoke** for confirmed-bad-grant
- Anomaly detection: **alert** on Misleading OAuth app name / Misleading publisher name / Malicious OAuth app consent

## Scope

- **Users:** all users (delegated-grant scope)
- **Apps:** all OAuth apps with grants in M365 / Google Workspace / Salesforce
- **Device posture:** N/A
- **Network position:** N/A
- **Traffic:** consent events + subsequent token-use events

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → OAuth apps (or App governance if add-on enabled). Built-in anomaly policies under Cloud apps → Policies → Policy management → Threat detections | Day 1: manual review of every High-permission, low-community-use, unverified-publisher app; manually ban known-malicious; do NOT enable auto-revoke on built-in anomaly policy for at least 4 weeks while measuring FP rate; restrict user consent to verified-publisher apps at the Entra layer in tandem | API-mode; covers M365 / Workspace / Salesforce delegated grants only | **Banning is tenant-wide and immediate, propagating on next token validation (CAE-aware apps revoke faster).** No staged rollout. Practitioners ban a flagged app at 11pm, wake up to dozens of broken integrations. **Application (client-credentials) OAuth grants are NOT in this view** — service-principal abuse is the dominant cloud-persistence pattern; Policy 2 covers ~30% of OAuth-grant abuse; service-principal class needs App Governance Predictive Risk / Entra Workload Identities |
| Netskope | `[unverified]` — Netskope OAuth governance | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security OAuth policies | | | |
| Skyhigh | `[unverified]` — Skyhigh OAuth visibility | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security OAuth | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + third-party app risk](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.9.x access control, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions), `DE.CM-06` (external service-provider activity) `[VERIFY]`

## False-positive risk

- Legitimate-but-lookalike publisher names — "Microsoft" vs "Microsoft Corporation" misclassified
- New legitimate ISVs with low community-use count — flagged by uncommon-use anomaly
- Verified-publisher status drift — vendor recently verified but cache stale

## Operational cost

- **Exception-handling load:** medium — every legitimate ISV onboarding requires review
- **Triage load:** medium — anomaly alerts need cross-reference with business-approved integrations list
- **End-user friction:** ban = high impact (every consenting user loses access); revoke = low impact (one user)

## Privacy / data-protection considerations

- Consent events catalogue user identity + grant scope — workforce-monitoring data
- Storm-0558 / Storm-2372-class supply-chain compromise: an approved-by-business ISV could turn malicious; documented kill-switch posture required

## Coverage gaps

- [OAuth blind spot](../../08-failure-modes/oauth-blind-spot.md) — application (client-credentials / app-only) grants are the dominant cloud-persistence pattern (~70% of OAuth abuse) and NOT in this policy's scope
- Personal-account OAuth (user signs into a third-party app with personal Gmail / personal MSA) — Entra never sees it
- First-time abuse of a never-before-anomalous app — Predictive Risk model has nothing to compare against
- Power Platform service-principal consents — under-surfaced in App Governance; compensating control = Power Platform Admin Center DLP

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
