# 03-policies/oauth

Policies governing **OAuth-app delegation** — third-party app grants made by users (or admins) to access M365 / Workspace / Salesforce data.

## Contents

| File | What it does |
|---|---|
| [`post-consent-cleanup-and-ban.md`](post-consent-cleanup-and-ban.md) | Catalogue + manually ban known-malicious; per-user revoke for confirmed-bad — covers ~30% of OAuth abuse |
| [`high-scope-grant-alert.md`](high-scope-grant-alert.md) | Behavioural detection on high-permission grants via App Governance Predictive Risk (MDA add-on) |

## The OAuth blind spot

These two policies cover the **delegated-permission** class (user-clicked-Consent). They DO NOT cover the **application-permission** class (admin-granted service-principal credentials) — which is the dominant cloud-persistence pattern (~70% of OAuth abuse).

See [`../../08-failure-modes/oauth-blind-spot.md`](../../08-failure-modes/oauth-blind-spot.md). Compensating controls live in Entra Workload Identities + Conditional Access for Workload Identities; not in CASB.

## Important reframing

OAuth post-consent governance via CASB is a **cleanup control, not a primary defence.** The primary control for `T1528 Steal Application Access Token` is the **Entra admin-consent workflow + verified-publisher restriction**, configured upstream in Entra (not MDA).

## Typical deployment cadence

- **Day 1:** `post-consent-cleanup-and-ban` — manual review + ban known-malicious
- **Day 30:** `high-scope-grant-alert` — App Governance Predictive Risk in notify-only mode for 4 weeks

## Cross-references

- [`../README.md`](../README.md) — policy-library overview
- [`../../02-capabilities/capability-matrix.md`](../../02-capabilities/capability-matrix.md) — OAuth-app discovery + governance row
- [`../../04-vendors/microsoft-defender-for-cloud-apps.md`](../../04-vendors/microsoft-defender-for-cloud-apps.md) — Policy 2a + 2b
- [`../../08-failure-modes/oauth-blind-spot.md`](../../08-failure-modes/oauth-blind-spot.md) — the ~70% the policy library does not cover
