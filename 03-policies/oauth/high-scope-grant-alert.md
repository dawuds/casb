# High-scope OAuth grant alert

> Status: v0.0 — MDA column from playbook v1 (Policy 2b — App Governance Predictive Risk + OAuth threshold, Day 30); other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [OAuth-app discovery + Anomalous-activity detection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector + App Governance add-on (MDA-specific) or vendor equivalent.
> MDA playbook reference: [Policy 2b](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Day 30 OAuth behavioural detection).

## Purpose

Alert on OAuth grants with high-permission scope (Mail.ReadWrite.All, Files.ReadWrite.All, Sites.FullControl.All, User.ReadWrite.All, Application.ReadWrite.All) regardless of publisher reputation. Covers the behavioural post-consent class — a legitimately-consented app suddenly expands scope, adds a new client credential, or starts behaving anomalously. Counters MITRE ATT&CK `T1098.001 Additional Cloud Credentials` (Detect), `T1550.001 Application Access Token` (Detect), `T1136.003 Cloud Account creation via service-principal` (Detect). Pairs with — does not replace — Entra admin-consent workflow + verified-publisher restriction, which is the primary preventive control for T1528.

## What organisations use this for

For most regulated tenants, the OAuth high-scope grant alert is the **first behavioural signal layered on top of the preventive Entra controls** (admin-consent workflow, verified-publisher restriction, app-consent policies). The preventive controls stop the easy cases; this policy catches what already exists and what changes over time — the long tail of consents granted before the policy went in, the service principals onboarded under legacy delegated admin, and the previously-clean app whose credentials get added to mid-life by an attacker.

Deployments are almost always **post-incident** or **pre-audit**, rarely greenfield. The catalysts after the 2024-2025 cycle have been the named clusters — Storm-0558, MIDNIGHT BLIZZARD service-principal abuse, Storm-2372 mass consent phishing, Storm-1283 — and the supervisory expectations these incidents bequeathed (workload-identity inventory, lifecycle attestation, anomaly detection on application-permission grants). The hardest single decision is not threshold tuning — it is what to do about the 200-800 pre-existing high-scope service principals the discovery scan surfaces on Day 1.

### Use case 1 — Tier-1 ASEAN universal bank, post-Storm-2372 service-principal sprawl response

- **Org type:** tier-1 ASEAN universal bank, ~30k employees, M365 E5 with App Governance add-on, BNM RMiT supervised, in-scope for PCI DSS v4.0
- **Trigger:** internal threat-hunt after Microsoft's Storm-2372 device-code phishing reporting (Feb 2025); discovered ~140 OAuth apps in the tenant with Mail.ReadWrite.All consented before admin-consent workflow was enforced (2022), several with unverified publishers and one with a typosquat publisher name. No confirmed compromise, but the inventory itself was the finding
- **Scope:** tenant-wide discovery first; alerting scoped to all new high-scope grants + credential additions on existing service principals; phased per-app risk-tier triage of the existing 140
- **Outcome:** ~140 apps triaged over 6 months — 12 banned (typosquat / abandoned / replaceable), ~30 retained-and-attested with owner sign-off, ~98 reduced-scope (replaced Mail.ReadWrite.All with Mail.Read where downstream consumer didn't need write). Steady-state ~3-5 alerts/week on credential additions; ~80% are legitimate ISV key-rotation events caught by the alert because no inventory exists of which apps rotate when

### Use case 2 — Power Platform service-principal governance, digital-native challenger bank

- **Org type:** digital-native challenger bank, ~800 employees, heavy Power Platform usage (citizen-developer-driven Power Automate flows + Power Apps for branch operations), M365 E5
- **Trigger:** internal audit finding — citizen developers had created ~60 Power Automate flows using personal service-principal credentials with Files.ReadWrite.All and Sites.FullControl.All. No central inventory; no one could answer the question "how many flows touch customer data". Catalyst was the bank's first SOC 2 Type II prep
- **Scope:** all Power Platform environments (Default + Production + Test); paired with Power Platform Admin Center DLP policy (the MDA-only approach undercovers Power Platform per the playbook caveat); alerting on any new service-principal consent in the production environment
- **Outcome:** 60 → 18 flows after consolidation under platform-team-owned service principals; Default environment locked down (no Microsoft connectors with high scope); auditor accepted the App Governance + PPAC DLP combination as the workload-identity governance evidence. Steady-state ~2-3 alerts/month, most from the test environment

### Use case 3 — Pre-audit application-permission inventory, regional insurer

- **Org type:** regional life insurer, ~4k employees, M365 E5, group-level BNM RMiT-equivalent supervised, ISO 27001 certified
- **Trigger:** ISO 27001 surveillance audit + IT-general-controls audit pre-work — auditor asked for "the inventory of third-party applications with access to customer data, with risk-tier and last-attestation date". The bank had no such inventory. Six weeks until the audit fieldwork
- **Scope:** tenant-wide; alert-only mode; output was the inventory itself rather than active enforcement. Owner-attribution exercise — every app needed a named human owner before audit
- **Outcome:** 312 OAuth apps surfaced; 47 high-scope; 8 had no identifiable internal owner (legacy / orphaned); auditor finding downgraded from "no inventory" (major) to "inventory established, attestation cadence pending" (minor — corrective action plan acceptable). Programme then re-scoped the alert as the ongoing detective control feeding annual attestation

### Use case 4 — M&A integration, holding-company cloud-app estate reconciliation

- **Org type:** mid-size BFSI holding group acquiring a smaller wealth-management subsidiary; consolidating two M365 tenants under a tenant-to-tenant migration (~6k employees combined post-merger)
- **Trigger:** pre-cutover diligence — acquiring tenant required an inventory of the target's OAuth grants before migration sign-off; target had no App Governance deployed, no inventory, mixed delegated and application-permission consents accumulated over 7 years
- **Scope:** target tenant first (read-only inventory via Graph API + temporary App Governance trial during the diligence window); acquirer ran alert-only mode on the target for 30 days pre-cutover
- **Outcome:** 89 OAuth apps in the target; 21 high-scope; 4 publisher-unverified with credential additions in the last 90 days flagged for separate investigation by the acquirer's IR team (no compromise confirmed, but two were retired from the migration scope entirely). Migration plan added a 90-day post-cutover monitoring window with tightened alerting before full inheritance into the acquirer's steady-state policy

## Implementation pattern

Typical 10-week rollout for a tier-2 BFSI tenant enabling App Governance for the first time, assuming Entra admin-consent workflow + verified-publisher restriction are already in place:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | License App Governance add-on; enable in tenant; document the SKU / cost in the security budget paper. Decide alert routing (SOC vs Identity team vs IRM analyst) | Add-on live; alert destination signed off |
| W2 | Pull baseline inventory of all existing OAuth apps via Graph API (`/servicePrincipals`, `/oauth2PermissionGrants`, `/appRoleAssignments`); flag high-scope set. Identify owner per app (HR / business / IT) — most painful step | Baseline inventory CSV; owner-attribution gap log |
| W3-W4 | Enable Predictive Risk policy in **notify-only mode**; enable OAuth threshold policy with starting values (new high-scope grant alert ON; credential-addition alert ON; geographic-anomaly alert ON). No auto-revoke | Policy live in alert mode; W1 alert volume baseline |
| W5 | Triage W3-W4 alert backlog; classify TP / FP / out-of-scope. Build per-app suppression list for legitimate burst-usage apps (DR test SPs, content-migration SPs) | First-tuning-pass FP-cause catalogue |
| W6-W7 | Phased existing-app remediation — for each high-scope app: contact owner, decide retain / ban / reduce-scope / replace. Document attestation. Banning is tenant-wide and **immediate** — query `CloudAppEvents` for current user-count first | Per-app disposition log; reduced high-scope inventory |
| W8 | Refine Predictive Risk + threshold values; add Power Platform Admin Center DLP companion policy (App Governance under-surfaces Power Platform SPs) | FP rate <15% target |
| W9 | Document the SOC triage runbook for OAuth alerts; document escalation chain (Identity admin → SOC L2 → IR if confirmed-bad); define quarterly attestation cadence | Runbook signed off |
| W10 | Selectively enable auto-revoke for the highest-confidence signal classes ONLY (e.g. malicious-consent verdict + Microsoft-confirmed pattern). Most alerts stay notify-only | Production-ready; auto-revoke scope documented |
| W11+ | Steady-state — quarterly attestation cycle; monthly FP-tuning; annual full inventory refresh | Quarterly metric on TP rate + attestation completeness |

The W2 inventory and owner-attribution is the work. Without it, Week 6 is impossible — there is no one to call when a high-scope app needs a retain / ban decision. Practitioners who skip W2 either ban indiscriminately (and break legitimate business workflows) or leave the inventory untriaged for years.

## Action

- Primary: **alert** in notify-only mode for 4 weeks minimum; flip to auto-revoke only on highest-confidence signal classes after FP tuning
- Escalation: **disable service principal** (via Entra `accountEnabled=false` on the SP object) only after corroborated signal — Microsoft-confirmed malicious-consent verdict OR confirmed credential-theft IR finding — AND human approval. Banning is tenant-wide and propagates on next token validation
- Never: **silent auto-revoke on first turn-on** — guarantees production breakage from legitimate ISV credential rotations

## Scope

- **Users:** all OAuth-consenting users + admins granting app-only permissions; application-permission grants by Global Admin / Application Admin / Cloud Application Admin in particular
- **Apps:** all OAuth-grantable SaaS — M365 (Graph), Workspace (Google OAuth), Salesforce (Connected Apps), GitHub OAuth where reachable
- **Device posture:** N/A — workload-identity layer
- **Network position:** N/A
- **Traffic:** consent + grant + service-principal-credential addition events; scope-expansion events on existing service principals

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → App governance → Predictive risk policies + OAuth threshold policies. (Re-verify 2026 portal path; App Governance may be a separate left-nav.) Plus Entra portal → Identity → Applications → Enterprise applications → Consent and permissions for the preventive layer | Enable Predictive Risk in notify-only mode for 4 weeks; tune; enable auto-revoke only on highest-confidence signal classes. OAuth threshold = count of grants per app per window + scope-expansion + geographic anomalies + credential-addition events on existing SPs. Recommended starting thresholds (illustrative — derive from W2 baseline): scope-expansion any high-scope = alert; new-credential-on-existing-SP = alert; geo-anomaly = alert if SP has no historical pattern from that region | App Governance is a paid add-on (verify SKU pricing per current `app-governance-licensing` doc); Predictive Risk shipped 2025 H2; behavioural class is post-consent. Banning via App Governance is tenant-wide and immediate; CAE-aware apps revoke faster. Discovery has a tenant-app-count ceiling — verify against current docs | Predictive Risk FP on legitimate burst usage (DR rehearsals, content-migration runs) — per-app suppression workflow required before auto-revoke. **First-time abuse of a never-before-anomalous app — model has nothing to compare against.** **Power Platform service-principal consents under-surfaced in App Governance — Power Platform Admin Center DLP required as companion.** Personal-account OAuth (user signs into a third-party app with personal MSA / Gmail) — Entra never sees it; no coverage. Service-principal-secret rotation by legitimate ISVs produces credential-addition alerts indistinguishable from T1098.001 without operator context |
| Netskope | `[unverified]` — Netskope OAuth governance with anomaly | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security OAuth threshold | | | |
| Skyhigh | `[unverified]` — Skyhigh OAuth anomaly | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security OAuth | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant after the 10-week ramp is complete:

```yaml
policy:
  name: "OAuth high-scope grant + behavioural alert — tenant-wide"
  type: AppGovernancePolicy
  predictive_risk:
    enabled: true
    sensitivity: Medium              # Low produces too few signals; High overwhelms
    action: Alert                    # notify-only; auto-revoke OFF at this baseline
    confidence_floor_for_auto_revoke: 95  # only if/when auto-revoke enabled
  oauth_threshold:
    new_grant_alerts:
      high_scope_set:
        - "Mail.ReadWrite.All"
        - "Mail.Send"
        - "Files.ReadWrite.All"
        - "Sites.FullControl.All"
        - "User.ReadWrite.All"
        - "Application.ReadWrite.All"
        - "Directory.ReadWrite.All"
        - "RoleManagement.ReadWrite.Directory"
      action: Alert
      include_admin_consent: true    # most regulated tenants restrict user consent
                                     # so admin grants are the primary alert path
    credential_addition_on_existing_sp:
      action: Alert
      exclude_known_isv_rotation:
        - "<ISV-app-id-1>"           # e.g. M365 backup vendor
        - "<ISV-app-id-2>"           # e.g. SIEM ingestion app
                                     # NEVER blanket-exclude a publisher;
                                     # exclude per app-id, with owner + expiry
    scope_expansion:
      action: Alert                  # high-fidelity signal class
    geographic_anomaly:
      action: Alert
      sp_history_window_days: 90
  user_filter:
    exclude_consenting_admin_upns: []  # do NOT exclude admins — they are the
                                       # most-likely-targeted consent path
  governance:
    auto_revoke: false               # NEVER true on first turn-on
    auto_disable_sp: false           # tenant-wide + immediate; requires sign-off
    notify_app_owner: true           # owner from W2 inventory
    notify_user: false               # do not tip off compromised account
  alerts:
    severity: High
    routing:
      - destination: "soc-l1@example.com"
      - destination: "identity-admin@example.com"
    sentinel_forward: true
    daily_alert_limit: 200
  attestation:
    cadence_days: 90
    owner_required: true             # an unowned high-scope app cannot pass
    evidence_retention_days: 730     # two-year audit window
  power_platform_companion:
    ppac_dlp_required: true          # MDA-only undercovers Power Platform SPs
```

The `sensitivity: Medium` + `confidence_floor_for_auto_revoke: 95` combination is the typical post-tuning baseline for regulated tenants. Lower sensitivity = misses behavioural drift; higher = unmanageable noise on legitimate ISV activity. The `exclude_known_isv_rotation` list must be **per-app-id with named owner and expiry**, never publisher-wide — Microsoft's verified-publisher list is large, and a compromised verified-publisher app is the MIDNIGHT BLIZZARD scenario.

## Variants

### Industry-specific

- **BFSI:** lowest scope-expansion tolerance; application-permission grants by admins explicitly in scope; Power Platform service-principal governance often the dominant operational fight; integration with vendor-risk programme mandatory (each high-scope ISV is a sub-processor under BNM RMiT / MAS TRM third-party arrangements `[VERIFY]`); auto-revoke disabled in most tenants because production breakage risk exceeds tolerated outage
- **Healthcare:** PHI scopes drive sensitivity — any app with Mail.* or Files.* over patient-record stores is in scope; HIPAA Security Rule mapping for the workload-identity catalogue; integration with Business Associate Agreement (BAA) attestation programme
- **Tech / SaaS:** highest tolerance for ISV-credential-rotation FPs (engineering teams ship internal apps weekly); risk profile inverted — most signals from internal-app developer behaviour, not external ISV abuse; integration with internal SDLC for app-registration governance
- **Retail:** loyalty-platform integrations + payment-gateway SPs dominate the inventory; FP fight is around seasonal-campaign SPs spun up for marketing partners; high churn on the SP estate; quarterly attestation often misses fast-moving inventory
- **Legal / professional services:** lower SP volume but higher per-app sensitivity (client-confidential matter scopes); matter-management workflow integrations need careful scoping; eDiscovery vendor apps with Mail.ReadWrite.All are a common high-scope retention

### Maturity-based

- **Immature:** Predictive Risk enabled but no W2 baseline inventory; alerts firing into a SOC queue that doesn't know what to do with them; no owner-attribution; alerts ignored by month 3; no companion Power Platform DLP; auto-revoke either disabled (no remediation) or enabled tenant-wide on first turn-on (broken production). Common at 6 months post-deployment for under-resourced programmes
- **Mature:** baseline inventory completed; per-app owner attribution documented; alert routing into Identity team with SOC ticketing for high-severity; quarterly attestation cycle running; Power Platform DLP companion policy in place; FP rate <15%; auto-revoke scoped to highest-confidence signal classes only; documented SOC triage runbook with named escalation chain
- **Advanced:** App Governance correlated with Entra ID Protection workload-identity risk signals; integration with SDLC for internal-app-registration governance (app-registration approvals routed via ticketing); per-SP risk-tier scoring driving differentiated attestation cadence (Tier-1 monthly, Tier-2 quarterly, Tier-3 annual); workload-identity inventory feeds the vendor-risk programme as a primary data source; per-ISV rotation calendars maintained to suppress legitimate-credential-rotation FPs proactively

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + third-party risk + technology risk management](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition — third-party arrangements + access management]`
- MAS TRM: workload-identity governance under access management + third-party risk `[VERIFY]`
- ISO 27017 control(s): [CLD.9.x access control, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27001:2022: A.5.15 (access control), A.5.16 (identity management), A.5.17 (authentication information), A.5.19 (information security in supplier relationships), A.8.2 (privileged access rights) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions managed), `DE.CM-06` (external service provider activity), `DE.AE-02` (event data analysed), `ID.AM-04` (external information systems catalogued) `[VERIFY]`
- PCI DSS v4.0: Req. 7 (need-to-know access; applies to service accounts including workload identities), Req. 8.2 (identification of users and accounts), Req. 12.8 (service-provider management) where the app touches CHD scope

## False-positive risk

- Legitimate burst usage — DR rehearsals, content-migration runs, planned-data-warehouse extracts
- New ISV integrations during onboarding — high-permission grant is part of legitimate setup, but indistinguishable from malicious consent at the moment of grant
- ISV credential rotations — vendor rotates client-secret on schedule; alert fires as credential-addition with no operator context
- Geographic anomaly on service principal that legitimately runs from multiple regions (e.g. ISV with global ops)
- Scope-expansion where ISV legitimately adds new permission for a new product feature (e.g. Microsoft adds new scopes to first-party services regularly)
- Power Platform default-environment service-principal noise (citizen-developer activity)

## Real-world FP experience

Typical FP-rate trajectory in a tier-2 BFSI tenant new to App Governance:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 60-80% | ISV credential rotations + legitimate burst usage during DR-test season + scope-expansion on first-party Microsoft apps + Power Platform default-environment noise |
| W4 | 30-45% | After first-pass per-app suppression list (top-10 noisy ISVs documented + DR-test windows registered + Power Platform DLP companion engaged) |
| W8 | 15-25% | After per-ISV rotation-calendar built + after owner-attribution allows targeted suppression with expiry + after scope-expansion FPs filtered for first-party Microsoft changes |
| W12 | 8-15% | Mature per-app suppression with named owners and expiry; quarterly attestation catches inventory drift; Power Platform DLP companion stable |
| Steady-state | 5-10% | Quarterly attestation + ISV-rotation-calendar maintenance; signal-to-noise sufficient for SOC to keep paying attention |

Named FP scenarios encountered repeatedly across deployments:

| Scenario | Mitigation |
|---|---|
| ISV rotates client-secret on quarterly schedule; alert fires as credential-addition | Per-app-id rotation-calendar suppression with documented owner + 12-month expiry; reaffirm at attestation |
| DR-test season — bulk service-principal activity from DR region triggers geo-anomaly + burst-usage | Pre-DR-test exception window registered by IT-DR team; named owner; time-bounded |
| Microsoft adds new scope to first-party app (e.g. Teams, Power Platform connector) — scope-expansion alert | Microsoft-first-party allowlist with monthly review of new scopes against published Microsoft release notes |
| Content-migration SP doing bulk Files.ReadWrite.All across SharePoint sites during a migration project | Migration-window suppression; documented project owner; time-bounded; reverts to active monitoring at project end |
| Power Platform default-environment citizen-developer flows generating SP-consent noise | PPAC DLP locks default environment; route citizen-developer activity to a controlled Production environment; alert only in Production |
| ISV with global ops legitimately operating from multiple Azure regions — geo-anomaly per region | Multi-region SP-allowlist per app-id; reaffirm geo-pattern at attestation |
| New verified-publisher ISV onboarded by procurement; user-consent path triggers grant alert | Onboarding workflow — procurement notifies Identity team in advance; pre-authorised app-id added to expected-grants list |
| Internal-developer creating a new app registration during build/test cycles | SDLC integration — app-registration approvals routed through ticketing; pre-approved app-ids suppressed in non-production |

## Operational cost

- **Exception-handling load:** high during W4-W12 ramp (15-30 exceptions per week typical, driven by ISV rotation discovery); medium steady-state (5-10 per week)
- **Triage load:** medium — every alert needs cross-reference with business operations (is this DR? migration? legitimate ISV?); typical 10-20 alerts per week per 5k users at FP rate ~15%; high during DR-test season + audit-evidence-pull windows
- **End-user friction:** minimal on alert; high on auto-revoke or SP-disable (broken automation, unclear remediation path); admin friction higher than end-user friction (admin-consent workflow + app-registration ticketing add latency to new-app onboarding)

Typical staffing: 0.3 FTE Identity admin during the 10-week ramp (heavy on the W2 inventory + W6-W7 remediation); 0.1 FTE steady-state. SOC L1 triage time ~0.1-0.2 FTE depending on tenant SP estate size. Annual attestation cycle adds ~0.3 FTE for 4-6 weeks per year.

## Privacy / data-protection considerations

- Workload-identity activity logged = no direct workforce-monitoring concern; but high-permission scopes give the app access to regulated PII / PHI / CHD, which is the substantive privacy concern
- Document workload-identity governance under PDPA / GDPR processor / sub-processor treatment — each high-scope ISV is a data processor under Art. 28 GDPR / equivalent
- The matching-content / event payload in App Governance can carry user identifiers + app-grant metadata — workforce data under PDPA / GDPR Art. 88 `[VERIFY]`; document access-control on Defender / Sentinel ingestion path
- DPIA scope: high-scope workload-identity inventory + retention of grant-event records + correlation with HR data (for app-owner attribution) typically a DPIA trigger
- Cross-border: App Governance event records live in the tenant's primary-data region (verify against current Microsoft Cloud regions doc); Japan East addition in 2025 H2 materially helps MY / SG / HK tenants

## Integration with broader programmes

- **Annual / external audit:** workload-identity inventory + attestation log + named-incident-investigation evidence feed ISO 27001 surveillance + SOC 2 + PCI DSS Req. 7 / Req. 12.8 evidence pulls. Quarterly inventory snapshot; annual attestation as primary audit artefact
- **BNM RMiT supervisory-engagement cycle:** workload-identity governance is a recurring supervisory-question theme post-Storm-0558 / Storm-2372; the inventory + attestation cadence + the named-incident response provide the response posture `[VERIFY]`
- **Vendor-risk programme:** every high-scope external ISV is a sub-processor (or processor) — workload-identity inventory becomes a primary input to the vendor catalogue, SOC 2 / ISO 27001 attestation pull, DPA review
- **Incident response runbook:** App Governance credential-addition alert + scope-expansion alert feed the suspected-account-compromise playbook (correlate with Entra sign-in risk, Defender XDR identity signal); Microsoft-confirmed-malicious-consent verdict feeds the OAuth-abuse-response runbook directly
- **Board / executive reporting:** quarterly metrics — total high-scope workload identities, % attested in cycle, named-incident count, attestation-overdue count; trend matters more than absolute number
- **DPIA refresh:** annual DPIA cycle reviews the workload-identity inventory scope, event-record retention, cross-correlation with HR data for owner attribution
- **SDLC / internal-app registration governance:** internal-developer app registrations route through a ticketing approval flow; pre-approved app-ids added to expected-grant list; reduces FP volume from internal-developer activity

## Anti-patterns specific to this policy

1. **"Enable auto-revoke on Day 1"** — guarantees production breakage from legitimate ISV credential rotations; trust in the policy collapses within the first month; programme rolls back to alert-only
2. **"Skip the W2 baseline inventory"** — every alert lands in a void; no owner to call; no way to classify TP / FP / out-of-scope. Programme dies of operational gravity
3. **"Ban the typosquat without checking user-count first"** — banning is tenant-wide and immediate (CAE-aware apps revoke faster); legitimate workflows break overnight; trust crisis with business teams. Query `CloudAppEvents` for usage before any ban
4. **"Treat App Governance as covering Power Platform"** — Power Platform SP consents are under-surfaced; the dominant exfil path in Power-Platform-heavy tenants is through citizen-developer Power Automate flows that App Governance only partly sees. PPAC DLP is mandatory companion
5. **"Exclude all admins from the grant-alert scope to reduce noise"** — admins are the **most-likely-targeted** consent path (MIDNIGHT BLIZZARD, Storm-2372 admin-consent phishing); excluding them inverts the threat model. Tolerate the noise; do not blanket-exclude
6. **"Blanket-exclude verified publishers"** — verified-publisher status is a useful signal, not an authoritative trust statement. MIDNIGHT BLIZZARD abused legitimate verified-publisher apps via credential-addition. Exclude per-app-id with named owner + expiry, never publisher-wide
7. **"Notify the user when alerted"** — for compromised-account scenarios, tips off the actor; investigation gets harder. Notify the app owner (W2 inventory) and the Identity team, not the consenting user
8. **"Rely on Predictive Risk alone"** — Predictive Risk has nothing to compare against for first-time-anomalous apps; threshold policies (credential-addition, scope-expansion, geo-anomaly) are the higher-fidelity signal classes for the dominant threat patterns
9. **"Skip owner-attribution because it's too hard"** — without an owner, no high-scope app can pass attestation; the unowned-app population grows; auditor finding follows. The owner-attribution work is what makes the rest of the programme possible
10. **"One-time inventory, no attestation cycle"** — the SP estate drifts; new high-scope apps accumulate without operator awareness; six months later the inventory is fiction. Quarterly attestation is the minimum survivable cadence

## Coverage gaps

- First-time abuse of a previously-clean app — Predictive Risk model has no baseline to anomaly-detect against
- App Governance covers some application-permission anomalies for M365 Graph; coverage on non-Graph scopes varies — verify per-scope against current docs
- **Power Platform service-principals** — under-surfaced; Power Platform Admin Center DLP required as companion (see Use case 2)
- **Personal-account OAuth** — user signs into a third-party app with personal MSA / Gmail / consumer-tier account; Entra and MDA never see the grant; not in scope of this policy
- **Workspace / Salesforce coverage** — App Governance is M365-Graph-centric; Workspace OAuth and Salesforce Connected Apps require parallel native-platform controls
- **OAuth blind spot** — see [OAuth blind spot](../../08-failure-modes/oauth-blind-spot.md); per the playbook framing, the full ~70% of OAuth abuse class is not solved by Predictive Risk alone — the preventive Entra layer (admin-consent workflow + verified-publisher restriction + app-consent policies) does most of the work
- **Service-principal sign-in risk for workload identities** — Entra Workload Identities P2 covers a different signal class; App Governance event records do not replace Workload Identity Premium telemetry
- **Token replay across regions for already-authorised SPs** — geo-anomaly on the SP itself helps, but T1550.001-on-SP-token is hard to detect without correlating SP sign-in telemetry

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
