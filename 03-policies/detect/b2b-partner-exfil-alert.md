# B2B / partner-tenant exfiltration alert

> Status: v0.0 — MDA column from playbook v1 (Policy 6b — Day 30 addition); other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [Anomalous-activity detection + B2B guest-user attribution](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 6b](../../04-vendors/microsoft-defender-for-cloud-apps.md) (B2B / partner-tenant exfiltration alert) — internal playbook numbering, not a Microsoft Learn nomenclature.

## Purpose

Detect external-domain guest-user file operations (download / share) against the tenant at volumes exceeding a threshold. Counters MITRE ATT&CK `T1199 Trusted Relationship` (Detect) and `T1213.002 SharePoint` (Detect). Catches Storm-0539 / Storm-2372-style consent-phishing-into-B2B attack chains, where the actor lands inside a partner tenant via consent abuse, then walks the trust relationship into the host tenant's SharePoint surface.

> **Practitioner inference:** Microsoft's published Storm-0539 reporting is gift-card-fraud-centric, and Storm-2372 reporting focuses on device-code phishing. The specific linkage to consent-phishing-into-B2B trust-walk against SharePoint is a practitioner pattern observed across BFSI deployments; it is consistent with the documented TTPs but is not itself a Microsoft-named attack chain. See Microsoft Security Blog Storm-0539 (Dec 2023, May 2024) and Microsoft Threat Intelligence Storm-2372 reporting (Feb 2025) for the verified TTPs `[VERIFY current URLs at learn.microsoft.com / microsoft.com/security/blog]`. Threat-actor names follow Microsoft canonical taxonomy [Microsoft Learn: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming].

