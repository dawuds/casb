# casb

Practitioner reference on **Cloud Access Security Broker (CASB) — what the capability set actually is in 2026, how it maps to vendor products, what policies it enforces, what regulators expect, and where it demonstrably fails.** Anchored Malaysia, with MAS / HKMA / EU / ISO as comparators.

**Thesis (locked 2026-06-10):** *"For each of the 5 CASB products in scope, what policies can a practitioner actually configure — and what does the tool not cover?"* — Microsoft Defender for Cloud Apps is the deepest worked example. See [`00-meta/thesis.md`](00-meta/thesis.md).

**Status:** v0.0 — canonical content populated 2026-06-10. Promotion log at [`00-meta/promotion-log.md`](00-meta/promotion-log.md). Outstanding items before v0.1 listed there.

## How to navigate

Start where the question lives:

| If you want | Read |
|---|---|
| **Orientation** — what CASB is, why it changed, where it sits in SSE | [`01-foundations/`](01-foundations/) — what-is, deployment modes, SSE positioning, market motion |
| **The cross-vendor capability picture** | [`02-capabilities/_capability-matrix.md`](02-capabilities/_capability-matrix.md) |
| **The cross-vendor policy archetype list** | [`03-policies/_policy-archetypes.md`](03-policies/_policy-archetypes.md) |
| **One vendor at depth (Microsoft Defender for Cloud Apps)** | [`04-vendors/microsoft-defender-for-cloud-apps.md`](04-vendors/microsoft-defender-for-cloud-apps.md) — 13 configurable policies (Day 1 / 30 / 90), the bilateral CAAC trap, the gaps-and-compensating-controls table, ATT&CK + ATLAS coverage matrix, KQL starter pack |
| **Vendor drafts** for Netskope / Zscaler / Palo Alto / Skyhigh | [`04-vendors/`](04-vendors/) — not at MDA depth yet; promoted as drafts |
| **Reference architectures** (proxy / API / hybrid; how CASB sits in SSE) | [`05-architecture/`](05-architecture/) |
| **Compliance / regulator mapping** | [`06-compliance/`](06-compliance/) — control-mapping framework + BNM RMiT / ISO 27017 / ISO 27018 framework pages |
| **How to actually implement** (programme readiness, phased rollout, SOC integration, metrics, change management, TCO) | [`07-implementation/`](07-implementation/) |
| **Where CASB demonstrably fails** | [`08-failure-modes/`](08-failure-modes/) — BYOD, OAuth blind spot, SSL/TLS breakage, API-mode-is-not-prevention, encrypted-upload bypass, over-blocking, vendor lock-in, cost blowout |
| **Research scratch + workflow output** | [`_research/`](_research/) |

## Component model

```
00-meta/             Thesis, lock log, three-lens review template, promotion log
01-foundations/      What CASB is and isn't; deployment modes; SSE/SASE positioning; market motion
02-capabilities/     Cross-vendor capability matrix
03-policies/         Cross-vendor policy archetypes
04-vendors/          5 vendors — Microsoft Defender for Cloud Apps (v1, lens-reviewed); Netskope / Zscaler / Palo Alto / Skyhigh (drafts)
05-architecture/     Proxy / API / hybrid patterns; SSE stack integration; BYOD architecture; GenAI overlay
06-compliance/       Control-mapping framework + BNM RMiT / ISO 27017 / ISO 27018 framework pages
07-implementation/   Programme readiness, phased rollout, SOC integration, success metrics, change management, cost of ownership
08-failure-modes/    BYOD, OAuth, SSL/TLS, API-mode, encrypted-upload, over-blocking, lock-in, cost
_sources/            Primary-source library (in development)
_research/           Working notes; multi-agent workflow outputs; v0 drafts and lens reviews
```

## The three-lens review pattern

Every synthesis page should pass through three lenses before being treated as stable:

- **Cybersecurity architect** — deployment, integration, what does *not* get covered
- **CASB product expert** — vendor claims vs vendor behaviour, deployment-mode caveats, console footguns
- **Cyber-compliance expert** — control mapping, audit evidence, regulator clock, privacy conflict

The MDA vendor file went through this plus a five-specialist review (QA / cyber+MITRE / MDA-deep-expert / system integration / documentation). See [`_research/2026-06-10-vendor-practitioner-reference/`](_research/2026-06-10-vendor-practitioner-reference/) for the full workflow output. See [`00-meta/three-lens-review.md`](00-meta/three-lens-review.md) for the prompt templates.

## What this repo does NOT cover

- **Vendor evaluation / RFP scoring** — the per-vendor files are practitioner references, not buying guides
- **Cross-vendor consulting deliverables** — the syntheses are draft-quality, not productized
- **Full regulator clause mapping** — `06-compliance/` has the framework pages; the full clause-by-clause mapping per regulator is a separate deliverable
- **Third-party attestation citations** — pulls from Trust Center / Service Trust Portal still required before treating any vendor claim as audit-evidenced
- **DPIA / TIA documents** — CAAC reverse-proxy and DSPM-for-AI prompt capture both require formal data-protection impact assessments before pilot; scaffolding is referenced but the documents themselves are separate

## Caveats

- **Material on regulatory clauses is illustrative; not legal/regulatory advice.** Confirm against gazetted text and your assurance lead.
- **Vendor capability claims are vendor-controlled.** Cited from vendor documentation; corroborate before treating as audit evidence.
- **The 2026 CASB / SSE landscape is moving fast.** GenAI-related capabilities especially are shifting quarterly. Verify the current-quarter state before relying on a 2026-H1 capability claim in production.

## Operating rules

See [`CLAUDE.md`](CLAUDE.md) for: citation discipline, sanitisation rules, file-creation discipline, common pitfalls, workflow, three-lens review pattern.
