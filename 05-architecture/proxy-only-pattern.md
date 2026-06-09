# Proxy-only deployment pattern

> Forward-proxy and reverse-proxy CASB topology. Traffic-steering options, coverage gaps, integration with the broader SSE stack. Vendor-agnostic.

## When proxy-only is the right choice

| Driver | Why proxy-only fits |
|---|---|
| Real-time DLP enforcement is mandatory (regulator or risk decision) | API mode is post-event; only proxy mode prevents pre-upload |
| Inline activity blocking required (block clipboard paste, block download, block print) | Only available in proxy mode |
| BYOD population is small or manageable via agent | Agent-based steering covers BYOD when present |
| Sanctioned SaaS list is small and stable | Per-app onboarding cost is bounded |

## When proxy-only is NOT the right choice

| Driver | Why |
|---|---|
| Heavy BYOD-mobile fleet | Mobile native apps bypass; proxy gives illusion of coverage |
| Cert-pinned business-critical apps | SSL inspection breaks them — silent bypass |
| Long-tail SaaS estate | Per-app onboarding doesn't scale |
| Personal-use-allowed corporate network | SSL-inspection privacy exposure |
| Multi-IdP estate | Multiple federation chains complicate inline routing |

For most of these, **hybrid** (proxy + API) is the right answer (see [`hybrid-pattern.md`](hybrid-pattern.md)).

## Forward-proxy reference topology

```
Managed endpoint (Windows/macOS)
   |
   | (1) Agent on endpoint OR PAC file in browser
   v
CASB SSE PoP (regional)
   |
   | (2) TLS termination + inspection
   v
CASB inline policy engine
   |
   | (3) Per-app session policies, DLP inspection, block/audit/allow
   v
SaaS application (sanctioned or otherwise)

Off-net devices:
   - Roaming agent maintains tunnel to nearest PoP
   - PAC file fails if browser cannot reach PAC server
   - Mobile BYOD: native apps bypass; mobile browser may or may not reach PoP
```

### Steering options

| Option | Coverage | Effort | Footgun |
|---|---|---|---|
| **Endpoint agent** | Best — works on/off-network, native + browser traffic | High — agent rollout, OS support matrix, coexistence with EDR/VPN | Multi-agent stacking causes endpoint perf issues |
| **PAC file** | Browser only; requires browser config | Low to configure, medium to maintain | Mobile + native apps + BYOD bypass |
| **IdP-mediated routing** | Sanctioned-app browser sessions only (reverse-proxy style) | Per-app onboarding | Browser-only; doesn't cover anything off the IdP path |
| **DNAT / SD-WAN / GRE tunnels from corp egress** | Corp-network traffic only | Network team effort | Off-net users bypass entirely |
| **DNS-based steering (DoH-aware)** | Coarse-grained; for category filtering | Low | Doesn't see HTTPS payload; rich SaaS policy unworkable |

### SSL inspection considerations

Forward-proxy CASB requires SSL inspection on app traffic. Decisions to make before deployment:

| Decision | Default position |
|---|---|
| Which apps are SSL-inspected? | All sanctioned + all unknown; bypass for banking, healthcare, government services |
| What's the CA story? | Internal CA, certificate distributed via MDM; managed devices trust it natively |
| What about BYOD? | Either selective inspection (BYOD bypasses sensitive categories) or no inspection on BYOD (loses much of the value) |
| Pinning audit | Test the inspection cert against the top 50 apps before go-live; document the bypass list |
| Privacy posture | Document the workforce-notice posture under PDPA MY 2024 / GDPR / HKPCPD; involve Legal/HR |

## Reverse-proxy reference topology

```
User signs in to sanctioned SaaS (browser)
   |
   v
IdP (Entra / Okta / Ping / Auth0)
   |
   | Conditional Access policy evaluates
   v
"Use Conditional Access App Control" session control fires
   |
   v
Sign-in redirects through CASB reverse-proxy endpoint (*.mcas.ms / vendor-equivalent)
   |
   | TLS termination at proxy
   v
URL-rewriting (subsequent SaaS URLs carry the proxy suffix)
   |
   v
CASB session-policy engine: block download / upload / paste / print / share
   |
   v
SaaS application (rewrites preserved through the session)
```

### Onboarding mechanics

