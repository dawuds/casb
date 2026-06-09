# Success metrics

> Programme-level KPIs / KRIs / KCIs for a CASB / SSE deployment. Defers metric-design methodology to the sibling repo [`cyberkpis`](../../cyberkpis/) (CASB-specific candidates only here).

## What this is and isn't

This file lists **candidate metrics specific to CASB / SSE operations** — Shadow IT reduction, DLP TP/FP rate, OAuth grant hygiene, etc. It does **not** specify thresholds, weights, or board-facing aggregation; that is metric-design work that belongs in [`cyberkpis`](../../cyberkpis/) where the KPI / KRI / KCI / KGI distinctions and audience-layering are codified.

Every metric below is paired with: definition, data source, why it bites, and a sensible default audience tier (ops / exec / board).

## Discovery + sanction taxonomy metrics

### M1 — Shadow IT reduction

| | |
|---|---|
| **Type** | KPI (programme outcome) |
| **Definition** | (Apps newly classified as Sanctioned or Unsanctioned in period) ÷ (Apps in Discovered/Untagged state at start of period). Trend: declining backlog of untagged apps |
| **Data source** | CASB cloud-app catalogue with tag history |
| **Audience** | Ops (weekly); Exec (quarterly trend) |
| **Why it bites** | Without movement on this metric, discovery is theatre — the dashboard exists, nothing changes |

### M2 — Sanctioned-app coverage

| | |
|---|---|
| **Type** | KCI (control coverage) |
| **Definition** | (Sanctioned apps with at least one API connector live) ÷ (Total sanctioned apps) |
| **Data source** | CASB connected-apps page |
| **Audience** | Ops; Exec |
| **Why it bites** | Sanctioned apps without API connection have no DLP coverage — only network-level discovery |

### M3 — Inline-protected app coverage

| | |
|---|---|
| **Type** | KCI |
| **Definition** | (Sanctioned apps onboarded to inline / proxy mode) ÷ (Sanctioned apps that *should* be onboarded by policy) |
| **Data source** | CAAC apps page (or vendor equivalent); programme scoping document |
| **Audience** | Ops |
| **Why it bites** | Inline mode is per-app onboarding effort; this metric quantifies onboarding throughput |

## Policy effectiveness metrics

### M4 — Policy true-positive rate (per policy)

| | |
|---|---|
| **Type** | KPI |
| **Definition** | (Alerts confirmed as TP by SOC triage) ÷ (Total alerts from policy) over a window. Per-policy, not aggregated |
| **Data source** | SOC triage records correlated to CASB policy events |
| **Audience** | Ops |
| **Why it bites** | Policies with TP rate <5% are noise generators; policies with TP rate >40% may be too narrow. Tuning target depends on policy class |

### M5 — Policy false-positive rate (per policy)

| | |
|---|---|
| **Type** | KRI |
| **Definition** | (Alerts confirmed as FP by SOC triage) ÷ (Total alerts from policy) |
| **Data source** | Same as M4 |
| **Audience** | Ops |
| **Why it bites** | FP rate >10% drives analyst fatigue; >25% drives alert ignoring; >40% is operationally untenable. Triggers tuning |

### M6 — Block decision rate (per policy)

| | |
|---|---|
| **Type** | KPI (when action = block) or KRI (signals friction) |
| **Definition** | Number of policy-block actions per period, per policy. Trend over time |
| **Data source** | CASB session-policy event records |
| **Audience** | Ops; Exec (quarterly aggregate) |
| **Why it bites** | Sharp spikes signal user-behaviour change (new training, new exec hire pattern) or platform misconfiguration. Steady decline signals successful coaching |

### M7 — Exception rate

| | |
|---|---|
| **Type** | KRI |
| **Definition** | (Approved exceptions in period) ÷ (Total policy hits that would otherwise block). Per policy and aggregate |
| **Data source** | Exception register (not the CASB itself) |
| **Audience** | Ops; Compliance |
| **Why it bites** | High exception rate signals that the policy isn't sized right. Or that the business case for the policy isn't well-communicated |

## OAuth + identity metrics

### M8 — High-risk OAuth grants outstanding

| | |
|---|---|
| **Type** | KRI |
| **Definition** | Count of OAuth grants with High-permission scope + unverified publisher + community-use=rare, not yet triaged. Should trend to 0 |
| **Data source** | CASB OAuth-apps page |
| **Audience** | Ops; Exec |
| **Why it bites** | Each one is an unmonitored exfil channel. The 2022-2024 OAuth-phishing campaigns made this a board-level risk |

### M9 — Banned OAuth app count + ban velocity

| | |
|---|---|
| **Type** | KPI |
| **Definition** | Count of OAuth apps banned this period; trend |
| **Data source** | CASB ban-action records |
| **Audience** | Ops |
| **Why it bites** | Throughput indicator — if zero in a quarter, either the org has zero new OAuth abuse (unlikely) or the team isn't triaging |

