# 03-policies

The **policy catalogue.** One file per policy archetype. Schema-driven.

A *policy* is what the firm **decides to enforce** — built on top of capabilities. Capabilities (what the tool can do) live in `02-capabilities/`. Strict separation.

A policy is not "block Shadow IT." That's a slogan. A real policy entry says: *which apps, which users, which action, under which conditions, with which exception path, evidencing which control, at what false-positive cost.*

## Schema (to write at `_schema.md` when first entry lands)

Each policy entry carries:

- **Name** — canonical name.
- **Purpose** — one sentence — what risk does this reduce?
- **Action** — block / coach / monitor / quarantine / encrypt / step-up auth / require labelling / DLP-redact.
- **Scope** — which users (group / OU / role), which apps (sanctioned / tolerated / unsanctioned / specific app), which device posture (managed / unmanaged / BYOD), which network position (on-prem / remote / mobile), which traffic (upload / download / share / API call).
- **Required capabilities** — links to `02-capabilities/` entries this policy depends on.
- **Deployment-mode requirement** — forward-proxy / API / hybrid.
- **Control mappings** — links to `06-compliance/` entries this policy generates evidence for (BNM RMiT clause, ISO 27017 control, NIST CSF subcategory, etc.).
- **False-positive risk** — high / medium / low + named scenarios.
- **Operational cost** — exception-handling load, triage load, end-user friction.
- **Privacy / data-protection considerations** — does this policy require SSL inspection on personal traffic? Does it surface PII to admins? PDPA / GDPR implications.
- **Coverage gaps** — what this policy demonstrably misses (mobile native, unmanaged endpoints, third-party-to-third-party flows, encrypted payloads).
- **Three-lens sign-off** — Architect / Product / Compliance.

## Policy archetypes to draft (initial — confirm at thesis-lock)

- Unsanctioned-app block (with coach-and-redirect-to-sanctioned variant)
- Tolerated-app monitor-only with risk-score threshold
- DLP — PII upload prevention to sanctioned SaaS
- DLP — PCI data upload prevention to sanctioned SaaS (note: BNM expectations on cardholder data)
- DLP — source-code upload prevention to GenAI tools
- External-share-link quarantine for sensitive document classes
- OAuth-app grant restriction (publisher / scope / risk-score gating)
- Anomalous-download response (geo / volume / time-of-day)
- Compromised-account auto-response (session kill, MFA step-up, admin alert)
- GenAI-app access gating (block / coach / monitor / DLP-redact prompt content)
- Data-residency enforcement (block uploads to non-MY data regions for tagged data)
- BYOD-conditional access (read-only / no-download from unmanaged devices)
