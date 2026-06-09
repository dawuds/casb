# OAuth post-consent cleanup and ban

> Status: v0.0 — MDA column lens-reviewed from playbook v1 (Policy 2a); other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [OAuth-app discovery and governance](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 2a](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Classic OAuth apps page anomalies).

## Purpose

Catalogue delegated-permission OAuth grants made by users to third-party apps in M365 / Workspace / Salesforce; ban known-malicious and per-user-revoke confirmed-bad. Counters MITRE ATT&CK `T1528 Steal Application Access Token` (Detect + Respond, post-consent), `T1550.001 Application Access Token` (Detect on subsequent token use), `T1567` / `T1071.001` (Detect, post-event exfil).

**Reframing:** this is a **cleanup control, not a primary defence.** Primary control for T1528 is the Entra admin-consent workflow + verified-publisher restriction, configured upstream in Entra (not MDA). MDA Policy 2a inspects what already happened.

## What organisations use this for

Most organisations stand this up *after* a consent-phishing incident — not before. The pattern is consistent: an internal IR ticket surfaces a suspicious "DocuSign" or "Adobe Acrobat" lookalike with `Mail.ReadWrite` + `offline_access` granted by 40 users in the last 72 hours; the SOC discovers there is no central inventory of OAuth grants and no workflow to revoke at scale; the OAuth-apps view in MDA becomes the cleanup surface, and from that incident onward it is policed continuously. The Storm-2372 device-code-phishing cluster (active across 2024-2025) and the TA453-class consent-phishing wave both produced this exact rollout trigger in regulated FS.

The second deployment pattern: this policy is the **compensating cleanup layer** beneath a newly-enabled Entra admin-consent workflow. Once admin-consent-required is switched on, the existing back-catalogue of user-granted consents does not disappear — those grants persist until someone revokes them. MDA Policy 2a is the inventory + revocation workflow that retires that legacy debt. Without it, the admin-consent workflow only covers new grants forward.

### Use case 1 — Tier-1 ASEAN universal bank, post-Storm-2372 mass-cleanup

- **Org type:** tier-1 ASEAN universal bank, ~30k employees, M365 E5, BNM RMiT supervised, App Governance add-on licensed
- **Trigger:** internal IR identified a Storm-2372-style device-code-phishing wave targeting RM-coded mailboxes. Initial scoping found 73 users had consented to a "Microsoft Teams Connector" lookalike with `Mail.ReadWrite` + `MailboxSettings.ReadWrite` + `offline_access` over a 6-week window. Tenant had never run a baseline OAuth-grant inventory; no ban workflow existed
- **Scope:** tenant-wide ban on the named app ID; per-user revoke for the 73 confirmed-bad grants; back-catalogue review of every High-permission OAuth app with `Mail.*` or `Files.*` scopes consented in the prior 12 months (~410 distinct apps surfaced)
- **Outcome:** named app banned within 90 minutes of IR confirmation; mail-forwarding rules + delegate additions audited via Entra AuditLogs for the 73 users; 18 additional lookalike app IDs surfaced and banned over the next 4 weeks; control evidence pack delivered to BNM under the post-incident notification process [VERIFY current RMiT incident-notification clause + clock]; admin-consent workflow rolled out 90 days later with this policy as the cleanup layer underneath

### Use case 2 — Merchant acquirer with multi-jurisdiction tenant, post-TA453-class wave

- **Org type:** payments processor operating across MY / SG / HK / TH; M365 E5; in-scope for PCI DSS + BNM RMiT + MAS TRM + HKMA SA-2
- **Trigger:** threat-intelligence brief from regional CERT identified TA453-class consent-phishing targeting payments-sector finance teams across ASEAN. Board ask within 48 hours for "what is our exposure and what would we do if we found it". Tenant had only the built-in MDA anomaly policies running, all in alert-only
- **Scope:** ran the back-catalogue review across all four jurisdictional sub-tenants; built named-runbook for the ban workflow (decision gate, communication plan to integration owners, regulator-notification trigger per jurisdiction); enabled App Governance Predictive Risk in notify-only as the forward-looking layer
- **Outcome:** 12 lookalike apps surfaced and banned across the four sub-tenants; no confirmed compromise from TA453 specifically, but the exercise surfaced 4 legitimately-consented ISVs with `Mail.ReadWrite` that the business had forgotten about — vendor-risk review triggered for all four; ban-runbook delivered to MAS / HKMA examiners on the next supervisory cycle as evidence of post-consent response capability

