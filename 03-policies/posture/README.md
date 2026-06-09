# 03-policies/posture

Policies that detect **SaaS configuration drift** from baseline — light SSPM via the CASB.

## Contents

| File | What it does |
|---|---|
| [`sspm-tenant-misconfig-drift.md`](sspm-tenant-misconfig-drift.md) | Continuous assessment of sanctioned-SaaS tenant security configuration against documented posture-rule library; alert on drift |

## Important framing

CASB SSPM coverage is **shallow**. For deep coverage, dedicated SSPM tools (Adaptive Shield, AppOmni, Obsidian, Wing) are the right tier. Verify per-vendor before procuring on CASB-only SSPM assumption.

The CASB SSPM coverage is typically narrow (M365, Salesforce, ServiceNow, GitHub, Okta, Box, Dropbox, Zendesk for MDA — vendor-dependent), with limited remediation actions and daily-at-best detection cadence.

## Typical deployment cadence

- **Day 90:** posture monitoring after the higher-priority policy families are stable

## Cross-references

- [`../README.md`](../README.md) — policy-library overview
- [`../../01-foundations/what-casb-is-and-isnt.md`](../../01-foundations/what-casb-is-and-isnt.md) — the CASB vs SSPM boundary
- [`../../04-vendors/microsoft-defender-for-cloud-apps.md`](../../04-vendors/microsoft-defender-for-cloud-apps.md) — App Governance posture features
