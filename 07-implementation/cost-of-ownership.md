# Cost of ownership

> TCO factors for a CASB / SSE deployment. Licensing is the visible cost; the hidden costs typically dominate. Vendor-agnostic.

## The seven cost lines

| Cost line | Typical relative weight |
|---|---|
| 1. Platform licensing | 30-50% of year-1 spend |
| 2. SIEM ingest + retention | 10-25% of year-1 spend; grows with use |
| 3. Implementation services | 15-30% of year-1 spend (one-off) |
| 4. Endpoint agent / steering footprint | 5-15% of year-1 spend |
| 5. Operational headcount | 15-25% of ongoing run-rate |
| 6. Exception-triage + tuning effort | 5-15% of ongoing run-rate (hidden) |
| 7. Adjacent-platform licence creep | 5-20% of ongoing run-rate (hidden) |

"Hidden" means: not in the vendor proposal; not in the procurement model; surfaces in year 2.

## 1. Platform licensing

| Pricing model | Notes |
|---|---|
| **Per user / month** | Most common. CASB tier within an SSE bundle (Netskope, Zscaler, Palo Alto, Skyhigh). MDA priced per-user/month as standalone or bundled into M365 E5 |
| **Per managed user** | Some vendors define "managed user" narrowly (active in last 30 days); others count everyone in the directory. Read the contract |
| **Per protected app** | Less common; some legacy CASB SKUs |
| **Per tenant / per region** | Skyhigh and some legacy Symantec-lineage; effectively unlimited users per tenant |

### What inflates the per-user line

- **Contractors and B2B guest users** — most vendors count them; tenants pay for users they don't think of as "their users"
- **BYOD-only users** — paying full licence for users with limited coverage anyway
- **Service accounts** — most vendors exclude; verify
- **Multi-tenant federations** — joint-venture users may be counted at both sides

### What can reduce it

- **SSE bundle vs CASB-only** — buying the broader SSE platform often makes per-user CASB cheaper than standalone
- **EA volume tiers** — the breakpoints matter; aiming for the next tier saves more than negotiation
- **3-year commitments** — vendor flexibility on price increases drops sharply past 1 year
- **Channel partner discount** — typically 10-25% off list

## 2. SIEM ingest + retention

| Component | Typical scale |
|---|---|
| **Ingest volume** | 50-200 GB/month per 1,000 users for "normal" CASB log forwarding. Higher (300-500 GB/month per 1,000 users) with clipboard inspection, DSPM-for-AI prompt capture, or verbose audit |
| **Ingest cost** | Sentinel ~$2-4/GB; Splunk Cloud ~$2-3/GB enterprise; Chronicle pricing is volumetric but tiered |
| **Retention** | In-product CASB retention (typically 30-90 days) is insufficient for BFSI audit. 1-7 year retention via SIEM is the typical defensible posture |
| **Archive tier** | Cold storage / archive blob (e.g. Sentinel Archive at ~$0.02/GB/month) for retention beyond hot SIEM |

### What inflates SIEM cost

- Verbose policies (clipboard inspection at scale — see MDA Policy 9 estimate of ~100 GB/month at 5k seats)
- Multiple connectors duplicating events (CASB native + Defender XDR + Office 365 connector — same event ingested three times)
- Wide-scope policies generating many low-value alerts

### What can reduce it

- **Connector consolidation** — one canonical connector per source; suppress duplicates at ingest
- **Tier the retention** — hot 90 days, warm 90-180 days, archive 1-7 years
- **Filter at source** — most CASBs let you filter events before forwarding; trade observability for cost

## 3. Implementation services

| Component | Typical scale |
|---|---|
| **Vendor PS / SI partner** | $200-400/day in MY; $1,500-2,500/day in US/UK; $800-1,500/day in SG/HK |
| **Implementation duration** | 8-16 weeks for a typical tier-2 BFSI deployment; longer (16-26 weeks) with multi-IdP / multi-tenant / heavy customisation |
| **Internal effort** | Easily 0.5-1.0 FTE of internal staff over the implementation window — often under-estimated |

### What inflates implementation

- **Multi-IdP federation** (Okta legacy + Entra) — per-app re-federation work, 2-4 weeks per app
- **Custom-app onboarding to inline mode** — non-mainstream SaaS apps need manual SAML / OIDC work
- **PKI for SSL inspection** — internal CA work; managed-device cert distribution; pinning audit
- **Multi-region deployment** — separate tenants or geo-specific config; coordination overhead
- **Migration from an existing CASB** — policy migration, classifier migration, log retention migration, change-management

### What can reduce it

- **Phased rollout discipline** — don't try to onboard 50 apps simultaneously; sequence
- **Vendor pre-built integrations** — most major SaaS apps have vendor-supplied onboarding guides; use them
- **Don't customise during deployment** — defer all customisation to post-go-live tuning

## 4. Endpoint agent / steering footprint

| Component | Cost / consideration |
|---|---|
| **Agent licensing** | Some CASB vendors (Netskope client, Zscaler ZCC) include the agent in the per-user price; some charge separately |
| **Agent deployment** | Endpoint team time; coexistence testing with EDR / VPN / other agents (multi-agent stacking is a known cause of endpoint perf issues) |
| **PAC management** | Per-region PAC file maintenance; testing on every browser / device class |
| **SSL-inspection cert PKI** | Internal CA infrastructure; managed-device certificate distribution; pinning audit for sensitive apps |
| **Steering mode trade-off** | IdP-routed (no agent, browser only) is lowest footprint but lowest coverage; agent-based steering is highest footprint but highest coverage |

