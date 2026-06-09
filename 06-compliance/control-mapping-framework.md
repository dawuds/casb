# Control mapping framework

> How to map CASB capabilities and policies to regulator and standards controls. The framework — not the full mapping. Vendor-agnostic, jurisdiction-agnostic. Material on regulation, audit, and supervisory expectations is illustrative only — never legal/regulatory advice.

## Why this framework matters

Auditors and regulators do not ask "what features does your CASB have?". They ask:

- For control X (named clause in named standard / regulation), how do you evidence that it is operating effectively?
- Show me the policy that enforces it. Show me the log artefact. Show me the retention.

Without a structured mapping, every audit cycle becomes a custom evidence pull. With a mapping, the audit-evidence package is reusable.

## The three-layer mapping

```
Layer 1: Control (regulator clause / standard control ID)
            |
            v
Layer 2: Policy (the firm's enforced rule, in the CASB)
            |
            v
Layer 3: Capability (the CASB feature that the policy uses)
```

Each layer has its own catalogue and lifecycle:

- **Controls** are external — defined by the standard or regulator; we don't choose them.
- **Policies** are internal — the firm chooses how to enforce. Multiple policies often address the same control.
- **Capabilities** are vendor-supplied — the firm doesn't define them.

The mapping makes each policy evidence one or more controls via one or more capabilities.

## Schema for a control entry

Every control gets a file in `06-compliance/<jurisdiction>/<instrument>/<clause-or-control-id>.md` with the following schema (created when the first control entry is written):

```markdown
# <Control ID> — <Control title>

## Source
- Instrument: <e.g. ISO/IEC 27017:2015>
- Clause: <e.g. CLD.12.4.5>
- Title: <verbatim>
- Version / edition: <e.g. 2015>
- Source URL: <published-source>
- Accessed date: <yyyy-mm-dd>

## Verbatim text
> <copy of the control text, exact, in quote block>

## In scope for CASB?
- YES / NO / PARTIAL

## Mapping
| CASB policy (from 03-policies/) | Capability used (from 02-capabilities/) | Evidence artefact |
|---|---|---|
| <policy-name> | <capability-name> | <table-or-log-event with field-level granularity> |

## Compensating controls (where CASB is partial)
- ...

## Auditor questions this control will draw
- ...

## Notes
- Any nuance, vendor-specific implementation, version drift
```

## What counts as evidence

The evidence floor for a regulated FI:

| Evidence type | Examples | Acceptable? |
|---|---|---|
| **Vendor product documentation** | Microsoft Learn page, Netskope docs, Zscaler help | NO as primary evidence (vendor self-attestation) |
| **Vendor configuration in your tenant** | Screenshot of policy, exported policy JSON | YES, paired with operational evidence |
| **Operational log records** | SIEM event records with policy ID, user ID, decision, timestamp | YES, primary evidence |
| **Audit log of policy state changes** | Change-management ticket + IdP / CASB audit log | YES, for control attestation |
| **Third-party attestation** | SOC 2 Type II report, ISO 27001 certificate scope | YES, for cloud-service-side claims (Microsoft / vendor side) |
| **Sub-processor list** | Vendor's published sub-processor list | YES, for processor-chain transparency |
| **Independent test report** | (Sometimes available — penetration test results, capability test reports) | YES |

For each control mapping, name which evidence types you can produce. Some controls (e.g. shared-responsibility model under ISO 27017 CLD.6.3.1) need both your-side configuration and vendor-side attestation.

## Standards / regulators in scope (initial)

For a Malaysian-anchored regulated FI, the initial mapping set:

| Instrument | Jurisdiction | Why in scope |
|---|---|---|
| **BNM RMiT** (current edition) | MY | Mandatory for licensed Malaysian FIs |
| **PDPA Malaysia (Act 709, 2010 + 2024 amendments)** | MY | Cross-border data, breach notification, DPO requirement |
| **Cybersecurity Act 854 (2024)** + **NACSA** sectoral directives | MY | CNII sectoral expectations relevant to BFSI |
| **MAS TRM** + Notice on Technology Risk Management | SG | If you operate in Singapore or rely on SG infrastructure |
| **HKMA SA-2** + General Principles for Technology Risk Management | HK | If you operate in Hong Kong |
| **PDPA Singapore (No. 26 of 2012, as amended)** | SG | If processing SG personal data |
| **GDPR** + **DORA** | EU | If processing EU personal data / EU FI footprint |
| **ISO/IEC 27001:2022** | International | Common certification baseline; Annex A controls |
| **ISO/IEC 27017:2015** | International | Cloud-services controls (specifically relevant for CASB) |
| **ISO/IEC 27018:2019** | International | Cloud-PII-processor controls |
| **NIST CSF 2.0** | International | Cross-jurisdiction backbone; subcategory mapping |
| **NIST SP 800-53 Rev 5** | International | Specific control families (AC, AU, SC, SI) |
| **PCI DSS v4.0** | International | If the firm processes cardholder data |
| **SOC 2** | International | Vendor-side attestation reference |

Material on regulatory clauses is illustrative — never legal/regulatory advice. Verify against the gazetted text of each instrument; confirm with your assurance lead before reliance.

