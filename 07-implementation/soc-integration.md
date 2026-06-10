# SOC integration

> Programme-level discipline within a CASB / SSE deployment. Vendor-agnostic.
> Reader audience: SOC lead, programme lead, SIEM / SOAR engineer.
> Most-directly governs which `03-policies/` files: alert-generating policies — all of [`detect/`](../03-policies/detect/) (5 files); [`oauth/`](../03-policies/oauth/) (2 files); [`posture/`](../03-policies/posture/) (1 file); the alert side of [`access-control/`](../03-policies/access-control/) + [`dlp/`](../03-policies/dlp/) + [`genai/`](../03-policies/genai/).

## Purpose

CASB alerts only matter if the SOC can triage them. Without integration, the platform becomes the loudest source of false-positive noise in the SIEM and gets ignored within a quarter. This discipline owns the alert-lifecycle plumbing — connector path, schema, retention, triage tiers, SOAR rules, ticketing path, and the alert-tuning cadence that determines whether the policy library remains a control or decays into noise.

## What organisations use this discipline for

SOC integration is the discipline that separates "we have CASB alerts" from "the SOC actually pays attention to them". Most failed CASB programmes do not fail because the policies were wrong — they fail because alert volume overwhelmed analyst capacity and the SOC stopped looking. The discipline owns three operational decisions: which alerts go where (connector + ticketing path), what gets automated (SOAR rules), and what the cadence is (daily / weekly / monthly tuning). Skip any of the three and the platform degrades in measurable ways within 90 days.

### Use case 1 — Managed-service-provider running tier-1 triage; in-house tier-2

- **Org type:** mid-cap insurance carrier, ~5k employees, M365 E5 + MDA + Sentinel, MSSP contract for tier-1 SOC, in-house tier-2 + tier-3 incident response
- **Trigger:** CASB programme rollout reached Phase 3 (enforcement); MSSP SOC playbook had no CASB-specific triage logic; ~80% of MDA alerts were being misclassified or auto-closed inappropriately
- **Scope:** SOC-integration discipline build — defined the SIEM ingest path (MDA → Sentinel via Defender XDR data connector → MSSP read access), defined the per-policy triage decision tree (each of the 19 policies got a tier-1 triage runbook), defined the escalation path from MSSP tier-1 to in-house tier-2, defined the SOAR automation boundary (what tier-1 can auto-close, what must escalate)
- **Outcome:** MSSP CASB-alert misclassification rate dropped from ~80% to ~12% over 8 weeks; MSSP signed off on the per-policy triage runbook as part of the SLA; in-house tier-2 throughput improved because the escalations were better-curated; ~30% reduction in MSSP triage hours after the discipline was embedded

### Use case 2 — Building in-house CASB triage capability

- **Org type:** tier-2 ASEAN bank, ~12k employees, in-house SOC with ~12 analysts across three shifts, M365 E5 + MDA, prior SIEM was Splunk Cloud (migrating to Sentinel)
- **Trigger:** the bank's SOC had not handled CASB alerts before; CASB programme was approaching Phase 2 (audit-mode); SOC lead asked for a structured ramp-up
- **Scope:** SOC-integration training programme — analysts trained on the CASB schema, the per-policy triage logic, the SIEM query library (Sentinel KQL), and the SOAR-automation boundary; SOC SLA defined for each alert class (mean-time-to-triage ≤ 4h for severity-high)
- **Outcome:** SOC analysts demonstrated tier-1 triage proficiency in 6 weeks; SIEM query library expanded to ~40 saved queries by month 3; SOAR automation enabled for known-FP-pattern alerts; MTTT for severity-high alerts settled at ~2.5h by month 4 (better than SLA)

### Use case 3 — Multi-vendor SIEM stack (Splunk + Sentinel + Chronicle parallel)

- **Org type:** multinational tech firm, ~10k employees, federated SIEM (engineering BU on Splunk, central security on Sentinel, M&A-acquired BU on Chronicle), M365 E5 + MDA + ChatGPT Enterprise
- **Trigger:** SIEM consolidation deferred 2-3 years; meanwhile CASB programme needed alerts in all three SIEMs because different BUs operated different SOCs
- **Scope:** triple-connector configuration — MDA → Sentinel via Defender XDR connector (primary); MDA → Splunk via Splunk Add-on for Microsoft Cloud Services (engineering BU); MDA → Chronicle via Chronicle Microsoft connector (acquired BU); schema-normalisation layer in each SIEM to produce a canonical event format; cross-SIEM dashboard for central security
- **Outcome:** multi-SIEM ingest worked but increased SIEM ingest cost ~40% (same events going to three places); central security got the cross-BU view; tech-debt acknowledged — triple-ingest is not the steady-state, but is the bridge until SIEM consolidation completes; documented as a known cost-of-ownership-line increase

