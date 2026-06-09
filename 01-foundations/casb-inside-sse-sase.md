# CASB inside SSE / SASE

> The 2026 reality: CASB capabilities are mostly delivered as features inside Security Service Edge (SSE) platforms, which are themselves part of Secure Access Service Edge (SASE) architectures. Vendor-agnostic.

## The acronym stack

| Acronym | What it is | Components |
|---|---|---|
| **SASE** (Secure Access Service Edge — Gartner 2019) | Network + security delivered as a cloud service. Combines SD-WAN with cloud-native security | Network: SD-WAN. Security: SSE |
| **SSE** (Security Service Edge — Gartner 2021) | The security subset of SASE; cloud-delivered network security | SWG + CASB + ZTNA + FWaaS (+ RBI, DEM, others) |
| **SWG** (Secure Web Gateway) | URL filtering, web malware scanning, generic web policy | One component of SSE |
| **CASB** | The SaaS-access control layer (this repo's scope) | One component of SSE |
| **ZTNA** | Identity-conditioned access to private apps (typically VPN replacement) | One component of SSE |
| **FWaaS** | Cloud-delivered firewall (typically L3-L7) | One component of SSE |
| **RBI** | Remote Browser Isolation | Often a sub-feature; sometimes a standalone product |
| **DEM** | Digital Experience Monitoring | Measures user-perceived performance of cloud-delivered apps; newer SSE component |

The acronyms came in waves; the underlying re-architecture is one trend: **move security from on-prem appliances to cloud-delivered services**.

## Where CASB sits

In an SSE / SASE deployment, CASB is **one of several control points** on the same traffic flow:

```
User device
   |
   v
   SD-WAN / VPN / agent / PAC                <- network steering
   |
   v
   SSE platform edge POP
       |
       v
       FWaaS              (L3-L7 firewall)
       |
       v
       SWG                (URL category, web malware)
       |
       v
       CASB inline        (SaaS-app-specific session policy, DLP, etc.)
       |
       v
       (optional RBI)     (browser isolation for risky categories)
       |
       v
   Internet / SaaS

   In parallel:
   ZTNA          (identity-conditioned access to private apps; bypasses SaaS path)
   CASB API      (out-of-band API connectors to sanctioned SaaS — works on events, not traffic)
```

Each control point is policy-owned, not just network-owned. The CISO / Risk lead decides which control point owns which decision; SSE platforms make consolidation easier but don't replace the policy work.

## The convergence story (2018-2026)

| Era | What was sold separately |
|---|---|
| **2012-2018** | Pure-play CASB (Adallom, Bitglass, Netskope, Skyhigh Networks, CipherCloud) — standalone product category |
| **2017-2020** | First wave of CASB acquisitions — Adallom→Microsoft (2015), Skyhigh→McAfee (2017), CloudLock→Cisco (2016), CipherCloud→Lookout (2021). CASB starts becoming a feature, not a product |
| **2019-2022** | SASE coined (2019); SSE coined (2021). Cloud-delivered security category emerges. Vendors with multiple components (CASB + SWG + ZTNA) reposition |
| **2022-2026** | SSE platforms dominate. Pure-play CASB largely gone. CASB capabilities are baseline expectations of any SSE vendor |

The drivers were real:
- Distributed workforce (post-2020) made on-prem security appliances structurally insufficient
- Cloud-native architecture (SaaS, IaaS) made on-prem inspection points the wrong shape
- Consolidation pressure (vendor count fatigue) drove buyers toward platform vendors

## Vendor positioning in 2026

| Vendor | SSE positioning | CASB depth |
|---|---|---|
| **Netskope** | SSE-leader (Gartner — analyst opinion, not measurement). CASB-native heritage | Deep — CASB Inline + CASB API are both first-party with long heritage |
| **Zscaler** | SSE-leader. SWG / ZTNA-heritage; CASB built later | Medium-deep — CASB is a feature inside ZIA; SaaS Security Posture and CASB API are separate sub-products |
| **Palo Alto Prisma Access** | SSE-leader. NGFW vendor extending into SSE | Medium — Inline CASB via Prisma Access; API CASB via SaaS Security (formerly Aperture, formerly CirroSecure) |
| **Microsoft Defender for Cloud Apps** | Not an SSE platform — sits alongside Microsoft Global Secure Access (the actual SSE play) | Deep API + reverse-proxy (CAAC); no forward-proxy. Heavy Entra + Defender XDR integration |
| **Skyhigh Security** | SSE-positioning since the Trellix spin-out (2022) | Deep — last credible stand-alone-CASB lineage; multi-mode (proxy + API + log-based) heritage |
| **Cisco Cloudlock / Umbrella** | SSE-positioning via the Umbrella platform | Lighter — Cloudlock is API-mode focused; not deep on inline |
| **Forcepoint ONE** | SSE platform (built on the Bitglass acquisition) | Medium — Bitglass heritage in reverse-proxy |
| **iboss, Lookout, Symantec/Broadcom** | Each present in SSE conversations; depth varies | Variable |

Vendor positioning is analyst-influenced and shifts annually. The above is a 2026-current snapshot; verify against current Gartner / Forrester / IDC publications when making procurement decisions.

## Implication for the policy library

If you're designing CASB policies in 2026, you must decide which control point owns the decision:

| Policy class | Most natural control point |
|---|---|
| **URL category blocking (gambling, adult, etc.)** | SWG layer |
| **Block-app-by-name (Shadow IT)** | SWG + CASB discovery layer |
| **DLP inside sanctioned SaaS** | CASB inline (proxy) or CASB API |
| **OAuth grant governance** | CASB API |
| **Anomalous-activity detection** | CASB API + IdP signal |
| **In-session activity block (paste, print, share)** | CASB inline (reverse-proxy or forward-proxy) |
| **Private-app access (VPN-replacement)** | ZTNA layer (not CASB) |
| **Network L3-L7 firewall** | FWaaS layer |
| **Browser isolation for risky pages** | RBI layer (sometimes inside SWG, sometimes separate) |
| **Endpoint-side DLP** | EDR / Endpoint DLP (outside SSE) |

The platform consolidation argument is that having all these in one vendor's stack reduces seam complexity. The counter-argument is concentration risk (see [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — concentration-risk section).

## What still doesn't fit cleanly into SSE

| Adjacent | Why it sits outside |
|---|---|
| **SSPM (SaaS configuration posture)** | Different control loop — SaaS *configuration* vs SaaS *traffic*. Most SSE platforms include shallow SSPM; specialists go deeper |
| **DSPM (Data Security Posture Management)** | Data classification across cloud data stores; intersects with CASB DLP but isn't the same |
| **CNAPP (cloud workload posture)** | Different layer — VM / container runtime; not CASB scope |
| **EDR (endpoint detection)** | On-device; not in the SSE traffic flow |
| **Email security** | Different transport layer; some SSE platforms add it (Mimecast-style functionality) but it's a separate discipline |
| **Identity Provider** | Upstream of everything; SSE depends on IdP, doesn't replace it |

The "everything in one platform" SSE pitch is real for the SSE-native components; the adjacent disciplines need separate decisions.

## The CASB-inside-SSE design questions

When designing a CASB-using-SSE deployment, the questions worth answering early:

1. **Which control point owns each policy?** Document, by policy, where the decision is made and where the audit evidence lives.
2. **How do the components compose?** SWG + CASB on the same traffic flow — which fires first, which has the last word, what does the SIEM see (one event or three)?
3. **What is the failure mode if one component degrades?** If CASB inline is down but SWG is up, what does the user experience? Fail-open or fail-closed?
4. **What is the unified audit-evidence story?** SSE platforms ship unified logs; verify the unification is real and the schema is stable.
5. **What is the platform-lock-in cost?** Switching SSE vendors is non-trivial — policy languages don't port, classifier definitions are vendor-specific. Document the exit posture before signing the EA.

## Reading order from here

- [`market-motion.md`](market-motion.md) — the consolidation history in more detail.
- [`deployment-modes.md`](deployment-modes.md) — the proxy / API / log-based trade-offs (mostly orthogonal to SSE-or-not).
- [`../05-architecture/`](../05-architecture/) — reference architectures showing CASB inside SSE.
