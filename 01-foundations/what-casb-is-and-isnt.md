# What CASB is (and isn't)

> Neutral definition; the four-pillar Gartner model; what CASB does and does not cover. Vendor-agnostic.

## Definition (2026-current)

A **Cloud Access Security Broker (CASB)** is a control point that sits between users and SaaS applications and enforces the firm's security policy on the access. In 2026 the term names a *capability set* rather than a *product category* — those capabilities are mostly delivered inside broader Security Service Edge (SSE) platforms (Netskope, Zscaler ZIA, Palo Alto Prisma Access, Skyhigh) or inside identity / productivity suites (Microsoft Defender for Cloud Apps inside Microsoft Defender XDR + Entra).

The original Gartner formulation (2012) named four pillars. Those pillars remain useful as a vocabulary, but the housing has shifted.

## The four pillars

| Pillar | What it means | Where it lives in 2026 |
|---|---|---|
| **Visibility** | Know which SaaS apps your users are using (Shadow IT discovery); know which apps hold which data classes; know who is doing what in each app | Network-log-based discovery + API-based activity logging |
| **Data security** | Prevent regulated data from leaving sanctioned SaaS in violation of policy; classify data inside SaaS; control external sharing | Inline DLP (proxy mode), API-mode DLP (post-upload scan), sensitivity labelling integration |
| **Threat protection** | Detect anomalous activity (impossible travel, mass download, malware in SaaS storage, OAuth-grant abuse); respond | UEBA-style anomaly detection, malware scanning, OAuth governance |
| **Compliance** | Generate evidence for regulator and audit obligations; map controls to standards (ISO 27017/27018, NIST CSF, sector-specific) | Activity logs, alert records, governance-action evidence |

The pillars overlap with each other and with adjacent disciplines (SSPM, CSPM, CNAPP). The pillar model is a vocabulary, not a procurement checklist.

## What CASB **is not**

| Adjacent discipline | What it owns | How it overlaps with CASB |
|---|---|---|
| **SSPM (SaaS Security Posture Management)** | The *configuration* posture of a sanctioned SaaS — admin settings, role assignments, security baselines (e.g. Salesforce sharing settings, M365 attack-surface configuration). Vendor examples: Adaptive Shield, AppOmni, Obsidian, Wing | CASB watches the *traffic*; SSPM watches the *configuration*. CASB does some configuration assessment for connected apps (MDA does ~10 apps); SSPM specialists do deep coverage on many more |
| **CSPM (Cloud Security Posture Management)** | The configuration posture of IaaS — AWS / Azure / GCP misconfiguration | Different layer — CASB is for SaaS, CSPM is for IaaS infra |
| **CNAPP (Cloud-Native Application Protection Platform)** | The runtime + posture of cloud workloads — containers, serverless, VM | Different layer — CASB does not touch workloads |
| **SWG (Secure Web Gateway)** | URL filtering, malware scanning, generic web-traffic policy enforcement | CASB sits *after* the SWG in many topologies — SWG filters categories, CASB enforces per-app policy. In SSE platforms the two are merged |
| **ZTNA (Zero Trust Network Access)** | Replaces VPN with identity-conditioned access to private apps (typically on-prem or IaaS) | Different surface — CASB is for SaaS, ZTNA is for non-SaaS apps. Both are SSE components |
| **DLP at the endpoint / email** | Data classification + enforcement at the user's device or in transit through mail | Different control points — CASB is the *SaaS* control point, Endpoint DLP is the *device* control point, Email DLP is the *message* control point. Most regulated FIs need all three |
| **Identity Provider (IdP) + Conditional Access** | Authentication, MFA, session controls at sign-in | CASB *depends on* the IdP (inline mode is IdP-mediated); CASB is not an IdP replacement |
| **RBI (Remote Browser Isolation)** | Renders web pages in a remote sandbox; user sees a stream | CASB *can* trigger RBI as an action; some SSE platforms include native RBI. MDA does not |
| **EDR (Endpoint Detection and Response)** | Threat detection and response on devices | Different layer; CASB does not run on the endpoint (with the partial exception of agent-based discovery) |

This taxonomy matters because vendor proposals routinely conflate CASB with adjacent disciplines. A "CASB" that lists "device posture management" or "container security" is using the term loosely.

## What CASB **does** cover (the capability set in scope of this repo)

See [`02-capabilities/`](../02-capabilities/) for the catalogue. Summary:

- Shadow IT discovery
- Sanctioned-app inventory + risk-scoring
- Inline DLP for sanctioned SaaS (proxy / session mode)
- API-mode DLP for sanctioned SaaS (post-upload scan)
- OAuth-app discovery and governance
- Anomalous-activity detection
- Compromised-account detection
- Data-residency / geo-policy enforcement
- Share-link governance
- Malware scanning in SaaS storage
- Encryption / tokenisation at gateway (legacy / niche)
- Risk-based session policy
- Generative-AI-app discovery + governance (2024-2026 feature wave)

Different CASB / SSE platforms cover this set with different depth and different deployment modes. The cross-vendor view is in [`02-capabilities/_capability-matrix.md`](../02-capabilities/_capability-matrix.md).

## What CASB **does not** cover (the unavoidable gaps)

| Gap | Why CASB cannot |
|---|---|
| **Apps not behind the IdP or not API-connected** | If the user doesn't sign in via the IdP and the SaaS has no API connector, the CASB has no chokepoint |
| **Mobile native apps with their own auth flows** | Inline mode requires IdP-mediated routing; mobile native bypasses |
| **Certificate-pinned apps** | SSL inspection breaks the pin |
| **User-encrypted content** | Cannot inspect what you cannot decrypt |
| **Image / audio content** | DLP engines do not OCR / transcribe at session layer (yet) |
| **SaaS-to-SaaS API traffic** | Slack → Jira via integration; Salesforce → DocuSign; Workday → ServiceNow. All bypass the user-traffic chokepoint |
| **Personal accounts on shared devices** | A user signing into ChatGPT with a personal Gmail account is invisible |
| **Application-permission OAuth grants** | Most CASBs see only delegated grants; service-principal grants are a separate surface |

See [`08-failure-modes/`](../08-failure-modes/) for the deeper treatment of each.

## The 2026 reality

- **Pure-play CASB is mostly gone as a product category** — absorbed into SSE platforms. Stand-alone "CASB" products that aren't part of an SSE bundle are increasingly rare (Skyhigh is the notable holdout; ex-pure-play lineages elsewhere are now SSE features).
- **The CASB capability set has expanded** — GenAI app discovery and prompt-content DLP are 2024-2026 additions that didn't exist in the original Gartner formulation.
- **The deployment mode question is unchanged** — proxy / API / hybrid is still the framing, and the trade-offs are still real (see [`deployment-modes.md`](deployment-modes.md)).
- **The compliance pillar has gotten harder, not easier** — regulator expectations on cloud data, AI use, cross-border transfer have all sharpened. CASB evidence depth must match.

## Reading order

If you are new to the discipline:

1. This page (orientation).
2. [`deployment-modes.md`](deployment-modes.md) — how the proxy / API / log-based / hybrid choice is made.
3. [`casb-inside-sse-sase.md`](casb-inside-sse-sase.md) — where the housing has gone.
4. [`market-motion.md`](market-motion.md) — the pure-play → SSE-feature consolidation history.
5. [`../02-capabilities/_capability-matrix.md`](../02-capabilities/_capability-matrix.md) — cross-vendor coverage.
6. [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — one vendor at depth.
