# 08-failure-modes

Where CASB **demonstrably fails.** Public-wedge candidate — contrarian, shareable, demolishes vendor decks. Each file names: the failure, what enables it, which vendors exhibit it, why the failure exists (technical), what compensates (NOT another CASB feature), and what practitioners should NOT promise.

## Contents

| File | The failure |
|---|---|
| [`byod-and-unmanaged-coverage-gap.md`](byod-and-unmanaged-coverage-gap.md) | The most under-acknowledged CASB failure. Mobile native + personal devices bypass; CASB alone is insufficient — MAM + label encryption pair required |
| [`oauth-blind-spot.md`](oauth-blind-spot.md) | ~30% delegated grants covered, ~70% application-permission (service-principal) grants uncovered; the dominant cloud-persistence pattern |
| [`ssl-tls-inspection-breakage.md`](ssl-tls-inspection-breakage.md) | Cert-pinning, personal-use HTTPS, TLS 1.3 / ESNI / ECH — silent partial failure on inspection |
| [`api-mode-is-not-prevention.md`](api-mode-is-not-prevention.md) | API mode detects post-event; vendor marketing conflates with prevention |
| [`encrypted-upload-bypass.md`](encrypted-upload-bypass.md) | 7zip-with-password + RMS-protected + zero-knowledge SaaS defeat DLP by class |
| [`over-blocking-and-user-circumvention.md`](over-blocking-and-user-circumvention.md) | Aggressive enforcement creates workaround culture; blocked traffic moves to unmonitored channels |
| [`vendor-lock-in.md`](vendor-lock-in.md) | Policy languages don't port between vendors; migration is 6-12 months of platform work |
| [`cost-blowout.md`](cost-blowout.md) | 2-3× procurement-time estimate by year 2; hidden lines (SIEM ingest growth, adjacent-platform creep) dominate |

## Sub-folder structural file

| File | What it contains |
|---|---|
| [`_failure-modes.md`](_failure-modes.md) | Cross-vendor failure-modes synthesis (legacy artefact from the 2026-06-10 workflow run). Mostly subsumed by the per-failure-mode files above |

## Reading order

Read in order of practitioner relevance for your environment:

- **BYOD-heavy enterprise:** start with [`byod-and-unmanaged-coverage-gap.md`](byod-and-unmanaged-coverage-gap.md)
- **Microsoft-stack-heavy:** start with [`oauth-blind-spot.md`](oauth-blind-spot.md) and [`api-mode-is-not-prevention.md`](api-mode-is-not-prevention.md)
- **Privacy-sensitive jurisdiction:** start with [`ssl-tls-inspection-breakage.md`](ssl-tls-inspection-breakage.md)
- **Procurement / financial planning:** start with [`cost-blowout.md`](cost-blowout.md) and [`vendor-lock-in.md`](vendor-lock-in.md)
- **Programme rollout planning:** start with [`over-blocking-and-user-circumvention.md`](over-blocking-and-user-circumvention.md)

## Wedge note

If the repo wedge expands to include a public failure-mode catalogue (one of the four candidate wedges in [`../CLAUDE.md`](../CLAUDE.md)), these files become the public-wedge content. Currently they are practitioner-residual-risk references.

## Cross-references

- [`../03-policies/`](../03-policies/) — each policy file's "Coverage gaps" section cross-links to this folder
- [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — the per-vendor Gaps table is the per-vendor view; this folder is the cross-vendor view
- [`../07-implementation/change-management.md`](../07-implementation/change-management.md) — communications to prevent the workaround pattern
- [`../README.md`](../README.md) — repo-level navigation
