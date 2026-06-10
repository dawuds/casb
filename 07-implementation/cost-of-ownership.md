# Cost of ownership

> Programme-level discipline within a CASB / SSE deployment. Vendor-agnostic.
> Reader audience: programme lead, finance, procurement, executive sponsor.
> Most-directly governs which `03-policies/` files: all 19 + adjacent-platform decisions.

## Purpose

TCO factors for a CASB / SSE deployment. Licensing is the visible cost; the hidden costs typically dominate. This discipline owns the TCO model, the adjacent-platform-creep tracker, the procurement-model requirements, and the year-by-year defensible spend trajectory. Programmes without TCO discipline routinely overspend by 2-3× the procurement estimate by year 2.

## What organisations use this discipline for

Cost-of-ownership discipline is what separates a defensible CASB business case from "we bought it, we'll figure out the cost later". Most CASB programmes deliver year-1 spend at or below estimate (the visible licence line is competitive) but exceed year-2 estimate by 2-3× because the hidden cost lines (SIEM ingest growth, adjacent-platform creep, operational headcount ramp, exception-triage effort) were not modelled at procurement time. The discipline owns three procurement artefacts: the 5-year TCO model (not just year-1 licence), the adjacent-platform-creep tracker (every "but you also need X" itemised), and the year-over-year cost-trend review (against the original model).

### Use case 1 — Pre-procurement TCO modelling

- **Org type:** tier-2 ASEAN bank, ~12k employees, M365 E3 → E5 upgrade decision, no current CASB
- **Trigger:** finance asked CISO for a defensible 5-year TCO forecast on the CASB programme before approving the M365 E5 upgrade budget
- **Scope:** built a 5-year TCO model covering all 7 cost lines + adjacent-platform creep (Entra ID P1, Purview IRM, MDE Network Protection, Sentinel ingest) + operational headcount ramp + exception-triage effort
- **Outcome:** 5-year TCO came out at $2.8M (vs vendor-supplied year-1 licence quote of $0.4M); finance approved the programme at the higher number because the cost was defensible (not surprise spend); programme started with realistic budget; year-2 spend tracked within 8% of model

### Use case 2 — Year-2 cost surprise, defending the budget

