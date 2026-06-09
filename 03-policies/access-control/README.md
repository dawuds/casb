# 03-policies/access-control

Policies that gate **who** can access **what**, under **which** conditions. The control-point question is identity-and-device-posture, evaluated before the user reaches the SaaS application.

## Contents

| File | What it does |
|---|---|
| [`block-unsanctioned-app-with-coach.md`](block-unsanctioned-app-with-coach.md) | Block discovered SaaS tagged Unsanctioned; coach user toward the sanctioned alternative |
| [`tenant-restriction-corporate-only.md`](tenant-restriction-corporate-only.md) | Block sign-in to M365 / Workspace / ChatGPT etc. using personal accounts or non-corporate tenants from corporate networks / managed devices |
| [`block-download-unmanaged-device.md`](block-download-unmanaged-device.md) | Prevent download of files labelled Confidential+ from sanctioned SaaS to BYOD / personal devices |
| [`geo-residency-block.md`](geo-residency-block.md) | Block upload to / download from data regions outside the firm's permitted geographic perimeter for tagged regulated data |

## Typical deployment cadence

- **Day 1:** `block-unsanctioned-app-with-coach` — discovery + ban known-bad
- **Day 30:** `tenant-restriction-corporate-only`, `block-download-unmanaged-device` — CAAC-dependent
- **Day 90:** `geo-residency-block` — depends on data classification maturity

(Cadence is a per-firm rollout property; see [`../../07-implementation/phased-rollout.md`](../../07-implementation/phased-rollout.md).)

## Cross-references

- [`../README.md`](../README.md) — policy-library overview
- [`../../02-capabilities/capability-matrix.md`](../../02-capabilities/capability-matrix.md) — capabilities these policies depend on
- [`../../05-architecture/proxy-only-pattern.md`](../../05-architecture/proxy-only-pattern.md) and [`../../05-architecture/hybrid-pattern.md`](../../05-architecture/hybrid-pattern.md) — deployment patterns
- [`../../08-failure-modes/byod-and-unmanaged-coverage-gap.md`](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — the dominant residual risk for this family
