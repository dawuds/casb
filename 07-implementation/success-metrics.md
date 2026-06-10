# Success metrics

> Programme-level discipline within a CASB / SSE deployment. Vendor-agnostic.
> Reader audience: programme lead, executive sponsor, compliance lead, finance.
> Most-directly governs which `03-policies/` files: ALL 19 — every policy has metrics that drive this discipline.

## Purpose

Programme-level KPIs / KRIs / KCIs for a CASB / SSE deployment. This discipline owns the metric catalogue, the per-audience aggregation cadence, and the anti-metric discipline (knowing what NOT to measure). Defers metric-design methodology to the sibling repo [`cyberkpis`](../../cyberkpis/) (CASB-specific candidates only here).

## What organisations use this discipline for

Most failed CASB programmes measure the wrong things. Discovered-app count goes up (looks like progress); alerts-generated goes up (looks like the platform is working); analyst-hours-spent goes up (looks like the SOC is engaged). None of these is a success signal. The discipline of measurement is the discipline of choosing the few metrics that actually correlate with risk reduction, ignoring the many that correlate with platform activity, and presenting them at the right cadence to the right audience.

### Use case 1 — Pre-IPO firm, board demands measurable security KPIs

- **Org type:** digital-native challenger bank, ~800 employees, M365 E5 + MDA, pre-IPO due diligence
- **Trigger:** lead investor's IT-due-diligence checklist required "measurable evidence of CASB programme effectiveness" — generic vendor metrics rejected as marketing rather than measurement
- **Scope:** programme-lead-and-CFO joint exercise to build a metric stack defensible at board level — selected M1 (Shadow IT reduction), M11 (DLP TP rate on regulated data classes), M12 (sensitive-data exposure trend), M16 (cost per active user)
- **Outcome:** quarterly board report carried 4-metric CASB dashboard plus residual-risk narrative; investor IT-due-diligence cleared; metric framework reused for SOC 2 Type II evidence pack; board signed off on the programme for the IPO prospectus security disclosure

### Use case 2 — BFSI under regulator inquiry, evidence pull

- **Org type:** tier-1 ASEAN universal bank, ~30k employees, M365 E5 + MDA + Sentinel, BNM RMiT supervised
- **Trigger:** BNM thematic review on cloud-data-leakage controls; the bank had to produce 24 months of measured-control-effectiveness evidence with 30 days notice
- **Scope:** retroactive evidence pull from existing CASB programme metrics (M4 TP rate per policy, M5 FP rate per policy, M11 DLP TP rate, M14 policy drift, M15 audit-evidence pull time); per-policy attestation evidence pack
- **Outcome:** evidence pack delivered within the 30-day window; per-policy effectiveness metric trends were the convincing element; review closed without finding; the discipline of measuring per-policy effectiveness (not aggregate platform metrics) is what made the response possible; pattern became the bank's documented audit-readiness baseline

### Use case 3 — Multi-year programme review, business-case validation

- **Org type:** multinational tech firm, ~10k employees, 3 years into the CASB programme, M365 E5 + MDA + Sentinel
- **Trigger:** vendor renewal cycle; finance asked "what value have we got from the spend?" — needed a defensible answer beyond compliance-checkbox framing
- **Scope:** retrospective metric analysis — M1 Shadow IT reduction trend (3-year); M11 DLP TP rate trend; M16 cost per active user trend; correlated CASB-detected events with public-disclosed incidents at peer firms (qualitative)
- **Outcome:** renewal approved at level pricing; programme team reframed the value narrative from "we detect bad stuff" to "we reduced uncontrolled-SaaS-adoption by ~40% and surfaced regulated-data sprawl in OneDrive that would have surfaced in audit otherwise"; metric trend evidence carried the renewal conversation

### Use case 4 — Programme-renewal cycle, "what value are we getting?"

- **Org type:** mid-cap insurance carrier, ~5k employees, 2 years into the CASB programme, finance scrutinising IT-security spend during corporate cost-reduction cycle
- **Trigger:** finance asked CISO to justify the CASB licence cost in concrete metrics; "we feel safer" rejected as insufficient
- **Scope:** built the M16 cost-per-active-user trend + M17 SIEM-ingest-cost trend + risk-reduction-narrative anchored to specific incidents prevented or caught early
- **Outcome:** licence retained at current scope; the programme team established quarterly value-narrative cadence to finance; the metric discipline survived the cost-reduction cycle; the discipline of cost-line measurement (per [`cost-of-ownership.md`](cost-of-ownership.md)) became part of the steady-state operations review

## The discipline

### What this is and isn't