- **Org type:** mid-cap insurance carrier, ~5k employees, year 2 of CASB programme; original procurement budget was year-1-licence-only
- **Trigger:** finance flagged year-2 spend was 2.4× year-1 plan; CISO asked to justify the over-run
- **Scope:** retrospective TCO build — identified that Sentinel ingest had grown 3× (verbose policies including [`../03-policies/genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md) at 5k seats), adjacent-platform licences (Purview IRM + MDE NP + App Governance) had been procured mid-year to unlock CASB capabilities, operational headcount had grown to ~2 FTE (from initial 0.5 FTE estimate)
- **Outcome:** programme retained at reduced future-spend trajectory; finance + CISO joint TCO review became quarterly cadence; programme team observation — the cost lines were not surprises to the implementation team but were not visible to finance; the discipline-failure was procurement-side not implementation-side

### Use case 3 — SSE consolidation business case

- **Org type:** multinational tech firm, ~10k employees, running Symantec endpoint DLP + Mimecast email security + Zscaler ZIA + Cisco Umbrella; renewal cycles overlapping in 18 months
- **Trigger:** finance asked "can we consolidate into one SSE vendor and reduce TCO?"
- **Scope:** SSE-consolidation TCO model — point-product vs SSE-platform comparison; included migration cost (training + per-app re-onboarding + parallel-run period); modelled 3-year TCO for two SSE candidates (Netskope and Zscaler) against the existing point-product stack
- **Outcome:** SSE consolidation forecast $1.2M net savings over 3 years (point-product spend $4.8M, SSE $3.6M); migration cost ~$0.5M one-off; net Year-1 was break-even, year-2 and year-3 saved; programme proceeded with SSE consolidation decision; year-1 reality matched within ~15% of model

### Use case 4 — Multi-vendor TCO modelling

- **Org type:** tier-1 ASEAN bank, ~30k employees, Microsoft-stack for M365 + Defender + Entra + Purview + MDA; Netskope for SSE + SWG + ZTNA on operational network; hybrid not unified
- **Trigger:** finance asked "what is our true CASB-equivalent spend across both vendors?" — neither vendor's invoice was a complete answer
- **Scope:** unified TCO model covering MDA (within M365 E5) + Microsoft adjacent platforms + Netskope SSE platform; SIEM ingest from both; operational headcount supporting both; documented the hybrid-vendor decision rationale
- **Outcome:** combined CASB-equivalent TCO came out at ~$1.4M / year (M365 E5 amortised CASB portion ~$0.5M + Netskope ~$0.6M + adjacent + headcount ~$0.3M); the hybrid-vendor reality was now visible to finance; quarterly cost-trend review surfaced when MDA and Netskope features overlapped (duplicative spend) — gradual rationalisation followed

## The discipline

### The seven cost lines

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

### 1. Platform licensing

| Pricing model | Notes |
|---|---|
| **Per user / month** | Most common. CASB tier within an SSE bundle (Netskope, Zscaler, Palo Alto, Skyhigh). MDA priced per-user/month as standalone or bundled into M365 E5 |
| **Per managed user** | Some vendors define "managed user" narrowly (active in last 30 days); others count everyone in the directory. Read the contract |
| **Per protected app** | Less common; some legacy CASB SKUs |
| **Per tenant / per region** | Skyhigh and some legacy Symantec-lineage; effectively unlimited users per tenant |

#### What inflates the per-user line

- **Contractors and B2B guest users** — most vendors count them; tenants pay for users they don't think of as "their users"
- **BYOD-only users** — paying full licence for users with limited coverage anyway
- **Service accounts** — most vendors exclude; verify
- **Multi-tenant federations** — joint-venture users may be counted at both sides

#### What can reduce it

- **SSE bundle vs CASB-only** — buying the broader SSE platform often makes per-user CASB cheaper than standalone
- **EA volume tiers** — the breakpoints matter; aiming for the next tier saves more than negotiation
- **3-year commitments** — vendor flexibility on price increases drops sharply past 1 year
- **Channel partner discount** — typically 10-25% off list

### 2. SIEM ingest + retention

| Component | Typical scale |
|---|---|
| **Ingest volume** | 50-200 GB/month per 1,000 users for "normal" CASB log forwarding. Higher (300-500 GB/month per 1,000 users) with clipboard inspection, DSPM-for-AI prompt capture, or verbose audit |
| **Ingest cost** | Sentinel ~$2-4/GB; Splunk Cloud ~$2-3/GB enterprise; Chronicle pricing is volumetric but tiered |
| **Retention** | In-product CASB retention (typically 30-90 days) is insufficient for BFSI audit. 1-7 year retention via SIEM is the typical defensible posture |
| **Archive tier** | Cold storage / archive blob (e.g. Sentinel Archive at ~$0.02/GB/month) for retention beyond hot SIEM |

#### What inflates SIEM cost

- Verbose policies (clipboard inspection at scale — see [`../03-policies/genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md) — ~100 GB/month at 5k seats)
- Multiple connectors duplicating events (CASB native + Defender XDR + Office 365 connector — same event ingested three times)
- Wide-scope policies generating many low-value alerts

#### What can reduce it

- **Connector consolidation** — one canonical connector per source; suppress duplicates at ingest
- **Tier the retention** — hot 90 days, warm 90-180 days, archive 1-7 years
- **Filter at source** — most CASBs let you filter events before forwarding; trade observability for cost

### 3. Implementation services

| Component | Typical scale |
|---|---|
| **Vendor PS / SI partner** | $200-400/day in MY; $1,500-2,500/day in US/UK; $800-1,500/day in SG/HK |
| **Implementation duration** | 8-16 weeks for a typical tier-2 BFSI deployment; longer (16-26 weeks) with multi-IdP / multi-tenant / heavy customisation |
| **Internal effort** | Easily 0.5-1.0 FTE of internal staff over the implementation window — often under-estimated |

#### What inflates implementation

- **Multi-IdP federation** (Okta legacy + Entra) — per-app re-federation work, 2-4 weeks per app
- **Custom-app onboarding to inline mode** — non-mainstream SaaS apps need manual SAML / OIDC work
- **PKI for SSL inspection** — internal CA work; managed-device cert distribution; pinning audit
- **Multi-region deployment** — separate tenants or geo-specific config; coordination overhead
- **Migration from an existing CASB** — policy migration, classifier migration, log retention migration, change-management

