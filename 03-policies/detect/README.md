# 03-policies/detect

Policies that **detect** anomalous activity in sanctioned SaaS — UEBA-style and rule-based. All are API-mode (post-event detection); none prevent.

## Contents

| File | What it does |
|---|---|
| [`mass-download-alert.md`](mass-download-alert.md) | Volumetric mass-download anomaly with alert (do NOT auto-suspend on single signal) |
| [`impossible-travel-alert.md`](impossible-travel-alert.md) | Naive credential-stuffing detection — geographic infeasibility check; bypassed by modern token-replay attacks |
| [`mass-delete-anomaly.md`](mass-delete-anomaly.md) | Ransomware-in-SaaS detection — high-velocity file deletion / overwrite |
| [`terminated-user-cross-saas.md`](terminated-user-cross-saas.md) | Cross-SaaS account activity check on every Entra-user termination |
| [`b2b-partner-exfil-alert.md`](b2b-partner-exfil-alert.md) | External-guest user file operations at threshold — trusted-relationship exfil detection |

## Honest framing

These are **detect-leg** controls. Each catches a specific named pattern but defeated-by classes exist:

- `mass-download-alert` defeated by slow-and-low (100 files/day × 30 days); pair with weekly-baseline Sentinel KQL
- `impossible-travel-alert` defeated by token-replay from residential-proxy infrastructure (T1550.001) — the dominant 2024-2026 attack pattern
- `terminated-user-cross-saas` 0% coverage when email identifiers don't match exactly between Entra and SaaS
- `mass-delete-anomaly` defeated by slow attacker pace

Read each file's "Coverage gaps" section before relying on a policy in this family as the primary control for any risk class.

## Typical deployment cadence

- **Day 1:** `mass-download-alert`, `impossible-travel-alert` — alert-only, no CAAC required
- **Day 30:** `mass-delete-anomaly`, `b2b-partner-exfil-alert`
- **Day 90:** `terminated-user-cross-saas` — depends on offboarding-runbook integration

## Cross-references

- [`../README.md`](../README.md) — policy-library overview
- [`../../02-capabilities/capability-matrix.md`](../../02-capabilities/capability-matrix.md) — Anomalous-activity / UEBA, Compromised-account detection
- [`../../04-vendors/microsoft-defender-for-cloud-apps.md`](../../04-vendors/microsoft-defender-for-cloud-apps.md) — Policies 3, 4, 10, 11 + Appendix B (slow-and-low KQL)
- [`../../08-failure-modes/api-mode-is-not-prevention.md`](../../08-failure-modes/api-mode-is-not-prevention.md) — structural detection-not-prevention limit
- [`../../07-implementation/soc-integration.md`](../../07-implementation/soc-integration.md) — SOC triage of these alerts