This file lists **candidate metrics specific to CASB / SSE operations** — Shadow IT reduction, DLP TP/FP rate, OAuth grant hygiene, etc. It does **not** specify thresholds, weights, or board-facing aggregation; that is metric-design work that belongs in [`cyberkpis`](../../cyberkpis/) where the KPI / KRI / KCI / KGI distinctions and audience-layering are codified.

Every metric below is paired with: definition, data source, why it bites, and a sensible default audience tier (ops / exec / board).

### Discovery + sanction taxonomy metrics

#### M1 — Shadow IT reduction

| | |
|---|---|
| **Type** | KPI (programme outcome) |
| **Definition** | (Apps newly classified as Sanctioned or Unsanctioned in period) ÷ (Apps in Discovered/Untagged state at start of period). Trend: declining backlog of untagged apps |
| **Data source** | CASB cloud-app catalogue with tag history |
| **Audience** | Ops (weekly); Exec (quarterly trend) |
| **Why it bites** | Without movement on this metric, discovery is theatre — the dashboard exists, nothing changes |
| **Governs** | [`../03-policies/access-control/block-unsanctioned-app-with-coach.md`](../03-policies/access-control/block-unsanctioned-app-with-coach.md) |

#### M2 — Sanctioned-app coverage

| | |
|---|---|
| **Type** | KCI (control coverage) |
| **Definition** | (Sanctioned apps with at least one API connector live) ÷ (Total sanctioned apps) |
| **Data source** | CASB connected-apps page |
| **Audience** | Ops; Exec |
| **Why it bites** | Sanctioned apps without API connection have no DLP coverage — only network-level discovery |

#### M3 — Inline-protected app coverage

| | |
|---|---|
| **Type** | KCI |
| **Definition** | (Sanctioned apps onboarded to inline / proxy mode) ÷ (Sanctioned apps that *should* be onboarded by policy) |
| **Data source** | CAAC apps page (or vendor equivalent); programme scoping document |
| **Audience** | Ops |
| **Why it bites** | Inline mode is per-app onboarding effort; this metric quantifies onboarding throughput |

### Policy effectiveness metrics

#### M4 — Policy true-positive rate (per policy)

| | |
|---|---|
| **Type** | KPI |
| **Definition** | (Alerts confirmed as TP by SOC triage) ÷ (Total alerts from policy) over a window. Per-policy, not aggregated |
| **Data source** | SOC triage records correlated to CASB policy events |
| **Audience** | Ops |
| **Why it bites** | Policies with TP rate <5% are noise generators; policies with TP rate >40% may be too narrow. Tuning target depends on policy class |
| **Governs** | All 19 policies — per-policy measurement |

#### M5 — Policy false-positive rate (per policy)

| | |
|---|---|
| **Type** | KRI |
| **Definition** | (Alerts confirmed as FP by SOC triage) ÷ (Total alerts from policy) |
| **Data source** | Same as M4 |
| **Audience** | Ops |
| **Why it bites** | FP rate >10% drives analyst fatigue; >25% drives alert ignoring; >40% is operationally untenable. Triggers tuning |
| **Governs** | All 19 policies — per-policy measurement |

#### M6 — Block decision rate (per policy)

| | |
|---|---|
| **Type** | KPI (when action = block) or KRI (signals friction) |
| **Definition** | Number of policy-block actions per period, per policy. Trend over time |
| **Data source** | CASB session-policy event records |
| **Audience** | Ops; Exec (quarterly aggregate) |
| **Why it bites** | Sharp spikes signal user-behaviour change (new training, new exec hire pattern) or platform misconfiguration. Steady decline signals successful coaching |
| **Governs** | Block-action policies (access-control / dlp / genai inline / posture remediation) |

#### M7 — Exception rate

| | |
|---|---|
| **Type** | KRI |
| **Definition** | (Approved exceptions in period) ÷ (Total policy hits that would otherwise block). Per policy and aggregate |
| **Data source** | Exception register (not the CASB itself) |
| **Audience** | Ops; Compliance |
| **Why it bites** | High exception rate signals that the policy isn't sized right. Or that the business case for the policy isn't well-communicated |
| **Governs** | All block-action policies; integrates with [`change-management.md`](change-management.md) exception process |

### OAuth + identity metrics

#### M8 — High-risk OAuth grants outstanding