## File structure

```
06-compliance/
   control-mapping-framework.md         <- this file
   _schema.md                            <- one file per control schema
   malaysia/
      bnm-rmit/                          <- one file per clause / appendix
         appendix-5-technology-risk-reporting.md
         section-10-data-loss-prevention.md
         section-11-cloud-services.md
         ...
      pdpa-2010-amended-2024/
      cybersecurity-act-854-2024/
   singapore/
      mas-trm/
      pdpa/
   hong-kong/
      hkma-sa-2/
   eu/
      gdpr/
      dora/
   international/
      iso-iec-27017-2015/
         cld-6-3-1-shared-responsibility.md
         cld-8-1-5-removal-of-assets-on-exit.md
         cld-12-4-5-monitoring-of-cloud-services.md
         ...
      iso-iec-27018-2019/
      nist-csf-2/
      nist-sp-800-53-rev5/
      pci-dss-v4/
```

Folder-level `_index.md` files describe per-jurisdiction scope. Per-clause files follow the schema above.

## Common mapping patterns

### Pattern 1 — One policy evidences multiple controls

A single MDA policy (e.g. Policy 5 — Block download to unmanaged) can evidence:
- ISO 27017 CLD.12.4.5 (monitoring of cloud services)
- BNM RMiT data leakage prevention clauses (verify against current edition)
- NIST CSF PR.AA-05 (access permissions/authorisations)
- PCI DSS Req. 7 (restrict access by need-to-know) — if PCI data is in scope

One mapping table per policy, referencing all evidenced controls.

### Pattern 2 — One control needs multiple policies

A single control (e.g. BNM RMiT third-party / outsourcing — verify against current edition) may need:
- Discovery policy (Shadow IT)
- OAuth governance policy
- Connector-state audit
- Vendor sub-processor list reference
- DR / fail-mode documentation

One mapping table per control, referencing all contributing policies.

### Pattern 3 — Control falls outside CASB scope

Some controls (e.g. NIST CSF GV.RM — risk management strategy) are organisational, not technical. CASB doesn't evidence them. State this clearly in the control file under "In scope for CASB? NO" with a pointer to where the control is owned.

### Pattern 4 — Vendor-side attestation needed

Some controls (e.g. ISO 27018 A.11 — processor obligations) are split: the firm side configures the policy, the vendor side attests the processor behaviour. For these, cite both your configuration AND the vendor's SOC 2 / ISO 27018 attestation from the Microsoft / vendor Trust Center.

## The audit-evidence package

A reusable evidence package per audit cycle:

| Section | Contents |
|---|---|
| **Control inventory** | List of controls in scope for this audit, with the mapping to policies / capabilities |
| **Policy state proof** | For each policy in scope: the policy export (JSON), the last-modified-date, the change-management ticket |
| **Operational log sample** | For each policy: a representative sample of SIEM event records (over the audit period) showing the policy firing |
| **Governance-action proof** | For confirmed-TP cases: the SIEM record, the governance action, the downstream effect (SCIM, label, suspension) |
| **Vendor attestation** | SOC 2 Type II report, ISO 27001 certificate scope, ISO 27017 / 27018 attestation — pulled from vendor Trust Center |
| **Sub-processor list** | Vendor's current sub-processor list with notification SLA |
| **Tenant data residency proof** | Trust Center / admin-centre screenshot or evidence of tenant primary-data region |
| **FP-rate metrics** | Documented per-policy false-positive rate for the audit period |
| **Exception register** | Active and expired exceptions with approval trail |
| **Incident response evidence** | If any CASB-detected incidents in the audit period: the IR runbook execution record, the notification clock met / missed |

Build the audit-evidence package incrementally — every quarter, refresh the sample log records and the policy-state snapshot. At audit time the package is mostly already assembled.

## Common mistakes

| Mistake | Consequence |
|---|---|
| **Citing vendor documentation as evidence** | Auditor rejection — vendor docs are product behaviour claims, not operational proof |
| **Quoting Microsoft Learn / Netskope docs without operational records** | Same — vendor self-attestation isn't independent evidence |
| **Map-once-and-forget** | Standards versions change (ISO 27017:2015 will eventually be superseded); regulator clauses get amended; mappings need refresh |
| **Map at the control-family level only** | Mapping to "ISO 27017 Section 9 — Access Control" is too coarse to defend; map at the clause level (CLD.9.5.1, CLD.9.5.2, etc.) |
| **Single-policy-evidences-everything claim** | A policy that "evidences" 20 controls usually evidences none of them well; the auditor will probe and find the depth issue |
| **No FP-rate measurement** | "We have a policy" without "we measure that it works" doesn't survive a control-effectiveness review |

## Sibling repos in scope

This framework defers operational-metric methodology to [`../../cyberkpis/`](../../cyberkpis/) — that repo owns the KPI / KRI / KCI / KGI distinction and the audience-layering. This file owns the CASB-specific mapping mechanics only.

For incident-response runbook structure that connects CASB detection to regulator-notification clocks, see [`../../CyberTTX/`](../../CyberTTX/) (sibling repo on tabletop exercise design — the runbook structure transfers).