### Use case 4 — Insider-risk programme integration

- **Org type:** tier-1 European bank, ~80k employees, mature SOC + Microsoft Purview IRM deployment, M365 E5 + MDA, EU DORA + GDPR supervised
- **Trigger:** IRM programme manager asked for CASB signals to feed the IRM Adaptive Protection scoring (per [`../03-policies/`](../03-policies/) Use case 4 patterns); siloed CASB + IRM produced duplicative alerts that exhausted analyst capacity
- **Scope:** SOC-integration upgrade — CASB activity-policy hits feed Purview IRM as boosted indicators; IRM-derived user risk score becomes a SOAR-rule input on CASB alerts (high-IRM-risk users have CASB alerts elevated automatically); cross-system correlation queries in Sentinel
- **Outcome:** standalone CASB alert TP rate ~3%; correlated CASB + IRM TP rate ~14%; analyst triage time per alert dropped because the IRM context arrived alongside the CASB alert; ~25% reduction in mean-time-to-investigate for insider-risk-class alerts; programme became the template for cross-system signal integration

## The discipline

### The alert lifecycle

```
CASB policy match → Alert generated → SIEM ingest → Triage queue
   → Decision (TP / FP / known issue / escalate)
   → If TP: incident response runbook + governance action
   → If FP: policy tuning ticket
   → If known issue: suppression with documented expiry
   → If escalate: handoff to IR / Identity / Legal / HR
```

Each transition is a documented step with a named owner. None is optional.

### SIEM ingest

| Component | What to verify |
|---|---|
| **Connector path** | Use the canonical CASB → SIEM connector, not log-screen-scrape. For Microsoft Defender for Cloud Apps → Sentinel, that is the **Microsoft Defender XDR data connector**. For Netskope / Zscaler / Palo Alto / Skyhigh, each vendor publishes a Splunk app + a Sentinel / Chronicle / QRadar connector — use the vendor's, not third-party |
| **Schema mapping** | Confirm `policy ID`, `user object ID`, `app`, `action`, `decision`, `verdict`, `device tag`, `IP`, `correlation ID` all map to expected SIEM fields. Test with a known synthetic event |
| **Retention** | CASB in-product retention (typically 30-90 days) is rarely enough for BFSI audit. Sentinel / Splunk / Chronicle forwarding for 1-7 year retention is mandatory; cost is non-trivial |
| **Throughput / quota** | Each CASB has documented ingest ceilings (events/sec/tenant; events/day/policy). Verify your peak hits don't auto-disable policies silently |
| **Schema versioning** | Vendor schema changes are routine. Pin schema version in SIEM queries; subscribe to the vendor's schema-change feed; quarterly re-test |
| **Tamper evidence** | For audit evidence, the log must be tamper-evident at rest. CASB-vendor logs forwarded to a Sentinel workspace under a different tenant boundary is the typical defensible posture |

### Triage tier

| Tier | Volume / day | Skills | Action |
|---|---|---|---|
| **Tier 1 — Triage** | High (hundreds to thousands of alerts) | Playbook-driven; analyst with access to SIEM, ticketing | Classify TP/FP/known. Close known-FP per documented suppression rules. Escalate confirmed TPs to Tier 2 |
| **Tier 2 — Investigation** | Lower (tens) | SOC analyst with IdP access, endpoint telemetry access | Correlate the CASB alert with sign-in logs, endpoint events, OAuth history. Decide containment action |
| **Tier 3 — Incident** | Single digits per quarter typically | Incident response lead | Run the IR runbook. Coordinate with Legal / HR / regulator if required |

Programmes that try to skip Tier 1 (analyst pre-triage) and route CASB alerts directly to Tier 2 end up with Tier 2 burnout and unread alerts within a quarter.

### SOAR / automation

