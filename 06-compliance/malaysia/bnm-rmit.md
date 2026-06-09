# BNM RMiT — CASB control mapping (framework page)

> Bank Negara Malaysia, Risk Management in Technology (RMiT). The current edition is the source — verify against the gazetted document at https://www.bnm.gov.my and confirm with your assurance lead. **Material on regulatory clauses is illustrative; not legal/regulatory advice.**

## Status

This page is a **framework**, not the full clause-by-clause mapping. The full clause mapping is a separate compliance deliverable; this file documents the approach and lists the clause families a CASB / SSE deployment is expected to evidence in a regulated Malaysian FI.

## Why BNM RMiT matters for CASB

BNM RMiT is the dominant technology risk supervisory document for licensed Malaysian financial institutions. It covers (among many things):
- Technology risk management governance
- Data loss prevention
- Cloud services and outsourcing
- Cybersecurity operations
- Third-party / vendor risk
- Incident reporting

CASB controls sit primarily in **data loss prevention**, **cloud services**, **cybersecurity operations**, and indirectly in **third-party risk** (the CASB vendor itself is a third party).

## Clause families CASB is expected to evidence

The following clause families typically interact with CASB controls. **Section numbers and exact clauses vary across editions — always verify against the current gazetted RMiT.** This list is an orientation, not a substitute for reading the document.

| Clause family (current edition) | CASB relevance |
|---|---|
| **Technology risk reporting** (Appendix area) | CASB programme produces some of the required reporting inputs — incident reporting metrics, control effectiveness measures |
| **Data Loss Prevention** | Direct mapping: CASB inline + API DLP, share-link governance, classification labelling, OAuth-grant governance |
| **Cloud services** | CASB is the cloud-access control point; tenant residency, sub-processor disclosure, exit strategy, third-party risk for the CASB vendor itself |
| **Third-party / outsourcing arrangements** | CASB vendor itself is an outsourced cloud service — requires due diligence, exit strategy, audit rights, sub-processor transparency |
| **Cryptographic controls** | CASB-applied encryption (via sensitivity labels), SSL inspection PKI, BYOK / customer-key questions |
| **Cybersecurity operations** | Detection (anomaly, mass-download, OAuth abuse, terminated-user activity), SOC integration, alert handling, incident escalation |
| **Identity and access management** | CASB-IdP integration; conditional access; access reviews |
| **Logging and monitoring** | CASB activity logs, SIEM forwarding for retention, audit-trail integrity |
| **Incident management** | Detection-to-notification clock; CASB-detected incidents may trigger BNM reporting (verify reporting timelines against the current gazetted RMiT and current BNM circulars) |

## Citation discipline

For each control mapping created against BNM RMiT:

| Requirement | Detail |
|---|---|
| **Cite the section / appendix / clause** | Specific reference; do not cite "BNM RMiT" without sub-reference |
| **State the edition** | RMiT has been updated multiple times; the November 2025 edition is the current edition as of this writing — verify the edition you cite is current |
| **Confirm against the gazetted document** | The English text gazetted by BNM is authoritative; vendor docs and consulting summaries are secondary |
| **Flag any clause where supervisory expectation diverges from gazetted text** | Where applicable; consult assurance lead |
| **Material is illustrative — not legal advice** | Mandatory disclaimer in any client-facing material |

## Mapping pattern

For each CASB policy (see [`../03-policies/`](../../03-policies/) and per-vendor playbooks):

```markdown
## BNM RMiT mapping for Policy <name>

| Clause | Edition | Subject of clause | How this policy evidences it | Evidence artefact |
|---|---|---|---|---|
| <section.subsection> | <edition date> | <verbatim or paraphrased subject> | <policy mechanism> | <SIEM event class + retention + field> |
```

Build one mapping table per policy. Do not aggregate — each policy needs its own mapping for auditor traceability.

## What an auditor will probe under RMiT

| Question | Evidence required |
|---|---|
| Where does the firm's regulated data reside in the cloud? | Tenant residency proof; data-classification + storage-location map |
| What controls prevent regulated data from leaving sanctioned cloud services? | Per-policy mapping; operational log records |
| What is the firm's exit strategy from the CASB / cloud vendor? | Documented exit-cost estimate; vendor contract clauses (per [`../../07-implementation/cost-of-ownership.md`](../../07-implementation/cost-of-ownership.md)) |
| What due diligence was performed on the CASB vendor? | Trust Center attestations (SOC 2, ISO 27001, ISO 27017, ISO 27018); risk assessment record |
| What is the audit-rights provision in the CASB contract? | Contract clause; vendor's audit-cooperation commitment |
| What sub-processors are involved? | Vendor's published sub-processor list; change-notification SLA |
| How is incident detection-to-reporting clocked? | Per-incident class: SOC SLA + escalation runbook + reporting timeline (verify against current RMiT clauses) |

## Sub-processor and concentration risk

For a tenant whose CASB vendor is Microsoft (MDA) plus the SSE platform is also Microsoft (GSA) and the IdP is also Microsoft (Entra) — that's a concentration risk that BNM RMiT third-party / outsourcing sections will probe.

Compensating measures:
- Independent SIEM (not the same vendor as the SSE platform)
- Independent IdP for break-glass
- Documented exit posture
- Multi-vendor redundancy for critical paths where feasible

## Cross-border transfer

Defender for Cloud Apps tenants land in EU / US / UK / AU / Japan East (as of 2025 H2; India on roadmap H2 2026). For a Malaysian tenant, the primary-data region matters under:
- BNM RMiT cloud-services clauses (verify against current edition)
- PDPA Malaysia (2010 + 2024 amendments) cross-border transfer regime (verify against gazetted Act A1709)

Pull the tenant primary-data region from the Microsoft Trust Center on the day of compliance attestation. Practitioner-memory paraphrasing is not evidence.

## Reading order

- BNM RMiT current edition (gazetted document)
- [`../control-mapping-framework.md`](../control-mapping-framework.md) for the mapping methodology
- [`../iso-27017.md`](../iso-27017.md) for the international-standard equivalent (often easier to cite first; RMiT references international standards in places)
- [`../../07-implementation/programme-readiness.md`](../../07-implementation/programme-readiness.md) for the readiness gate
