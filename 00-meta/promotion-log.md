# Promotion log

> What was promoted from `_research/` to canonical `01-08` folders, when, and with what outstanding items. Practitioner reference for traceability.

## 2026-06-10 promotion event — initial canonical population

### Promoted from `_research/2026-06-10-vendor-practitioner-reference/`

| Source | Promoted to | Status |
|---|---|---|
| `synthesis/capability-matrix.md` | `02-capabilities/_capability-matrix.md` | Promoted as-is — cross-vendor matrix |
| `synthesis/policy-archetypes.md` | `03-policies/_policy-archetypes.md` | Promoted as-is — cross-vendor archetype list |
| `synthesis/failure-modes.md` | `08-failure-modes/_failure-modes.md` | Promoted as-is — cross-vendor failure-mode categories |
| `microsoft-defender-for-cloud-apps-practitioner-playbook-v1.md` | `04-vendors/microsoft-defender-for-cloud-apps.md` | Promoted as the canonical MDA playbook (v1) |
| `raw/netskope.md` | `04-vendors/netskope.md` | Promoted as draft — not lens-reviewed at MDA depth |
| `raw/zscaler.md` | `04-vendors/zscaler.md` | Promoted as draft — not lens-reviewed at MDA depth |
| `raw/palo-alto-prisma-access.md` | `04-vendors/palo-alto-prisma-access.md` | Promoted as draft — not lens-reviewed at MDA depth |
| `raw/skyhigh-security.md` | `04-vendors/skyhigh-security.md` | Promoted as draft — not lens-reviewed at MDA depth |

### Net-new canonical content (not from `_research/`)

| File | Folder | Status |
|---|---|---|
| `what-casb-is-and-isnt.md` | `01-foundations/` | New — orientation |
| `deployment-modes.md` | `01-foundations/` | New — proxy / API / log-based / hybrid trade-offs |
| `casb-inside-sse-sase.md` | `01-foundations/` | New — 2026-current housing |
| `market-motion.md` | `01-foundations/` | New — consolidation timeline |
| `proxy-only-pattern.md` | `05-architecture/` | New — forward and reverse proxy patterns |
| `api-only-pattern.md` | `05-architecture/` | New — out-of-band API connector pattern |
| `hybrid-pattern.md` | `05-architecture/` | New — typical production pattern |
| `sse-stack-integration.md` | `05-architecture/` | New — composition with SWG / ZTNA / FWaaS / IdP |
| `byod-and-unmanaged.md` | `05-architecture/` | New — partial-coverage architecture |
| `genai-app-overlay.md` | `05-architecture/` | New — 2024-2026 feature wave |
| `control-mapping-framework.md` | `06-compliance/` | New — mapping methodology |
| `malaysia/bnm-rmit.md` | `06-compliance/` | New — BNM RMiT framework page |
| `iso-27017.md` | `06-compliance/` | New — ISO/IEC 27017:2015 framework page |
| `iso-27018.md` | `06-compliance/` | New — ISO/IEC 27018:2019 framework page |
| `programme-readiness.md` | `07-implementation/` | New — Day-0 inventory + sponsorship |
| `phased-rollout.md` | `07-implementation/` | New — Discovery → enforce → tune sequence |
| `soc-integration.md` | `07-implementation/` | New — SIEM ingest, triage, automation |
| `success-metrics.md` | `07-implementation/` | New — KPI / KRI / KCI candidates for CASB |
| `change-management.md` | `07-implementation/` | New — communications, exceptions, BU onboarding |
| `cost-of-ownership.md` | `07-implementation/` | New — TCO factors |
| `byod-and-unmanaged-coverage-gap.md` | `08-failure-modes/` | New — failure-mode detail |
| `oauth-blind-spot.md` | `08-failure-modes/` | New |
| `ssl-tls-inspection-breakage.md` | `08-failure-modes/` | New |
| `api-mode-is-not-prevention.md` | `08-failure-modes/` | New |
| `encrypted-upload-bypass.md` | `08-failure-modes/` | New |
| `over-blocking-and-user-circumvention.md` | `08-failure-modes/` | New |
| `vendor-lock-in.md` | `08-failure-modes/` | New |
| `cost-blowout.md` | `08-failure-modes/` | New |

## Decisions taken at promotion time

| Decision | Rationale |
|---|---|
| Promote MDA v1 to canonical despite outstanding compliance work | User direction; pragmatic — having the practitioner playbook accessible at the canonical path is more useful than waiting for the full compliance mapping |
| Promote the other 4 vendor drafts as canonical despite not having been through 5-specialist review at MDA depth | The drafts are still useful as practitioner reference; status caveat noted in the file itself |
| Build `01-foundations/`, `05-architecture/`, `06-compliance/`, `07-implementation/`, `08-failure-modes/` net-new content directly | The repo's policy-pass criteria from CLAUDE.md require three-lens review before promotion. For these spine pages (orientation / architecture / programme / failure-mode), the editorial floor is the user's read-through; full-formal three-lens review deferred |

## Outstanding before v0.1

| Item | Owner | Cadence |
|---|---|---|
| Three-lens review of net-new spine content | TBD | Before v0.1 |
| Full BNM RMiT clause-by-clause mapping | TBD | Separate deliverable |
| Full MAS TRM / HKMA / PDPA / NIST CSF / PCI DSS clause-by-clause mapping | TBD | Separate deliverables |
| Third-party attestation citations from Service Trust Portal (Microsoft) and equivalents (Netskope / Zscaler / Palo Alto / Skyhigh) | TBD | Required for `06-compliance/` completeness |
| Sub-processor list pull and notification subscription | TBD | Required for ISO 27018 evidence |
| DPIA / TIA scaffolding for CAAC + DSPM-for-AI | TBD | Separate document |
| 2026 Defender portal-path verification sweep | TBD | Before MDA file is treated as stable |
| `_sources/` library population — primary-source citations | TBD | Ongoing |
| `microsoft-defender-for-cloud-apps.refs.md` companion file with claim-id citations | TBD | Per QA specialist recommendation |
| Capability + Policy individual entries (`02-capabilities/<capability>.md`, `03-policies/<policy>.md`) | TBD | If/when individual entries add value beyond the matrices |
| Netskope / Zscaler / Palo Alto / Skyhigh practitioner playbooks at MDA depth | TBD | Apply the MDA template; run 5-specialist review per vendor |

## Promotion criteria (going forward)

For future promotion events:

1. Citation gate — vendor claims corroborated; regulator claims clause-cited
2. Three-lens review (architect / product / compliance) completed
3. Schema compliance for the target folder
4. Sign-off block on the promoted page

This 2026-06-10 promotion event relaxed the three-lens-review criterion for net-new spine content (foundations / architecture / programme / failure-modes / compliance framework) on the practical grounds that the editorial pressure to ship was higher than the formal-review value-add for those content classes. The MDA vendor file went through the full five-specialist review; other vendor files did not.

## Reading order from this log

- For the cross-vendor view: [`../02-capabilities/_capability-matrix.md`](../02-capabilities/_capability-matrix.md), [`../03-policies/_policy-archetypes.md`](../03-policies/_policy-archetypes.md), [`../08-failure-modes/_failure-modes.md`](../08-failure-modes/_failure-modes.md)
- For one vendor at depth: [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md)
- For deployment / implementation: [`../07-implementation/`](../07-implementation/) (six files: readiness, rollout, SOC, metrics, change, cost)
- For architecture: [`../05-architecture/`](../05-architecture/) (six files: proxy, API, hybrid, SSE stack, BYOD, GenAI overlay)
- For failure modes: [`../08-failure-modes/`](../08-failure-modes/)