| Action | Automate? |
|---|---|
| **Enrich** (lookup user role, IP reputation, app risk score) | Yes, always — at SIEM ingest or first analyst touch |
| **Notify** (email, Teams/Slack channel, ticket creation) | Yes — but with documented suppression to avoid noise on known issues |
| **Quarantine file / revoke OAuth grant / require password reset** | **No, not on first signal.** Require human-in-the-loop until the policy has run with measured FP rate ≤1% for ≥3 months |
| **Suspend user** | **Never automate on a single CASB signal.** Require corroborating signal (Entra ID Protection sign-in risk + UEBA + CASB) AND human approval. The blast radius (SCIM cascade per [`../03-policies/detect/mass-download-alert.md`](../03-policies/detect/mass-download-alert.md)) is too high |
| **Block app at endpoint / SWG** | Yes for confirmed-malicious; manual for tag-as-unsanctioned propagation |
| **Open ticket in ITSM** | Yes — every alert generates a ticket, even if auto-closed in Tier 1 |

### The three correlation queries every SOC needs

These three Sentinel KQL patterns (or vendor equivalents) cover ~80% of useful CASB triage:

#### 1. Three-record audit join for any CAAC / inline policy match

```kql
let session_window = 30s;
CloudAppEvents
  | where PolicyId == "<policy-id-of-interest>"
  | join kind=inner (
      SigninLogs | project SigninTime = TimeGenerated, UserId, IPAddress, ConditionalAccessStatus, CorrelationId
    ) on $left.AccountObjectId == $right.UserId
  | where abs(datetime_diff('second', TimeGenerated, SigninTime)) < session_window
  | join kind=inner (
      AuditLogs | project AuditTime = TimeGenerated, OperationName, CorrelationId
    ) on $left.CorrelationId == $right.CorrelationId
```

Without all three records correlated, an auditor will reject the evidence. The CASB alert alone is insufficient.

#### 2. Slow-and-low exfil detection (per-user weekly baseline)

See [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) Appendix B.1. The Day 1 mass-download policy (per [`../03-policies/detect/mass-download-alert.md`](../03-policies/detect/mass-download-alert.md)) catches fast-and-loud; the weekly-baseline query catches slow-and-low. Both are required.

#### 3. OAuth grant audit trail

```kql
CloudAppEvents
  | where TimeGenerated > ago(30d)
  | where ActionType has "Consent"
  | where RawEventData has_any ("Mail.ReadWrite", "Files.ReadWrite.All", "Sites.FullControl.All")
  | project TimeGenerated, AccountObjectId, Application, RawEventData
  | join kind=leftouter (
      AuditLogs | where OperationName == "Consent to application"
        | project AuditTime = TimeGenerated, InitiatedBy_user, CorrelationId
    ) on CorrelationId
```

High-permission OAuth grants are a high-value audit query for both insider risk and supply-chain compromise.

### Ticketing integration

For most BFSI deployments, the SOC works in ServiceNow / Jira / a vendor-specific platform. The CASB → SOC integration is **never** email-parse — it is a connector:

| SIEM → Ticketing path | Notes |
|---|---|
| **Sentinel → ServiceNow ITSM connector** | Use Event Management vs Security Incident Response carefully — they map to different ServiceNow tables |
| **Defender XDR → ServiceNow via Sentinel SOAR / Logic Apps** | Adds latency but allows enrichment before ticket creation |
| **Power Automate → ServiceNow custom connector** | Requires Power Automate Premium licence; outbound IPs change weekly (use service tag `AzureCloud.{region}`); orphan flows under personal accounts on offboarding |
| **Splunk SOAR / Cortex XSOAR / Chronicle SOAR** | Vendor-platform-native — typically better-engineered than the Microsoft stack alternatives for non-MS-shop tenants |
| **Email-to-ticket** | Worst option. Doesn't survive an audit. Don't ship |

Document the chosen path; do not let "we will figure out tickets later" persist into Phase 3.

### Alert tuning cadence

| Cadence | Activity |
|---|---|
| **Daily** | Tier 1 triage clears queue; SOAR suppression rules updated for known-noise patterns |
| **Weekly** | Tier 2 review of escalated TPs; trend analysis on FP rate per policy |
| **Monthly** | Policy tuning sprint — adjust thresholds, exclusion lists, scope of any policy with FP rate >5% |
| **Quarterly** | Schema-change re-validation; SOAR playbook refresh; alert-volume vs SOC capacity review |

A CASB programme without this cadence becomes shelfware within a year. Alert volume drifts up, FP rate drifts up, analyst attention drifts down, the platform ends up tuned to "everything to a generic inbox" and the SOC ignores it.

### What an auditor will ask