### Use case 3 — Digital-native challenger bank, Entra admin-consent workflow rollout

- **Org type:** neobank, ~800 employees, high-velocity engineering culture, M365 E5, board-mandated cloud-first posture
- **Trigger:** internal audit finding — engineering teams had been freely consenting to GitHub/Vercel/Linear/observability OAuth apps with broad scopes. ~140 distinct OAuth apps had `Mail.*` or `Files.*` grants from at least one user. Audit recommended admin-consent workflow with a documented exception path
- **Scope:** Entra admin-consent workflow switched on for all apps requesting any *.ReadWrite or *.FullControl scope; MDA Policy 2a deployed as the back-catalogue cleanup layer + the ongoing inventory layer; named "ISV onboarding desk" workflow created in ServiceNow with a 5-day SLA for admin review
- **Outcome:** ~140 back-catalogue apps reviewed over 8 weeks; 27 banned (lookalikes, ghost apps with no business owner, vendor-test apps left over from POCs); 113 admin-consented at tenant scope with documented business owner; ISV onboarding desk has handled ~8-12 new requests per month at steady state. Engineering friction surfaced during W2-W4 and was mitigated by the 5-day SLA + executive sponsorship from the CTO

### Use case 4 — Regulated FI back-catalogue cleanup as compensating control

