# GenAI app overlay

> The 2024-2026 feature wave: discovering and gating Generative-AI apps via CASB; prompt-content DLP; vendor-by-vendor capability variance. The category is moving fast; expect this page to age quickly.

## The problem

Employees paste source code, customer data, financial models into ChatGPT / Bard / Claude / Perplexity / Copilot. Consumer-tier accounts may train on the content. Once leaked, the data is in the model. The firm's data-protection programme needs to address this without blocking productive AI use.

## What CASB can do (in 2026)

| Capability | Notes |
|---|---|
| **Discover GenAI app usage** | All major vendors have a Generative-AI category in their cloud-app catalogue; volume + user-count per app |
| **Risk-score per GenAI app** | Vendor-opaque scoring; based on terms-of-service, training-data policy, compliance certifications, geographic footprint |
| **Sanction taxonomy** | Tag-as-sanctioned / tolerated / unsanctioned; auto-unsanction above risk threshold |
| **Block at endpoint via MDE Network Protection** (Microsoft stack) | Requires MDE deployed + NP in block mode + Windows / macOS supported version |
| **Block at SWG layer** | All SSE platforms have GenAI-category blocking |
| **Inline DLP on browser sessions to sanctioned GenAI** | Block clipboard paste + sent message + uploaded file when content matches DLP signatures |
| **DSPM-for-AI prompt visibility** (Microsoft Purview) | The actual prompts and responses are captured for ChatGPT Enterprise / Copilot Chat + supported AI sites |

## Capability depth by AI tool

The depth of prompt-content inspection varies by AI tool:

