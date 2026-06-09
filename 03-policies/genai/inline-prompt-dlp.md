# Inline prompt DLP

> Status: v0.0 — MDA column lens-reviewed from playbook v1 (Policy 9 — Day 90); other vendors `[unverified]`.
> Required capabilities: [Risk-based session policy + Inline DLP + GenAI app governance](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC). Entra ID P1 required. SAML federation prerequisite on the AI vendor side.
> MDA playbook reference: [Policy 9](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Block clipboard paste of sensitive data into ChatGPT).

## Purpose

Block clipboard paste (and optionally Cut / Copy / Sent message) of cardholder data, SSN-class PII, source code, internal project codenames into sanctioned GenAI browser sessions. Counters MITRE ATT&CK `T1567 Exfiltration Over Web Service` narrowed to clipboard sub-channel (Prevent — partial). **Not T1052 (Physical Medium)** — the original draft's mapping was wrong.

## Action

- Primary: **block** with custom user-education message
- Tuning variant: **audit** action initially, escalate to **block** per SIT after FP tuning

## Scope

- **Users:** all employees (excluding documented exceptions)
- **Apps:** ChatGPT Enterprise / Team (SAML-federated only); Copilot Chat; other GenAI tools that support tenant SAML federation
- **Device posture:** managed (CAAC reverse-proxy only sees browser sessions on Entra-federated devices)
- **Network position:** any (browser session via Entra)
- **Traffic:** clipboard Paste (and optionally Cut / Copy / Sent message)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create Session policy. Plus matching Entra CA policy with "Use Conditional Access App Control" | Session control type = Block activities; App = Custom ChatGPT Enterprise (Manual onboarding); Activity type = Clipboard Paste (+ Sent message with content inspection); Inspection method = DCS; SIT = Credit Card, SSN, PII + custom SIT for internal project codenames; Action = Block with custom user-education message; start with Audit, escalate to Block per SIT after FP tuning | **ChatGPT must be onboarded to CAAC via SAML federation. Consumer ChatGPT (chat.openai.com with free or personal Plus account) does NOT support SAML.** Deployable ONLY against ChatGPT Enterprise or ChatGPT Team plan with SSO configured (verify against OpenAI's current SSO-by-plan page on day of deployment) | Clipboard inspection is **high-intrusion processing** under GDPR Art. 35 / equivalent — DPIA before pilot. Audit log may retain truncated content snippets — itself regulated content. At 5k seats, clipboard inspection inflates Sentinel ingest by ~100 GB/month. Policy only covers the browser session on CAAC-onboarded ChatGPT URL — opening ChatGPT in different browser / desktop app (Mac/Windows clients exist) / personal device / personal account = bypass |
| Netskope | `[unverified]` — Netskope inline GenAI policy with prompt-content inspection | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access inline GenAI controls | | | |
| Skyhigh | `[unverified]` — Skyhigh inline GenAI | | | |
| Zscaler | `[unverified]` — Zscaler ZIA + Data Protection inline GenAI | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage + AI governance overlay](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- PDPA MY 2024: personal-data protection at the prompt boundary
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.DS-05` (data leak protection), `DE.CM-09` `[VERIFY]`
- MITRE ATLAS: AML.T0024 (Exfiltration via ML Inference API) — partial via clipboard inspection

## False-positive risk

- Clipboard content inspection produces a lot of noise initially — code snippets, 16-digit identifiers, employee IDs all trigger Credit Card SIT
- Internal-project-codename SIT may fire on legitimate publications and external comms

## Operational cost

- **Exception-handling load:** medium during tuning; lower steady-state
- **Triage load:** high during audit phase
- **End-user friction:** medium — users see paste blocks; coaching message + sanctioned alternative essential

## Privacy / data-protection considerations

- Clipboard inspection on the user's browser is high-intrusion processing — DPIA mandatory under GDPR Art. 35 / equivalent
- Truncated content snippets in audit log = regulated content — itself subject to storage controls
- Workforce-notice posture: users must understand clipboard activity is inspected when accessing sanctioned AI tools
- Sentinel ingest cost line — not just privacy — at scale

## Coverage gaps

- Image / screenshot / audio of the same data (DCS does not OCR / transcribe at session layer)
- **Typing instead of pasting (no paste event fires)** — fundamental bypass class
- Desktop ChatGPT apps (Mac / Windows clients now exist)
- Personal account on sanctioned-app domain
- Non-Edge browsers without CAAC routing
- 2026 alternative: Microsoft Edge for Business `gen-ai.app-control` (H2 2026 GA) — browser-layer enforcement without SAML-federation prerequisite, for Edge-on-Windows estates

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
