# BYOD and unmanaged endpoint coverage gap

> The most under-acknowledged CASB failure mode. Vendor marketing implies coverage; the technical reality is partial at best.

## The failure

A user accesses a sanctioned SaaS app from a personal device — laptop or mobile. The CASB cannot enforce inline policy on most of this traffic. The result: the data-loss prevention surface is full of holes wherever the user is not on a managed device.

## What enables it

| Mechanism | Why it bypasses CASB |
|---|---|
| **No agent on the personal device** | Forward-proxy CASB depends on agent/PAC traffic steering. Personal devices don't have the agent |
| **Native mobile apps with own auth flows** | Reverse-proxy CASB (CAAC / equivalent) intercepts IdP-mediated sign-in. Native mobile apps that don't authenticate via the IdP bypass entirely |
| **Browser sessions on non-corporate networks without PAC** | PAC file isn't deployed on the personal device's browser |
| **Cert-pinned mobile apps** | SSL inspection breaks the pin; apps refuse the connection or silently bypass |
| **Mobile SDK auth flows** | Many mobile SaaS apps use OAuth flows that don't traverse the corporate IdP |

The result: a BYOD-heavy environment has policy coverage on roughly the managed Windows / macOS browser sessions and almost nothing else.

## Which vendors exhibit it

| Vendor | BYOD posture |
|---|---|
| **MDA** | Severe — no Defender-for-Cloud-Apps endpoint agent. Coverage is browser-via-CAAC on managed devices only. Mobile native is essentially uncovered |
| **Netskope / Zscaler / Palo Alto** | Better than MDA — endpoint agents available; mobile clients exist; still limited for cert-pinned apps and non-agent BYOD |
| **Skyhigh** | Multi-mode heritage; agent + agentless coverage; still hits the cert-pinning wall |

Vendors with endpoint agents do better than vendors without, but none claim full BYOD coverage honestly.

## Why the failure exists (technical)

CASB enforcement requires a chokepoint. The chokepoints are:

1. **An endpoint agent** — sees all traffic but requires installation; impossible on truly-personal-devices
2. **An IdP-mediated proxy** — sees only browser sessions that go through the IdP
3. **A network egress proxy** — sees only traffic from the corporate network
4. **An API connector** — sees post-event activity, not pre-event prevention

For BYOD on non-corporate network using native mobile apps with their own auth, none of these chokepoints exist. There is nothing for the CASB to do.

## What compensates (NOT another CASB feature)

The compensating controls are at different layers:

| Layer | Control |
|---|---|
| **Identity** | Entra Conditional Access requiring approved client app; device-compliance grant on the IdP layer |
| **Mobile** | **Intune App Protection Policies (MAM)** with copy-paste / save-to-personal-cloud restrictions inside the managed app — this is the actual answer for the mobile case |
| **Data** | Microsoft Purview labels with encryption — protects the file regardless of where it ends up |
| **Network** | Block BYOD entirely from sensitive resources (the heavy-handed answer) |
| **Process** | Read-only access from BYOD; require download from a managed device |

The pattern: CASB + MAM + label encryption together cover the BYOD case. CASB alone does not.

## Public-incident notes

The 2022-2024 wave of corporate-data leaks via personal-Gmail-signed-in-to-ChatGPT illustrated the BYOD-style coverage gap at scale. Users on corporate networks but on personal accounts to consumer-tier SaaS bypassed every network-layer control because the auth was through a personal IdP that the firm's CASB / SSE never saw.

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "CASB protects our BYOD users" | It doesn't, mostly |
| "We have mobile coverage via the CASB agent" | Mobile agent coverage is partial; cert-pinned apps + own-auth-flow apps bypass |
| "Reverse-proxy CASB sees all browser sessions" | Only those that authenticate via the integrated IdP |
| "If they're on the corporate network, we cover them" | Only if their traffic flows through the CASB; personal device on the same network may not |

## Promotion candidate

This page is a promotion candidate to a public-wedge piece if the repo wedge ever expands to include the public failure-mode catalogue ([`CLAUDE.md`](../CLAUDE.md) candidate wedge "Failure-mode catalogue"). Currently the locked wedge is the practitioner-per-vendor reference, so this is a residual-risk reference, not the public artefact.