| | |
|---|---|
| **Type** | KRI |
| **Definition** | Count of OAuth grants with High-permission scope + unverified publisher + community-use=rare, not yet triaged. Should trend to 0 |
| **Data source** | CASB OAuth-apps page |
| **Audience** | Ops; Exec |
| **Why it bites** | Each one is an unmonitored exfil channel. The 2022-2024 OAuth-phishing campaigns made this a board-level risk |
| **Governs** | [`../03-policies/oauth/post-consent-cleanup-and-ban.md`](../03-policies/oauth/post-consent-cleanup-and-ban.md), [`../03-policies/oauth/high-scope-grant-alert.md`](../03-policies/oauth/high-scope-grant-alert.md) |

#### M9 — Banned OAuth app count + ban velocity

| | |
|---|---|
| **Type** | KPI |
| **Definition** | Count of OAuth apps banned this period; trend |
| **Data source** | CASB ban-action records |
| **Audience** | Ops |
| **Why it bites** | Throughput indicator — if zero in a quarter, either the org has zero new OAuth abuse (unlikely) or the team isn't triaging |

#### M10 — Time from termination to cross-SaaS account closure

| | |
|---|---|
| **Type** | KPI (operational) |
| **Definition** | Median time from Entra user deletion to confirmation that all named connected-SaaS local accounts have been disabled. Per termination event |
| **Data source** | Offboarding ticketing + connected-SaaS admin audits |
| **Audience** | Ops; Compliance |
| **Why it bites** | The offboarding gap is the dominant insider-access-after-departure risk. Most regulated FIs have a documented expectation (e.g. 24h or 48h) |
| **Governs** | [`../03-policies/detect/terminated-user-cross-saas.md`](../03-policies/detect/terminated-user-cross-saas.md) |

### DLP + data-protection metrics

#### M11 — DLP TP rate on sensitive-data classes

| | |
|---|---|
| **Type** | KPI |
| **Definition** | (Confirmed-TP DLP hits on regulated data classes — PCI, PII, KYC, etc.) ÷ (Total DLP hits). Per data class |
| **Data source** | SOC triage + CASB DLP event records |
| **Audience** | Ops; Compliance |
| **Why it bites** | Per-class breakdown matters — PCI false-positive rates are very different from generic-PII or source-code false-positive rates |
| **Governs** | All `dlp/` policies |

#### M12 — Sensitive-data exposure trend (external sharing)

| | |
|---|---|
| **Type** | KRI |
| **Definition** | Count of files with sensitivity label ≥ Confidential currently shared externally in OneDrive/SharePoint/Google Drive. Trend |
| **Data source** | CASB file-policy scans |
| **Audience** | Exec; Compliance |
| **Why it bites** | Sharing drift is a dominant insider data-loss vector. Trend matters more than absolute count |
| **Governs** | [`../03-policies/dlp/external-share-link-quarantine.md`](../03-policies/dlp/external-share-link-quarantine.md), [`../03-policies/dlp/auto-label-pci-data.md`](../03-policies/dlp/auto-label-pci-data.md) |

### Operational metrics

#### M13 — Mean time to triage (MTTT)

| | |
|---|---|
| **Type** | KPI |
| **Definition** | Median time from alert generation to first-touch triage decision. Per alert class |
| **Data source** | SOC ticketing |
| **Audience** | Ops; Exec |
| **Why it bites** | Discipline metric for the SOC. Drifting MTTT signals capacity or alert-volume issue |
| **Governs** | All alert-generating policies; integrates with [`soc-integration.md`](soc-integration.md) |

#### M14 — Policy drift

| | |
|---|---|
| **Type** | KCI |
| **Definition** | Number of CASB policies modified out-of-band (not via change ticket) detected via audit log review. Should trend to 0 |
| **Data source** | CASB audit log + change-management ticketing |
| **Audience** | Compliance |
| **Why it bites** | An undocumented policy change is an audit finding regardless of the change's content |

#### M15 — Audit-evidence pull time

| | |
|---|---|
| **Type** | KCI (operational maturity) |
| **Definition** | Time to produce an audit-evidence package for a named control on request — policy state + relevant event records + correlated identity logs |
| **Data source** | Audit-prep runbook |
| **Audience** | Compliance |
| **Why it bites** | If this takes >5 working days, the SIEM forwarding setup is wrong, the query library is missing, or the schema mapping is unstable. Indirect indicator of platform health |

### Cost metrics

#### M16 — Cost per active user per month

| | |
|---|---|
| **Type** | KPI (FinOps) |
| **Definition** | Total platform cost (licensing + Sentinel ingest + admin headcount allocation) ÷ active users |
| **Data source** | Finance + ops cost reports |
| **Audience** | Exec; Finance |
| **Why it bites** | Per-user pricing scales unfavourably with contractors, BYOD, partners. Trending CPMU signals whether the deployment is scaling efficiently |
| **Governs** | Programme economics; integrates with [`cost-of-ownership.md`](cost-of-ownership.md) |

