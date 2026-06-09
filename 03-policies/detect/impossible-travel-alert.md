# Impossible-travel alert

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Required capabilities: [Compromised-account detection (impossible travel)](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector. Entra-as-IdP for governance signal.
> MDA playbook reference: [Policy 4](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Impossible travel + activity from infrequent country).

## Purpose

Detect sign-in patterns that are geographically infeasible within the observed time window — credit-stuffing botnet / compromised-credential pattern. **Important:** the dominant 2024-2026 intrusion pattern (T1566.002 → T1528 → T1550.001 token-replay from residential-proxy infrastructure) produces zero impossible-travel signal. Treat this control as **naive credential-stuffing-only**.

## Action

- Primary: **alert-only** (do NOT auto-suspend)
- Pair with Entra ID Protection sign-in risk for higher-fidelity correlated signal

## Scope

- **Users:** all (exclude documented Frequent travellers group with named owner + review cadence)
- **Apps:** sign-in events to any connected app (Entra-mediated)
- **Device posture:** any
- **Network position:** any
- **Traffic:** sign-in events

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Anomaly detection policy → Impossible travel (built-in; edit, do not create) | Sensitivity slider = Low or Medium initially; Scope = all users; Exclude documented "Frequent travellers" group; Action = Alert only | API-mode; country-level granularity only; ML-based VPN / tenant-common-location suppression per Microsoft (independent FP rate not published) | **Country-level only — no alerts within the same country or between bordering countries.** KL ↔ SG ↔ JB produces no signal — material gap for MY FIs with cross-border insider scenarios. Defeated by T1550.001 token replay from residential-proxy infrastructure (modern intrusions produce zero signal). VPN-detection enrichment source may be silently unavailable — no documented operator notification for enrichment outage |
| Netskope | `[unverified]` — Netskope UEBA with impossible travel | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security anomaly | | | |
| Skyhigh | `[unverified]` — Skyhigh UEBA | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security UEBA | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT cybersecurity operations](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `DE.AE-02` (analyse event data), `DE.CM-07` (unauthorised personnel monitored) `[VERIFY]`

## False-positive risk

- VPN endpoints geographically distant from user — registers as wrong country
- Cloud-provider IPs (no physical location)
- Frequent business travellers — name an exception group with documented owner and review cadence

## Operational cost

- **Exception-handling load:** low — Frequent-travellers group updated quarterly
- **Triage load:** medium — alerts need correlation with sign-in risk; high FP rate during travel periods
- **End-user friction:** low if alert-only; high if auto-suspend (broken account during legitimate travel)

## Privacy / data-protection considerations

- Sign-in location data = workforce-location processing; PDPA / GDPR Art. 88 / equivalent treatment required
- Compensating-control framing: practitioners must understand this is naive-credential-stuffing-only, not modern-attack coverage

## Coverage gaps

- T1550.001 token replay from residential-proxy infrastructure — bypass class
- T1556 Modify Authentication Process — attacker registers their MFA method before exfil, no impossible-travel signal at exfil time
- Same-country attacks — fundamental fidelity gap
- Compensating controls: Entra Token Protection (verify GA status), Continuous Access Evaluation, sign-in-risk policies forcing step-up

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