| AI tool | Deep inspection (prompt + response body capture) | Shallow inspection (metadata only) | Not inspectable |
|---|---|---|---|
| **ChatGPT Enterprise / Team (SSO'd)** | Yes (via Purview DSPM-for-AI; SSO required) | | |
| **Microsoft 365 Copilot / Copilot Chat / Copilot Studio** | Yes (native Purview integration) | | |
| **Anthropic Claude (claude.ai web, SSO'd Enterprise)** | (Vendor-dependent — some support; some shallow) | (Most CASBs: metadata only) | |
| **Google Gemini for Workspace** | (Limited deep capture) | Yes | |
| **Perplexity Enterprise** | (Limited deep) | Yes | |
| **Consumer ChatGPT (no SSO)** | | | Yes — bypasses |
| **Personal-account access to any consumer AI** | | | Yes — bypasses |
| **ChatGPT / Claude / etc. on personal device, off-network** | | | Yes — bypasses |

For deep inspection, the SSO federation is the gate. Without SSO, the AI tool is essentially uncovered by CASB.

## Reference policy architecture for GenAI

### Layer 1 — Sanction taxonomy

| Category | Default treatment |
|---|---|
| **Enterprise AI tools with SSO + tenant-isolated data** (ChatGPT Enterprise, Copilot, Gemini for Workspace) | Sanctioned — allow with DLP guardrails |
| **Enterprise AI tools without SSO support** | Tolerated — monitor; consider inline DLP if browser-session-onboardable |
| **Consumer AI tools** | Unsanctioned — block at SWG / MDE NP; coach users to use the sanctioned alternatives |
| **AI image / video / audio generators** | Per-tool decision; image generators bypass clipboard DLP (output is not text) |
| **AI coding assistants** (GitHub Copilot, Cursor, etc.) | Sanctioned with source-code protection policy |

### Layer 2 — Discovery and risk-scoring

| Activity | Frequency |
|---|---|
| Discovery of new GenAI apps in tenant | Continuous |
| Risk-score review of newly-discovered apps | Weekly |
| Sanction decision | Per-app; documented |
| Auto-unsanction above risk threshold (with allowlist for known-approved) | Continuous |

### Layer 3 — DLP guardrails on sanctioned AI

| Policy | Action |
|---|---|
| Block clipboard paste of cardholder data / SSN / source-code into ChatGPT Enterprise | Block at session policy with DLP inspection |
| Block file upload of Confidential-labelled files to AI tools | CAAC session policy |
| Audit all prompt content matching sensitive-data signatures | DSPM-for-AI logging |
| Alert on prompt patterns matching internal project codenames | Custom SIT in DLP |

### Layer 4 — Block unsanctioned

| Path | Control |
|---|---|
| Endpoint (Windows + MDE) | NP in block mode; tag-as-unsanctioned auto-propagates |
| Endpoint (non-MDE) | SWG block; PAC routing if available |
| Mobile native AI apps | MAM block; MDM-blacklist if managed |
| Off-network personal-device personal-account | No control; deterrent only |

### Layer 5 — Approved-AI allowlist governance

| Discipline | Cadence |
|---|---|
| Documented allowlist owner | Single named owner |
| Quarterly allowlist review | Approved AI vendors remain approved; risk-score check; supply-chain compromise contingency |
| Kill-switch posture | Documented response if an approved AI vendor is compromised (Storm-class supply-chain) |
| ATLAS AML.T0010 (ML Supply Chain Compromise) consideration | Each approved vendor a third-party concentration-risk decision |

## What CASB cannot do for GenAI

| Gap | Compensating control |
|---|---|
| **Personal-account sign-in to consumer AI** | None at CASB layer; deterrent + Acceptable Use Policy |
| **Image / audio prompts** | DLP engines don't OCR / transcribe at session layer |
| **Voice / video AI assistants** | Same — not text-channel |
| **Typing instead of pasting** (defeats clipboard inspection) | No control; can only detect pasted content |
| **Native AI desktop apps** (Mac / Windows ChatGPT clients) | MAM if managed; otherwise bypasses CAAC |
| **AI accessed via API** (developer integration) | Application-permission OAuth governance + secrets management |
| **AI agents invoking OAuth on user's behalf** (T1528-via-agent) | Currently undetected technique class; emerging gap |
| **Indirect prompt injection causing OAuth grant escalation** (ATLAS AML.T0051) | Emerging gap; no clear control |

## 2026 platform alternatives to CASB-only GenAI control

| Platform | What it adds |
|---|---|
| **Microsoft Edge for Business `gen-ai.app-control`** (H2 2026 GA, verify) | Browser-layer enforcement without the SAML-federation prerequisite of CAAC; covers Edge-on-Windows estates |
| **Microsoft Global Secure Access (GSA) native AI category enforcement** | SSE-native GenAI controls; alternative path to MDE NP block for unsanctioned AI |
| **Microsoft Purview Adaptive Protection for GenAI** (2026) | Risk-adaptive DLP — strictness adjusts based on user risk score |
| **Vendor-specific: Netskope / Zscaler / Palo Alto / Skyhigh** | Each has shipped GenAI-specific features in 2024-2026; capability varies; verify against current vendor docs |

For Edge-on-Windows estates licensed for GSA, prefer those over the MDA → MDE NP chain — fewer dependencies, no audit-mode footgun.

## Privacy treatment for GenAI

| Concern | Posture |
|---|---|
| **Prompt content is personal data when it contains user identifiers** | DSPM-for-AI prompt capture stores this; treat as regulated content |
| **Response content may be personal data of third parties** | When the user asks the AI about someone, the response may identify them |
| **Workforce notice for prompt logging** | Required under PDPA / GDPR / equivalent; clipboard inspection is high-intrusion processing |
| **DPIA for DSPM-for-AI prompt capture** | Mandatory for high-risk processing |
| **Cross-border transfer of prompt content** | Prompt content may flow through the AI vendor's infrastructure outside the firm's tenant region |

Document the privacy posture for GenAI controls before pilot. The 2024-2026 wave of AI deployment landed faster than the privacy regulation; the gap is closing rapidly.

## Operating model

| Activity | Cadence | Owner |
|---|---|---|
| Newly-discovered GenAI app review | Weekly | Security + IT |
| Approved AI vendor allowlist review | Quarterly | Risk + IT |
| GenAI DLP policy FP-rate tuning | Monthly | Platform team |
| DSPM-for-AI prompt-content review | Per incident | Risk + Privacy |
| Supply-chain compromise contingency for approved AI vendors | Tabletop annually | IR + Risk |

## Reading order

- [`../03-policies/`](../03-policies/) — `genai-access-gating.md` and related policy entries
- [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — Policy 8 (auto-unsanction GenAI) and Policy 9 (clipboard block to ChatGPT) as worked examples
- [`../08-failure-modes/`](../08-failure-modes/) — `over-blocking-and-user-circumvention.md` for the "block consumer AI" failure mode
