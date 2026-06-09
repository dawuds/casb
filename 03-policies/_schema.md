# Policy entry schema

Every file in `03-policies/<family>/<policy>.md` follows the schema below. Sections appear in this order. Headings are exact (`## Purpose`, etc.) so cross-file regex tooling works.

## Depth tiers

**Tier 1 — Skeletal** (the v0.0 default at first ship): structural sections only — Status, Purpose, Action, Scope, Vendor implementation grid, Control mappings, FP risk, Operational cost, Privacy considerations, Coverage gaps, Three-lens sign-off.

**Tier 2 — Deep-dive** (the target for promoted policies): Tier 1 plus seven additional sections — Use cases, Implementation pattern, Worked configuration example, Variants (industry + maturity), Real-world FP experience, Integration with broader programmes, Anti-patterns specific to this policy.

At v0.0, only a handful of exemplar policies are at Tier 2. Other policies are Tier 1 with `> Depth: Tier 1 — skeletal; Tier 2 expansion outstanding` flagged in the status block. The outstanding-list lives in [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md).

## Full Tier-2 schema

```markdown
# <Policy name>

> Status: <v0.0 — MDA-tier corroborated only / draft / etc.>
> Depth: Tier 1 — skeletal | Tier 2 — deep-dive
> Required capabilities: link to `../../02-capabilities/capability-matrix.md` row(s)
> Deployment-mode requirement: <forward-proxy / reverse-proxy / API / hybrid>
> MDA playbook reference: link to relevant policy section in `../../04-vendors/microsoft-defender-for-cloud-apps.md`

## Purpose

One sentence. Risk reduced. Cite ATT&CK / ATLAS technique.

## What organisations use this for (Tier 2)

1-2 paragraph framing of the practitioner reality. Then 3-4 named use cases, each with:

### Use case <N> — <named pattern>

- **Org type:** industry vertical + size + regulatory context
- **Trigger:** what made them deploy this (incident, audit, regulatory change, board ask)
- **Scope:** where they scoped it (BU / data class / phase)
- **Outcome:** what changed measurably (or what's claimed; flag if unverified)

## Implementation pattern (Tier 2)

Typical rollout sequence with timeline + dependencies. Use a table:

| Week | Activity | Output / gate |
|---|---|---|
| W1-W2 | ... | ... |

## Action

block / coach / monitor / quarantine / encrypt / step-up / require labelling / DLP-redact

## Scope

- Users: which group / OU / role
- Apps: sanctioned / tolerated / unsanctioned / specific
- Device posture: managed / unmanaged / BYOD
- Network position: on-prem / remote / mobile
- Traffic: upload / download / share / API call

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | ... | ... | ... | ... |
| Netskope | `[unverified]` | | | |
| Palo Alto Prisma Access | `[unverified]` | | | |
| Skyhigh | `[unverified]` | | | |
| Zscaler | `[unverified]` | | | |

## Worked configuration example (Tier 2)

Anonymised tier-2 BFSI baseline configuration. YAML-ish property list, not vendor-product literal syntax. Show the key fields a practitioner would set.

## Variants (Tier 2)

### Industry-specific
- **BFSI:** what's different (regulator-driven sensitivities, data classes, FP fights)
- **Healthcare:** what's different
- **Tech:** what's different
- **Retail:** what's different

### Maturity-based
- **Immature deployment** looks like: ...
- **Mature deployment** looks like: ...
- **Advanced deployment** looks like: ...

## Control mappings

- BNM RMiT clause(s): link to `../../06-compliance/malaysia/bnm-rmit.md`
- ISO 27017 control(s): link to `../../06-compliance/iso-27017.md`
- ISO 27018 control(s): link to `../../06-compliance/iso-27018.md`
- NIST CSF 2.0 subcategory(ies): reference
- PCI DSS Req(s).: where applicable

## False-positive risk

Named scenarios — not a rating.

## Real-world FP experience (Tier 2)

FP-rate trajectory by week (typical):

| Week | Typical FP rate | Cause |
|---|---|---|
| W1 | high | initial overshoot |
| W4 | medium | after first tuning pass |
| W12 | low | mature exclusion lists |

Plus named FP scenarios encountered in production deployments.

## Operational cost

- Exception-handling load: named scenarios
- Triage load: alerts/day estimate where possible
- End-user friction: named UX consequences

## Privacy / data-protection considerations

- SSL inspection on personal traffic — PDPA / GDPR implications
- PII surfaced to admins — access governance on the policy's evidence record
- DPIA / TIA trigger?

## Integration with broader programmes (Tier 2)

How this policy feeds:
- Audit cycles (which Req., which clauses, on what cadence)
- Board / executive reporting (which metrics this drives)
- Incident response runbooks (which signals feed which playbooks)
- Other compliance deliverables (DPIA, vendor risk assessments, etc.)

## Anti-patterns specific to this policy (Tier 2)

What goes wrong in deployment of *this specific policy*. Distinct from generic CASB anti-patterns (which live in [`../../08-failure-modes/`](../../08-failure-modes/)).

## Coverage gaps

Cross-link to `../../08-failure-modes/<mode>.md` entries.

## Three-lens sign-off

- Architect: pending / findings + resolution
- Product: pending / findings + resolution
- Compliance: pending / findings + resolution
```

## Notes

- **Tier 1 default at first ship** — every new policy starts skeletal. Avoid speculative use-case content (the file-creation-discipline rule in [`../CLAUDE.md`](../CLAUDE.md) line 87).
- **Tier 2 requires real-world grounding** — use cases must come from practitioner knowledge of how organisations deploy the policy, not invented. Where the practitioner-knowledge floor isn't met, leave the section skeletal with `[TBD — depth pending]` rather than confabulate.
- **No client-specific names** — sanitisation rules in `CLAUDE.md` apply. Use archetype framing ("tier-1 ASEAN universal bank", "merchant acquirer with multi-jurisdiction CHD", etc.).
- **FP rate numbers** are typical-trajectory ranges, not measurements from a specific deployment. State clearly when a number is illustrative.
