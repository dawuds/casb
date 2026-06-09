# Deployment modes

> Forward-proxy / reverse-proxy / API / log-based discovery / hybrid. Trade-offs, what each can and cannot enforce, traffic-steering options. Vendor-agnostic.

## The four modes

| Mode | How it sees traffic | What it can enforce | What it cannot |
|---|---|---|---|
| **Forward-proxy** | Endpoint agent / PAC / IdP routing sends user traffic through the CASB proxy | Real-time inspection + block; inline DLP; download/upload control; clipboard inspection (vendor-specific) | Anything that doesn't traverse the proxy (BYOD without agent, mobile native, cert-pinned apps, encrypted content) |
| **Reverse-proxy** | IdP redirects sign-in to the CASB proxy; subsequent session URL-rewritten through the proxy | Real-time inspection + block on browser sessions; per-activity blocks (paste, print, share) | Native desktop apps with their own auth; cert-pinned apps; apps not onboarded; anything that doesn't sign in through the IdP |
| **API (out-of-band)** | CASB connects to the SaaS via vendor API; receives events after the fact | Post-upload scan + remediate; sensitivity labelling; OAuth governance; share-link cleanup; activity-policy detection | Real-time prevention (events are delivered after the action); apps without API connectors |
| **Log-based discovery** | CASB ingests logs from SWG / firewall / EDR / cloud-provider | Visibility only (Shadow IT discovery, risk-scoring, app-categorisation) | No enforcement at all — discovery is precondition, not control |

## Forward-proxy (deep dive)

### How traffic gets steered

| Steering method | Coverage |
|---|---|
| **Agent on the endpoint** | High — covers managed devices on any network; struggles on BYOD |
| **PAC file** | Medium — covers browsers configured to use it; native apps + mobile bypass; BYOD opt-in only |
| **IdP-routed (e.g. Microsoft GSA, Cloudflare Access in some configs)** | Medium — covers what the IdP can route; mobile + non-IdP apps bypass |
| **Network DNAT / SD-WAN policy** | Network-bound — covers traffic from corp network; off-net users bypass |
| **GRE / IPsec tunnels from on-prem** | Bound to the corp network egress; doesn't follow the user |

The choice of steering method is the largest single deployment decision in a forward-proxy CASB. Each method makes a different population uncovered.

### SSL/TLS inspection trade-off

Forward-proxy CASB requires SSL inspection to see encrypted application content. SSL inspection:

- **Breaks cert-pinned apps.** Many mobile banking apps, vendor SDK clients, and some desktop apps refuse the inspection cert and silently bypass or fail
- **Inspects personal-use HTTPS.** Treating an employee's personal banking session or medical-portal session as inspectable traffic creates PDPA / GDPR / HKPCPD exposure
- **Computes at scale.** TLS termination is non-trivial CPU; per-user latency adds up; certificate PKI maintenance is its own discipline

The privacy posture must be documented before SSL inspection is enabled. See [`../07-implementation/change-management.md`](../07-implementation/change-management.md) for the workforce-privacy treatment.

## Reverse-proxy (deep dive)

Microsoft's Conditional Access App Control (CAAC) is the canonical reverse-proxy implementation in 2026 — IdP-mediated, browser-only, URL-rewritten to `*.mcas.ms`. Other vendors implement reverse-proxy differently but the model is the same: the IdP sees the sign-in and redirects the session through the CASB proxy.

### What reverse-proxy can do that forward-proxy can't easily

- Onboard a specific app for in-session controls without changing network topology
- Apply per-app controls without re-routing all internet traffic
- Survive PAC / agent gaps for sanctioned-app coverage

### What it can't do

