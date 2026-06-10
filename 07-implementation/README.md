# 07-implementation

Programme view — vendor-agnostic guidance on how to actually deploy a CASB / SSE programme. The "what is required to implement" content. Each file in this folder is at **Tier-2 depth** matching the [`../03-policies/`](../03-policies/) policy library, with the same shape: use cases + implementation pattern + variants + integration + anti-patterns.

## Contents

| File | What it owns |
|---|---|
| [`programme-readiness.md`](programme-readiness.md) | Four readiness axes (SaaS estate / Identity posture / Endpoint posture / Regulatory perimeter); pre-vendor-selection inventory; sponsorship + named owners; locked-before-vendor-selection decisions |
| [`phased-rollout.md`](phased-rollout.md) | Discovery → risk-scoring → enforcement → tuning sequence (Phase 1-5); per-policy phase assignment for the 19 `03-policies/` files; done-criteria gates between phases |
| [`soc-integration.md`](soc-integration.md) | Alert lifecycle, SIEM ingest, triage tiers, SOAR automation rules, three correlation queries, ticketing integration paths, alert tuning cadence |
| [`success-metrics.md`](success-metrics.md) | 17 candidate KPIs / KRIs / KCIs covering discovery, policy effectiveness, OAuth + identity, DLP, operations, cost; per-audience aggregation cadence; anti-metrics |
| [`change-management.md`](change-management.md) | End-user impact rank-order, communications-plan template, tiered exception process (L1-L4), business-unit onboarding, workforce-privacy posture, roll-back protocol |
| [`cost-of-ownership.md`](cost-of-ownership.md) | Seven cost lines (licensing / SIEM / impl / endpoint / ops headcount / exception triage / adjacent-platform creep); 5-year TCO framing; geographic pricing variance; procurement-model requirements |
| [`_schema.md`](_schema.md) | The canonical Tier-2 schema each file follows |

## Reading order

Linear — each builds on the previous:

1. [`programme-readiness.md`](programme-readiness.md) — go/no-go gate before vendor selection
2. [`phased-rollout.md`](phased-rollout.md) — the canonical sequence; per-phase done-criteria
3. [`soc-integration.md`](soc-integration.md) — what the SOC needs to be capable of
4. [`success-metrics.md`](success-metrics.md) — how to measure the programme
5. [`change-management.md`](change-management.md) — how to deploy without losing the user population
6. [`cost-of-ownership.md`](cost-of-ownership.md) — what you're actually committing to financially

## Alignment with 03-policies

Each implementation file is structured to mirror the Tier-2 schema of the policy library:

| Implementation file owns | 03-policies/ files it most-directly governs |
|---|---|
| `programme-readiness` | All 19 — readiness gates whether any policy ships |
| `phased-rollout` | All 19 — per-policy phase assignment documented |
| `soc-integration` | `detect/` (5 files) + `oauth/` (2 files) + `posture/` (1 file) + alert side of others |
| `success-metrics` | All 19 — per-policy M4 / M5 / M11 metrics; programme-level M1 / M16 |
| `change-management` | All user-visible block policies — `access-control/` (4 files) + user-facing `dlp/` + `genai/inline-prompt-dlp.md` + auto-suspend `detect/` policies |
| `cost-of-ownership` | All 19 + adjacent-platform decisions (Entra ID P1 / Purview IRM / MDE NP / Sentinel ingest) |

## What this folder does NOT cover

- **Vendor selection / RFP scoring** — out of scope (this is a practitioner reference, not a buying guide)
- **Per-vendor configuration steps** — see [`../04-vendors/`](../04-vendors/)
- **Clause-by-clause regulator mapping** — see [`../06-compliance/`](../06-compliance/)
- **Policy details (what to enable)** — see [`../03-policies/`](../03-policies/) (vendor-neutral) or [`../04-vendors/<vendor>.md`](../04-vendors/) (vendor-specific Day 1 / 30 / 90)
- **Reference architecture patterns** — see [`../05-architecture/`](../05-architecture/)

## Cross-references

- [`../03-policies/`](../03-policies/) — policy library; each policy's "Integration with broader programmes" section cross-links to this folder
- [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — the gold-standard worked example; Day 1 / 30 / 90 mirrors the `phased-rollout.md` pattern
- [`../08-failure-modes/`](../08-failure-modes/) — what to watch out for; `over-blocking-and-user-circumvention.md` is the failure mode `change-management.md` exists to prevent
- [`../../cyberkpis/`](../../cyberkpis/) (sibling repo) — KPI / KRI methodology that `success-metrics.md` defers to
- [`../../cyber-business/`](../../cyber-business/) (sibling repo) — vendor unit-economics for `cost-of-ownership.md` benchmarking
- [`../README.md`](../README.md) — repo-level navigation