### Microsoft-specific

MDA has **no Defender-for-Cloud-Apps endpoint agent** — Defender for Endpoint feeds discovery data but does not route traffic. Inline coverage is browser-only via CAAC. For roaming inline coverage in a Microsoft-stack deployment, **Microsoft Global Secure Access** is the additional component (separate licence, separate agent).

## 5. Operational headcount

| Role | Typical FTE allocation for a 5k-10k seat deployment |
|---|---|
| **Platform engineer** (the CASB itself) | 0.5-1.0 FTE |
| **Security policy analyst** (rules, exceptions, tuning) | 0.5-1.0 FTE |
| **SOC analyst time** (alert triage) | 0.25-0.5 FTE if alerts properly tuned; 1.0+ FTE if not |
| **Compliance analyst** (audit evidence, control mapping) | 0.25 FTE ongoing; spikes at audit cycle |
| **Programme manager** | 0.25-0.5 FTE in steady state |

For a programme to deliver value, this headcount is non-negotiable. Programmes that try to run on 0.5 FTE total drift into shelfware regardless of how much was spent on licences.

## 6. Exception-triage + tuning effort (the hidden line)

| Activity | Why hidden in proposals |
|---|---|
| **Per-policy FP-rate tuning** | Vendor demos assume policies are pre-tuned; reality is they require 3-6 months of per-environment tuning |
| **Exception review cycle** | Monthly review of every active exception; quarterly expiry / renewal pass |
| **OAuth grant triage** | New consent events review; ban decisions; user notification |
| **SaaS estate maintenance** | New SaaS onboarding to sanctioned/tolerated as the business adopts new tools |
| **Schema-change re-validation** | Vendor schema changes break SIEM queries; quarterly re-test |
| **Compatibility regression** | Per-app CAAC regression when the SaaS vendor changes their UI (`window.parent.postMessage` issues, etc.) |

Programmes that don't budget for this find that the SOC absorbs it; the SOC then deprioritises CASB alerts; the platform's effective TP rate drops.

## 7. Adjacent-platform licence creep

Most CASB capabilities have a "but also" — to fully use them you also need:

| CASB feature | Adjacent licence requirement |
|---|---|
| **MDA CAAC session policies** | Entra ID P1 |
| **MDA auto-labelling** | Microsoft Purview Information Protection (lives in M365 E5 if you have it; otherwise standalone) |
| **MDA GenAI prompt visibility** | Microsoft Purview Insider Risk Management |
| **MDA endpoint enforcement of unsanctioned apps** | Defender for Endpoint (Network Protection in block mode) |
| **MDA long-term log retention** | Microsoft Sentinel or external SIEM |
| **Netskope / Zscaler / PAN clipboard inspection** | Vendor-specific premium tier; often a bolt-on |
| **Inline GenAI prompt DLP** | Most vendors require a specific premium tier |
| **App-Governance (post-consent OAuth behavioural)** | Often a separate add-on across vendors |

Procurement that buys the base platform and discovers these in implementation either pays mid-cycle or operates with degraded coverage. Names these in the procurement model upfront.

## Geographic pricing variance

Vendor list pricing is global but realised pricing varies sharply:

| Region | Typical day-rate for implementation | List-price discount range |
|---|---|---|
| **US** | $1,500-2,500/day | 15-25% off list typical; up to 40% on multi-year |
| **UK / EU** | £900-1,800/day | 10-20% off list |
| **SG / HK** | $1,000-2,000/day | 15-25% off list (more aggressive in MY/Thailand/Vietnam) |
| **MY / Thailand / VN / Philippines** | $200-600/day | 25-40% off list common for multi-year EA |

For an MY tier-1 bank, a list-price US deal model under-prices the local market by 30-50%. Negotiation in this region is meaningful.

## Five-year TCO framing

| Year | Cost shape |
|---|---|
| **Year 1** | Licensing + SIEM + implementation + initial headcount. Most expensive year per-spend |
| **Year 2** | Licensing + SIEM (grows) + tuning effort (peaks) + adjacent-platform additions surface |
| **Year 3** | Steady-state. Look for the adjacent-licence consolidation opportunity |
| **Year 4** | Vendor renewal / re-evaluation. SSE category consolidation typical (multiple point products → one platform) |
| **Year 5** | If staying with the vendor: contracted price increases bite. If switching: migration cost on top |

A defensible 5-year TCO for a tier-2 BFSI 5k-seat deployment is in the $1.5-4M range depending on geography, SSE bundle, and SIEM model. Vendor proposals typically cite Year 1 platform licence only; multiply by 3-5× for honest 5-year TCO.

## What a procurement model should require

| Line | Source |
|---|---|
| **Licence cost per user per year, 3-year fixed** | Vendor proposal with named tier breakpoints |
| **Bolt-on / add-on premium tiers for each capability** | Itemised; named per capability above |
| **Implementation cost ceiling** | Fixed-price or capped time-and-materials |
| **SIEM ingest forecast** | Independent estimate (not the CASB vendor's) based on 50-200 GB/month per 1k users + verbose-policy uplift |
| **Adjacent licence cost** | Each "but also" item itemised |
| **Year 4-5 renewal price-cap** | Negotiated CAP on annual increase |
| **Exit cost** | Documented policy export, log export, transition support — required for outsourcing-regulator alignment (BNM RMiT / MAS Outsourcing illustrative) |

Procurement models that don't capture exit cost end up with one-sided lock-in by year 3. The CASB market is small enough that switching vendors is non-trivial.
