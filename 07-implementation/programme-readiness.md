# CASB programme readiness

> Programme-level discipline within a CASB / SSE deployment. Vendor-agnostic.
> Reader audience: programme lead, executive sponsor, risk lead.
> Most-directly governs which `03-policies/` files: ALL 19 — readiness is the gate before any policy ships.

## Purpose

What an organisation must know about itself before a CASB / SSE platform deployment will succeed. Discovery, classification, and policy decisions only land on a base that has these answers. Readiness is the discipline of producing that base — the inventory, the named owners, the regulatory perimeter, and the locked-before-vendor-selection decisions that everything downstream depends on.

## What organisations use this discipline for

Programme readiness is the discipline that distinguishes a vendor-purchase from a programme. Most regulated FI procurements buy the licence first and discover the readiness gap when implementation services start; readiness done well surfaces the gap before the SOW is signed, and is the dominant predictor of whether the licence cost realises any value. The hardest single output of the readiness phase is not the inventory itself — it is the **locked-and-signed scoped-requirements document** that names BYOD position, personal-use position, and exception process before vendor demos.

### Use case 1 — Tier-2 ASEAN universal bank, board-mandated CASB programme

- **Org type:** tier-2 ASEAN universal bank, ~12k employees, M365 E3 (no E5 yet), no current CASB, BNM RMiT supervised
- **Trigger:** board cyber report flagged Shadow IT visibility as a Category-2 finding from external assurance; board approved a CASB programme contingent on a defensible business case
- **Scope:** full readiness phase before vendor selection — 6-week sprint including SaaS-estate inventory, identity-posture audit (Entra primary + Okta legacy on a subset of legacy apps), BYOD-population study, regulatory inventory; sponsor = Head of IT Security, programme lead = senior security architect, compliance lead = Risk's BNM-liaison officer
- **Outcome:** 1,200-line scoped-requirements document; identified that 35% of in-scope apps were Okta-fronted (forcing per-app re-federation work in the implementation phase); identified that BYOD-mobile coverage was the dominant risk class but inline-mobile-CASB was not viable — pivoted to Intune MAM as the BYOD compensating control; saved an estimated $400k in mis-procured premium-tier features that would not have been usable

### Use case 2 — Multinational tech firm replacing legacy Symantec DLP with M365 E5

- **Org type:** B2B SaaS company, ~8k employees, M365 E5 newly acquired, existing Symantec DLP endpoint deployment, light regulatory load (SOC 2 + privacy law in operating regions)
- **Trigger:** Symantec DLP renewal cycle; finance asked "do we keep paying for Symantec or use the Purview DLP we already have in E5?"
- **Scope:** parallel-run readiness — assess Purview + MDA capability against Symantec's current control set; identify migration path; sponsor = CISO, technical lead = Purview-experienced principal engineer (newly hired), compliance lead = General Counsel delegate
- **Outcome:** readiness identified that Purview DCS classifier-set diverged materially from Symantec EDM on custom classifiers (~30% of in-use classifiers had no clean Purview equivalent); decision: retain Symantec at endpoint for 18 months parallel-run; add MDA + Purview for SaaS coverage; full Symantec retirement deferred 24 months; readiness saved the firm from a "rip and replace" approach that would have created a 6-month coverage gap

### Use case 3 — Post-OAuth-phishing-incident emergency readiness

- **Org type:** digital-native challenger bank, ~800 employees, M365 E5, no prior CASB
- **Trigger:** Storm-2372-class device-code-phishing campaign successfully consented three malicious OAuth apps into the M365 tenant over a 4-hour window; post-incident review surfaced no tenant-wide CASB visibility into OAuth grants; CISO convened emergency readiness on a 2-week sprint
- **Scope:** compressed readiness — focus on OAuth-grant inventory, admin-consent posture, tenant-restriction posture, MDA deployment readiness; defer broader inventory to phase 2
- **Outcome:** MDA deployed in 3 weeks (vs typical 8-12); App Governance add-on procured under emergency budget; tenant-restriction headers configured via GSA; emergency readiness was effective but produced known technical debt (incomplete BYOD assessment, no SOC integration plan) addressed in the standard readiness phase at month 6