| Stage | Activity |
|---|---|
| 1 | Enable the IdP-CASB integration (Entra-to-MDA, Okta-to-Skyhigh, etc.) |
| 2 | Configure the Conditional Access policy / IdP policy with "use session control" |
| 3 | Add the target SaaS app to the CASB's onboarded-apps list |
| 4 | For first-user touch, the app moves from "Discovered" to "Enabled" (vendor-specific terminology) |
| 5 | Build session policies for the app |
| 6 | Run compatibility regression test (per [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) Pre-CAAC compatibility test) |
| 7 | Pilot on a small user group, escalate to broader scope |

### What breaks under reverse-proxy

Per the MDA CAAC compatibility list, the dominant 2026 breakage class is **`window.parent.postMessage` cross-origin failure** in single-page applications. Named affected SaaS:

- Salesforce Lightning
- ServiceNow Polaris
- Workday Extend
- Box file-preview
- Atlassian Compass
- Confluence whiteboard

Each must be regression-tested before onboarding. Some configurations have no reverse-proxy workaround and must fall back to API mode.

## Integration with the broader SSE stack

Proxy-only CASB doesn't typically deploy alone in a regulated-FI architecture. It composes with:

| Component | Role | Composition |
|---|---|---|
| **SWG (Secure Web Gateway)** | URL category, web malware, generic web policy | Inline before CASB — SWG handles "no, you can't go to gambling.com" before CASB sees the traffic |
| **ZTNA (Zero Trust Network Access)** | Private-app access (VPN replacement) | Parallel path — ZTNA traffic doesn't pass through the CASB proxy |
| **DLP at endpoint** | Pre-encryption data classification + enforcement on device | Pair with CASB for pre-proxy DLP (catches what CASB cannot see when content is encrypted client-side) |
| **DLP in email** | Message-body DLP for Exchange / Workspace | CASB doesn't cover email body content; pair |
| **EDR** | Endpoint threat detection + response | Outside SSE; correlate alerts at SIEM |
| **IdP + MFA + Conditional Access** | Sign-in security | Upstream of CASB; the inline-mode story collapses without a clean IdP |
| **SIEM + SOAR** | Detection correlation and response orchestration | Downstream of CASB; canonical SIEM connector required |

The composition determines which control point owns which policy decision. Document this per-policy.

## Failure modes specific to proxy-only

| Failure | Symptom | Mitigation |
|---|---|---|
| BYOD-mobile coverage gap | Mobile users exfil sensitive data through native apps; no policy match | Pair with Intune MAM (App Protection Policies) for the mobile estate |
| Cert-pinned app breakage | Banking app refuses connection through proxy | Document bypass list; communicate to users |
| `window.parent.postMessage` SaaS app breakage | SaaS app silently malfunctions in CAAC session | Regression-test catalogue; fallback to API mode for affected apps |
| PAC file regional drift | Roaming users connect to wrong PoP, latency spikes | Auto-detection via vendor agent; PAC file fallback for non-agent devices |
| TLS-inspection compute saturation | Latency spikes during peak hours | Sizing exercise based on peak traffic; PoP capacity reservation |
| SSL-pinning certificate rotation | Internal CA expires; managed devices reject the cert mid-session | PKI lifecycle management; renewal monitored separately from CASB |
| Personal-use legal exposure | HR / Legal pushback on inspection of personal traffic | Privacy posture documented; selective bypass for personal categories; workforce notice |

## When to retire proxy-only for hybrid

A proxy-only deployment typically evolves to hybrid within 12-18 months as:

- Long-tail SaaS apps need governance but inline onboarding doesn't scale
- API-mode capabilities (auto-labelling, share-link governance) become attractive
- The team builds the regression-test discipline to run both modes

The shift is incremental — add API connectors to the existing proxy deployment, not a re-platform.

## Costs specific to proxy-only

| Cost line | Notes |
|---|---|
| **PoP latency** | Inline traffic adds RTT. Quantify per-region before regional rollout |
| **TLS inspection compute** | At scale, the inspection budget is non-trivial; vendor proposals often under-estimate |
| **Cert PKI maintenance** | Internal CA, distribution, rotation, audit. Often under-budgeted |
| **Per-app onboarding effort** | 1-3 days standard SaaS via Entra; 2-4 weeks for Okta-federated apps; some apps unsupportable behind reverse-proxy |
| **Regression-test discipline** | Per-app, per-SaaS-vendor-update. Ongoing |
