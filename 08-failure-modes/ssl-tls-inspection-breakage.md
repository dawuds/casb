# SSL / TLS inspection breakage

> Forward-proxy CASB requires SSL inspection. SSL inspection breaks things. The breakage is silent, partial, and politically contentious.

## The failure

To enforce inline DLP on encrypted SaaS traffic, the CASB proxy terminates TLS, inspects the cleartext, then re-establishes TLS to the SaaS. This works only when:

- The endpoint trusts the proxy's CA certificate
- The SaaS app doesn't pin its certificate
- The user accepts that personal-use HTTPS is being inspected
- The inspection compute keeps up at peak load

When any of those is false, the inspection either silently fails or breaks user-visible functionality.

## What enables it

| Mechanism | Breakage |
|---|---|
| **Certificate pinning** | Many mobile banking apps, vendor SDK clients, and some desktop apps refuse to trust the inspection cert. The app silently fails, bypasses, or refuses to connect |
| **App-server certificate transparency monitoring** | Some SaaS apps detect that their certs are being substituted and refuse the session |
| **Cross-origin protections** | Browsers and some SaaS apps detect TLS-MITM-style behaviour and apply security restrictions |
| **Personal-use HTTPS** | Inspection of personal banking, medical portals, government services creates legal and HR exposure |
| **TLS 1.3 + ESNI / ECH** | Newer TLS features intentionally resist inspection; CASB vendors are catching up |
| **Encrypted Client Hello** | Hides SNI; complicates routing decisions before TLS handshake |

## Which vendors exhibit it

All vendors with forward-proxy or reverse-proxy capability:

| Vendor | Inspection model |
|---|---|
| **Netskope / Zscaler / Palo Alto / Skyhigh** | Forward-proxy with SSL inspection; vendor-managed CA cert distributed via MDM; bypass list maintained per-tenant |
| **MDA** | Reverse-proxy (CAAC) only — TLS termination at the `*.mcas.ms` endpoint. Doesn't suffer the forward-proxy SSL-inspection problems but inherits the cert-pinning failure on managed devices with strict pinning profiles |

Reverse-proxy reduces the inspection footprint (only inspecting onboarded apps) but doesn't eliminate the pinning issue.

## Why the failure exists (technical)

TLS was designed to resist interception. SSL inspection is, technically, a controlled MITM. The TLS designers added pinning specifically to prevent this — initially as a privacy-and-integrity defence against malicious proxies. Inspection proxies use the same techniques attackers would; pinning treats them the same.

The newer TLS features (1.3, ESNI/ECH, TLS-ECH-enabled clients) make inspection harder. CASB / SSE vendors track these but the gap between "TLS spec releases new feature" and "CASB inspection supports it" is ~6-18 months.

## What compensates

| Control | Where |
|---|---|
| **Bypass list for pinned apps** | Maintained per-tenant; vendor catalogues are starting points only |
| **MAM (Intune App Protection Policies)** for mobile native apps | Catches the data at the app layer before TLS bypass matters |
| **Endpoint DLP** for pre-encryption inspection | Inspects content on the user's device before it ever hits the TLS layer |
| **Identity-layer controls (Entra Conditional Access)** | Block access from non-compliant devices; complements traffic inspection |
| **Selective inspection by category** | Bypass personal-use categories; inspect work-context categories |
| **Workforce-privacy notice + DPIA** | Document the inspection posture under PDPA / GDPR / equivalent; required regardless of technical answer |

## Privacy / legal exposure

For a regulated FI in MY, SG, HK, EU:

| Concern | Treatment required |
|---|---|
| **PDPA Malaysia (2010 + 2024 amendments)** | Workforce notice; lawful basis (employment context); DPO involvement (verify against gazetted Act A1709). Illustrative; not legal advice |
| **GDPR Art. 88** + **EDPB Guidelines 05/2020** | Lawful basis can't rely on consent in employment context; legitimate interest balancing test; DPIA mandatory for SSL-MITM-equivalent on workforce |
| **HKPCPD employer-monitoring guidance** | Disclosure before monitoring; proportionality test |
| **MAS / BNM** | Operational risk if inspection itself becomes a data-handling concern |

Pushing SSL inspection live without these in place gets the programme rolled back at the first HR / Legal escalation.

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "Forward-proxy gives us complete visibility" | Cert-pinned apps + newer TLS features carve holes |
| "SSL inspection has no user impact" | Personal-use inspection has user impact; pinned-app breakage is silent |
| "The CASB vendor maintains a complete bypass list" | The list is a starting point; the long tail of pinned apps in any tenant requires per-tenant maintenance |

## What it looks like operationally

| Symptom | Cause | Fix |
|---|---|---|
| Mobile banking app stops working on managed devices | Cert pinning rejects inspection cert | Bypass the app; pair with MAM if the app is corporate |
| Vendor SDK client (e.g. medical-device console, trading-platform plugin) fails | Same | Bypass per-app |
| Browser shows certificate warning on personal Gmail / medical portal | Inspection cert is being substituted | Bypass personal categories; document the workforce-privacy posture |
| App's anti-bot / WAF blocks the inspection proxy IPs | App vendor's protection | Vendor-side allowlist of CASB egress IPs (vendor publishes service tags) |
| SaaS-to-SaaS API calls fail (mid-session integrations) | TLS pinning in API client libraries | Bypass server-to-server traffic; inspect user-traffic only |

## Promotion candidate

Documentation-grade content for a public-wedge piece. Currently a residual-risk reference.
