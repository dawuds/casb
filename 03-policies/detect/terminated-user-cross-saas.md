# Terminated user — cross-SaaS activity

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**
> Required capabilities: [Compromised-account detection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector. Entra-as-IdP for the deletion signal.
> MDA playbook reference: [Policy 10](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Terminated-user activity in connected SaaS).

## Purpose

Detect cross-SaaS account activity after a user's Entra account is deleted. Catches the offboarding-gap class — terminated user retains a local account in a connected SaaS under a different email format, or holds non-user credentials (PATs, service-principal secrets, MFA bypass methods) added before departure. Counters MITRE ATT&CK `T1078.004 Cloud Accounts` (persistence), `T1136 Create Account` (where the user created a SaaS-local account before leaving), `T1098.001 Additional Cloud Credentials` (pre-termination credential additions), `T1556` (MFA-method additions pre-termination).

## What organisations use this for

For most regulated firms this is a **closure control**, not a primary detection — the assumption is that IT offboarding works, the SaaS-local-account audit catches what HR-IT integration misses, and this policy catches what neither catches. It earns its keep two ways: (a) on the day an auditor asks "how do you know terminated users have no residual SaaS access?", the activity log from this policy is the answer; (b) when something genuinely slips through — most often a Salesforce / Workday / GitHub local account under a non-UPN email — this is usually how it surfaces.

Practitioner reality: the built-in is necessary but not sufficient. The Entra-UPN-to-SaaS-account match is exact-string by design, and the three offboarding sub-cases that fall outside it (service-principal credentials added pre-termination, MFA-method additions, accounts the leaver created themselves in connected SaaS) cause more real incidents than the cross-SaaS-email mismatch does. The policy that actually works is **the built-in + an offboarding runbook that handles the three sub-cases + a quarterly orphan-account reconciliation against HR**. Skipping any of the three leaves a control gap the auditor will eventually find.

### Use case 1 — Tier-1 ASEAN universal bank, post-Storm-2372 OAuth cleanup

- **Org type:** tier-1 ASEAN universal bank, ~30k employees, M365 E5, BNM RMiT supervised, BCM regulated as a domestic systemically important bank
- **Trigger:** internal incident-response review after a Storm-2372 device-code-phishing campaign hit several peer banks in the region; the bank's own threat-modelling exercise identified offboarding gap as the highest-rated residual access risk (departed staff retaining credentials in SaaS that survived the Entra disable). Board-level ask for "demonstrable offboarding coverage across all connected SaaS"
- **Scope:** all employees + contractors; all 14 connected SaaS apps (M365 + Salesforce + Workday + ServiceNow + AWS + GitHub Enterprise + Box + Atlassian + the rest); built-in MDA policy + Sentinel KQL for MFA-method-additions + Entra App Governance for service-principal credential adds
- **Outcome:** in the first six months the built-in policy fired 11 times; 4 were genuine offboarding misses (3 Salesforce local accounts under a `firstname.lastname@bank.com.my` format that did not match the Entra `firstinitial-lastname` UPN, 1 GitHub non-EMU account); 7 were guest-scenario FPs. The Sentinel MFA-addition KQL caught 2 cases of pre-termination MFA-method additions that would otherwise have been invisible. Auditor accepted the combined log as evidence of BNM RMiT IAM control effectiveness `[VERIFY clause]`

### Use case 2 — M&A integration with overlapping accounts

- **Org type:** mid-size acquirer integrating a digital-native challenger bank, ~8k staff combined; both tenants M365 E5, acquiree carried separate Salesforce + GitHub tenants
- **Trigger:** acquirer's CISO insisted on full offboarding-coverage proof before formal Day-1 integration cutover; M&A workstream identified ~120 leavers across the acquired entity in the 6 months bracketing the deal, many with informal access arrangements
- **Scope:** all leavers from the acquired tenant; both pre-merger SaaS estates; correlation across two Entra tenants in the same connected-MDA group via Cross-Tenant Access Settings
- **Outcome:** surfaced ~30 orphaned SaaS-local accounts that had survived the acquiree's HR-IT process; ~15 service-principal credentials with no documented owner; integration team adopted the policy as the standing "did we get them all" check during subsequent BU divestitures. Cost: ~3 weeks of identity-team time to manually revoke + document the surfaced accounts

### Use case 3 — Departed-RM credential reuse incident (post-incident deployment)

- **Org type:** wealth-management arm of a regional FI, ~5k employees, M365 E5; relationship managers (RMs) carry deep client-data access in SharePoint + Salesforce
- **Trigger:** confirmed incident — an RM who departed for a competitor was later observed accessing the acquiring-firm's Salesforce instance via a `legacy.<name>@firm.com` local account that had been created during a 2022 data-migration project and never cleaned up. Discovered when client retention-call data appeared at the competitor. Triggered an HR / Legal preservation hold, regulatory disclosure consideration, and a board-level remediation programme
- **Scope:** initially all RMs (~600); extended within 90 days to all client-facing roles (~2k); extended within 6 months to all staff. Threshold for governance action: alert + manual revoke + HR / Legal notification within 4 hours
- **Outcome:** programme documented as an audit response; surfaced 7 additional historical orphaned accounts that had escaped earlier reviews; FP rate ~12% (driven by ex-employees retained as consultants under different account scopes); the policy is now the documented closure check for every RM departure

### Use case 4 — HR-IT integration programme, auditor finding remediation

- **Org type:** mid-cap retail-banking subsidiary of a regional group, ~12k staff, M365 E5, Workday-as-HRIS
- **Trigger:** internal audit issued a "high" finding — "offboarding effectiveness cannot be evidenced for non-Entra-authenticated SaaS accounts." Finding referenced ~40 orphaned accounts surfaced during a one-off audit sweep. Audit committee required closure within one cycle
- **Scope:** all staff; integration of Workday termination event → Entra delete → MDA policy fires → ServiceNow ticket auto-created → IAM analyst manually revokes per-SaaS local account. Quarterly reconciliation between Workday active-employee roster + per-SaaS user lists became standing operational control
- **Outcome:** auditor closed the finding after one quarterly cycle of evidence; the ServiceNow ticket-creation workflow became reusable for other identity controls; ongoing operational cost ~0.3 FTE for the IAM-analyst manual-revoke work

## Implementation pattern

Typical 10-week rollout for a tenant new to the built-in policy plus the supplementary detections:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Enumerate connected SaaS app inventory; identify which apps have local-account creation outside SCIM; document the per-app account-identifier format (UPN vs email vs SaaS-internal-ID); identify owners of each app | App inventory + identifier-format matrix |
| W2 | Verify Entra-as-IdP precondition; pull 90-day baseline of `Delete user` Entra Audit events; sample-test the UPN-to-SaaS-account match for 10 known leavers across the top 5 SaaS | Identifier-mismatch sample data — drives the supplementary-detection scope |
| W3 | Enable built-in anomaly policy (edit, do not recreate); scope = all users; per-app governance action = Notify admin (no auto-suspend); severity = High; alerts routed to identity admin + SOC | Built-in policy live in alert mode |
| W4 | Author Sentinel KQL for the three sub-cases — (a) service-principal credentials added by user pre-termination; (b) MFA-method additions in the 7 days before deletion (Appendix B.4 in MDA playbook); (c) per-SaaS local-account-creation events by leaving users in the 30 days pre-termination | Three KQL queries scheduled in Sentinel |
| W5 | Integrate Workday (or equivalent HRIS) termination event into the offboarding runbook; document the HR-IT-CASB sequencing; identify the SLA between HRIS termination and Entra delete (target: <4 hours) | Documented offboarding runbook v1 |
| W6 | Build the per-SaaS manual-revoke playbook — for each connected SaaS, document where the IAM analyst goes to disable a local account; SCIM-cascade table (MDA Appendix A) lifted into the runbook | Per-SaaS revoke playbook |
| W7 | Run the first quarterly orphan-account reconciliation — Workday active-employee list vs each SaaS user list; baseline orphan-count surfaced | Baseline orphan count; remediation backlog |
| W8 | Wire alert routing — identity admin + SOC L1 + automated ServiceNow ticket creation; document escalation path | Alert-routing live + tested |
| W9 | Tabletop: simulate a departed-user-credential-reuse incident; verify the policy fires, the alert routes, the manual revoke completes in <24 hours | Tabletop debrief + runbook refinements |
| W10 | Document the policy as production-ready; define quarterly reconciliation cadence + auditor-evidence-pull procedure | Steady-state ops handoff |
| W11+ | Quarterly orphan-reconciliation; annual auditor-evidence pull; runbook refresh after every connected-app change | Quarterly metric on detected leavers + orphans closed |

The W2 identifier-mismatch sampling is the critical step. If 30% of the sample shows UPN-to-SaaS-account mismatches, the built-in policy alone covers only ~70% of the offboarding population — the supplementary KQL + the quarterly reconciliation must close the rest. Skipping W2 produces a deployment that looks complete but silently under-covers.

## Action

- Primary: **alert + manual revoke** per app (no auto-suspend for non-Entra accounts — Entra suspend cannot disable a non-Entra local account)
- Notify identity admin + SOC; auto-create ServiceNow / Jira ticket
- For service-principal credential additions surfaced via supplementary KQL: **suspend service principal** + rotate credential; treat as a separate IR class

## Scope

- **Users:** all terminated employees (Entra-deletion event is the trigger); extended via supplementary detections to all leavers (pre-termination signals)
- **Apps:** all connected SaaS where activity logs surface user identifiers (SharePoint, OneDrive, Salesforce, AWS, GitHub, ServiceNow, Workday, Box, Atlassian, etc.)
- **Device posture:** N/A
- **Network position:** any
- **Traffic:** activity-log events with user identifier; Entra `AuditLogs` for the sub-cases

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Anomaly detection policy → Activity performed by terminated user (built-in; edit, do not create) | Scope = all users; Per-app governance action = Notify admin + manual revoke; Severity = High; Alert to identity admin + SOC; no auto-suspend on non-Entra local accounts | API-mode; built-in anomaly; requires Entra-as-IdP for the deletion signal; correlation depends on shared identifier (usually email) between Entra deleted-user and SaaS-app account | **Correlation depends on exact-match email between Entra deleted-user and SaaS-app account.** If Entra UPN = `user.surname@corp.com` but Salesforce account = `surname.user@corp.com`, no match, no alert. Three additional offboarding sub-cases NOT covered by this built-in: (a) service-principal credentials / PATs / API keys added pre-termination — check Entra Audit `AppRoleAssignment` and `Add service principal credentials` events; (b) MFA-method additions / backup-email changes (T1556) — Entra Audit `User registered alternative authentication device`; (c) accounts created by terminated user in connected SaaS (T1136) — per-SaaS admin audit, not in MDA |
| Netskope | `[unverified]` — Netskope terminated-user correlation | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security identity correlation | | | |
| Skyhigh | `[unverified]` — Skyhigh terminated-user detection | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration for a tier-2 ASEAN BFSI tenant with the built-in policy plus supplementary detections:

```yaml
policy:
  name: "Terminated user — cross-SaaS activity (built-in)"
  type: AnomalyDetectionPolicy
  built_in: true
  scope:
    users: all
    apps:
      - SharePoint Online
      - OneDrive for Business
      - Salesforce
      - Workday
      - ServiceNow
      - AWS
      - GitHub Enterprise
      - Box
      - Atlassian
  correlation:
    identifier_match: ExactEmail            # the hard constraint — see W2 mismatch-sampling
    fallback_strategy: SupplementaryKQL     # do not rely on built-in alone
  governance:
    action: NotifyAdminOnly                 # NEVER auto-suspend non-Entra local accounts
    per_app_governance: ManualRevoke
    auto_suspend: false
  alerts:
    severity: High
    email_recipients:
      - identity-admin@example.com
      - soc-l1@example.com
    sentinel_forward: true
    auto_ticket_create: true                # ServiceNow integration via Logic App
    notify_user: false                       # user is terminated
  retention:
    alert_record_days: 365                  # auditor-cycle retention
    cross_correlate_to: EntraAuditLogs

supplementary_detections:
  - name: "Sub-case (a) — service-principal credentials added pre-termination"
    source: Sentinel
    schedule: hourly
    kql_ref: "Appendix B.4 variant — AddServicePrincipalCredentials"
    severity: High
  - name: "Sub-case (b) — MFA-method additions in 7 days pre-deletion"
    source: Sentinel
    schedule: daily
    kql_ref: "MDA playbook Appendix B.4"
    severity: High
  - name: "Sub-case (c) — accounts created by leaver in connected SaaS"
    source: PerSaaSAdminAudit
    schedule: weekly
    severity: Medium

operational_controls:
  quarterly_reconciliation:
    sources:
      - Workday active-employee roster
      - Per-SaaS user list (top 9 connected apps)
    target_orphan_count: 0
    sla_remediation_days: 30
  offboarding_runbook_link: "/runbooks/offboarding-v3.md"
  hr_it_termination_to_entra_delete_sla_hours: 4
```

The `NotifyAdminOnly` governance action is non-negotiable for non-Entra local accounts. Entra "Suspend user" disables the Entra object; it does nothing to a Salesforce-local account whose only relationship to Entra is a string-matching email. Auto-suspend on this policy is a category error.

## Variants

### Industry-specific

- **BFSI:** quarterly orphan-reconciliation is typically mandated by internal audit; the BNM RMiT IAM clause expectation (`[VERIFY]`) drives documented evidence of offboarding closure across all SaaS. RM-class departures (wealth management, private banking) are treated as elevated-risk and often get a same-day enhanced check rather than the standard quarterly cycle. Service-principal credential rotation discipline is a hard requirement
- **Healthcare:** focus on EHR-adjacent SaaS where local accounts persist (departing clinicians retaining portal access); HIPAA workforce-access termination expectation drives faster SLA than BFSI; quarterly orphan reconciliation often monthly
- **Tech:** dominant gap is GitHub Enterprise non-EMU accounts (email-linked rather than SCIM-cascaded), AWS IAM accounts created by engineering outside Identity Center, and developer-created service-principal credentials in Azure / GCP. Personal access tokens (PATs) survive Entra deletion entirely
- **Retail:** lower per-leaver risk but higher volume; the operational cost of manual revoke per SaaS becomes the binding constraint; some tenants accept a 7-day SLA rather than 4-hour
- **Legal / professional services:** consultant / contractor lifecycle dominates; many "leavers" are actually engagement endings where the user returns 6 months later — the policy must distinguish "permanent departure" from "engagement gap" in HRIS feed

### Maturity-based

- **Immature:** built-in policy enabled with default scope; no supplementary KQL; no quarterly reconciliation; no integration with HRIS termination event; no documented per-SaaS revoke playbook. The policy fires when it fires; alerts may or may not get actioned; auditor finding likely on first cycle. Common at 6 months post-deployment for tenants that turned on the built-in without an offboarding programme behind it
- **Mature:** built-in policy + supplementary KQL for all three sub-cases + quarterly reconciliation against HRIS + documented per-SaaS revoke playbook + ServiceNow ticket integration. SLA from termination to evidence-of-revoke <24 hours. Auditor-evidence pull is a documented quarterly artefact. FP rate ~10-15%, driven by legitimate guest-account scenarios
- **Advanced:** all of the above + automated revoke for SCIM-cascaded apps (within the SCIM 40-min latency window) + Purview IRM correlation (terminated-user signal boosted when other indicators present — see MDA Policy 12) + per-leaver risk-tiering (RMs / privileged users get same-day enhanced check) + Entra ID Governance Access Reviews as a forward-looking compensating control. Coverage extends to contractor / vendor / partner identities via Cross-Tenant Access Settings audit

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + offboarding](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY clause — IAM lifecycle / privileged-access lifecycle]`
- MAS TRM clause(s): access termination expectations `[VERIFY]`
- ISO 27017 control(s): [CLD.9.x access control overlay; CLD.8.1.5 removal of assets](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation)
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions), `PR.AA-04` (identities removed when no longer needed), `DE.CM-09` `[VERIFY]`
- PCI DSS v4.0 Req. 7.2 / 8.2 (user access revocation timeliness) where the firm is in CHD scope `[VERIFY]`

## False-positive risk

- Legitimate guest-account scenarios (user retains a vendor-side account legitimately due to ongoing engagement)
- Service-account UPN that resembles a former user
- Re-hired employees whose previous account was deleted but whose old SaaS-local account survived
- Contractors whose Entra account is deleted at engagement end but whose SaaS-local account is intentionally retained for an upcoming re-engagement
- M&A and divestiture scenarios where account states are temporarily inconsistent across systems

## Real-world FP experience

Typical FP-rate trajectory:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 35-50% | Guest-account-of-different-email scenarios; ex-employees retained as contractors with renamed accounts; M&A-in-flight identifier inconsistencies |
| W4 | 20-30% | After documented exception list for contractor / vendor / guest scenarios; after HR-IT data quality cleanup |
| W8 | 10-15% | After per-SaaS identifier-format mapping is documented + after HRIS-Entra delete SLA is tightened |
| W12 | 8-12% | Mature exception list; quarterly reconciliation closes the long tail of stale orphans |
| Steady-state | 5-10% | Most remaining FPs are unavoidable lifecycle edge cases (re-hires, contractor-to-employee transitions) |

The FP-rate trajectory is gentler than for volumetric policies like mass-download — the population (terminated users) is small and clearly defined. The harder problem is **false negatives** (silent under-coverage from UPN-mismatch), not false positives. FN rate is not measurable from the policy itself; only the quarterly reconciliation surfaces it.

Named FP scenarios encountered repeatedly:

| Scenario | Mitigation |
|---|---|
| Ex-employee retained as contractor under renamed account, original Entra account deleted | Document re-engagement exception list; HRIS termination workflow distinguishes Permanent vs Engagement-end |
| Salesforce "deactivated" user reactivated as a Guest licence for a one-off project | Per-app guest-management workflow; documented exception with expiry date |
| Hybrid-AD edge case — `onPremisesSyncEnabled=true` user "deleted" in cloud but still active on-prem | Validate via `IdentityInfo` table (`onPremisesSyncEnabled`); fix at the hybrid-AD writeback layer rather than the policy layer |
| Service account with UPN matching a former employee (rare but happens after BU restructuring) | Quarterly service-account inventory + naming convention enforcement |
| M&A-in-flight: account exists in acquirer tenant after acquiree-tenant delete | Document M&A integration exception window; sunset on integration cutover |
| Re-hired employee with old SaaS-local account still present | HR re-hire workflow includes orphan-reconciliation step before account re-provisioning |
| Long-departed employee whose SaaS-local account appears in a quarterly reconciliation, not in the live alert stream | Treat as historical-cleanup backlog; close per remediation SLA |

## Operational cost

- **Exception-handling load:** low to medium — terminated-user events are clearly defined; most ops time goes into per-SaaS manual revoke + documenting guest-account exceptions (~3-10 exceptions per quarter for a 10k-user tenant)
- **Triage load:** low — typical alert volume 5-20 per quarter for a 10k-user tenant; manual revoke per SaaS adds 30-60 minutes per leaver across the connected-app estate
- **End-user friction:** N/A (user is terminated)
- **Reconciliation cost:** quarterly orphan-reconciliation is the dominant operational cost — ~0.3 FTE for the IAM analyst at a 10k-user tenant, lower for smaller tenants

Typical staffing: 0.1 FTE platform admin (policy tuning + supplementary KQL maintenance); 0.3 FTE IAM analyst (manual revoke + quarterly reconciliation + auditor-evidence pull); 0.05 FTE compliance analyst (annual control-effectiveness review).

## Privacy / data-protection considerations

- Cross-SaaS activity correlation involves processing of former-employee data — workforce-monitoring scope under PDPA / GDPR Art. 88 / equivalent applies
- Lawful basis: post-employment data handling under PDPA / GDPR — document retention and deletion treatment for the alert record + the supplementary KQL output
- DPIA scope: cross-correlation of HRIS termination event with SaaS activity logs constitutes workforce monitoring; document under DPIA refresh cycle
- For EU / UK workforce: works-council / employee-representative notification typically required before deployment of cross-system monitoring of departed staff
- Retention discipline: the alert record contains former-employee identity + activity metadata; align retention with employment-record retention policy + auditor-evidence-cycle needs (typically 1-3 years for BFSI)

## Integration with broader programmes

- **HR-IT integration programme:** the policy is one of the documented closure checks in the HRIS-Entra-SaaS termination chain. Workday termination event → Entra delete SLA (<4 hours typical target) → built-in policy fires within hours of any cross-SaaS residual activity → manual revoke per SaaS within SLA
- **Internal audit cycle:** quarterly orphan-reconciliation + annual policy-effectiveness review feed the IAM-controls section of the annual internal audit (mapping to BNM RMiT IAM expectations / equivalent regional supervisory expectations). Auditor-evidence pull: the alert log + the quarterly reconciliation worksheet + the ServiceNow ticket trail
- **Incident response runbook:** any policy-fire that resolves to a TP triggers the "departed-user credential reuse" IR playbook — preservation hold, HR / Legal notification, investigation under privileged process. The IR runbook is shared with the mass-download-alert policy's IR runbook
- **Board / executive reporting:** quarterly metric — "leavers with detected cross-SaaS residual activity / total leavers" — trend matters more than absolute count; named-incident summary (sanitised) where TP rate produces a confirmed event
- **Insider Risk Management:** terminated-user signal becomes a Purview IRM indicator in mature deployments (per MDA Policy 12); correlation with pre-departure mass-download alerts, anomalous-email-forward, anomalous-print produces high-fidelity pre-resignation patterns
- **Vendor-risk programme:** the policy is a downstream consumer of HRIS data quality + Entra delete-event timeliness — both are dependencies that the vendor-risk register tracks
- **Identity governance:** Entra ID Governance Access Reviews + Lifecycle Workflows are the forward-looking compensating controls; this policy is the backstop when those fail or miss a non-Entra local account
- **Regulatory disclosure path:** for tier-1 BFSI in MY / SG / HK, a confirmed departed-employee credential-reuse incident with material customer-data implication may trigger supervisory notification (BNM / MAS / HKMA). The alert log is the primary timeline evidence — illustrative only, not legal/regulatory advice `[VERIFY]`

## Anti-patterns specific to this policy

1. **"The built-in policy is enabled, therefore offboarding is covered"** — exact-string UPN-to-SaaS-account matching produces silent under-coverage in any environment with non-uniform identifier formats. Without the W2 identifier-mismatch sampling + supplementary KQL + quarterly reconciliation, coverage is whatever the lowest-common-denominator format produces. Often this is 60-80%, not 100%
2. **"Set auto-suspend as the governance action"** — Entra suspend disables the Entra object, not the non-Entra SaaS-local account that this policy detects. Auto-suspend on this signal is a category error; for the Entra-resident account the user is already deleted, for the non-Entra-resident account the suspend is a no-op
3. **"Ignore the three sub-cases because the built-in covers the main pattern"** — service-principal credentials added pre-termination, MFA-method additions, and accounts created by leavers cause more real incidents than UPN-mismatch does. The Sentinel KQL for these three is not optional in a mature deployment
4. **"Skip the quarterly orphan reconciliation because the live policy will catch anything"** — the live policy only catches *active* residual access. Dormant accounts that nobody is using right now still exist as latent persistence vectors; only the reconciliation surfaces them
5. **"Tie the alert to the SOC, not to identity admin"** — SOC analysts triage volumetric and threat-actor-pattern signals; identity admins own the per-SaaS revoke. Routing exclusively to SOC produces alerts that age out because nobody with the right access does the revoke
6. **"Assume HRIS-to-Entra termination SLA is fast enough"** — at many BFSI tenants the HR termination event lands in Workday on departure day, but the Entra delete trails by 24-72 hours because of manual-approval steps. During the gap, the user can still cross-SaaS-access freely with valid Entra credentials. Fix the HRIS-Entra SLA first; the policy depends on it
7. **"Use the built-in policy for contractor offboarding without modification"** — contractor lifecycles are messier than employee lifecycles (engagement-end vs permanent departure vs re-engagement). FP rate balloons unless the HRIS feed distinguishes engagement-end from permanent termination
8. **"Treat the alert log as the only evidence"** — auditors typically want both the alert log (showing the policy fired) and the per-SaaS revoke confirmation (showing closure). Maintain both — ServiceNow ticket trail is the canonical link
9. **"Configure the policy and walk away because terminated users are a rare event class"** — connected-app inventories change quarterly; identifier-format conventions drift; HR feeds change schema after HRIS upgrades. Without a quarterly review of the policy itself, the coverage silently erodes
10. **"Skip the M&A and divestiture documentation"** — these scenarios produce the worst FP / FN profile, both at once. Document the M&A exception window with sunset date; document the divestiture orphan-account-handling procedure. Otherwise integration teams reinvent the wheel under time pressure

## Coverage gaps

- Non-email-matched accounts (different identifier format between Entra and SaaS local) — built-in policy silently misses; only supplementary KQL + quarterly reconciliation catches
- Service-principal abuse — pre-departure credential additions; T1098.001 not covered by this built-in (see [OAuth blind spot](../../08-failure-modes/oauth-blind-spot.md))
- MFA-method additions before departure (T1556) — separate Entra Audit-log query required (MDA playbook Appendix B.4)
- Per-SaaS account creation by leaver pre-termination (T1136) — per-SaaS admin audit, not in MDA
- Personal-account exfil before departure — invisible to this control; see mass-download-alert policy for the volumetric pre-departure pattern
- Hybrid-AD users with `onPremisesSyncEnabled=true` and no writeback — Entra delete silently fails; supplementary `IdentityInfo` validation needed
- GitHub Enterprise non-EMU setups — email-based linking, not SCIM-cascaded; manual revoke required
- Personal Access Tokens / API keys / OAuth refresh tokens issued to the user — survive Entra deletion in most SaaS until rotated
- Dormant orphan accounts that nobody is actively using — invisible to the live signal; only quarterly reconciliation surfaces them

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