### M10 — Time from termination to cross-SaaS account closure

| | |
|---|---|
| **Type** | KPI (operational) |
| **Definition** | Median time from Entra user deletion to confirmation that all named connected-SaaS local accounts have been disabled. Per termination event |
| **Data source** | Offboarding ticketing + connected-SaaS admin audits |
| **Audience** | Ops; Compliance |
| **Why it bites** | The offboarding gap is the dominant insider-access-after-departure risk. Most regulated FIs have a documented expectation (e.g. 24h or 48h) |

## DLP + data-protection metrics

### M11 — DLP TP rate on sensitive-data classes

| | |
|---|---|
| **Type** | KPI |
| **Definition** | (Confirmed-TP DLP hits on regulated data classes — PCI, PII, KYC, etc.) ÷ (Total DLP hits). Per data class |
| **Data source** | SOC triage + CASB DLP event records |
| **Audience** | Ops; Compliance |
| **Why it bites** | Per-class breakdown matters — PCI false-positive rates are very different from generic-PII or source-code false-positive rates |

### M12 — Sensitive-data exposure trend (external sharing)

| | |
|---|---|
| **Type** | KRI |
| **Definition** | Count of files with sensitivity label ≥ Confidential currently shared externally in OneDrive/SharePoint/Google Drive. Trend |
| **Data source** | CASB file-policy scans |
| **Audience** | Exec; Compliance |
| **Why it bites** | Sharing drift is a dominant insider data-loss vector. Trend matters more than absolute count |

## Operational metrics

### M13 — Mean time to triage (MTTT)

| | |
|---|---|
| **Type** | KPI |
| **Definition** | Median time from alert generation to first-touch triage decision. Per alert class |
| **Data source** | SOC ticketing |
| **Audience** | Ops; Exec |
| **Why it bites** | Discipline metric for the SOC. Drifting MTTT signals capacity or alert-volume issue |

### M14 — Policy drift

| | |
|---|---|
| **Type** | KCI |
| **Definition** | Number of CASB policies modified out-of-band (not via change ticket) detected via audit log review. Should trend to 0 |
| **Data source** | CASB audit log + change-management ticketing |
| **Audience** | Compliance |
| **Why it bites** | An undocumented policy change is an audit finding regardless of the change's content |

### M15 — Audit-evidence pull time

| | |
|---|---|
| **Type** | KCI (operational maturity) |
| **Definition** | Time to produce an audit-evidence package for a named control on request — policy state + relevant event records + correlated identity logs |
| **Data source** | Audit-prep runbook |
| **Audience** | Compliance |
| **Why it bites** | If this takes >5 working days, the SIEM forwarding setup is wrong, the query library is missing, or the schema mapping is unstable. Indirect indicator of platform health |

## Cost metrics

### M16 — Cost per active user per month

| | |
|---|---|
| **Type** | KPI (FinOps) |
| **Definition** | Total platform cost (licensing + Sentinel ingest + admin headcount allocation) ÷ active users |
| **Data source** | Finance + ops cost reports |
| **Audience** | Exec; Finance |
| **Why it bites** | Per-user pricing scales unfavourably with contractors, BYOD, partners. Trending CPMU signals whether the deployment is scaling efficiently |

### M17 — Sentinel / SIEM ingest cost trend

| | |
|---|---|
| **Type** | KRI |
| **Definition** | $/month CASB-driven SIEM ingest |
| **Data source** | Cloud cost reports |
| **Audience** | Ops; Finance |
| **Why it bites** | Verbose policies (clipboard inspection at scale) inflate ingest sharply. A 5k-seat deployment with full Policy 9 (clipboard) added ~100 GB/month per the MDA playbook estimate. Naming the cost early avoids surprise |

## Anti-metrics (do not measure these)

| Metric | Why not |
|---|---|
| **Total alerts generated** | Volume isn't quality. High alert count with low TP rate is a problem disguised as productivity |
| **Number of policies configured** | More policies isn't better protection; well-tuned policies are. Many policies, no tuning, = shelfware with a metric |
| **Discovered-app count** | Cardinality is not progress. M1 (sanction-decision rate) matters; the raw count doesn't |
| **Vendor-supplied "platform health score"** | Vendor scoring of their own platform is not measurement |

## Aggregation cadence

| Audience | Cadence | What they see |
|---|---|---|
| **Operations team** | Weekly | M1, M4, M5, M6, M7, M11, M13 with per-policy detail |
| **Programme lead** | Monthly | Above + M2, M3, M8, M9, M10, M14, M15 trends |
| **Executive sponsor** | Quarterly | Aggregate of programme lead's view + M16, M17 |
| **Board** | Semi-annually | 3-5 metrics maximum (typically M1, M11, M12, M16, plus a residual-risk narrative) |

Defer the aggregation methodology and the audience-layering principles to [`cyberkpis`](../../cyberkpis/) — this file owns the candidate-metric content; that repo owns the metric-design methodology.