#### M17 — Sentinel / SIEM ingest cost trend

| | |
|---|---|
| **Type** | KRI |
| **Definition** | $/month CASB-driven SIEM ingest |
| **Data source** | Cloud cost reports |
| **Audience** | Ops; Finance |
| **Why it bites** | Verbose policies (clipboard inspection at scale) inflate ingest sharply. A 5k-seat deployment with full [`../03-policies/genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md) added ~100 GB/month per the MDA playbook estimate. Naming the cost early avoids surprise |

### Anti-metrics (do not measure these)

| Metric | Why not |
|---|---|
| **Total alerts generated** | Volume isn't quality. High alert count with low TP rate is a problem disguised as productivity |
| **Number of policies configured** | More policies isn't better protection; well-tuned policies are. Many policies, no tuning, = shelfware with a metric |
| **Discovered-app count** | Cardinality is not progress. M1 (sanction-decision rate) matters; the raw count doesn't |
| **Vendor-supplied "platform health score"** | Vendor scoring of their own platform is not measurement |

### Aggregation cadence

| Audience | Cadence | What they see |
|---|---|---|
| **Operations team** | Weekly | M1, M4, M5, M6, M7, M11, M13 with per-policy detail |
| **Programme lead** | Monthly | Above + M2, M3, M8, M9, M10, M14, M15 trends |
| **Executive sponsor** | Quarterly | Aggregate of programme lead's view + M16, M17 |
| **Board** | Semi-annually | 3-5 metrics maximum (typically M1, M11, M12, M16, plus a residual-risk narrative) |

Defer the aggregation methodology and the audience-layering principles to [`cyberkpis`](../../cyberkpis/) — this file owns the candidate-metric content; that repo owns the metric-design methodology.

## Implementation pattern

Typical 6-week metric-discipline build-out alongside Phases 2-3 of the rollout:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Metric selection workshop — confirm which of M1-M17 are in scope; defer those not relevant | Selected metric list per phase |
| W2 | Data-source verification — for each in-scope metric, confirm the data exists in CASB / SIEM and can be queried | Data-source-to-metric mapping; gaps logged |
| W3 | Per-metric query / dashboard build — Sentinel KQL or equivalent for each metric; weekly + monthly + quarterly views | Operational dashboard scaffolding |
| W4 | Per-audience report template — ops weekly, programme-lead monthly, exec quarterly, board semi-annually | Report-template library |
| W5 | First-pass measurement on Phase-2 audit-mode policies; baseline data collected | Per-policy baseline values |
| W6 | Cadence committed with exec sponsor; metric ownership assigned per audience tier | Metric-discipline operational |
| Quarterly thereafter | Metric review — add / retire metrics; tune thresholds; review against [`cyberkpis`](../../cyberkpis/) methodology updates | Steady-state metric governance |

## Variants

### Industry-specific

- **BFSI:** M11 (DLP TP rate on PCI / KYC) + M15 (audit-evidence pull time) dominate; regulator-driven cadence (semi-annual or annual board cyber report under BNM / MAS / HKMA expectations); M16 cost trend reviewed under operational-risk-capital lens
- **Healthcare:** M11 on PHI classes; M12 (sensitive-data exposure) is BAA-adjacent metric; M13 MTTT tighter for PHI alerts; HIPAA Privacy Officer in the metric-review audience
- **Tech:** M1 (Shadow IT reduction) is the dominant exec-level metric; M5 (FP rate) drives engineering culture pressure; M16 cost-per-user is the board-level discipline metric for cost-conscious tech firms
- **Retail:** M11 on PCI is the audit-driver; M7 exception rate is high-attention due to operational scale; M16 CPMU sensitivity is high under retail margin pressure
- **Manufacturing:** M1 + M12 dominate; trade-secret-class DLP metrics override generic PII focus; M&A scenarios create cadence resets per acquired-firm
- **Higher education / research:** federal-grant compliance regimes drive M15 (audit-evidence pull time); per-grant cadence overrides default cadences
- **Public sector:** statutory records-retention drives M12 and a custom retention-compliance metric; the metric stack often expanded with public-disclosure-readiness measures

### Maturity-based

- **Immature:** measuring discovered-app count, total alerts, analyst-hours-spent (all anti-metrics); board sees a 10-row metric table that means nothing; CISO defends the programme on the basis of activity
- **Mature:** documented M1 / M11 / M16 + 3-4 other metrics per audience tier; quarterly cadence; per-metric trend reported; metric review every quarter; new metrics added selectively
- **Advanced:** metric-discipline integrated with [`cyberkpis`](../../cyberkpis/) methodology; KPI / KRI / KCI / KGI distinctions applied per metric; FAIR-style quantification on top metrics; risk-reduction narrative grounded in measured-trend evidence; metrics versioned and audit-tracked; metric retirement is a formal decision not an oversight

## Real-world experience

Typical patterns from production metric programmes:

| Pattern | Frequency |
|---|---|
| Programme team measures total-alerts-generated as the headline metric | ~50% of immature programmes |
| Board metrics include 10+ rows; board cannot identify which 3 matter | ~40% of programmes |
| M11 (DLP TP rate) reported as aggregate not per-data-class — masks low-quality classifiers | ~50% of programmes |
| M16 (cost per user) not reported to exec — programme defends spend reactively | ~60% of programmes |
| M15 (audit-evidence pull time) not measured — discovered at first audit-evidence-pull crisis | ~70% of programmes |
| Quarterly cadence drops after first year; metric reports become annual or stop | ~45% of programmes |
| Anti-metrics persist (discovered-app count being the public victory) for >12 months | ~55% of programmes |

## Integration with broader programmes

- **`programme-readiness.md`:** readiness business case is the initial metric stack; locked-decisions document determines which policies generate which metrics
- **`phased-rollout.md`:** per-phase done-criteria are metric-driven; metric baseline established in Phase 2
- **`soc-integration.md`:** M4 / M5 / M13 / M14 are SOC-integration-managed
- **`change-management.md`:** M6 / M7 are change-management-managed
- **`cost-of-ownership.md`:** M16 / M17 are TCO-discipline-managed
- **All 19 `03-policies/` files:** each policy has a "Real-world FP experience" section feeding M4 / M5; each policy's Integration-with-broader-programmes section feeds the board / audit metric flow
- **[`cyberkpis`](../../cyberkpis/) (sibling repo):** owns KPI / KRI / KCI / KGI methodology and audience-layering principles
- **Board cyber report:** semi-annual cadence; 3-5 metrics + residual-risk narrative
- **Audit-evidence pack:** annual cadence; M15-based; per-policy attestation evidence

## Anti-patterns specific to metric design

1. **Anti-metric dominance** — discovered-app count / total-alerts / analyst-hours-spent reported as success measures; the metric story tells stakeholders the platform is busy, not effective
2. **Aggregate-not-per-policy DLP rate** — M11 reported as "we have a 70% DLP TP rate" obscures that one policy is 95% and another is 8%; per-policy breakdown is the discipline
3. **Board metric overload** — 10+ rows in the board report; board cannot identify priorities; metric discipline degrades to "metric supply" rather than "metric decision"
4. **No M15 measurement** — audit-evidence-pull capability not measured until the first crisis pull; programme discovers it cannot produce evidence within the audit window
5. **No M16 cost trend** — programme defends spend reactively under finance pressure; lacks the trend evidence to argue against cuts
6. **Metric retirement informal** — old metrics persist beyond relevance because nobody decided to retire them; new metrics added on top; reporting bloat masks signal
7. **Vendor-supplied metrics treated as measurement** — "Microsoft Secure Score" or vendor "platform health score" carried in board report as the success signal; vendor self-attestation laundered as measurement
8. **Cadence drift** — quarterly cadence becomes "we do it when we remember"; metric trend story breaks
9. **No per-audience tiering** — same metric stack reported to ops and to board; either ops gets a board-level summary (too coarse) or board gets ops-level detail (too noisy)
10. **No risk-reduction narrative** — metrics presented without "what bad thing did we prevent / catch / make less likely" framing; board cannot translate numbers into risk decisions

## Cross-references

- [`programme-readiness.md`](programme-readiness.md) — metric stack derived from readiness business case
- [`phased-rollout.md`](phased-rollout.md) — per-phase done-criteria are metric-driven
- [`soc-integration.md`](soc-integration.md) — SOC-managed metrics (M4 / M5 / M13 / M14)
- [`change-management.md`](change-management.md) — change-management-managed metrics (M6 / M7)
- [`cost-of-ownership.md`](cost-of-ownership.md) — TCO-discipline metrics (M16 / M17)
- [`../03-policies/`](../03-policies/) — all 19 policies; each "Real-world FP experience" section feeds M4 / M5
- [`../../cyberkpis/`](../../cyberkpis/) (sibling repo) — KPI / KRI methodology this defers to
- [`../README.md`](../README.md) — repo-level navigation

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