### Use case 4 — Post-M&A combined-firm readiness

- **Org type:** tier-1 ASEAN bank (~30k pre-acquisition) acquiring a regional fintech (~3.5k employees, mixed M365 + Google Workspace estate)
- **Trigger:** post-close integration; acquired firm had no CASB; parent firm had mature MDA deployment; combined-tenant approach required readiness assessment
- **Scope:** acquired-firm readiness phase (parent's CASB already mature); identify federation seams, sub-tenant scoping, sanction-taxonomy alignment; sponsor = combined-firm CIO, programme lead = parent's CASB programme lead with acquired-firm liaison
- **Outcome:** identified acquired-firm Google Workspace tenant as in scope (parent had M365-only CASB); broader vendor scope decision; readiness produced a 12-month integration plan with explicit milestones for sanction-taxonomy alignment, identity unification, and BYOD-policy harmonisation; acquired-firm BYOD posture (more permissive) was preserved through transition then unified at month 18

## The discipline

### The four readiness axes

| Axis | Question | Why it bites if unanswered |
|---|---|---|
| **SaaS estate** | What sanctioned + tolerated + unknown SaaS does the firm use, and which contain regulated data classes? | CASB onboarding effort scales with the SaaS estate. Without an inventory, the rollout is reactive, the policy library is sized wrong, and the licence cost is wrong |
| **Identity posture** | Is there a single IdP source of truth? Where is the federation seam (Entra primary, Okta legacy, ADFS, Ping)? Is MFA enforced for all access? | CASB inline policies depend on the IdP. Identity drift is the dominant cause of policy gaps. Multi-IdP estates pay 3-5× the onboarding cost per app |
| **Endpoint posture** | Managed vs unmanaged distribution. Mobile BYOD population. MDM/MAM coverage. SSL-inspection compatibility (cert pinning, banking apps) | Every CASB deployment trade-off (forward-proxy vs API, block vs read-only on BYOD) collapses to the endpoint posture answer |
| **Regulatory perimeter** | Which obligations actually bite (BNM RMiT, MAS TRM, HKMA, PDPA MY 2024, GDPR, DORA, sectoral)? What is the audit cadence? What records survive an examination? | Drives the must-have policies, the audit evidence requirements, and the residency/cross-border decisions. Treated as an afterthought it forces re-architecture late |

### Inventory checklist (before vendor selection)

| Check | Owner | Outcome required |
|---|---|---|
| Top-50 SaaS by user count (from IdP sign-in logs) | Identity team | Named list with data-classification tag per app |
| Top-50 SaaS by traffic volume (from SWG / firewall logs) | Network team | Named list including the long-tail Shadow IT not behind the IdP |
| Sanctioned vs tolerated vs unsanctioned classification | Risk + Business | Documented, with sponsoring business owner per sanctioned app |
| Data classification scheme | Information Protection team | Named classes (PII, PCI, KYC, etc.) with examples and labelling guidance — not just "Confidential / Internal / Public" |
| Sensitive-data exposure assessment | Risk + Information Protection | Where the regulated data classes actually live across the SaaS estate (Microsoft Purview / Google Vault / equivalent already deployed?) |
| IdP inventory | Identity team | Primary IdP, secondary IdP, federation chain, app coverage per IdP |
| MFA enforcement state | Identity team | What % of users / apps are MFA-protected. Gap analysis against CASB session-policy assumptions |
| Endpoint management state | Endpoint team | % managed (MDM enrolled), % unmanaged, % BYOD-mobile, MDM platform (Intune, Workspace ONE, Jamf, Kandji) |
| SSL-inspection feasibility | Network + Endpoint | Cert pinning audit on key apps (banking, vendor SDK clients, mobile native). Personal-traffic inspection legal/HR position |
| Egress topology | Network team | Where users exit to the internet (corp network, VPN, BYOD direct, SD-WAN). Determines where the CASB proxy can sit |
| SOC capability | Security operations | SIEM in use, SOAR available, alert handling capacity (alerts/analyst/day), current FP rate on existing controls |
| Regulatory inventory | Risk + Legal | Named obligations with article/clause references. Cross-border data-flow constraints. Audit cadence per regulator |
| Existing DLP estate | Information Protection | What is already enforcing (endpoint DLP, email DLP, service-side SaaS DLP). Where is the duplication risk |
| Change-management capacity | IT operations | How fast can you push agent/PAC/policy changes. Communications channels for end-user impact |

If five or more rows above are blank or "unknown", **CASB selection is premature** — close the inventory gap first. Tooling buyers who skip this end up paying for capabilities they cannot deploy.

### Sponsorship and ownership

A CASB programme needs four named owners signed off before kickoff:

| Role | Owns |
|---|---|
| **Executive sponsor** | The business case; the budget; the cross-functional unlock when Legal / HR / business push back |
| **Programme lead** | The roadmap; the phased rollout; vendor management; the politics of policy decisions |
| **Technical lead** | Platform architecture; integration plumbing; the on-call rotation for the CASB itself |
| **Compliance lead** | Control mapping; audit evidence; the regulator conversation; sign-off on policies that touch privacy |

Programmes with three of four end up with mis-scoped policies (no compliance lead), shelfware (no exec sponsor), or controls that work technically but fail audit (no compliance lead). Programmes with two of four fail.

### Decisions that must be locked before policy design

| Decision | What it determines |
|---|---|
| **In-scope user populations** | Employees only? Contractors? B2B partners? Service accounts? Each population has different policy semantics |
| **In-scope SaaS scope** | Sanctioned only? Sanctioned + tolerated? All-discovered? Sets the licensing budget and the implementation surface |
| **BYOD position** | Allow with controls / block / read-only / device-trust-required. The single most political decision in a CASB rollout |
| **Personal-use position** | Are users allowed to access personal SaaS (personal Gmail, personal cloud drives) on corporate networks / devices? Determines the SSL-inspection privacy posture |
| **Action-class default** | Audit / coach / block. Most programmes start in audit, escalate. Block-first programmes generate a workaround culture that defeats the controls |
| **Exception process** | Who can grant an exception? On what evidence? For how long? Audit trail of exceptions |
| **Privacy posture** | What end-user notice is given? What is logged about user activity? Where does the log live? How is access governed? Critical under PDPA MY 2024 / GDPR / HKPCPD |

Don't run vendor selection without these locked. Vendor demos look identical if the requirements are vague; they look different when the requirements name a policy decision.

## Implementation pattern

Typical 4-6 week readiness phase. Compressed (2-3 weeks) variants exist for emergency-readiness scenarios but produce known technical debt; standard 6-week is the defensible posture.

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Sponsor + named-owner kickoff; charter signed; scope locked at the four readiness axes | Charter approved; four owners named with calendar commitment |
| W1-W2 | Top-50 SaaS inventory (IdP-derived); Top-50 traffic-derived inventory; reconcile; classify Sanctioned / Tolerated / Unsanctioned candidate list | Initial inventory; per-app business-owner provisional list |
| W2-W3 | Identity posture audit (IdP inventory, MFA enforcement state, federation map); endpoint-posture audit (managed/unmanaged distribution, BYOD-mobile population, SSL-inspection feasibility) | Identity-posture report + endpoint-posture report |
| W3-W4 | Regulatory perimeter inventory — named clauses per obligation, audit cadence, evidence requirements, cross-border-data-flow constraints | Regulatory inventory + control-mapping outline |
| W4-W5 | Locked decisions workshop — in-scope user populations, BYOD position, personal-use position, action-class default, exception-process design, privacy posture | Scoped-requirements document signed by exec sponsor |
| W5-W6 | Gap analysis vs current controls (endpoint DLP, email DLP, IdP, SIEM); business case finalisation; vendor-selection criteria derived from locked decisions | Business case approved; vendor RFI / RFP issued |

The W4-W5 workshop is the political bottleneck. Without exec sponsor and BU representatives in the room, BYOD-and-personal-use decisions get deferred; without those locked, vendor selection produces a tool that cannot deploy.

## Variants

### Industry-specific

- **BFSI:** regulatory perimeter is the long-pole axis (BNM RMiT / MAS TRM / HKMA / DORA each demand specific evidence patterns); BYOD posture often "blocked from regulated-data scope, allowed with controls elsewhere"; longer readiness phase (6-8 weeks) for the regulator-conversation prep
- **Healthcare:** HIPAA + state-level AI rules drive regulatory perimeter; PHI-handling scope decision dominates; BYOD usually restrictive in clinical contexts; physician-population workflow nuances often missed in initial inventory
- **Tech / SaaS:** light regulatory load; BYOD often permissive (developer culture); engineering-team workflow patterns dominate; readiness focus shifts to source-code + secrets-protection classes
- **Retail:** PCI DSS + privacy regs drive regulatory perimeter; high-turnover workforce creates contractor / temp-staff edge cases; BYOD posture varies by role; loyalty-data scope often under-mapped
- **Manufacturing / industrial:** trade-secret + IP-protection classes; supplier-collaboration patterns; M&A frequency creates inherited-estate readiness scenarios; OT/IT boundary often interacts with CASB scope
- **Higher education / research:** federal-grant compliance regimes (US) or equivalent national regimes elsewhere; multi-jurisdiction research collaboration; BYOD permissive; readiness focuses on research-data-handling scope
- **Public sector:** government-ID classifiers; statutory records-retention drives data lifecycle; Approved-Software-List culture; readiness aligns with procurement-control regimes

### Maturity-based

- **Immature:** readiness skipped or compressed to one week; SaaS inventory derived from finance procurement-records (incomplete); BYOD position undecided; vendor selection driven by analyst-MQ position rather than requirements; outcome — implementation phase discovers the gap and either rebuilds readiness retrospectively or proceeds with known holes
- **Mature:** documented 4-6 week readiness phase; named owners with calendar commitment; locked decisions signed by exec sponsor; vendor selection driven by scoped requirements; ongoing readiness-refresh cadence (quarterly review of inventory drift, annual review of locked decisions)
- **Advanced:** readiness is institutionalised as a quarterly capability assessment; SaaS-estate inventory auto-refreshed from CASB Discovery feed; identity + endpoint posture continuously measured (Microsoft Secure Score-style index); regulator-perimeter updates trigger automated re-assessment; vendor-portfolio decisions made annually against the refreshed readiness baseline

## Real-world experience

Typical patterns from production readiness phases:

| Pattern | Frequency in readiness assessments |
|---|---|
| BYOD position decided in W6 (deferred), forcing rework in implementation | ~40% of programmes |
| SaaS estate inventory missing 15-25% of in-use apps (long-tail unknown) | ~70% of programmes |
| Identity posture has at least one undocumented federation chain (e.g. legacy ADFS still federating one app) | ~60% of programmes |
| Endpoint posture overstated — "all devices managed" turns out to mean "all corporate-owned Windows; macOS partial; mobile largely unmanaged" | ~50% of programmes |
| Regulatory inventory misses cross-border-data-flow constraints | ~50% of programmes |
| Compliance lead named but under-resourced (no calendar commitment) | ~35% of programmes |
| Exception process unspecified at readiness end; created reactively during implementation | ~45% of programmes |
| Personal-use position deferred to "we will figure it out" | ~55% of programmes |

These patterns are the readiness-quality signal — programmes with three or more of these unresolved at readiness-end typically encounter the corresponding implementation-phase failure within 3-6 months.

## Integration with broader programmes

- **Vendor procurement:** readiness scoped-requirements document is the procurement input; vendor RFP scored against locked decisions, not feature checklists
- **Board reporting:** readiness business case feeds the quarterly cyber report; risk-reduction framing (per [`success-metrics.md`](success-metrics.md)) starts here
- **All 19 `03-policies/` files:** readiness gates which policies can ship at which phase; in-scope SaaS scope determines which policies are deployable; BYOD position determines whether `access-control/block-download-unmanaged-device.md` can ever ship; exception process design determines whether `dlp/external-share-link-quarantine.md` can ship without help-desk overload
- **`06-compliance/control-mapping-framework.md`:** regulatory inventory feeds the control mapping; readiness output is the input to the compliance discipline
- **`05-architecture/`:** identity + endpoint posture determines which deployment patterns ([`proxy-only-pattern.md`](../05-architecture/proxy-only-pattern.md) / [`api-only-pattern.md`](../05-architecture/api-only-pattern.md) / [`hybrid-pattern.md`](../05-architecture/hybrid-pattern.md)) are viable
- **`phased-rollout.md`:** readiness sign-off is the Phase 0 gate; without readiness complete, no phase can start
- **`change-management.md`:** locked-before-vendor-selection decisions (BYOD / personal-use / exception process) are the change-management input
- **`cost-of-ownership.md`:** readiness business case is the TCO base; readiness gap-analysis is the input to the procurement model

## Anti-patterns specific to readiness

1. **"We'll figure out the policies in implementation"** — six months in, the policy library is half-defined and the rollout is stalled on internal disagreement. Treated CASB as a tooling decision, not a policy decision
2. **"The IT team will run it"** — alerts accumulate; nobody triages; the platform becomes shelfware. No SOC integration plan; no operational capacity sized in readiness
3. **"We'll start with everything in block mode"** — high user friction within two weeks, business escalates to executive, programme is rolled back. No graduated action-class plan in readiness
4. **"BYOD is allowed but unmanaged"** — policies cover ~30% of actual exfil risk; auditor flags the gap. BYOD position wasn't decided in readiness; controls inherited a "do what you can" scope
5. **"The vendor will handle the regulator question"** — first audit cycle exposes that no control mapping exists; vendor docs cited as evidence. No compliance lead in readiness
6. **Single-owner readiness** — one architect runs the readiness alone; output is technically accurate but lacks BU and Legal sign-off; locked decisions are not actually locked
7. **Inventory-only readiness** — produces a 200-row SaaS spreadsheet, no decisions; vendor selection then runs on the spreadsheet as if it were the scoped-requirements document
8. **Skipping the personal-use position** — most-deferred locked decision; produces the GenAI-rollout political crisis 6 months later (per [`change-management.md`](change-management.md))
9. **Compressed readiness becoming the norm** — emergency-readiness (per Use case 3) is acceptable post-incident but the standard 6-week readiness must be completed retrospectively. Compressed-readiness-as-policy produces structurally-flawed programmes

## Outputs of the readiness phase

When this phase is genuinely complete, you have:

1. A **scoped requirements document** that names: user populations, SaaS scope, BYOD/personal-use position, action-class defaults, exception process, privacy posture.
2. A **populated SaaS estate inventory** with risk-classification, data-classification, and ownership per app.
3. A **named team** — sponsor, programme, technical, compliance — with calendar commitment.
4. A **regulatory inventory** with named clauses per obligation, audit cadence, evidence requirements.
5. A **gap analysis** between current controls (endpoint DLP, email DLP, IdP, SIEM) and the CASB target — where the duplication is, where the seams are, what gets retired.
6. A **business case** the executive sponsor signs that names the risk reduction (not the feature count).

Without these six outputs, vendor selection is theatre.

## Cross-references

- [`phased-rollout.md`](phased-rollout.md) — the next phase after readiness sign-off
- [`change-management.md`](change-management.md) — locked decisions become change-mgmt inputs
- [`success-metrics.md`](success-metrics.md) — readiness business case feeds the metric design
- [`cost-of-ownership.md`](cost-of-ownership.md) — readiness gap-analysis informs TCO modelling
- [`../03-policies/`](../03-policies/) — all 19 policies depend on readiness output
- [`../06-compliance/control-mapping-framework.md`](../06-compliance/control-mapping-framework.md) — regulatory inventory feeds the framework
- [`../../cyberkpis/`](../../cyberkpis/) (sibling repo) — KPI / KRI methodology
- [`../README.md`](../README.md) — repo-level navigation

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
