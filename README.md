# casb

Practitioner reference for **Cloud Access Security Broker (CASB)** controls in 2026 — capability set, configurable policies, vendor coverage, regulator mapping, and where the tooling demonstrably fails. Anchored Malaysia (BNM RMiT) with MAS / HKMA / EU / ISO as comparators.

**Status:** v0.0 — canonical content populated 2026-06-10. **Microsoft Defender for Cloud Apps is the only vendor at v1 lens-reviewed depth**; Netskope / Zscaler / Palo Alto Prisma Access / Skyhigh are draft-quality only. See [`00-meta/promotion-log.md`](00-meta/promotion-log.md). Outstanding items before v0.1 listed there.

## Start here

Three reading paths. Pick yours.

| If you are | Read in this order |
|---|---|
| **New to CASB** | [`01-foundations/what-casb-is-and-isnt.md`](01-foundations/what-casb-is-and-isnt.md) → [`01-foundations/deployment-modes.md`](01-foundations/deployment-modes.md) → [`07-implementation/programme-readiness.md`](07-implementation/programme-readiness.md) |
| **Already running Microsoft Defender for Cloud Apps** and need to know what to switch on | [`04-vendors/microsoft-defender-for-cloud-apps.md`](04-vendors/microsoft-defender-for-cloud-apps.md) — Day 0 preconditions → Gaps table → 13 numbered policies (Day 1 / 30 / 90), each with console path, trap, defeated-by, audit evidence, SIEM artefact |
| **Planning a CASB programme** (any vendor) | [`07-implementation/programme-readiness.md`](07-implementation/programme-readiness.md) → [`07-implementation/phased-rollout.md`](07-implementation/phased-rollout.md) → [`04-vendors/`](04-vendors/) for your vendor → [`08-failure-modes/`](08-failure-modes/) |

Everything else is reference; see "Folder map" below.

## How the content is split

- **Capability** = what the tool *can* do. Lives in [`02-capabilities/`](02-capabilities/) (one cross-vendor matrix).
- **Policy** = what the firm has *decided to enforce*, expressed as a configurable, vendor-neutral spec with a per-vendor implementation grid. Lives in [`03-policies/`](03-policies/) (one file per policy, sub-foldered by control family).
- **Vendor playbook** = the full per-vendor configuration walkthrough (Day 1 / 30 / 90 rollout cadence, console paths, traps). Lives in [`04-vendors/`](04-vendors/).

The per-vendor file is canonical for "how do I configure this". The policy file is canonical for "what does this policy look like across vendors". The capability matrix is canonical for "which vendors can do this at all".

## Folder map

| Folder | One-line role |
|---|---|
| [`00-meta/`](00-meta/) | Thesis lock, promotion log, three-lens review template, glossary |
| [`01-foundations/`](01-foundations/) | What CASB is and isn't; deployment modes; where it sits in SSE; market motion |
| [`02-capabilities/`](02-capabilities/) | Cross-vendor capability matrix — what each tool can do, cell-by-cell with caveats |
| [`03-policies/`](03-policies/) | Vendor-neutral policy library — one file per policy, with a 5-row vendor-implementation grid |
| [`04-vendors/`](04-vendors/) | Per-vendor playbooks — MDA at v1 lens-reviewed depth; Netskope / Zscaler / Palo Alto / Skyhigh as drafts |
| [`05-architecture/`](05-architecture/) | Reference architectures — proxy / API / hybrid; SSE stack; BYOD; GenAI overlay |
| [`06-compliance/`](06-compliance/) | Control mapping framework + BNM RMiT / ISO 27017 / ISO 27018 framework pages |
| [`07-implementation/`](07-implementation/) | Programme readiness, phased rollout, SOC integration, metrics, change management, cost of ownership |
| [`08-failure-modes/`](08-failure-modes/) | Where CASB demonstrably fails — BYOD, OAuth, SSL/TLS, API-mode, encrypted upload, over-blocking, lock-in, cost |
| [`_research/`](_research/) | Working notes, multi-agent workflow outputs, lens reviews — not citation-gated |
| [`_sources/`](_sources/) | Primary-source library (in development) |

## What this repo does NOT cover

- **Vendor evaluation / RFP scoring** — the per-vendor files are practitioner references, not buying guides
- **Cross-vendor consulting deliverables** — the syntheses are draft-quality, not productized
- **Full regulator clause mapping** — `06-compliance/` has framework pages; clause-by-clause mapping is a separate deliverable
- **Third-party attestation citations** — pulls from Trust Center / Service Trust Portal still required before treating any vendor claim as audit-evidenced
- **DPIA / TIA documents** — CAAC reverse-proxy and DSPM-for-AI prompt capture both require formal data-protection impact assessments before pilot

## Caveats

- **Material on regulatory clauses is illustrative; not legal/regulatory advice.** Confirm against gazetted text and your assurance lead.
- **Vendor capability claims are vendor-controlled.** Cited from vendor documentation; corroborate before treating as audit evidence.
- **The 2026 CASB / SSE landscape is moving fast.** GenAI-related capabilities especially are shifting quarterly. Verify the current-quarter state before relying on a 2026-H1 capability claim in production.

## For collaborators

See [`CLAUDE.md`](CLAUDE.md) for citation discipline, sanitisation rules, file-creation discipline, the three-lens review pattern, and the workflow. The thesis lock and rationale are at [`00-meta/thesis.md`](00-meta/thesis.md). The promotion log at [`00-meta/promotion-log.md`](00-meta/promotion-log.md) records what was promoted from `_research/` and what is outstanding before v0.1.