| Question | What you need |
|---|---|
| Show me the policy that detected this event | Policy ID + policy version + last-modified-by + change-management record |
| Prove the policy was active at T0 | Audit log of policy state from CASB + SIEM forwarding integrity |
| Prove the alert was triaged within SLA | Triage ticket with timestamp; SOC SLA documented |
| Prove the governance action was applied | The action record in CASB + the downstream SCIM / IdP event |
| Prove the user was notified (where required) | User-facing message log; HR or Legal communication record |
| Prove the false-positive rate is measured | Monthly tuning report; FP / TP / unknown classification stats per policy |
| Prove the SOC has capacity | Alert volume vs analyst headcount; documented capacity metric trend |

If you cannot answer all seven in evidence form (not slide form), the audit conversation will go badly regardless of how good the CASB platform is.

## Implementation pattern

Typical 6-8 week SOC-integration build-out alongside Phase 2 of the rollout:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Connector selection + tenant-boundary decision; SIEM-side workspace provisioning | Canonical connector path documented |
| W2 | Schema mapping validation with synthetic events; SIEM-side dashboard scaffolding | Schema fields mapped; synthetic-event test pass |
| W3 | Per-policy triage runbook drafting (one runbook per policy class); SOAR-rule scope decision (what auto-closes, what escalates) | Tier-1 runbook library v1 |
| W4 | SOC analyst training on the runbook library + Sentinel query library; SOAR rules deployed in audit mode | Analyst proficiency baseline |
| W5 | First alert wave (Phase 2 policies live in audit mode); SOC measures triage throughput + MTTT | Operational baseline data |
| W6 | Tuning cycle — refine SOAR suppression rules based on W5 noise patterns; update runbooks where ambiguity surfaced | Tuned baseline; SLA candidate values |
| W7 | Ticketing integration deployed; ServiceNow / Jira / SOAR connector tested end-to-end | Ticketing flow operational |
| W8 | SLA approval by exec sponsor + SOC lead; cadence (daily / weekly / monthly / quarterly) committed | SOC integration operational; Phase 2 → 3 ready |

The W3 runbook drafting is the discipline-defining work. Without per-policy runbooks, tier-1 analysts default to "escalate everything" and tier-2 burnout follows.

## Variants

### Industry-specific

- **BFSI:** SOC SLA is tight (MTTT ≤ 1-2h for severity-high); regulator-driven SLA evidence requirements; SIEM retention long (5-7 years typical); SOC integration tied to incident-reporting clock (BNM / MAS / HKMA)
- **Healthcare:** PHI-handling alerts route through compliance officer in parallel with SOC; HIPAA Privacy Officer is in the escalation chain; SOAR automation more restricted (PHI cannot be auto-handled by SOAR rules without human review)
- **Tech:** SOC is often part of platform engineering rather than security; SOC integration uses developer-facing tooling (Jira instead of ServiceNow); SOAR automation more aggressive (engineering-culture tolerance for automation)
- **Retail:** PCI cycle drives SOC SLA for cardholder-data alerts; alert volume high due to high transaction volume; SOAR-suppression discipline critical
- **Manufacturing:** OT/IT boundary creates split-SOC scenarios; CASB alerts route to IT SOC, OT alerts to a separate operations team; cross-pollination disciplines required
- **Higher education / research:** SOC often understaffed; runbook quality matters more than analyst count; multi-jurisdiction operations create per-jurisdiction triage workflows
- **Public sector:** Approved-Software-List culture creates clean alert-FP-suppression workflow; longer audit-cycle creates tolerance for slower MTTT

### Maturity-based

- **Immature:** CASB alerts route to a generic security inbox; no per-policy runbook; SOC analyst triages CASB alerts as best they can alongside other alert classes; FP-rate not measured; alerts age into "auto-closed because old" within 3 months
- **Mature:** documented per-policy runbooks; tier-1 / tier-2 / tier-3 split; SOAR rules deployed for known-FP patterns; monthly tuning cadence maintained; SLA measured monthly; MTTT trends published quarterly
- **Advanced:** SOC integration extends to IRM signal-boost (per Use case 4); cross-SIEM correlation queries normalised; SOAR automation extended to cross-system actions (CASB alert + endpoint-DLP signal + identity-risk score → automated containment); per-policy SLA differentiated; runbook library version-controlled; analyst-rotation schedule includes CASB-specialty period

## Real-world experience

Typical patterns from production SOC-integration build-outs:

| Pattern | Frequency |
|---|---|
| W3 runbook drafting compressed; tier-1 analysts default to "escalate everything" | ~40% of programmes |
| SOAR automation deployed before FP-rate is measured; automation amplifies bad alerts | ~25% of programmes |
| Ticketing path documented but not actually tested end-to-end | ~30% of programmes |
| Monthly tuning cadence drops within 6 months; alert volume drifts up | ~50% of programmes |
| Schema-change re-validation skipped; quarterly schema change breaks SIEM queries silently | ~35% of programmes |
| MTTT meets SLA in first quarter, drifts up by month 9 | ~45% of programmes |
| Auto-suspend on a CASB-only signal causes user-disable cascade (rare but high-severity) | ~5% of programmes; 100% of those that occur become a programme-rollback event |

## Integration with broader programmes

- **`programme-readiness.md`:** SOC capability assessment is a readiness inventory item; without SOC capacity, no enforcement phase ships
- **`phased-rollout.md`:** SOC integration completed by end of Phase 2; gates Phase 3
- **`change-management.md`:** SOC triage decisions feed back into policy tuning; tuning cycle interacts with change-management cadence
- **`success-metrics.md`:** SOC metrics M4 (TP rate), M5 (FP rate), M13 (MTTT), M14 (policy drift) are SOC-integration metrics
- **`cost-of-ownership.md`:** SOC analyst headcount (cost line 5) is the dominant ongoing ops cost
- **All alert-generating `03-policies/` files:** SOC integration is the operational arm of every detect / oauth / posture policy; without SOC integration, the detect policies generate noise without action
- **Sentinel / Splunk / Chronicle:** the SIEM is the SOC's working surface; SOC integration is the SIEM-integration discipline at the alert level
- **IRM / Insider Risk Management:** Purview IRM Adaptive Protection consumes CASB signals; SOC integration is the upstream feed
- **Incident Response runbook:** confirmed-TP escalations from tier-2 feed the IR runbook; SOC integration is the trigger-and-handoff layer

## Anti-patterns specific to SOC integration

1. **"We'll figure out the triage runbook in production"** — without runbooks, tier-1 analysts apply ad-hoc judgement; FP / TP classification is inconsistent; tuning is impossible
2. **Auto-suspend on single CASB signal** — destroys a hybrid-AD user account silently (writeback issue); breaks SCIM-cascaded apps; legitimate-but-misclassified user has their entire work day broken; rolled back within hours
3. **Email-to-ticket as the SIEM-to-SOC integration** — alerts are buried in an inbox; tickets are not created; audit evidence has no triage trail
4. **No daily-cadence Tier 1 review** — alerts age; the queue grows; SOC stops looking; the platform becomes wallpaper
5. **SOAR rules deployed in audit-mode then forgotten** — the rules exist but never enforce; SOC believes they are enforcing; gap discovered at audit
6. **Schema-change blind** — vendor changes a field name; SIEM queries silently produce empty results; "no alerts" interpreted as "all clear" when it actually means "broken integration"
7. **No SLA on Tier 1 MTTT** — analyst capacity not measured; queue grows; quarterly SOC capacity-review missing
8. **Skipping the synthetic-event test in W2** — connector reports success but schema mapping is wrong; not surfaced until first real alert; live debugging during incident
9. **Multi-SIEM ingest without de-duplication** — same event in two SIEMs counted twice; FP / TP metrics distorted; cost-of-ownership inflated
10. **No tabletop exercise** — SOC has never run an IR scenario on a CASB-detected event; first real event is also the rehearsal

## Cross-references

- [`programme-readiness.md`](programme-readiness.md) — SOC capability assessment
- [`phased-rollout.md`](phased-rollout.md) — Phase 2 dependency
- [`change-management.md`](change-management.md) — alert-volume-driven user friction
- [`success-metrics.md`](success-metrics.md) — SOC metrics M4 / M5 / M13 / M14
- [`cost-of-ownership.md`](cost-of-ownership.md) — SOC headcount cost line
- [`../03-policies/detect/`](../03-policies/detect/) — 5 alert-generating policy files
- [`../03-policies/oauth/`](../03-policies/oauth/) — 2 alert-generating policy files
- [`../03-policies/posture/sspm-tenant-misconfig-drift.md`](../03-policies/posture/sspm-tenant-misconfig-drift.md) — drift-alert policy
- [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — Appendix B.6 (the three-way audit join KQL)
- [`../README.md`](../README.md) — repo-level navigation

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
