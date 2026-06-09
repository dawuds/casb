# 07-implementation

Programme view — vendor-agnostic guidance on how to actually deploy a CASB / SSE programme. The "what is required to implement" content.

## Contents

| File | What it contains |
|---|---|
| [`programme-readiness.md`](programme-readiness.md) | Four readiness axes (SaaS estate / Identity posture / Endpoint posture / Regulatory perimeter); pre-vendor-selection inventory checklist; sponsorship and named owners; decisions that must be locked before policy design |
| [`phased-rollout.md`](phased-rollout.md) | Discovery → risk-scoring → enforcement → tuning sequence (Phases 1-5). Why this order, what breaks if reversed. Done-criteria per phase |
| [`soc-integration.md`](soc-integration.md) | Alert lifecycle, SIEM ingest, triage tiers, SOAR/automation rules (what to automate, what NOT to), three correlation queries every SOC needs, ticketing integration paths, alert tuning cadence |
| [`success-metrics.md`](success-metrics.md) | 17 candidate KPIs / KRIs / KCIs covering discovery, policy effectiveness, OAuth + identity, DLP, operations, cost. Anti-metrics. Aggregation cadence per audience |
| [`change-management.md`](change-management.md) | End-user impact rank-order, communications-plan template, exception process with tiered authority, business-unit onboarding, workforce-privacy posture, roll-back protocol, anti-patterns |
| [`cost-of-ownership.md`](cost-of-ownership.md) | Seven cost lines (licensing / SIEM ingest / impl. services / endpoint footprint / ops headcount / exception triage / adjacent-platform creep). 5-year TCO framing. Geographic pricing variance. What a procurement model should require |

## Reading order

Linear — each builds on the previous:

1. [`programme-readiness.md`](programme-readiness.md) — go/no-go gate before vendor selection
2. [`phased-rollout.md`](phased-rollout.md) — the canonical sequence
3. [`soc-integration.md`](soc-integration.md) — what the SOC needs to be capable of
4. [`success-metrics.md`](success-metrics.md) — how to measure the programme
5. [`change-management.md`](change-management.md) — how to deploy without losing the user population
6. [`cost-of-ownership.md`](cost-of-ownership.md) — what you're actually committing to financially

## What this folder does NOT cover

- **Vendor selection / RFP scoring** — out of scope (this is a practitioner reference, not a buying guide)
- **Per-vendor configuration steps** — see [`../04-vendors/`](../04-vendors/)
- **Clause-by-clause regulator mapping** — see [`../06-compliance/`](../06-compliance/)
- **Policy details (what to enable)** — see [`../03-policies/`](../03-policies/) (vendor-neutral) or [`../04-vendors/<vendor>.md`](../04-vendors/) (vendor-specific Day 1 / 30 / 90)

## Cross-references

- [`../03-policies/`](../03-policies/) — vendor-neutral policies that get enabled during phased rollout
- [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — the gold-standard worked example with Day 1 / 30 / 90 cadence
- [`../08-failure-modes/`](../08-failure-modes/) — what to watch out for during the programme
- [`../../cyberkpis/`](../../cyberkpis/) (sibling repo) — KPI / KRI methodology that this folder defers to
- [`../README.md`](../README.md) — repo-level navigation
