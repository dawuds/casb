# SSE stack integration

> How CASB composes with SWG, ZTNA, FWaaS, DLP-at-rest, and identity inside a Security Service Edge platform. Policy ownership boundaries. Vendor-agnostic.

## Component map

```
                        Identity Provider (Entra / Okta / Ping)
                                     |
                                     v
        +----------------------------+----------------------------+
        |                            |                            |
        v                            v                            v
   User device                  SSE platform                  Private apps
                                                                   ^
                                     |                             |
                                     v                             |
                              +-------------+                      |
                              |   FWaaS     |  L3-L7 firewall      |
                              +-------------+                      |
                                     |                             |
                                     v                             |
                              +-------------+                      |
                              |    SWG      |  URL category +      |
                              |             |  web malware         |
                              +-------------+                      |
                                     |                             |
                                     v                             |
                              +-------------+                      |
                              |   CASB      |  Per-app SaaS        |
                              |  (inline)   |  policy, DLP         |
                              +-------------+                      |
                                     |                             |
                                     v                             |
                              +-------------+                      |
                              | RBI (opt)   |  Risky-category      |
                              |             |  browser isolation   |
                              +-------------+                      |
                                     |                             |
                                     v                             |
                              Internet / SaaS                      |
                                                                   |
                              +-------------+                      |
                              |    ZTNA     |----------------------+
                              | (parallel)  |  Identity-conditioned
                              +-------------+  access to private apps

                              +-------------+
                              | CASB (API)  |  Out-of-band to sanctioned SaaS
                              +-------------+
```

The diagram is simplified. In production, the components may be ordered differently per vendor; some may be merged into single decision-engines; the order can be policy-configurable. The principle: each component owns a layer of policy decision.

## Component ownership per policy class

| Policy class | Owner |
|---|---|
| **URL category block** (gambling, adult, etc.) | SWG |
| **Web malware download block** | SWG (sometimes paired with CASB inline for sanctioned-SaaS file scanning) |
| **Block by app name (Shadow IT)** | SWG + CASB discovery feed |
| **DLP inside sanctioned SaaS upload** | CASB inline |
| **DLP inside sanctioned SaaS share / external link** | CASB API (post-event) |
| **Block clipboard paste to a SaaS app** | CASB inline (reverse-proxy or forward-proxy) |
| **Block download of Confidential file to unmanaged device** | CASB inline + IdP conditional access |
| **OAuth grant discovery / governance** | CASB API |
| **Anomalous activity detection (UEBA-style)** | CASB API + IdP signal |
| **Sign-in security (MFA, conditional access)** | IdP (upstream of all SSE components) |
| **L3-L7 firewall policy** | FWaaS |
| **Private-app access (VPN replacement)** | ZTNA |
| **Browser isolation (risky page sandbox)** | RBI |
| **DLP at endpoint (pre-encryption)** | EDR / Endpoint DLP (outside SSE) |
| **DLP in email body** | Email security / Purview DLP for Exchange |
| **Workload identity / service principal governance** | IdP (Entra Workload Identities) — outside CASB scope |

Document this mapping per-policy in your environment. The mapping is the source of truth for "who owns this decision" when an incident review asks.

## Why composition matters

| Symptom | Cause | Fix |
|---|---|---|
| Two policies firing on the same event with conflicting outcomes | SWG and CASB both have an opinion on the request | Document precedence; pick which component is authoritative for the policy class |
| Duplicate alerts in SIEM | Same event reported by both SWG and CASB | SIEM dedup rules; or accept the volume for evidence completeness |
| Unclear "what failed" in incident review | Multiple components in the path; no single decision point | Single-pane-of-glass logging in the SSE platform (where the platform supports it) |
| Gaps where neither component fires | Policy responsibility wasn't assigned | The component-ownership table catches this at design time |

## Vendor-platform unification

SSE platforms increasingly ship unified policy engines where CASB / SWG / ZTNA / FWaaS use the same policy language and produce the same event schema. Vendor-specific:

| Vendor | Unification level |
|---|---|
| **Netskope** | Strong — single policy engine, single dashboard for the SSE bundle |
| **Zscaler** | Strong — ZIA + ZPA share infrastructure; CASB inside ZIA |
| **Palo Alto Prisma Access** | Medium — unified at platform level; some component-specific consoles remain |
| **Microsoft (Defender for Cloud Apps + Global Secure Access + Defender XDR)** | Distributed — multiple consoles; unification at the SIEM layer (Sentinel) more than at the platform layer |
| **Skyhigh** | Less SSE-bundle-coupled than the leaders; integrates more loosely |