- **Org type:** tier-2 regional bank, ~12k employees, M365 E5, recent regulatory inspection finding on "third-party application risk in cloud productivity stack"
- **Trigger:** examiner finding from the prudential supervisor citing lack of inventory and lifecycle management for third-party OAuth integrations in the cloud productivity stack. 90-day remediation clock [VERIFY against the supervisor's actual finding-resolution mechanic, not the citation here]
- **Scope:** back-catalogue review of every OAuth grant with `Mail.*`, `Files.*`, `Sites.*`, `Calendars.*`, or `User.ReadBasic.All`+ scope; bilateral revocation/approval workflow with named business owner sign-off per app; quarterly attestation cycle with output to the third-party risk register; integration with vendor-risk DPIA refresh
- **Outcome:** 320 OAuth-app inventory surfaced; 41 banned (no business owner identifiable, lookalikes, dormant apps with no activity in 12 months); 60 approved-with-conditions (requiring DPA refresh or sub-processor verification); 219 approved as-is. Remediation evidence pack accepted by examiner

## Implementation pattern

Typical 8-week rollout for a tenant new to OAuth governance, **post-incident** or **as compensating control under an Entra admin-consent workflow**:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Inventory pull: every OAuth grant in OAuth-apps view across M365 / Workspace / Salesforce connectors. Categorise by permission scope (High = `*.ReadWrite`, `*.FullControl`, `Mail.*`; Medium = `*.Read.All`; Low = `User.Read`); by publisher verified-status; by user-count | Inventory CSV with risk-tiering; back-catalogue baseline frozen |
| W2 | Cross-reference inventory against the firm's known-good ISV list (vendor-risk register / procurement records). Tag each app as Known-good / Unknown / Suspicious. Build SOC runbook: criteria for ban-decision, communication plan to integration owners pre-ban, rollback path | Triage queue with categorised apps; ban-decision criteria documented |
| W3 | Manual review of every High-permission, unverified-publisher, low-community-use app. Identify confirmed-malicious for immediate ban. Identify Unknowns for business-owner outreach | First-wave ban list (typically 5-30 apps for a fresh deployment); business-owner outreach in flight |
| W4 | Execute first-wave bans during a documented change window. Communicate to integration owners 24h before; monitor for breakage during + 48h after | First-wave bans completed; breakage triaged; ban-runbook validated |
| W5 | Enable MDA built-in anomaly policies (Misleading OAuth app name / Misleading publisher name / Malicious OAuth app consent) in **alert-only** mode. Document SOC triage criteria | Built-in anomaly alerts flowing to SOC; baseline FP rate being measured |
| W6 | Tune anomaly-policy sensitivity; whitelist verified ISVs surfaced as FPs in W5 (lookalike publisher-name false matches, etc.). Begin App Governance Predictive Risk onboarding if licensed (notify-only) | First-tuning-pass anomaly policies; Predictive Risk baseline started |
| W7 | Document ban-vs-revoke decision criteria (tenant-wide vs per-user); establish quarterly attestation cadence; integrate inventory into vendor-risk register | Steady-state runbook signed off; vendor-risk-register integration live |
| W8 | First full attestation cycle dry-run; produce evidence pack template; align with audit cycle | Attestation evidence-pack template; audit-cycle integration documented |
| W9+ | Steady-state: quarterly attestation; continuous anomaly-policy triage; ban-decisions logged to change record. Auto-revoke decisions only after 8+ weeks of measured FP rate | Quarterly metric on grant-count by scope-tier; banned-app log; FP-rate trend |

The W1 inventory is the work that determines whether this policy ever produces audit-grade output. Skip W1 and the ban-decisions are unauditable; the SOC defaults to "ignore anomaly alerts because we cannot tell good from bad".

## Action

- Primary: **manual ban** (tenant-wide kill) for known-malicious apps; **per-user revoke** for confirmed-bad-grant
- Anomaly detection: **alert** on Misleading OAuth app name / Misleading publisher name / Malicious OAuth app consent
- Auto-revoke: **only after 8+ weeks of measured FP rate** on the built-in anomaly policy AND only for the highest-confidence signal classes (Malicious OAuth app consent)

## Scope

- **Users:** all users (delegated-grant scope)
- **Apps:** all OAuth apps with grants in M365 / Google Workspace / Salesforce
- **Device posture:** N/A
- **Network position:** N/A
- **Traffic:** consent events + subsequent token-use events

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → OAuth apps (or App governance if add-on enabled). Built-in anomaly policies under Cloud apps → Policies → Policy management → Threat detections | Day 1: manual review of every High-permission, low-community-use, unverified-publisher app; manually ban known-malicious; do NOT enable auto-revoke on built-in anomaly policy for at least 4 weeks while measuring FP rate; restrict user consent to verified-publisher apps at the Entra layer in tandem | API-mode; covers M365 / Workspace / Salesforce delegated grants only | **Banning is tenant-wide and immediate, propagating on next token validation (CAE-aware apps revoke faster).** No staged rollout. Practitioners ban a flagged app at 11pm, wake up to dozens of broken integrations. **Application (client-credentials) OAuth grants are NOT in this view** — service-principal abuse is the dominant cloud-persistence pattern; Policy 2a covers ~30% of OAuth-grant abuse; service-principal class needs App Governance Predictive Risk / Entra Workload Identities |
| Netskope | `[unverified]` — Netskope OAuth governance | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security OAuth policies | | | |
| Skyhigh | `[unverified]` — Skyhigh OAuth visibility | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security OAuth | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration for a tier-2 ASEAN BFSI tenant after first-tuning-pass on the built-in anomaly policies; ban workflow documented; back-catalogue inventory complete:

```yaml
oauth_governance:
  inventory:
    connected_apps:
      - M365                        # primary
      - Salesforce
      - Google Workspace            # if dual-stack tenant
    scope_tiers:
      high:
        - "Mail.*"
        - "Files.ReadWrite.All"
        - "Sites.FullControl.All"
        - "User.ReadWrite.All"
        - "Directory.*"
      medium:
        - "Files.Read.All"
        - "Mail.Read"
        - "Calendars.ReadWrite"
      low:
        - "User.Read"
        - "openid"
        - "profile"
    refresh_cadence: weekly         # CSV pull into vendor-risk register

  anomaly_policies:
    misleading_oauth_app_name:
      enabled: true
      action: Alert
      severity: High
      email_recipients: [soc-l2@example.com, identity-admin@example.com]
      auto_revoke: false            # NEVER true on first turn-on
    misleading_publisher_name:
      enabled: true
      action: Alert
      severity: High
      auto_revoke: false
    malicious_oauth_app_consent:
      enabled: true
      action: Alert
      severity: High
      auto_revoke: false            # candidate for true after W12+ FP-tuning

  ban_workflow:
    decision_authority: "SOC L2 + Identity Admin (dual-control)"
    pre_ban_check:
      - "Query CloudAppEvents for user-count + recent token-use volume"
      - "Cross-reference business-owner register (vendor-risk-register integration)"
      - "Confirm not on documented business-approved ISV list"
    change_window: "Documented change record; off-hours preferred for High-impact apps"
    communication:
      pre_ban_notice_hours: 24      # to integration owners where business-owned
      exception: "Confirmed-malicious → immediate ban, post-hoc notice"
    post_ban_monitoring_hours: 48
    rollback_path: "Approve + reinstate via OAuth-apps view; user re-consent required"

  per_user_revoke:
    decision_authority: "SOC L2 (single-control acceptable)"
    trigger: "Confirmed-bad-grant during IR investigation"
    cascade: "Document in IR ticket; correlate with Entra AuditLogs for mail-forwarding rule additions in same session"

  attestation:
    cadence: quarterly
    output: "OAuth-grant inventory CSV with business-owner attestation per High-permission app"
    audit_destination:
      - vendor-risk-register
      - third-party-risk-committee
```

The `auto_revoke: false` across all three built-in anomaly policies is the post-W4 production baseline. Auto-revoke is **never** enabled on the first turn-on; the FP profile of the Misleading-name signals in particular is high enough in the first 4 weeks that auto-revoke would generate a credibility incident.

## Variants

### Industry-specific

- **BFSI:** highest scrutiny on `Mail.*` and `Files.*` grants because of CHD / customer-data / fraud-investigation-data sensitivity; tightest integration with third-party risk register and supervisor-notification clock; back-catalogue cleanup typically takes 8-12 weeks because the historical grant volume is high; ban-decision authority typically requires dual-control with Identity Admin + SOC L2
- **Healthcare:** PHI-bearing scopes (`Mail.*`, `Calendars.ReadWrite` where appointment data = PHI under HIPAA) get the same scrutiny as `Files.*`; ban-runbook integrates with HIPAA Security Rule incident-response procedure; patient-portal-OAuth (where the EHR vendor consents to M365 for clinician scheduling) is a high-FP source
- **Tech:** highest legitimate-grant velocity — developer-tool ISVs (GitHub, Vercel, Linear, Datadog, observability stack) regularly request broad scopes; admin-consent workflow rollout is the central event; back-catalogue is large but well-documented because procurement records exist; engineering-friction management is the main rollout challenge
- **Retail:** loyalty-platform integrations, marketing-automation OAuth (Mailchimp, Klaviyo, etc.) dominate the inventory; ban-decisions are commercial decisions because revenue depends on the integration; ban-runbook requires marketing-ops sign-off
- **Public sector:** lower legitimate-grant volume; admin-consent-required is often the policy default from day one; back-catalogue cleanup is smaller but ban-decisions face procurement governance (cannot ban an app under active contract without contractual notice)

### Maturity-based

- **Immature:** no OAuth-grant inventory; built-in anomaly policies in alert-only with no SOC triage runbook; bans happen ad-hoc during IR; no business-owner register; no integration with vendor-risk register; ban-decisions are made by one person under time pressure with no rollback path. Common at 12 months post-deployment for under-resourced programmes
- **Mature:** quarterly inventory with risk-tiering; built-in anomaly policies tuned; ban-runbook documented with dual-control authority; admin-consent workflow enabled at Entra layer; back-catalogue cleanup complete; vendor-risk-register integration live; ban-decisions logged to change record
- **Advanced:** App Governance Predictive Risk live with measured FP rate; OAuth threshold policies tuned per-scope-tier; auto-revoke enabled selectively on highest-confidence signal classes; CASB OAuth signal correlated into Purview IRM as indicator; per-ISV DPA refresh on a documented cadence; SaaS-to-SaaS OAuth (Salesforce → Slack via inter-tenant consent) governed via Cross-Tenant Access Settings; Power Platform service-principal hygiene addressed via Power Platform Admin Center DLP as compensating control

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + third-party app risk](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]`
- ISO 27017 control(s): [CLD.9.x access control, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.7 (third-party processing), A.10 (use limitation) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions), `DE.CM-06` (external service-provider activity), `ID.SC-04` (suppliers + third-party providers monitored) `[VERIFY]`
- MAS TRM (where applicable) — third-party access management clauses `[VERIFY]`
- HKMA SA-2 — third-party risk management `[VERIFY]`

## False-positive risk

- Legitimate-but-lookalike publisher names — "Microsoft" vs "Microsoft Corporation" misclassified by Misleading-publisher-name anomaly
- New legitimate ISVs with low community-use count — flagged by uncommon-use anomaly during the ISV's first 90 days of tenant onboarding
- Verified-publisher status drift — vendor recently verified by Microsoft Publisher Verification but MDA cache stale
- Internal-developed line-of-business apps registered in Entra but not flagged as verified — appear as "unknown publisher" with no community-use signal
- Multi-tenant ISVs where the tenant being reviewed is the first in the region to consent — community-use baseline biased by geography

## Real-world FP experience

Typical FP-rate trajectory on the built-in anomaly policies for a tier-2 BFSI tenant new to OAuth governance:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 50-70% | Misleading-publisher-name firing on legitimate ISVs with publisher names that partially match Microsoft/Google/Salesforce naming patterns; Uncommon-use firing on every new ISV onboarding |
| W4 | 25-40% | After whitelisting verified ISVs surfaced as FPs in W1-W3; after exclusion of business-approved app IDs |
| W8 | 12-20% | After cross-reference with vendor-risk register; after publisher-name allowlist matures |
| W12 | 5-12% | Documented business-owner register integrated; first-quarter attestation cycle catches drift before it becomes FP volume |
| Steady-state | 3-8% | Quarterly attestation + continuous business-owner-register sync; Predictive Risk (if licensed) provides higher-confidence signal class |

Named FP scenarios encountered repeatedly across deployments:

| Scenario | Mitigation |
|---|---|
| Legitimate ISV publisher name resembles Microsoft / Google / Salesforce (e.g. "Microsoft Power Tools Ltd" — actual third party) | Whitelist by app ID after publisher-verification check; document in business-approved-ISV register |
| Newly-onboarded enterprise ISV with low community-use count in the first 30-90 days post-publisher-verification | Time-bounded whitelist for first 90 days post-onboarding; quarterly review |
| Internal LOB app registered in Entra without going through Microsoft Publisher Verification | Permanent allowlist by app ID; document as internal app in inventory |
| Verified-publisher status drift — vendor was verified by Microsoft but MDA's cached community-use signal is stale | Manual override + business-owner attestation; ticket with Microsoft to refresh cache |
| Multi-region ISV where this tenant is the first in the region to consent | Time-bounded whitelist; cross-reference with regional peer firms via ISAC if active |
| Power Platform connector consents (Premium connectors requiring `*.ReadWrite`) | Power Platform Admin Center DLP as the primary governance surface; MDA whitelist for the Power Platform service-principal pattern |
| Salesforce → Slack inter-tenant OAuth (or similar SaaS-to-SaaS) | Document as expected pattern in inventory; whitelist if business-approved |
| Test-tenant or sandbox app IDs left from POC work | Cleanup as part of W1 back-catalogue; ban the dormant test app IDs |

## Operational cost

- **Exception-handling load:** medium during W3-W12 back-catalogue cleanup (10-30 exceptions per week typical for a 10k-user tenant); low-medium steady-state (3-8 per week)
- **Triage load:** medium during W5-W12 anomaly-policy ramp (5-15 anomaly alerts per week per 10k users); low-medium steady-state once allowlist + business-owner register are mature
- **End-user friction:** ban = high impact (every consenting user loses access to the banned app; integrations break; help-desk surge during the first 48h); revoke = low impact (one user; user re-consents if legitimate)
- **ISV-onboarding-desk load (where admin-consent workflow is live):** 5-15 requests per month per 10k users; 5-day SLA typical; requires sustainable triage capacity in the Identity team

Typical staffing: 0.2 FTE Identity admin during the 8-week ramp + back-catalogue cleanup; 0.1 FTE steady-state. SOC L2 contributes 0.1 FTE to anomaly-alert triage. Third-party risk analyst contributes ~0.05 FTE for the quarterly attestation cycle.

## Privacy / data-protection considerations

- Consent events catalogue user identity + grant scope + timestamp — workforce-monitoring data under PDPA / GDPR Art. 88 / equivalent
- The OAuth-apps view exposes which user consented to which third-party app — admin-access to that view itself needs governance (it can surface a user's tooling preferences, productivity-app inventory, sometimes religious/political app patterns)
- DPIA scope: the OAuth-grant inventory + back-catalogue cleanup is a documented processing activity; the cross-reference to vendor-risk-register involves data-flow mapping
- Storm-0558 / Storm-2372-class supply-chain compromise: an approved-by-business ISV could turn malicious; documented kill-switch posture required as a control-design decision, not as a runbook footnote
- Per-user revocation cascades into the user's downstream session state — does not revoke already-issued refresh tokens until the next token validation; document the revocation lag

## Integration with broader programmes

- **Vendor / third-party risk register:** OAuth-grant inventory feeds the third-party processor inventory; each High-permission app with consented `Mail.*` or `Files.*` scope is a sub-processor relationship in DPIA terms. Quarterly reconciliation cadence
- **Incident response runbook:** consent-phishing-confirmed signal (from SOC IR ticket) is the named trigger for the ban-workflow; integrates with the IR playbook for credential-compromise (T1528 → T1550.001 token-replay → T1114 mailbox-exfil chain). The ban-runbook is a documented IR sub-procedure
- **Entra admin-consent workflow:** this policy is the compensating cleanup layer beneath admin-consent. Coupling document required — admin-consent governs new grants forward; MDA Policy 2a governs the back-catalogue and provides the inventory + revocation workflow
- **App Governance (where licensed):** Policy 2a hands off the behavioural-anomaly detection (T1098.001 / T1136.003 / T1550.001 post-consent) to Policy 2b. Document the boundary; ensure no double-attestation overhead
- **Audit cycle:** quarterly OAuth-grant inventory + named-incident-investigation evidence feed the annual control-effectiveness review (under SOX 404 / NIS2 / DORA Art. 5-15 third-party-risk equivalent operational-risk regimes) `[VERIFY against the specific regime in scope]`
- **Board reporting:** quarterly metric — count of High-permission grants by business-owner-attested vs not; count of bans this quarter; named consent-phishing incidents handled (sanitised). The metric to avoid: raw grant-count, which is meaningless without scope-tier classification
- **Supervisory notification clock:** where a confirmed consent-phishing incident affects customer data, the prudential supervisor's incident-notification clock applies. Material is illustrative only — clock is regime-specific and not legal/regulatory advice [VERIFY]
- **Purview IRM (Day 90, where deployed):** OAuth-anomaly signal can boost Insider Risk scoring when correlated with HR-departure or unusual-activity signals from the same user

## Anti-patterns specific to this policy

1. **"Ban at 11pm without 24-hour notice to integration owners"** — wakes up to dozens of broken integrations; help-desk surge; trust crisis with business owners; ban-policy gets rolled back under pressure. Confirmed-malicious is the only ban-without-notice exception
2. **"Enable auto-revoke on the built-in anomaly policies on day 1"** — Misleading-publisher-name and Uncommon-use generate 50-70% FP rate in W1; auto-revoke at that FP rate breaks legitimate ISVs at scale within the first week
3. **"Skip the W1 back-catalogue inventory"** — ban-decisions go in without business-owner context; every ban produces a help-desk ticket from someone in the business who depended on the integration; ban-workflow loses credibility
4. **"Treat Policy 2a as the OAuth-governance answer"** — covers delegated grants only (~30% of OAuth-abuse surface); service-principal / client-credentials grants (the dominant cloud-persistence pattern, T1098.001) are NOT in this view. Document the gap and the compensating control (App Governance + Entra Workload Identities)
5. **"Ban without checking user-count and recent token-use"** — the OAuth-apps view shows users + community-use; the ban-runbook MUST require pre-ban check of CloudAppEvents for actual usage. Banning a 4-user shadow integration is low-risk; banning a 4,000-user business-approved ISV is a P1 incident
6. **"No documented business-owner register"** — every ban becomes an investigation; "is this real or shadow?" cannot be answered; ban-decisions take days instead of minutes; in a real consent-phishing incident, the attacker's window stays open while the SOC tries to figure out whether the app is legitimate
7. **"Personal-account OAuth is in scope"** — Entra never sees a user signing into a third-party app with personal Gmail / personal MSA. Treating Policy 2a as covering personal-account exfil produces a false sense of coverage. The compensating control is endpoint-DLP / DSPM for AI / network-egress, not this policy
8. **"Per-user revoke without correlating mail-forwarding rule additions"** — T1528 → T1550.001 → T1114 chain. The consent grant is the entry; the actor typically adds a mail-forwarding rule + delegate within the same session to maintain access after the token expires. Revoke without correlating the secondary persistence = the attacker is still in
9. **"Ignore the Power Platform sub-class"** — Power Platform service-principal consents are under-surfaced in this view AND in App Governance. Heavy Power Platform tenants must additionally configure Power Platform Admin Center DLP; treating Policy 2a as covering Power Platform is the documented gap
10. **"Run the ban-runbook as a single-person decision"** — dual-control (SOC L2 + Identity Admin) for tenant-wide ban; single-control acceptable for per-user revoke during IR. Single-control on tenant-wide ban is how a disgruntled or compromised SOC analyst kills production integrations

## Coverage gaps

- [OAuth blind spot](../../08-failure-modes/oauth-blind-spot.md) — application (client-credentials / app-only) grants are the dominant cloud-persistence pattern (~70% of OAuth abuse) and NOT in this policy's scope
- Personal-account OAuth (user signs into a third-party app with personal Gmail / personal MSA) — Entra never sees it
- First-time abuse of a never-before-anomalous app — Predictive Risk model has nothing to compare against; pair with Policy 2b once App Governance is licensed
- Power Platform service-principal consents — under-surfaced in App Governance; compensating control = Power Platform Admin Center DLP
- SaaS-to-SaaS OAuth (e.g. Salesforce app consenting to Slack via inter-tenant flow) — Cross-Tenant Access Settings is the governance surface, not this policy
- Refresh-token revocation lag — per-user revoke does not invalidate already-issued refresh tokens until next validation; CAE-aware apps revoke faster but not all apps are CAE-aware
- Token-replay from residential-proxy infrastructure (T1550.001) — even after revoke, attacker on stolen access-token within validity window can continue
- Storm-0558-class Microsoft service-side compromise — token-forgery at the IdP layer leaves no in-tenant detection in this view

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
