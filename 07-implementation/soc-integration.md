# SOC integration

> CASB alerts only matter if the SOC can triage them. Without this, the platform becomes the loudest source of false-positive noise in the SIEM and gets ignored. Vendor-agnostic.

## The alert lifecycle

```
CASB policy match → Alert generated → SIEM ingest → Triage queue
   → Decision (TP / FP / known issue / escalate)
   → If TP: incident response runbook + governance action
   → If FP: policy tuning ticket
   → If known issue: suppression with documented expiry
   → If escalate: handoff to IR / Identity / Legal / HR
```

Each transition is a documented step with a named owner. None is optional.

## SIEM ingest

| Component | What to verify |
|---|---|
| **Connector path** | Use the canonical CASB → SIEM connector, not log-screen-scrape. For Microsoft Defender for Cloud Apps → Sentinel, that is the **Microsoft Defender XDR data connector**. For Netskope / Zscaler / Palo Alto / Skyhigh, each vendor publishes a Splunk app + a Sentinel / Chronicle / QRadar connector — use the vendor's, not third-party |
| **Schema mapping** | Confirm `policy ID`, `user object ID`, `app`, `action`, `decision`, `verdict`, `device tag`, `IP`, `correlation ID` all map to expected SIEM fields. Test with a known synthetic event |
| **Retention** | CASB in-product retention (typically 30-90 days) is rarely enough for BFSI audit. Sentinel / Splunk / Chronicle forwarding for 1-7 year retention is mandatory; cost is non-trivial |
| **Throughput / quota** | Each CASB has documented ingest ceilings (events/sec/tenant; events/day/policy). Verify your peak hits don't auto-disable policies silently |
| **Schema versioning** | Vendor schema changes are routine. Pin schema version in SIEM queries; subscribe to the vendor's schema-change feed; quarterly re-test |
| **Tamper evidence** | For audit evidence, the log must be tamper-evident at rest. CASB-vendor logs forwarded to a Sentinel workspace under a different tenant boundary is the typical defensible posture |

## Triage tier

| Tier | Volume / day | Skills | Action |
|---|---|---|---|
| **Tier 1 — Triage** | High (hundreds to thousands of alerts) | Playbook-driven; analyst with access to SIEM, ticketing | Classify TP/FP/known. Close known-FP per documented suppression rules. Escalate confirmed TPs to Tier 2 |
| **Tier 2 — Investigation** | Lower (tens) | SOC analyst with IdP access, endpoint telemetry access | Correlate the CASB alert with sign-in logs, endpoint events, OAuth history. Decide containment action |
| **Tier 3 — Incident** | Single digits per quarter typically | Incident response lead | Run the IR runbook. Coordinate with Legal / HR / regulator if required |

Programmes that try to skip Tier 1 (analyst pre-triage) and route CASB alerts directly to Tier 2 end up with Tier 2 burnout and unread alerts within a quarter.

## SOAR / automation

| Action | Automate? |
|---|---|
| **Enrich** (lookup user role, IP reputation, app risk score) | Yes, always — at SIEM ingest or first analyst touch |
| **Notify** (email, Teams/Slack channel, ticket creation) | Yes — but with documented suppression to avoid noise on known issues |
| **Quarantine file / revoke OAuth grant / require password reset** | **No, not on first signal.** Require human-in-the-loop until the policy has run with measured FP rate ≤1% for ≥3 months |
| **Suspend user** | **Never automate on a single CASB signal.** Require corroborating signal (Entra ID Protection sign-in risk + UEBA + CASB) AND human approval. The blast radius (SCIM cascade per `04-vendors/microsoft-defender-for-cloud-apps.md` Policy 3) is too high |
| **Block app at endpoint / SWG** | Yes for confirmed-malicious; manual for tag-as-unsanctioned propagation |
| **Open ticket in ITSM** | Yes — every alert generates a ticket, even if auto-closed in Tier 1 |

## The three correlation queries every SOC needs

These three Sentinel KQL patterns (or vendor equivalents) cover ~80% of useful CASB triage:

### 1. Three-record audit join for any CAAC / inline policy match

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

### 2. Slow-and-low exfil detection (per-user weekly baseline)

See `04-vendors/microsoft-defender-for-cloud-apps.md` Appendix B.1. The Day 1 mass-download policy catches fast-and-loud; the weekly-baseline query catches slow-and-low. Both are required.

### 3. OAuth grant audit trail

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

## Ticketing integration

For most BFSI deployments, the SOC works in ServiceNow / Jira / a vendor-specific platform. The CASB → SOC integration is **never** email-parse — it is a connector:

| SIEM → Ticketing path | Notes |
|---|---|
| **Sentinel → ServiceNow ITSM connector** | Use Event Management vs Security Incident Response carefully — they map to different ServiceNow tables |
| **Defender XDR → ServiceNow via Sentinel SOAR / Logic Apps** | Adds latency but allows enrichment before ticket creation |
| **Power Automate → ServiceNow custom connector** | Requires Power Automate Premium licence; outbound IPs change weekly (use service tag `AzureCloud.{region}`); orphan flows under personal accounts on offboarding |
| **Splunk SOAR / Cortex XSOAR / Chronicle SOAR** | Vendor-platform-native — typically better-engineered than the Microsoft stack alternatives for non-MS-shop tenants |
| **Email-to-ticket** | Worst option. Doesn't survive an audit. Don't ship |

Document the chosen path; do not let "we will figure out tickets later" persist into Phase 3.

## Alert tuning cadence

| Cadence | Activity |
|---|---|
| **Daily** | Tier 1 triage clears queue; SOAR suppression rules updated for known-noise patterns |
| **Weekly** | Tier 2 review of escalated TPs; trend analysis on FP rate per policy |
| **Monthly** | Policy tuning sprint — adjust thresholds, exclusion lists, scope of any policy with FP rate >5% |
| **Quarterly** | Schema-change re-validation; SOAR playbook refresh; alert-volume vs SOC capacity review |

A CASB programme without this cadence becomes shelfware within a year. Alert volume drifts up, FP rate drifts up, analyst attention drifts down, the platform ends up tuned to "everything to a generic inbox" and the SOC ignores it.

## What an auditor will ask

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
