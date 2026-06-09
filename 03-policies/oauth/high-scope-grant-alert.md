# High-scope OAuth grant alert

> Status: v0.0 — MDA column from playbook v1 (Policy 2b — App Governance Predictive Risk + OAuth threshold, Day 30); other vendors `[unverified]`.
> Required capabilities: [OAuth-app discovery + Anomalous-activity detection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector + App Governance add-on (MDA-specific) or vendor equivalent.
> MDA playbook reference: [Policy 2b](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Day 30 OAuth behavioural detection).

## Purpose

Alert on OAuth grants with high-permission scope (Mail.ReadWrite.All, Files.ReadWrite.All, Sites.FullControl.All, User.ReadWrite.All) regardless of publisher reputation. Covers the behavioural post-consent class — a legitimately-consented app suddenly expands scope or behaves anomalously. Counters MITRE ATT&CK `T1098.001 Additional Cloud Credentials` (Detect), `T1550.001` (Detect), `T1136.003 Cloud Account creation via service-principal` (Detect).

## Action

- Primary: **alert** in notify-only mode for 4 weeks; flip to auto-revoke only on highest-confidence signal classes after FP tuning

## Scope

- **Users:** all OAuth-consenting users + admins granting app-only permissions
- **Apps:** all OAuth-grantable SaaS — M365, Workspace, Salesforce
- **Device posture:** N/A
- **Network position:** N/A
- **Traffic:** consent + grant + service-principal-credential addition events

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → App governance → Predictive risk policies + OAuth threshold policies | Enable Predictive Risk in notify-only mode for 4 weeks; tune; enable auto-revoke only on highest-confidence signal classes; OAuth threshold = count of grants per app per window + scope-expansion alerts + geographic anomalies | App Governance is a paid add-on (verify SKU); Predictive Risk shipped 2025 H2; behavioural class is post-consent | Predictive Risk false-positives on legitimate burst usage (DR rehearsals, content-migration runs) — per-app suppression workflow required before auto-revoke. First-time abuse of a never-before-anomalous app — model has nothing to compare against. Power Platform service-principal consents under-surfaced in App Governance |
| Netskope | `[unverified]` — Netskope OAuth governance with anomaly | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security OAuth threshold | | | |
| Skyhigh | `[unverified]` — Skyhigh OAuth anomaly | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security OAuth | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + third-party risk](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.9.x access control, CLD.12.4.5](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05`, `DE.CM-06`, `DE.AE-02` `[VERIFY]`

## False-positive risk

- Legitimate burst usage (DR rehearsals, content-migration runs)
- New ISV integrations during onboarding — high-permission grant is part of legitimate setup
- Geographic anomaly on a service principal that legitimately runs from multiple regions

## Operational cost

- **Exception-handling load:** medium — per-app suppression workflow
- **Triage load:** medium — behavioural alerts need cross-reference with business operations
- **End-user friction:** N/A for service-principal class; per-user notification if user-consent-driven anomaly

## Privacy / data-protection considerations

- Workload-identity activity logged = no direct workforce-monitoring concern; but high-permission scopes may give the app access to regulated PII
- Document workload-identity governance under PDPA / GDPR processor treatment

## Coverage gaps

- First-time abuse of a previously-clean app — model has no baseline
- App Governance covers some application-permission anomalies for M365 Graph; coverage on non-Graph scopes varies — verify per-scope
- Power Platform service-principals — Power Platform Admin Center DLP required as companion
- [OAuth blind spot](../../08-failure-modes/oauth-blind-spot.md) — the full ~70% of OAuth abuse class is not solved by Predictive Risk alone

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
