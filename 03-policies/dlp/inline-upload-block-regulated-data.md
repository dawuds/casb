# Inline upload block — regulated data

> Status: v0.0 — MDA column from playbook v1; other vendors `[unverified]`.
> Required capabilities: [Inline DLP for sanctioned SaaS](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC) or forward-proxy. Entra ID P1 required for the MDA path.
> MDA playbook reference: [Policy 3](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Block upload of unlabelled files containing PII).

## Purpose

Prevent upload of files containing regulated data (PCI / PII / source / secrets) to sanctioned SaaS before the file lands. The pre-event prevention complement to API-mode DLP (post-event detect-and-remediate). Counters MITRE ATT&CK `T1530` and `T1567` on the upload leg.

## Action

- Primary: **block** upload
- Coaching variant: **block** + custom user message naming the regulated-data class detected and how to handle it (label first, use approved channel, etc.)

## Scope

- **Users:** all employees (excluding documented break-glass and exception groups)
- **Apps:** all CAAC-onboarded apps (M365 + extended onboarding); restrict to specific apps initially
- **Device posture:** any (the file content drives the decision, not the device)
- **Network position:** any (CAAC-mediated browser session)
- **Traffic:** upload (file or paste-content where supported)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create Session policy | Session control type = Control file upload (with inspection); App = CAAC-onboarded list; Inspection method = Microsoft Purview Data Classification Service; SIT = Credit Card Number / U.S. SSN / EU passport / custom; Minimum violations ≥ 5 (not 1); Action = Block + custom user message | Browser-only via CAAC; reverse-proxy; bilateral CAAC trap (CA policy + session policy required) | DCS regex on SSN matches many 9-digit strings (employee IDs, order numbers) — high FP. Always include `Always apply if data cannot be scanned` = false unless prepared for legitimate-encrypted-file blocks. Per-SIT confidence threshold tuning required (Match accuracy + minimum unique-match count, not just `minimum_violations = 1`) |
| Netskope | `[unverified]` — Netskope CASB Inline with native DLP | | Forward-proxy + reverse-proxy support; native DLP engine | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access inline DLP via Enterprise DLP add-on | | | |
| Skyhigh | `[unverified]` — Skyhigh multi-mode DLP | | | |
| Zscaler | `[unverified]` — ZIA inline DLP via Data Protection add-on | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage prevention](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- PDPA MY 2024 (Act A1709): personal-data protection at the upload boundary
- ISO 27017 control(s): [CLD.12.4.5 monitoring of cloud services](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.12 (information security)
- NIST CSF 2.0 subcategory(ies): `PR.DS-05` (protections against data leaks), `DE.CM-06` (external service-provider activity) `[VERIFY]`
- PCI DSS Req. 3 (protect cardholder data) where PCI in scope; Req. 4 (in-transit encryption)

## False-positive risk

- DCS regex on PII SITs fires on test fixtures, employee IDs, order numbers, sample data uploaded for legitimate analysis
- Excel spreadsheets with numeric columns matching SSN format — uploads of legitimate operational data blocked
- Format-shifting (PDF of a spreadsheet, image of a document) where DCS extraction is imperfect

## Operational cost

- **Exception-handling load:** high initially during tuning; medium steady-state
- **Triage load:** high — every blocked upload generates a triage event with the matching content preview
- **End-user friction:** medium — users see blocks on legitimate work until tuning closes the FP gap; coaching message + clear alternative path critical

## Privacy / data-protection considerations

- Content inspection on upload = high-intrusion processing under GDPR Art. 35 / equivalent; DPIA mandatory
- The 100-character matching-content preview snippet is itself regulated content — itself stored in the CASB's incident metadata, subject to PCI DSS 3.4 / 3.5 storage controls when matched content is cardholder data
- Workforce-notice posture required: users must understand content of their uploads is being inspected

## Coverage gaps

- [Encrypted-upload bypass](../../08-failure-modes/encrypted-upload-bypass.md) — user-encrypted archives, password-protected files, user-applied RMS labels evade content inspection
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — apps not CAAC-onboarded fall to post-upload API-mode only (or no detection)
- Image-format PII (photo of a document, screenshot of a spreadsheet) — DCS does not OCR at session layer
- Native desktop / mobile sync clients bypass — only browser sessions covered

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
