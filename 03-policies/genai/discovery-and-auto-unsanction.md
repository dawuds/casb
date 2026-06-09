# GenAI discovery and auto-unsanction

> Status: v0.0 — MDA column lens-reviewed from playbook v1 (Policy 8 — Day 90); other vendors `[unverified]`.
> Required capabilities: [GenAI-app discovery + Risk-based session policy + endpoint enforcement integration](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Log-based discovery + endpoint enforcement (MDE Network Protection in block mode) OR SWG GenAI-category block.
> MDA playbook reference: [Policy 8](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Auto-unsanction GenAI above risk threshold).

## Purpose

Continuously discover GenAI app usage; auto-unsanction apps in the Generative-AI category exceeding a risk threshold; propagate the unsanction to endpoint enforcement to block at the device. Reduces data leakage to consumer-tier GenAI. Counters MITRE ATT&CK `T1567 Exfiltration Over Web Service` (Detect on Discovery; Prevent only via MDE NP block).

## Action

- Primary: **tag as Unsanctioned + alert**; MDE Network Protection propagates the block to managed Windows / macOS endpoints
- Allowlist: approved AI vendors (ChatGPT Enterprise, Copilot, internal AI services) exempted by app ID

## Scope

- **Users:** all (excluding documented exception groups)
- **Apps:** GenAI sub-categories (LLM / AI Coding Assistant / AI Image Generator / AI Video Generator / AI Voice-Avatar) — the 2025 H2 sub-category split
- **Device posture:** managed endpoints with MDE Network Protection in block mode; BYOD bypasses
- **Network position:** corporate network + roaming via MDE agent
- **Traffic:** all (block by domain/IP via Network Protection)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Shadow IT → Create App discovery policy | Category filter = GenAI sub-categories (LLM, AI Coding Assistant, etc.) — use sub-categories, not the monolithic Generative AI; Risk score ≤ 5 (where lower = higher risk on Microsoft's 1-10 scale); Users per day > 10; Action = Tag as Unsanctioned + Alert; Allowlist approved AI vendors by app ID; **MDE Network Protection in block mode (not audit)** | Endpoint enforcement requires MDE deployed; Windows/macOS supported builds per current MDE NP supported-OS page; Endpoints reporting to same tenant | "Generate block script for SWG" produces flat IP/domain list that does not match modern GenAI vendor egress (CDN-fronted, JA3-fingerprinted, dynamic domains) — partial fail. Microsoft's risk score is opaque (uncited "90+ factors") — auditor cannot accept as sole basis for automated enforcement; document parallel internal risk-acceptance / appeal process. Approved-AI allowlist is a third-party concentration-risk decision — quarterly review + supply-chain-compromise contingency (ATLAS AML.T0010) required |
| Netskope | `[unverified]` — Netskope CCI + native SSE GenAI-category block | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access GenAI security | | | |
| Skyhigh | `[unverified]` — Skyhigh GenAI controls | | | |
| Zscaler | `[unverified]` — Zscaler ZIA GenAI category | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT third-party risk + DLP + AI governance overlay](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `GV.SC` (cybersecurity supply chain), `ID.AM-04` (services from suppliers), `PR.DS-05` (data leak protection), `DE.CM-09` `[VERIFY]`
- MITRE ATLAS coverage: AML.T0010 (ML Supply Chain Compromise) — partial via allowlist governance

## False-positive risk

- Approved-by-business AI vendor scoring ≤5 due to "SOC 2 = No" in Microsoft's catalogue but enterprise contract exists — exempt by app ID
- New legitimate AI tools in catalogue Unknown class — may be tagged Unsanctioned during legitimate trial
- Image / video / voice AI generators lumped with LLMs in the monolithic category — different risk class, different policy approach

## Operational cost

- **Exception-handling load:** medium — quarterly allowlist review + new-vendor onboarding
- **Triage load:** low — block at endpoint is silent
- **End-user friction:** high if blocked AI was in legitimate use — coaching message + sanctioned alternative essential

## Privacy / data-protection considerations

- AI vendor discovery includes user identity tied to AI app usage — workforce-monitoring data
- Approved-AI vendor's data-handling posture matters: training-data policy, customer-isolation, jurisdictional location
- DSPM-for-AI prompt capture (separate policy) requires DPIA

## Coverage gaps

- BYOD / hotspot — off-corp network, MDE NP not in path
- Personal MSA or personal Google sign-in to ChatGPT — Entra never sees session
- CDN-fronted vendor domains (T1090.002, T1568.002) — SWG block script defeated by class
- Image / audio prompts — DCS does not OCR / transcribe at session layer
- Supply-chain compromise of an approved AI vendor (AML.T0010) — allowlist becomes the attack surface
- 2026 alternatives: Microsoft Edge for Business `gen-ai.app-control` (H2 2026 GA — verify) + GSA native GenAI category block supersede MDA→MDE NP chain for Edge-on-Windows estates

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