> **June 2025 built-in anomaly-policy wave (boilerplate for detect/* files):** Several built-in anomaly policies were transitioned to a dynamic-threat-detection model in June 2025 and renamed — Activity from suspicious IP addresses; Suspicious inbox forwarding; Suspicious inbox manipulation rules; Activity from anonymous IP addresses; Ransomware activity; Suspicious file access activity by user; Unusual ISP for an OAuth App; Suspicious email deletion activity. This B2B-partner-exfil policy is a **custom Activity Policy** and is not directly affected, but downstream correlation (e.g. cross-referencing the dynamic-detection signals against guest-user behaviour) should account for the renames. Source: [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy] (Important notice).

## What organisations use this for

The policy exists because the partner-tenant surface is the surface most security teams have least visibility into. Identity sits in the partner tenant, the device is the partner's device, the access decision was historically made by the partner — and yet the data being read or shared lives in your tenant. A standard insider-risk threshold (per [`./mass-download-alert.md`](./mass-download-alert.md)) does not catch this class because the actor is a guest user, not an employee, and most CASB activity policies are scoped against the employee population.

The deployment is rarely greenfield. It is almost always part of a **Cross-Tenant Access Settings (cross-tenant access settings, abbreviated XTAS here) hardening programme** — the policy is the detect leg that sits behind the XTAS allow-list, catching what XTAS lets through. Without XTAS first, the alert volume is unmanageable. Without the alert, XTAS is a static control with no telemetry on whether the allow-list still reflects current behaviour. (Note: Microsoft Learn uses "cross-tenant access settings" lowercase; XTAS / CTAS abbreviations are practitioner convention.)

### Use case 1 — Tier-1 ASEAN universal bank, post-Storm-2372 OAuth-cleanup wave

- **Org type:** tier-1 ASEAN universal bank, ~30k employees, M365 E5, ~140 active B2B partner tenants (correspondent-banking, fintech integrations, professional-services firms, regtech vendors); BNM RMiT supervised
- **Trigger:** following the 2024 Storm-2372 device-code-phishing campaign, internal review identified that 60+ partner tenants had token-grant patterns inconsistent with their documented engagement scope. CISO requested a detect-leg signal on guest-user activity in the host tenant, paired with a XTAS-hardening workstream
- **Scope:** all SharePoint sites + OneDrive; tenant-wide guest-user activity; threshold initially permissive (5,000 downloads / 24h per guest) and tightened over three quarters to 1,000 / 24h after XTAS reduced the inbound guest population
- **Outcome:** XTAS programme reduced active inbound partner tenants from ~140 to ~85 (Q1 → Q4); alert volume dropped from ~40/week to ~6/week; one confirmed compromised-partner incident detected (a regtech vendor's tenant was compromised; their guest account in the bank's tenant attempted a 2,400-file pull at 02:14 local — flagged, blocked at session level, partner notified)

### Use case 2 — Merchant acquirer, post-incident vendor-collaboration tightening

- **Org type:** payments processor operating across MY / SG / HK / TH; ~3k employees; ~60 partner-tenant integrations for merchant-onboarding, KYC, fraud-vendor data sharing; PCI DSS + BNM RMiT + MAS TRM + HKMA SA-2 in scope
- **Trigger:** vendor-side breach at a KYC-data-sharing partner — partner's tenant was compromised, attacker pivoted into the acquirer's SharePoint via the guest account, exfiltrated a folder of merchant-application files before detection. Post-incident review: no detection rule existed for high-volume guest activity
- **Scope:** all sanctioned SharePoint sites with external-guest access; OneDrive scoped to specific BUs only (Finance, Merchant Ops, Risk); per-partner-tenant differentiated thresholds (KYC-vendor tenants stricter; legal-counsel tenants more permissive)
- **Outcome:** documented detect-leg control for the supervisory follow-up; quarterly XTAS attestation cycle paired with this policy's metric; alert TP rate ~8% measured over 6 months; identified two further partner tenants with anomalous access patterns that turned out to be misconfigured automation (their RPA bots were running under guest accounts — flagged, partner remediated)

### Use case 3 — Consulting firm, audit-driven XTAS programme

- **Org type:** professional-services firm, ~10k employees, M365 E5, project-based external collaboration is core business; ~300 active client-tenant relationships at any time
- **Trigger:** external audit observation — "external collaboration is operational reality but external-user activity has no monitoring layer beyond the underlying SharePoint audit log; no alerting on volumetric anomaly"; remediation commitment to deploy guest-activity monitoring within 90 days
- **Scope:** SharePoint Online tenant-wide; threshold tuned high (10,000 downloads / 24h per guest) because legitimate engagement-end client-side downloads dominate; XTAS partner-tenant allow-list scoped per-engagement
- **Outcome:** ~70% of year-1 alerts were legitimate engagement-end "client copies down the deliverables" patterns; documented exception workflow at engagement-close became the operational norm; ~20% were misclassified-as-guest internal-via-personal-MSA users (employees using personal accounts as guests in client tenants — surfaced a separate identity-governance fix); the remaining ~10% included two genuine partner-side incidents

### Use case 4 — Digital-native challenger bank, multi-tenant SaaS architecture

- **Org type:** neobank, ~500 employees, M365 E5, heavy reliance on partner-tenant collaboration for engineering / product / data partnerships; ~25 active B2B partner tenants
- **Trigger:** pre-IPO due diligence flagged the partner-tenant surface as an unmonitored exposure; board ask — "what is our visibility on partner-side compromise propagating to us?"
- **Scope:** tenant-wide; threshold initially aggressive (500 downloads / 24h) given small legitimate-collaboration baseline; per-partner thresholds for the three high-volume partners (data-analytics vendor, design-collaboration vendor, security-vendor research-data sharing)
- **Outcome:** alert volume manageable (~3/week); pair-policy with Entra ID Protection guest-user risk signal added in Q2; one near-miss caught — design-collab vendor's tenant had a credential-stuffing incident; their guest account in the neobank tried to enumerate file metadata at 03:40 (3,100 metadata reads, sub-threshold for download but enough to trigger a related anomaly rule); incident contained before file pull

## Implementation pattern

Typical 10-week rollout. Assumes M365 connector is live and Entra B2B is in use; XTAS may or may not already be configured.

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Inventory active partner tenants (Entra → External Identities → Cross-tenant access settings → inbound activity); pull 90-day baseline of guest activity from `CloudAppEvents` filtered by `IsExternalUser=true`; identify high-volume partner tenants by `HomeTenantId` | Partner-tenant inventory; baseline statistics per partner |
| W2 | Cross-walk inventory against vendor-risk register, contract status, owner-of-record; flag orphaned tenants (guest accounts where the host-side sponsor has left); flag stale tenants (no activity in 90+ days) | Cleansed partner inventory; XTAS scope decision |
| W3 | XTAS hardening pass — configure inbound access per partner (default deny; per-tenant allow); document any blanket-allow exceptions with named owner; identify partners whose users should be members not guests (and route to identity-governance fix) | XTAS configured; pre-policy guest population reduced |
| W4 | Define per-partner threshold tiers: standard guests (default threshold), high-volume partner-tenant allow-list (raised threshold + named owner), restricted partners (low threshold + alert on any download) | Threshold tiering documented |
| W5 | Configure activity policy: scope to `IsExternalUser=true`, threshold = baseline-derived (start permissive, P95 of guest baseline + headroom), action = Alert only, no auto-suspend; route to identity admin + SOC | Policy live in alert-only mode |
| W6 | SOC + identity admin triage of W5 alerts; classify TP / FP / partner-misconfigured; cross-check against XTAS allow-list | First-tuning-pass classifier of FP causes |
| W7-W8 | Refine threshold per partner tier; add partner-tenant-specific exception groups for known legitimate high-volume engagements (project-end client-deliverable download windows, regtech bulk-report pulls) | FP rate <15% target |
| W9 | Pair-policy hookups — correlate with Entra ID Protection guest-user risk; correlate with partner-tenant home-domain reputation; document SOC + identity-admin shared runbook (partner-side compromise has a different escalation chain from insider-risk) | Cross-signal correlation live |
| W10 | Document the policy as production-ready; define quarterly XTAS re-attestation cycle; define partner-tenant addition / removal workflow that updates both XTAS and policy exception lists | Steady-state ops handoff; quarterly attestation gate |
| W11+ | Quarterly XTAS attestation + threshold re-tuning + new-partner onboarding flow | Quarterly metric on TP / FP rate; quarterly XTAS-drift report |

The W1-W3 inventory + XTAS hardening is the work. Deploying the activity policy without the XTAS pre-work produces a flood of alerts on partner tenants that should not have inbound access at all — the policy ends up substituting for missing access control, which is a category error.

## Action

- Primary: **alert + email** to identity admin and SOC L2 (not L1 — the triage path involves contacting the partner tenant, which is not an L1 task)
- Secondary: **session-level block** at the SharePoint app where corroborated with Entra ID Protection guest-user risk High AND partner-tenant home-domain on the watch-list — human approval before any block
- Do not **auto-suspend the guest account** — disabling a guest user via MDA governance breaks the partner relationship and tips off the partner's compromised account holder; the right escalation is partner-tenant-side incident notification

## Scope

- **Users:** external / guest users (`UserType = Guest` in Entra; `IsExternalUser = true` in `CloudAppEvents`)
- **Apps:** sanctioned SaaS where B2B sharing is enabled (SharePoint Online + OneDrive primary; Teams shared-channels secondary)
- **Device posture:** N/A — the user is external; their device claim comes from the home tenant and cannot be authoritatively asserted by the host tenant
- **Network position:** N/A — partner-tenant guest sessions originate from arbitrary network positions
- **Traffic:** download + share events by external users (file enumeration counted separately as a lower-severity signal where supported)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → Policies → Policy management → Threat detections → Create Activity policy | Activity type = Download file OR Share file; User type = External user; Threshold = baseline-derived; starting point 1,000 in 24h (BFSI) or 5,000 in 24h (consulting); Action = Alert + email to identity admin and SOC; Scope = all sanctioned SharePoint + OneDrive sites; partner-tenant differentiation via `AccountObjectId` filter where per-tenant tiering is desired | API-mode; post-event; classification of user as External depends on the user's home-tenant claim — pre-existing federation misconfigurations propagate into this filter | "External user" classification depends on the user's home tenant claim — mis-classification at federation time means the filter misses them. A compromised partner-tenant account behaving within threshold per session = slow-and-low B2B exfil (sub-threshold). **Activity-policy auto-disable thresholds (commonly cited as 200k/day, 100k/3h)** — **`[VERIFY against current Microsoft Learn `activity-policy` / `data-protection-policies` Limitations pages]`**; could not confirm a public Microsoft Learn URL at QA time. Practitioner-observed behaviour: partner-tenant bulk events can saturate; silent saturation looks like alert decline. Pair with XTAS restricting partner scope, NOT as a substitute for XTAS. Guest accounts with `UserType` manually flipped to Member (a common workaround for partners with frequent access) silently drop off the filter |
| Netskope | `[unverified]` — Netskope external-user attribution + activity policy | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security external-user policies | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB external-user detection | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security external-user activity | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant with ~90 active inbound partner tenants after XTAS hardening:

```yaml
policy:
  name: "B2B partner-tenant exfil — guest-activity threshold"
  type: ActivityPolicy
  activity_filters:
    activity_type:
      - FileDownloaded
      - FileShared
      - FileSyncDownloadedFull        # OneDrive sync-client download events
    apps:
      - SharePoint Online
      - OneDrive for Business
  user_filter:
    include:
      - IsExternalUser: true          # primary filter
      - UserType: Guest               # secondary corroboration; both required where supported
  match_parameters:
    type: RepeatedActivity
    count_threshold: 1000             # standard-tier; baseline-derived P95 + 30% headroom
    time_window_hours: 24
    scope: SinglePartnerTenantSingleApp     # one partner-tenant, one app, one user
  partner_tenant_tiers:
    restricted:                       # partners under elevated scrutiny — KYC-vendor, regtech
      home_tenants:
        - kyc-vendor-a.onmicrosoft.com
        - regtech-vendor-b.onmicrosoft.com
      count_threshold: 200            # tighter
    standard:                         # default — all other allow-listed partners
      count_threshold: 1000
    high_volume:                      # documented bulk-collaboration partners
      home_tenants:
        - legal-counsel.onmicrosoft.com
        - audit-firm.onmicrosoft.com
      count_threshold: 5000
      named_owner: "external-collaboration-governance@example.com"
  governance:
    action: Alert
    auto_suspend: false               # NEVER auto-disable a guest — partner-side handling required
    notify_user: false                # do not tip off the actor
  alerts:
    severity: High
    email_recipients:
      - identity-admin@example.com
      - soc-l2@example.com
    sentinel_forward: true
    enrichment:
      - partner_home_tenant_id
      - partner_home_domain
      - user_principal_name
      - file_paths_top_50
    power_automate_flow: "partner-tenant-incident-create"
  cross_signal_correlation:
    entra_id_protection:
      guest_user_risk_threshold: Medium    # boost severity to Critical if guest-user-risk High
    xtas_drift_check:
      flag_if_partner_not_in_allowlist: true
  retention:
    policy_match_record_days: 180     # longer than insider-risk; partner-side incidents take longer to confirm
```

The `count_threshold: 1000` value should be **derived from the partner-tenant inventory baseline**, not used as a literal default. Partner populations with high legitimate-collaboration volumes (consulting / legal / audit-firm tenants) need raised thresholds; partner populations under elevated scrutiny (KYC vendors, regtech, security vendors with research-data flows) need tightened thresholds. A single tenant-wide threshold misframes the problem.

## Variants

### Industry-specific

- **BFSI:** partner-tenant inventory is dominated by correspondent banks, KYC / AML vendors, regtech, and professional-services firms; XTAS governance under BNM RMiT third-party-risk expectations; per-partner tiering with named owner-of-record is standard; auto-suspend never enabled (partner-relationship management is the right channel)
- **Healthcare:** partner population includes HIE (health-information-exchange) tenants, research collaborators, clinical-trial sponsors; legitimate-bulk-download patterns dominate (dataset packs for collaborative analysis); per-trial / per-protocol allowlist; HIPAA business-associate-agreement (BAA) status drives partner-tenant tiering
- **Tech / SaaS:** partner-tenants for joint-engineering, design partners, customer-side admin tenants accessing vendor's support workspaces; project-end "grab everything" pattern dominates FPs; engagement-end exception workflow is the operational reality
- **Retail:** loyalty-programme partners, marketing-agency partners, supply-chain partners; partner-tenant inventory tends to be large and stale (orphaned partners post-engagement); XTAS hygiene is the main fight before policy deployment is meaningful
- **Public sector / Legal:** inter-agency collaboration tenants, external legal-counsel tenants, matter-coordinator workflows; Purview eDiscovery / matter-management workflows produce legitimate-bulk-download events; matter-coordinator UPN allowlist; protective-marking-scheme cross-walk [Microsoft Learn: https://learn.microsoft.com/en-us/purview/ediscovery-overview] (the classic eDiscovery experiences were retired 2025-08-31; the new Purview eDiscovery is the supported surface)

### Maturity-based

- **Immature:** activity policy deployed without XTAS hardening first; tenant-wide single threshold; no partner-tenant inventory; alerts dominated by unknown-partner-tenant traffic that should not have been allow-listed in the first place; FP rate >40%; SOC ignores by month 3; XTAS configuration still default-permissive
- **Mature:** XTAS hardened with per-partner allow-list; per-partner threshold tiers in place; documented exception workflow for known high-volume collaborations; quarterly XTAS attestation cycle running; FP rate <15%; identity-admin + SOC shared runbook; partner-side incident-notification process documented
- **Advanced:** per-partner-tenant behavioural baseline (each partner's P95 of last 90 days drives that partner's threshold); cross-signal correlation with Entra ID Protection guest-user risk + partner-home-domain reputation + content-class signal (per [auto-label-pci-data](../dlp/auto-label-pci-data.md) labels accessed by guest); partner-tenant addition / removal flow integrated with vendor-risk register; XTAS-drift detection automated (alert when a partner tenant gains activity beyond the documented engagement scope); contractual partner-side notification SLA embedded in MSA

## Control mappings

- **CIS Microsoft 365 Foundations Benchmark v5.0.0** (30 April 2025) — direct maps for this policy:
  - **CIS 5.1.6.1 (L2)** Ensure that collaboration invitations are sent to allowed domains only — primary upstream control that constrains the inbound guest population this policy then monitors
  - **CIS 5.1.6.2 (L1)** Ensure 'External sharing' of calendars is not available — peripheral but part of the external-collaboration surface
  - **CIS 5.1.6.3 (L2)** Ensure 'External Identities' inbound restriction is configured — the cross-tenant access settings control surface
  - **CIS 5.3.2 (L1)** Ensure 'Privileged Identity Management' is used to manage roles — peripheral; relevant where partner-tenant abuse escalates to host-tenant role assignment
  - **CIS 7.2.5 (L2)** Ensure that SharePoint guest users cannot share items they don't own — pairs directly with this detect leg to constrain the action a compromised guest can take
  - **CIS 2.4.3 (L2)** umbrella — "Ensure additional security mechanisms are configured" (Microsoft Defender for Cloud Apps as the named MDA-dependent detection layer)
- **Microsoft Secure Score** — Identity group ("Restrict guest user permissions", "Allow only specific external domains") + Apps group ("Block sharing with external users on documents not labeled") [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score] and [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]. Note: Microsoft Secure Score restructured to a four-group model (Identity / Device / Apps / Data) — earlier five-group references including Infrastructure are stale.
- BNM RMiT clause(s): [BNM RMiT third-party risk + DLP + cybersecurity operations](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]` (illustrative only — not regulatory advice)
- ISO 27017 control(s): [CLD.6.3.1 shared responsibility; CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation); A.11 (third-party relationships)
- NIST CSF 2.0 subcategory(ies): `DE.CM-06` (external service-provider activity monitored), `PR.AA-05` (access permissions), `ID.SC-04` (suppliers monitored), `GV.SC-07` (supplier risk management) `[VERIFY]`
- DORA (EU) ICT third-party-risk articles where applicable `[VERIFY]` (illustrative only — not legal/regulatory advice)
- MAS TRM external-party-risk sections where applicable `[VERIFY]`

## False-positive risk

- Legitimate B2B partner collaboration projects with high-volume file exchange — name an exception group with named owner
- Misclassified guest users (set up as guest but functioning as member; or `UserType` manually flipped)
- Engagement-end "client copies down all deliverables" pattern in consulting / legal / audit relationships
- Bulk-data-sharing partners under BAA / DPA (data-analytics, AML, fraud-vendor flows)
- Partner-side automation (RPA, integration bots, scheduled-pull scripts) running under guest accounts instead of dedicated service identities
- Personal-account-as-guest scenarios — individual contractor or contingent worker using personal MSA as a guest identity

## Real-world FP experience

Typical FP-rate trajectory for a tier-2 BFSI tenant deploying after XTAS hardening. **Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.**

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 40-60% | Pre-XTAS-cleanup residual partner traffic; misclassified guests; partner-side automation running under guest accounts |
| W4 | 20-30% | After XTAS-allow-list reconciliation; after partner-side automation re-identified |
| W8 | 10-15% | After per-partner-tier thresholds + documented engagement-end exception windows |
| W12 | 6-12% | Per-partner-tenant behavioural baseline; XTAS-drift detection live |
| Steady-state | 4-8% | Quarterly attestation cycle; new-partner onboarding flow ties XTAS + policy exception list |

If the policy is deployed **without** XTAS hardening first, W1 FP rates of 70-90% are normal and the policy usually gets rolled back within two months.

Named FP scenarios encountered repeatedly across deployments:

| Scenario | Mitigation |
|---|---|
| Engagement-end client-side download of project deliverables (consulting / legal) | Engagement-management workflow integration; named owner; time-bounded exception window |
| Partner-side RPA / automation running under a guest account | Identity-governance fix — partner moves to dedicated service identity, not guest user; until then, named UPN exception with renewal cadence |
| Audit-firm bulk-pull during annual audit window | Pre-audit coordination; named audit-firm tenant in the high-volume tier; time-bounded if narrowly scoped |
| Legal-counsel matter-document pull for active matter | Matter-coordinator allowlist; matter-management workflow integration |
| KYC-vendor batch-export of merchant-application files | Tiered per-partner threshold; differentiated authorisation chain; PCI / merchant-data flow documented under DPA |
| Misclassified guest (`UserType=Guest` but functioning as member) | Identity-governance audit; route to member-vs-guest correction; not a policy fix |
| Personal-account-as-guest (employee using personal MSA against client tenant) | Identity-governance fix at the home tenant; surface to identity admin |
| Sync-client download events from a legitimate guest-side OneDrive sync configuration | Document the sync configuration as expected; raise threshold for that partner tenant or exclude `FileSyncDownloadedFull` for documented sync partners |

## Operational cost

- **Exception-handling load:** medium-high during W4-W12 ramp (5-10 partner-side exceptions per week typical, plus engagement-end exceptions in consulting / legal); medium steady-state (2-4 per week) — partner population is dynamic
- **Triage load:** medium ongoing — every alert generates an identity-admin + SOC L2 ticket; typical 3-8 alerts per week per 100 partner tenants at FP rate 10%
- **End-user friction:** N/A in alert-only mode (the user is external); high if session-block is enabled — partner relationship management implications

Typical staffing: 0.1-0.2 FTE identity admin for XTAS attestation + partner-tenant onboarding flow; 0.1 FTE SOC L2 for triage; quarterly cycle requires ~2-3 days for the cross-XTAS-and-policy reconciliation. Vendor-risk programme analyst typically owns the partner-tenant inventory at ~0.1 FTE.

## Privacy / data-protection considerations

- External-user activity audit-log = third-party processing record — document under PDPA / GDPR processor-controller treatment; partner tenant is the controller of the user identity, host tenant is the controller of the file content
- DPA / BAA terms with the partner determine whether host-side activity-monitoring is contractually permitted; some partner contracts require partner-side notification of host-side monitoring of partner-tenant users
- Cross-border processing concerns when partner-tenant home region differs from host-tenant data residency
- DPIA scope: activity-log retention + cross-correlation with vendor-risk-register data + partner-tenant identity attributes
- Workforce notice does NOT apply (the user is external); partner-side notice depends on the partner's own workforce-monitoring posture
- For PDPA MY 2024: data-sharing arrangements with external parties subject to documented controller / processor mapping; this policy generates evidence relevant to that mapping (illustrative; not legal advice)

## Integration with broader programmes

- **Vendor-risk programme:** the partner-tenant inventory feeds vendor-risk register reconciliation; alert events feed vendor-incident-notification workflow; quarterly XTAS attestation is a control-effectiveness signal for the vendor-risk programme
- **XTAS attestation cycle:** policy alerts surface partners whose actual activity deviates from documented engagement scope — primary input to quarterly XTAS allow-list review
- **Audit cycle:** activity-policy coverage + measured TP/FP rate + named-partner-incident evidence feed the annual third-party-risk control-effectiveness review (under BNM RMiT third-party-risk / DORA ICT TPRM / MAS TRM external-party-risk regimes) `[VERIFY clause-specific obligations]` (illustrative only — not legal/regulatory advice)
- **Board / executive reporting:** quarterly metric — "confirmed partner-side incidents per quarter"; "XTAS-drift detections per quarter" (partners whose activity gained scope without governance approval); trend matters more than absolute number
- **Incident response runbook:** B2B-partner-side incident has a distinct playbook from insider-risk — partner-tenant notification + joint-investigation + potential contract clause invocation; this policy is the primary detect signal feeding that playbook
- **DPIA refresh:** every annual DPIA cycle includes a review of the external-user-activity processing scope, cross-correlation data sources, and partner-controller mapping
- **Contract / MSA refresh:** partner-side notification SLA + joint-investigation obligation embedded in MSA at next renewal; this policy's existence is documented as a host-side control in the DPA

## Anti-patterns specific to this policy

1. **"Deploy the activity policy without XTAS hardening first"** — produces 70-90% FP rates in W1 because the partner-tenant population is unvalidated; programme rolls back within two months; the policy gets blamed when the real failure is upstream access control
2. **"Auto-suspend the guest account on alert"** — disabling the partner's identity in your tenant breaks the partner relationship without coordination; if the partner was genuinely compromised, the right channel is partner-tenant-side incident notification, not unilateral guest-disable; if the alert was a FP, the host has created a contractual incident
3. **"Tenant-wide single threshold across all partners"** — legitimate-volume varies by 50× between an audit-firm partner-tenant during annual audit and a regtech partner under normal operation; single threshold guarantees either alert fatigue (low) or missed events (high)
4. **"Treat XTAS as one-time-set-and-forget"** — partner relationships churn; engagements end; orphaned guest accounts accumulate; quarterly XTAS-drift detection is the operational complement to this policy
5. **"Route alerts to L1 SOC only"** — triage involves contacting the partner-tenant identity admin, which is not an L1 task; the alert ages out or escalates incorrectly; route to identity-admin + SOC L2 jointly with shared runbook
6. **"Flip `UserType` from Guest to Member to reduce friction for high-volume partners"** — silently drops them off the activity-policy filter; common workaround when partners complain about per-session access flows, and it kills the detect signal completely
7. **"Use Cross-Tenant Access Settings as substitute for the activity policy"** — XTAS is the access-control leg, this policy is the detect leg; XTAS alone has no telemetry on whether the allow-list still reflects current behaviour, the activity policy alone is reactive after access is too broad
8. **"Notify the guest user on alert"** — tips off the actor on the partner side; if the partner-tenant identity is genuinely compromised, the actor accelerates the remaining transfer; the partner-tenant security contact is the correct notification path, not the user
9. **"Treat all partner-side anomalies as partner-side compromise"** — most alerts are misconfigured automation, engagement-end legitimate pulls, or misclassified-guest scenarios; the partner-compromise subset is small and requires triage discipline
10. **"Skip the partner-side incident-notification SLA in the MSA"** — when a partner-side compromise is detected, contractual ambiguity on notification + joint-investigation obligation slows the response by days; the policy generates a signal that the legal framework cannot act on

## Coverage gaps

- Slow-and-low B2B exfil within threshold (e.g. 50 files/day per guest over weeks) — bypass class; pair with weekly-baseline Sentinel KQL on guest activity
- Cross-Tenant Access Settings drift — partner-tenant scope changes without policy update; cross-link to [`../../08-failure-modes/xtas-drift.md`](../../08-failure-modes/xtas-drift.md) `[verify path]`
- B2B guest device claims come from home tenant — device-posture checks unreliable for external users; not a policy fix, an identity-architecture limitation
- Personal-account-as-guest scenarios — guest accounts where the home tenant is an MSA consumer tenant; weak identity-assurance; partner-tenant security posture is unknowable
- Misclassified guests (`UserType` flipped to Member) — silently drop out of the filter; cross-link to [`../../08-failure-modes/user-type-misclassification.md`](../../08-failure-modes/user-type-misclassification.md) `[verify path]`
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — detection is post-event
- Teams shared-channel external-member activity may produce different event class — verify coverage against current MDA event-schema documentation `[VERIFY]`
- Partner-tenant compromise that uses guest-account session-token replay from new infrastructure — Entra ID Protection guest-user-risk signal is the corroboration leg, not this policy

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