- See traffic to apps not onboarded
- See native desktop / mobile app traffic (only browser sessions)
- Survive `window.parent.postMessage`-heavy app architectures without breaking (see the MDA playbook's CAAC compatibility list)
- Avoid the `*.mcas.ms` URL-rewrite cascade (bookmarks break, SIEM rules pattern-matching on canonical URLs break, anti-CSRF tokens tied to host header break)

## API mode (deep dive)

| Connector | Typical capabilities |
|---|---|
| **Microsoft 365** | Full — read activity, scan files, apply labels, quarantine, suspend users, revoke OAuth (typically the deepest connector across all CASB vendors) |
| **Google Workspace** | Strong — read activity, scan files (Drive), some governance actions; depth varies by vendor |
| **Salesforce** | Good — read activity, scan attachments; configuration assessment in some vendors |
| **Box / Dropbox** | Good — read activity, scan files, quarantine |
| **ServiceNow / Workday** | Variable — read activity; less depth on governance |
| **GitHub Enterprise** | Read activity, secrets scanning, repo policy enforcement (vendor-dependent) |
| **AWS / GCP / Azure** | IaaS resource activity; not full CASB scope |

API-mode coverage depth varies by vendor. The vendor-doc capability matrix usually tells the truth here; cross-corroborate with practitioner reports if a specific governance action is critical to your design.

### The "API is not prevention" rule

API-mode policies operate on **events after they happen**. Typical latency:
- M365 SharePoint / OneDrive: near-real-time per Microsoft (no published SLA — independent measurement varies; plan for <15 min normal, longer during throttling)
- M365 Power BI / Dynamics 365: 24-72 hours
- Google Drive: minutes
- Salesforce: minutes to hours depending on connector

For inline prevention on the same SaaS, you need reverse-proxy / CAAC onboarding. API + inline is the hybrid model.

## Log-based discovery (deep dive)

Discovery is the easy entry point for any CASB deployment — no traffic steering, no agent, no per-app onboarding. Just pipe egress logs in.

### Source quality matters

| Source | Discovery fidelity |
|---|---|
| **EDR network telemetry (e.g. Defender for Endpoint, CrowdStrike, SentinelOne)** | High — endpoint perspective covers off-network roaming; carries user identity |
| **SWG logs (Zscaler / Netskope / Palo Alto / Cisco Umbrella)** | High when on corp network or VPN; lower for off-net users |
| **Firewall / NGFW logs** | Medium — high-volume; less rich user attribution |
| **Cloud provider VPC flow logs** | Useful for IaaS-side discovery; less useful for SaaS-from-endpoint |
| **Manual log upload** | Snapshot only; useful for one-off assessment |

Multi-source ingest gives the most complete picture. Single-source discovery (e.g. SWG-only) leaves the off-network user population invisible.

### What discovery can and cannot do

| Can | Cannot |
|---|---|
| Enumerate apps in use | Block apps |
| Categorise risk | Enforce policy |
| Identify users with high Shadow IT count | Take action against them |
| Show traffic volume per app | See encrypted content |
| Drive sanction taxonomy decisions | Replace inline / API control |

## Hybrid (the typical production deployment)

A typical regulated-FI CASB deployment is **hybrid**:

- **Log-based discovery** for Shadow IT visibility — always on.
- **API connectors** for the named sanctioned SaaS — depth varies per app.
- **Inline / proxy mode** (forward or reverse) for a subset of sanctioned SaaS where real-time prevention is required.

The decision of which apps go behind inline mode is a Phase 2-3 decision (see [`../07-implementation/phased-rollout.md`](../07-implementation/phased-rollout.md)). Default: M365 (always inline if you have CAAC); Salesforce and HR system if you handle regulated data there; long-tail apps stay API-only.

## Mode-to-capability matrix

| Capability | Forward-proxy | Reverse-proxy | API | Log-based |
|---|---|---|---|---|
| Shadow IT discovery | (Yes, but agent-bound) | No | (Partial — sanctioned only) | **Yes (primary)** |
| Inline DLP (pre-upload prevention) | **Yes** | **Yes** (browser only) | No | No |
| Post-upload DLP | (Yes) | (Yes) | **Yes (primary)** | No |
| Block download | **Yes** | **Yes** (browser only) | No (post-event only) | No |
| Block clipboard paste | (Yes, vendor-specific) | **Yes** (browser only) | No | No |
| Apply sensitivity label on download | **Yes** | **Yes** | (Yes — file-policy auto-label) | No |
| OAuth grant discovery | No | No | **Yes (primary)** | No |
| Anomalous activity detection | (Yes, via session events) | (Yes, via session events) | **Yes (primary)** | No |
| Compromised-account detection | (Yes) | (Yes) | **Yes (primary)** | No |
| Malware scan in SaaS storage | (Yes, on upload) | (Yes, on upload) | **Yes (primary, post-upload)** | No |
| Share-link governance | No | No | **Yes (primary)** | No |
| BYOD coverage (managed device) | Yes (with agent) | Yes (browser) | Yes | (Yes — if BYOD endpoint reports to discovery feed) |
| BYOD coverage (unmanaged mobile native) | No | No | (Yes — for sanctioned SaaS) | No |

The matrix is the answer to "which mode do I need" — work backwards from the capabilities you must support.

## Choosing modes by use-case

| Goal | Minimum mode |
|---|---|
| "I just want to know what SaaS my users use" | Log-based discovery |
| "I want to govern OAuth grants and clean up sharing" | API |
| "I want to block download of sensitive files to BYOD" | Reverse-proxy or forward-proxy with agent |
| "I want to block GenAI prompts from leaking source code" | Forward-proxy with agent + DLP inspection, OR reverse-proxy on the GenAI app (limited to SAML-supporting plans) |
| "I want all of the above" | Hybrid (all four modes, sequenced) |

## What practitioners get wrong about modes

| Mistake | Consequence |
|---|---|
| Assuming API mode = real-time enforcement | Sensitive data leaves before the policy fires; remediation is post-event quarantine |
| Assuming forward-proxy covers BYOD by default | Mobile native + cert-pinned apps bypass; the residual exposure is the BYOD mobile fleet |
| Assuming reverse-proxy covers all apps the IdP knows | Only browser sessions are covered; native apps using the same IdP often bypass |
| Assuming log-based discovery is a control | Discovery is precondition; nothing is prevented until enforcement is layered on |
| Treating SSL inspection as a free lunch | Privacy posture, cert PKI, pinned-app gaps all bite |
| Skipping the IdP question | The whole inline-mode story depends on identity. If the IdP isn't clean, the CASB isn't reliable |