#### What can reduce it

- **Phased rollout discipline** — don't try to onboard 50 apps simultaneously; sequence per [`phased-rollout.md`](phased-rollout.md)
- **Vendor pre-built integrations** — most major SaaS apps have vendor-supplied onboarding guides; use them
- **Don't customise during deployment** — defer all customisation to post-go-live tuning

### 4. Endpoint agent / steering footprint

| Component | Cost / consideration |
|---|---|
| **Agent licensing** | Some CASB vendors (Netskope client, Zscaler ZCC) include the agent in the per-user price; some charge separately |
| **Agent deployment** | Endpoint team time; coexistence testing with EDR / VPN / other agents (multi-agent stacking is a known cause of endpoint perf issues) |
| **PAC management** | Per-region PAC file maintenance; testing on every browser / device class |
| **SSL-inspection cert PKI** | Internal CA infrastructure; managed-device certificate distribution; pinning audit for sensitive apps |
| **Steering mode trade-off** | IdP-routed (no agent, browser only) is lowest footprint but lowest coverage; agent-based steering is highest footprint but highest coverage |

#### Microsoft-specific

MDA has **no Defender-for-Cloud-Apps endpoint agent** — Defender for Endpoint feeds discovery data but does not route traffic. Inline coverage is browser-only via CAAC. For roaming inline coverage in a Microsoft-stack deployment, **Microsoft Global Secure Access** is the additional component (separate licence, separate agent).

### 5. Operational headcount

| Role | Typical FTE allocation for a 5k-10k seat deployment |
|---|---|
| **Platform engineer** (the CASB itself) | 0.5-1.0 FTE |
| **Security policy analyst** (rules, exceptions, tuning) | 0.5-1.0 FTE |
| **SOC analyst time** (alert triage) | 0.25-0.5 FTE if alerts properly tuned; 1.0+ FTE if not |
| **Compliance analyst** (audit evidence, control mapping) | 0.25 FTE ongoing; spikes at audit cycle |
| **Programme manager** | 0.25-0.5 FTE in steady state |

For a programme to deliver value, this headcount is non-negotiable. Programmes that try to run on 0.5 FTE total drift into shelfware regardless of how much was spent on licences.

### 6. Exception-triage + tuning effort (the hidden line)

| Activity | Why hidden in proposals |
|---|---|
| **Per-policy FP-rate tuning** | Vendor demos assume policies are pre-tuned; reality is they require 3-6 months of per-environment tuning |
| **Exception review cycle** | Monthly review of every active exception; quarterly expiry / renewal pass |
| **OAuth grant triage** | New consent events review; ban decisions; user notification |
| **SaaS estate maintenance** | New SaaS onboarding to sanctioned/tolerated as the business adopts new tools |
| **Schema-change re-validation** | Vendor schema changes break SIEM queries; quarterly re-test |
| **Compatibility regression** | Per-app CAAC regression when the SaaS vendor changes their UI (`window.parent.postMessage` issues, etc.) |

Programmes that don't budget for this find that the SOC absorbs it; the SOC then deprioritises CASB alerts; the platform's effective TP rate drops.

### 7. Adjacent-platform licence creep

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

Procurement that buys the base platform and discovers these in implementation either pays mid-cycle or operates with degraded coverage. Name these in the procurement model upfront.

### Geographic pricing variance

Vendor list pricing is global but realised pricing varies sharply:

| Region | Typical day-rate for implementation | List-price discount range |
|---|---|---|
| **US** | $1,500-2,500/day | 15-25% off list typical; up to 40% on multi-year |
| **UK / EU** | £900-1,800/day | 10-20% off list |
| **SG / HK** | $1,000-2,000/day | 15-25% off list (more aggressive in MY/Thailand/Vietnam) |
| **MY / Thailand / VN / Philippines** | $200-600/day | 25-40% off list common for multi-year EA |

For an MY tier-1 bank, a list-price US deal model under-prices the local market by 30-50%. Negotiation in this region is meaningful.

### Five-year TCO framing