Multi-vendor SSE deployments (e.g. Microsoft for IdP + Defender XDR + MDA API + Netskope for SWG + inline CASB) require deliberate per-component policy ownership assignment.

## Identity is upstream of everything

All SSE components depend on the IdP signal. Inline CASB cannot enforce a session policy without knowing the user. SWG cannot apply user-specific URL category rules. ZTNA cannot authenticate. FWaaS user-level policy cannot apply.

Implication: **IdP architecture decisions dominate every SSE decision**. Get the IdP architecture wrong (multi-IdP without federation strategy, MFA gaps, conditional access drift) and every downstream SSE policy is unreliable.

| IdP architecture | Downstream impact |
|---|---|
| Single-IdP (Entra-only or Okta-only) | SSE integration straightforward; per-app onboarding scales |
| Dual-IdP (Entra + Okta legacy) | Per-app federation chain; inline CASB onboarding costs 2-4× higher for Okta-fronted apps |
| Three-or-more IdPs | Reconsider before SSE — the integration complexity is likely the dominant cost |

## What an SSE platform does NOT replace

| Capability | Where it lives |
|---|---|
| IdP / MFA / Conditional Access | Outside SSE (Entra / Okta / Ping) |
| EDR / Endpoint DLP | On the device |
| Email security (anti-phish, mail-body DLP) | Email security platform |
| SIEM / SOAR | Detection + response platform |
| Data classification governance (Purview / equivalent) | Information protection platform |
| SSPM (SaaS configuration posture) | Outside CASB scope |
| CSPM / CNAPP | Outside CASB scope |
| HRIS / IGA (joiner-mover-leaver process) | Identity governance |

A regulated FI's full security architecture includes all of the above plus the SSE stack. SSE is one significant component, not the whole.

## Single-pane-of-glass reality

Vendor pitches typically promise "single pane of glass" for SSE policy + alerts. Reality:

| Reality | Notes |
|---|---|
| Single policy console **within the SSE platform** | Mostly delivered by SSE-native vendors (Netskope, Zscaler, PAN) |
| Single console **across SSE + IdP + EDR + email + SIEM** | Not delivered by any vendor in 2026 |
| Single console **across multi-vendor SSE** (e.g. Microsoft + third-party) | Not delivered |
| Single SIEM unification | Achievable with deliberate connector configuration; not vendor-provided out-of-box |

Plan SOC operations around realistic multi-console reality, not the marketing single-pane promise.

## Concentration risk (re-stated)

Picking one vendor for SSE = picking one vendor for SWG + CASB + ZTNA + FWaaS. Multiply by one vendor for IdP + EDR + Email + SIEM and the firm is single-vendor for most of the security stack. This is operationally efficient but a concentration-risk pattern under BNM RMiT / MAS Outsourcing / EU DORA third-party recitals (verify against current editions).

Cover the concentration risk with:
- Independent SIEM with raw log forwarding (not the same vendor as the SSE platform)
- Independent IdP for break-glass
- Non-Microsoft signing key for critical secrets management
- Documented exit posture per vendor

## Failure modes when integration is broken

| Failure | Symptom | Cause |
|---|---|---|
| CASB policy doesn't fire on an app that traverses SWG | Component ownership unclear; SWG dropping traffic before CASB sees it | Document precedence; verify chain order |
| ZTNA path bypasses CASB inspection | ZTNA was meant to bypass CASB (private apps); but user accessed a SaaS via the ZTNA path | Network topology audit; explicit policy that ZTNA path is for non-SaaS only |
| User identity differs between SWG and CASB | Different IdP signal source | Unified IdP integration; verify both components see the same user object |
| Alert deduplication fails | Same event reported with different schema by SWG and CASB | Connector consolidation; SIEM-level normalisation |

## Anti-patterns

| Anti-pattern | Consequence |
|---|---|
| **Deploy CASB without considering SWG / ZTNA composition** | Coverage gaps at component boundaries |
| **Treat the SSE platform as a CASB replacement** | The CASB feature inside SSE has different depth than a pure-play; verify |
| **Skip the IdP discipline** | Inline policies unreliable; multi-IdP estate doubles or triples implementation effort |
| **Buy unified SSE without an exit strategy** | Year-3 vendor lock-in across multiple security components |