| Year | Cost shape |
|---|---|
| **Year 1** | Licensing + SIEM + implementation + initial headcount. Most expensive year per-spend |
| **Year 2** | Licensing + SIEM (grows) + tuning effort (peaks) + adjacent-platform additions surface |
| **Year 3** | Steady-state. Look for the adjacent-licence consolidation opportunity |
| **Year 4** | Vendor renewal / re-evaluation. SSE category consolidation typical (multiple point products → one platform) |
| **Year 5** | If staying with the vendor: contracted price increases bite. If switching: migration cost on top |

A defensible 5-year TCO for a tier-2 BFSI 5k-seat deployment is in the $1.5-4M range depending on geography, SSE bundle, and SIEM model. Vendor proposals typically cite Year 1 platform licence only; multiply by 3-5× for honest 5-year TCO.

### What a procurement model should require

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

## Implementation pattern

Typical 4-6 week TCO discipline build-out alongside readiness phase:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Vendor-proposal review — itemise all 7 cost lines; flag missing items | Cost-line inventory |
| W2 | Independent SIEM-ingest forecast — not from the vendor; use 50-200 GB/month/1k-users baseline + verbose-policy uplift | SIEM ingest cost model |
| W3 | Adjacent-platform-creep inventory — every "but also" item itemised; named per capability | Adjacent-platform cost line |
| W3-W4 | Operational headcount sizing — by role + by phase (Year-1 ramp vs Year-3 steady state) | Headcount cost model |
| W4 | Exception-triage + tuning effort estimate — hidden line; usually 5-15% of run-rate | Hidden-line cost model |
| W5 | Geographic-pricing-variance application — list-price discount applied by region | Realised-price model |
| W6 | 5-year TCO assembly + exit-cost line; finance sign-off | 5-year TCO model approved by finance |
| Quarterly thereafter | Cost-trend review against the model | Cost-discipline cadence |

The W2 independent SIEM-ingest forecast is the discipline-critical step. Without it, the procurement model takes the vendor's "typical ingest" claim as input — which understates by 2-3× for verbose-policy deployments.

## Variants

### Industry-specific

- **BFSI:** regulator-driven SIEM retention (5-7 years) dominates cost line 2; exit-cost discipline rigorous (BNM RMiT outsourcing exit-strategy expectations); 5-year TCO is the procurement minimum
- **Healthcare:** HIPAA retention requirements vary by record-class; PHI-handling sub-processor list adds adjacent-licence creep (BAA-required vendors); cost-of-ownership cycle tighter due to healthcare margin pressure
- **Tech:** licensing-per-user inflated by contractor + freelance population; SIEM-ingest growth tracks verbose-policy deployment more closely than other sectors; SSE consolidation business case more common
- **Retail:** PCI cycle drives retention and audit-evidence costs; high-turnover workforce skews per-user pricing (lots of seats for short tenure)
- **Manufacturing:** OT/IT boundary creates dual cost-modelling (CASB scope vs OT-security scope); M&A frequency triggers per-acquired-firm cost-model refresh
- **Higher education / research:** federal-grant compliance regimes (US) drive specific retention costs; per-research-project SaaS-tenant subscriptions add long-tail licensing
- **Public sector:** procurement-control regimes drive vendor-selection cycle length (multi-year); cost-trend review aligned to budget cycles rather than fiscal years

### Maturity-based

- **Immature:** Year-1 licence cost is the procurement budget; surprise spend in years 2-3; finance discovers the over-run reactively; programme defends spend in crisis mode
- **Mature:** 5-year TCO model approved at procurement; quarterly cost-trend review against the model; deltas explained per variance; adjacent-platform creep itemised in advance
- **Advanced:** TCO model continuously refined with actual-vs-model data; benchmarking against peer firms (where data available); cost-trend metric reported to board quarterly; cost-discipline integrated with vendor-renewal cycle (annual reset against the model); SSE-consolidation business case maintained as a standing option

## Real-world experience

Typical patterns from production TCO disciplines:

| Pattern | Frequency |
|---|---|
| Procurement model is Year-1 licence cost only; Year-2 over-runs the procurement by 2-3× | ~60% of programmes |
| SIEM ingest growth tracked but the cost forecast was vendor-supplied (under-estimated by 2-3×) | ~50% of programmes |
| Adjacent-platform creep (Entra ID P1 / Purview IRM / MDE NP / etc.) surfaces in Year-2; procurement model had not itemised | ~70% of programmes |
| Operational headcount under-estimated at Year-1; ramps to ~2× procurement-model estimate by Year-2 | ~55% of programmes |
| Exception-triage effort absorbed by SOC without dedicated headcount; SOC reprioritises away from CASB alerts | ~40% of programmes |
| Exit cost not modelled; programme discovers Year-3 lock-in when considering vendor switch | ~70% of programmes |
| Geographic pricing variance applied incorrectly (US prices in non-US tenders or vice versa) | ~30% of programmes |
| SSE consolidation business case considered but not executed due to migration-cost unfamiliarity | ~50% of programmes |
| Vendor renewal price increase >10% without resistance because no price-cap clause was negotiated at Year-1 | ~45% of programmes |

## Integration with broader programmes

- **`programme-readiness.md`:** readiness business case is the initial TCO base; locked-decisions document shapes the cost model
- **`phased-rollout.md`:** phased rollout determines per-year spend trajectory (Year-1 = Phases 1-3; Year-2 = Phase 4; Year-3 = Phase 5)
- **`soc-integration.md`:** SOC analyst headcount (cost line 5) tracks the SOC-integration discipline maturity
- **`change-management.md`:** change-management staffing is part of cost line 5; exception-triage is part of cost line 6
- **`success-metrics.md`:** M16 (cost per active user) and M17 (SIEM ingest cost trend) are the TCO-discipline metrics
- **All 19 `03-policies/` files:** verbose policies (clipboard inspection, prompt-content capture, file-policy scans) drive cost line 2 (SIEM ingest); policy class governs the cost trajectory
- **Vendor procurement:** TCO model is the procurement-side scorecard; vendor renewal is the TCO-refresh trigger
- **Finance:** quarterly cost-trend review with finance; annual TCO model refresh; SSE-consolidation business case maintained as a standing option

## Anti-patterns specific to cost-of-ownership

1. **Year-1 licence as the procurement-model headline** — finance approves on Year-1 cost; Year-2 over-runs the budget by 2-3×; programme defends spend reactively
2. **Vendor-supplied SIEM-ingest forecast taken as input** — vendor under-estimates verbose-policy uplift by 2-3×; SIEM cost surprise in Year-2
3. **No adjacent-platform creep itemisation** — Entra ID P1 / Purview IRM / MDE NP / App Governance procured mid-year on emergency budget; cost line invisible to procurement
4. **Operational headcount estimated as Year-3 steady-state only** — Year-1 ramp under-staffed; programme struggles operationally; remediation in Year-2 adds headcount cost
5. **Exception-triage effort absorbed by existing SOC** — SOC reprioritises away from CASB; effective TP rate drops; platform value degrades while cost increases
6. **No exit-cost line in procurement model** — Year-3 lock-in discovery; vendor renewal pricing unconstrained
7. **No price-cap clause at Year-1 EA** — Year-4 renewal pricing >10% increase; no negotiation leverage
8. **Geographic pricing variance ignored** — US list price applied in non-US tender (over-spend) or local pricing applied in US tender (under-budget)
9. **SSE-consolidation business case considered but not modelled** — programme team has the hypothesis but no defensible TCO comparison; finance cannot decide
10. **TCO model not refreshed annually** — model becomes stale; cost-trend review references a fictional baseline; discipline degrades

## Cross-references

- [`programme-readiness.md`](programme-readiness.md) — readiness business case feeds TCO
- [`phased-rollout.md`](phased-rollout.md) — per-phase spend trajectory
- [`soc-integration.md`](soc-integration.md) — SOC headcount cost line
- [`change-management.md`](change-management.md) — change-mgmt staffing cost
- [`success-metrics.md`](success-metrics.md) — M16 / M17 are TCO metrics
- [`../03-policies/genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md) — verbose-policy SIEM ingest example
- [`../03-policies/`](../03-policies/) — all policies; their verbose-policy weighting feeds cost line 2
- [`../../cyber-business/`](../../cyber-business/) (sibling repo) — vendor unit-economics + market mechanics for benchmarking
- [`../README.md`](../README.md) — repo-level navigation

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
