# Mass-delete anomaly

> Status: v0.0 — MDA column from playbook v1 (Policy 11 — Day 30 addition); other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [Anomalous-activity detection (volumetric)](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 11](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Mass-delete / mass-overwrite anomaly).

> **June 2025 built-in-anomaly-policy disablement / rename wave (boilerplate):** Several Microsoft built-in anomaly policies were transitioned to a dynamic-threat-detection model in June 2025 and renamed — Activity from suspicious IP addresses; Suspicious inbox forwarding; Suspicious inbox manipulation rules; Activity from anonymous IP addresses; Ransomware activity; Suspicious file access activity by user; Unusual ISP for an OAuth App; Suspicious email deletion activity. In particular, the built-in **Ransomware activity** detection was renamed to **"Ransomware payment instruction file uploaded to {Application}"** — practitioners must not rely on the legacy name in detection-engineering documentation. Source: [Microsoft Learn — anomaly-detection-policy (Important notice)](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy).

## Purpose

Detect high-velocity file deletion or modification events against sanctioned cloud storage — the ransomware-in-SaaS class. Counters MITRE ATT&CK `T1485 Data Destruction` (Detect) and `T1486 Data Encrypted for Impact` (SaaS variant). The Microsoft-named cluster **Storm-0501** has been documented executing this class of action against SaaS-resident content during ransom negotiation. **Practitioner inference:** "BlackCat-on-SaaS" variants are practitioner shorthand for the same broader behaviour pattern; Storm-0501 is the verified Microsoft cluster, and BlackCat-on-SaaS is not a Microsoft-canonical attribution. Source: [Microsoft Security Blog — Storm-0501](https://www.microsoft.com/en-us/security/blog/2025/08/27/storm-0501s-evolving-techniques-lead-to-cloud-based-ransomware/) `[VERIFY URL against current Microsoft Security Blog index]`.

## What organisations use this for

For most regulated FIs, mass-delete is the **second cloud-storage activity policy** they deploy — usually after mass-download — and the trigger is rarely a feature ask. It is almost always **post-incident learning** (their own or a peer's), an auditor's question about ransomware recovery for SaaS-resident data, or a board-level brief on the SaaS-ransomware class after a named-actor incident (Storm-0501, BlackCat-on-SharePoint (practitioner shorthand; not Microsoft-canonical), the 2024-2025 wave of OneDrive-encryption-by-ransomware variants). Unlike mass-download, where the dominant scenario is insider exfil, mass-delete sits at the intersection of **insider-sabotage** (disgruntled departures, contractor disputes, M&A integration friction) and **adversary-driven destruction** (ransomware that has pivoted from on-prem encryption into SaaS, OAuth-token-replay actors deleting evidence trails, post-compromise wipe to cover lateral movement).

The hardest design decisions are not the threshold value (SharePoint version-history offers a recovery floor that mass-download does not) but **scope discipline around legitimate bulk-delete patterns** — SharePoint site archival, mailbox cleanup, document-retention automation — and the **integration with backup / version-history / IR runbooks**. Without those, the alert is just noise; with them, it is the earliest in-tenant signal of SaaS-resident destructive action.

### Use case 1 — Tier-1 ASEAN universal bank, post-Storm-0501-class peer incident

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, mature on-prem ransomware programme (immutable backup, isolated recovery environment, tabletop-exercised IR)
- **Trigger:** a peer institution in the region took a hit from a Storm-0501-class actor that pivoted from on-prem to SharePoint Online, deleting and overwriting team-site content as a pressure tactic during ransom negotiation. Board asked "would we see this in our tenant within the first hour?" — answer was no
- **Scope:** tenant-wide on SharePoint + OneDrive; threshold derived from 30-day baseline at P99 + 50% headroom (~600 deletions / 30 min per user); migration / archival accounts excluded by UPN regex; signal feeds Sentinel + IR runbook for the SaaS-ransomware play
- **Outcome:** Day-1 deploy caught two service-account misconfigurations producing legitimate bulk-deletes that had been running undetected for months (retention-policy automation under a poorly-scoped account); no actual destructive incident in year 1; tabletop validated that the alert + version-history-restore path met the bank's 4-hour RTO for SaaS-resident content. Treated as **detection-effectiveness evidence for BNM RMiT cyber-resilience expectations** `[VERIFY against current BNM RMiT edition]` (illustrative — not regulatory advice) rather than as a productivity tool

### Use case 2 — Digital-native challenger bank, departing-engineer sabotage scenario

- **Org type:** neobank, ~600 employees, heavy SharePoint usage as engineering knowledge base + product documentation, M365 E5
- **Trigger:** post-incident review — a departing engineer in the platform team had deleted ~2,000 files across team sites in their last week before notice was accepted. Caught by accident (a colleague noticed missing docs); restored from version history; HR / Legal got involved late. Programme question: "why did we find this by accident?"
- **Scope:** engineering + product team sites (~200 users initially); threshold 200 deletions / 30 min (deliberately tight given low legitimate bulk-delete in engineering content); signal piped to Insider Risk Management (Purview IRM) for correlation with the existing pre-resignation watchlist
- **Outcome:** in year 1, surfaced 3 pre-departure deletion bursts (2 confirmed sabotage-intent, 1 legitimate clean-up of personal scratch space); built the pattern into the offboarding runbook (HR notice → MDA scope-flag → 30-day heightened monitoring window). Cost was real — ~5 FP-related exceptions per quarter from engineers cleaning up old project content

### Use case 3 — Healthcare provider with SharePoint-resident clinical workflows, post-IR build-out

- **Org type:** mid-size healthcare provider, ~8k employees, SharePoint Online holds clinical-protocol documentation + patient-facing material, M365 E5, HIPAA-equivalent + local health-data regulation
- **Trigger:** an internal IR exercise simulated a ransomware variant deleting clinical-protocol content on SharePoint. The blue team had no detection layer between "user opened a ticket about missing files" and "IT looked at the SharePoint recycle bin" — typical lag ~6 hours
- **Scope:** clinical-protocol SharePoint sites + clinical-team OneDrives (~3k users); threshold 300 deletions / 30 min; integrated with backup-restore runbook (Veeam-for-M365 in this case); excluded the records-retention automation account by UPN
- **Outcome:** detection lag dropped from ~6 hours to ~15 minutes (alert-to-triage SLA); used in tabletop reporting to the clinical-governance committee as a measurable control improvement; the policy itself triggered ~2 alerts per month, ~90% legitimate (records-retention job after exclusion lapsed; a clinician cleaning out a defunct sub-site)

### Use case 4 — Professional-services firm, post-OAuth-token-replay (Storm-2372-class) compromise

- **Org type:** mid-size consulting firm, ~5k employees, project-based work in SharePoint, M365 E5
- **Trigger:** a Storm-2372-class device-code-phishing attack landed against a partner-level user; the attacker exfiltrated proposal content via OAuth-app token replay over several days, then deleted ~1,500 files from the partner's OneDrive to cover trail before the IR team identified the compromise. Post-IR retrospective named "no SaaS-delete detection" as a top-3 gap
- **Scope:** all users (~5k); SharePoint + OneDrive; threshold 400 deletions / 30 min; signal correlated with OAuth-grant-additions (per MDA playbook Appendix B.2 KQL) to detect the token-replay → exfil → delete chain
- **Outcome:** in year 1, no comparable incident; the policy fired ~3 times per month, all legitimate (project-end cleanups, departing-staff content removal); the **correlation rule** (mass-delete + recent high-scope OAuth grant + non-typical sign-in IP) is the actual control of value — the standalone policy is the signal, the correlation is the alert that gets escalated

## Implementation pattern

Typical 8-week rollout for a tenant new to mass-delete detection (M365 connector assumed live):

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Inventory legitimate bulk-delete patterns (records-retention automation, archival projects, mailbox-cleanup jobs); document service / migration / archival account UPNs; confirm SharePoint version-history retention setting per site (default 500 versions but tenant policy varies); confirm M365 backup posture (native version history vs Veeam / Rubrik / AvePoint third-party) | Bulk-delete actor inventory; backup-restore runbook reference; exclusion-list draft |
| W2 | Pull 30-day baseline of delete + modify events per user from `CloudAppEvents` (`ActionType in ("FileDeleted", "FileModified")` grouped by `AccountObjectId`, `bin(TimeGenerated, 30m)`); compute P95 + P99 per role family; identify outliers and confirm they are exclusion-list candidates | Baseline statistics; threshold candidate (P99 + 50% headroom typical); refined exclusion list |
| W3 | Configure policy with starting threshold (500 deletions / 30 min as MDA-default; adjust per W2 baseline); Action = Alert + email to SOC; **explicitly NOT auto-suspend** on first turn-on; severity = High; pair with IR runbook for the SaaS-ransomware play | Policy live in alert mode; IR-runbook reference linked |
| W4 | SOC triage of week-1 alerts; classify TP / FP / business-as-usual; surface any service / archival accounts missed in W1 inventory | First-tuning-pass FP classifier; updated exclusion list |
| W5 | Refine threshold based on W4 evidence; document the standard SOC response (verify governance-action context, check version-history availability, confirm or escalate within 30 min); add the OAuth-correlation KQL in Sentinel as a higher-fidelity rule (mass-delete + recent high-scope OAuth grant + atypical sign-in IP) | FP rate target <15%; Sentinel correlation rule live |
| W6 | Tabletop the SaaS-ransomware scenario end-to-end: alert fires → SOC triages → version-history-restore initiated → business owner notified → containment (suspend user, revoke tokens, isolate device) → recovery → post-incident review | Tabletop output; runbook updates; documented RTO/RPO for the SaaS-resident-content case |
| W7 | Extend scope if appropriate (additional SaaS connectors with delete-event visibility — Google Workspace, Box where licensed); document the policy as production-ready; quarterly tuning cadence agreed | Steady-state operations handoff |
| W8 | Onboard the alert into the broader Insider Risk Management correlation (per use case 2 / 4); confirm the policy feeds the quarterly insider-risk metric pack | Integration sign-off; metric pack updated |
| Q+1 | Quarterly: re-baseline (legitimate bulk-delete patterns shift as the org evolves); re-confirm version-history retention; refresh exclusion list against current service-account inventory | Quarterly tuning record (auditable artefact) |

The W6 tabletop is the make-or-break gate. A mass-delete alert without a confirmed-working restore path is theatre.

## Action

- Primary: **alert + manual governance review** (do NOT auto-suspend on first turn-on)
- Escalation: **auto-suspend Entra user** only on corroborated signal classes (mass-delete AND Entra ID Protection sign-in risk High AND not on exclusion list) AND with human approval. **Architectural attribution:** The suspend decision is owned by Entra (Identity Zero Trust pillar, Objective V "Other security controls integrated") via ID Protection user-risk boost + Conditional Access enforcement; MDA provides the activity signal. The architectural chain is: MDA alert → Entra ID Protection user-risk boost → Conditional Access policy enforces session controls / sign-in block. Source: [Microsoft Learn — Identity Zero Trust deployment objectives](https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity).
- Compensating: SharePoint version history retains pre-deletion content during the retention period; mailbox-recoverable-items retains deleted-mailbox-item recovery during the configured period; third-party M365 backup (Veeam / Rubrik / AvePoint) for longer-window recovery

## Scope

- **Users:** all (exclude documented migration / archival / records-retention-automation accounts by UPN regex)
- **Apps:** SharePoint, OneDrive (M365 connector); extend to other connected apps with delete-event visibility (Google Drive, Box where licensed)
- **Device posture:** any
- **Network position:** any
- **Traffic:** delete + modify events (the modify case catches in-place-overwrite ransomware variants)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → Policies → Policy management → Threat detections → Create Activity policy | Activity type = File deleted OR File modified; Apps = SharePoint + OneDrive; Match parameters = Repeated activity, count > 500 in 30 minutes by same user, single app (starting heuristic — derive P95/P99 from 30-day baseline); Action = Alert + manual governance review; **do NOT enable auto-suspend on first turn-on**; Exclude service / migration / archival / retention-automation accounts by UPN regex | API-mode; M365 near-real-time per Microsoft (no published SLA); some bulk operations arrive as single events with item-count, not per-item enumeration — the threshold can under-count for bulk-API-driven deletes | Legitimate bulk-delete events (SharePoint site archival, content migrations, mailbox cleanup, records-retention automation) are the dominant FP class — must be excluded by UPN. Slow-and-low destruction (100 files/hour over days) defeats the threshold; pair with weekly-baseline Sentinel KQL. Ransomware-in-place that overwrites (not delete then create) only fires on the `FileModified` arm; some variants use copy-overwrite-delete patterns that arrive as multiple event classes — confirm event-class fidelity per variant. Auto-suspend has the same hybrid-AD `onPremisesSyncEnabled=true` silent-fail mode documented in Policy 3. **Activity-policy auto-disable thresholds** of approximately 200,000 events / day or 100,000 events / 3-hour window are practitioner-observed; could not be located in Microsoft Learn at the date of QA — `[VERIFY against the current activity-policies Microsoft Learn page]`. **SCIM cascade** for cross-SaaS suspend propagation is approximately 40 min in practitioner observation; not Microsoft-documented — `[VERIFY against current SCIM provisioning docs]` |
| Netskope | `[unverified]` — Netskope UEBA delete anomaly | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security activity policies | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB anomaly | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant after W2 baseline derivation and W4 tuning pass:

```yaml
policy:
  name: "Mass-delete anomaly — tenant-wide, alert-only"
  type: ActivityPolicy
  activity_filters:
    activity_type:
      - FileDeleted
      - FileModified           # catches in-place overwrite ransomware variants
    apps:
      - SharePoint Online
      - OneDrive for Business
  match_parameters:
    type: RepeatedActivity
    count_threshold: 600       # derived from W2 baseline: P99 + 50% headroom
    time_window_minutes: 30
    scope: SingleUserSingleApp
  user_filter:
    exclude_groups:
      - "Service Accounts"
      - "Migration Accounts"
      - "Records Retention Automation"
      - "DR Test Accounts"
    exclude_upn_regex:
      - "^svc-.*@example\\.com$"
      - "^app-.*@example\\.com$"
      - "^retention-bot@example\\.com$"
      - "^migration-.*@example\\.com$"
  governance:
    action: Alert
    auto_suspend: false          # NEVER true on first turn-on
    notify_user: false           # do not tip off actor in insider-sabotage scenarios
  alerts:
    severity: High
    email_recipients:
      - soc-l1@example.com
      - ir-on-call@example.com   # IR co-owner for the SaaS-ransomware play
    sentinel_forward: true
    power_automate_flow: "soc-ticket-create + ir-runbook-link"
    daily_alert_limit: 50
  correlated_detection:           # Sentinel-side, not MDA-native
    correlation_rule: "mass-delete + recent high-scope OAuth grant (Appendix B.2) + atypical sign-in IP"
    elevated_severity: Critical
  retention:
    policy_match_record_days: 90  # steady-state; Sentinel forward for BFSI 7y retention
  recovery_dependencies:           # documented for auditor + IR
    sharepoint_version_history_versions: 500
    onedrive_recycle_bin_days: 93
    third_party_backup: "Veeam-for-M365, 7y retention"
```

The `count_threshold: 600` is illustrative — it must be derived from the W2 baseline, not used as a literal. A tenant where archival-job UPNs cannot be cleanly excluded will run with a higher threshold and tighter SOC triage; a low-archival-activity tenant (engineering-heavy, low-records-management) will run with a much tighter threshold and accept more triage work.

## Variants

### Industry-specific

- **BFSI:** lower thresholds typical (300-700 deletions / 30 min) because legitimate-bulk-delete is rare outside records-retention automation; tight IR integration with the SaaS-ransomware play; explicit RTO commitment for SharePoint-resident regulatory documentation. Third-party-concentration concern (per playbook) — the destruction path and the detection path both live in the same Microsoft stack. **Attribution note:** "concentration risk" is BNM RMiT third-party-concentration and DORA Art. 28-30 supervisory framing — not Microsoft's own positioning; Microsoft frames the same integration as "unified XDR / end-to-end security" per the [Microsoft Cybersecurity Reference Architecture (April 2025 refresh)](https://learn.microsoft.com/en-us/security/cybersecurity-reference-architecture/mcra). Material on regulatory clauses is illustrative — not regulatory advice
- **Healthcare:** similar to BFSI on thresholds but the FP class shifts — clinical-records-retention automation, EHR-integration cleanup, patient-record-merging workflows produce legitimate bulk-deletes. Recovery RTO often tighter (clinical-workflow dependency). HIPAA-equivalent audit cycle expects deletion-event audit trail completeness
- **Tech:** higher legitimate-bulk-delete rates (engineering scratch cleanup, sprint-end project tidy-up, deprecated-codebase removal); per-team thresholds rather than tenant-wide; sabotage-by-departing-engineer is the dominant TP class (per use case 2). Lower priority on ransomware-in-SaaS unless the tenant is a high-target vendor
- **Retail:** loyalty-data and transaction-history sit elsewhere (not in SharePoint) typically; SharePoint mass-delete tends to be marketing / merchandising / store-ops content. Lower regulatory pressure; the policy is often deferred or scoped narrowly to corporate-HQ users
- **Public sector / Legal:** records-management discipline produces both more legitimate bulk-deletes (statutory disposal schedules) and tighter audit expectations on the deletion trail. The mass-delete policy is usually paired with an explicit Records-Disposal-Authorised user group; alerts on disposal-group activity are routed to records management, not SOC

### Maturity-based

- **Immature:** one tenant-wide threshold left at MDA default (500 / 30 min); no W2 baseline derivation; thin exclusion list (just "Service Accounts" group); FP rate >40%; alerts ignored by SOC by month 3 because the dominant signal is the records-retention bot. No tabletop of the SaaS-ransomware scenario. Common at 6-12 months post-deployment for under-resourced programmes
- **Mature:** baseline-derived threshold; documented exclusion list refreshed quarterly; FP rate <15%; SOC has a documented triage runbook including the version-history-restore step; IR runbook for SaaS-ransomware tabletop-exercised at least annually; quarterly tuning cycle running; metric on alert → triage → close SLA reported into operational risk forum
- **Advanced:** per-role-family or per-user-baseline thresholds; the mass-delete signal is correlated in Sentinel with OAuth-grant additions + sign-in-risk + endpoint signals before triggering the high-severity path; auto-suspend enabled only on the corroborated path with documented human-approval gate; signal feeds Purview IRM Adaptive Protection (per Policy 12 in the playbook) raising DLP strictness on the implicated user automatically — Adaptive Protection now wires into DLP, Data Lifecycle Management (DLM), and Conditional Access, not just CA. Cite [Microsoft Learn — Adaptive Protection](https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection). Restore-path RTO measured quarterly via game-day-style validation

## Control mappings

- **CIS Microsoft 365 Foundations Benchmark v5.0.0** (30 April 2025): **CIS 2.4.3 (L2)** "Ensure Microsoft Defender for Cloud Apps is enabled and configured" — umbrella; **CIS 3.1.1 (L1)** "Ensure 'Audit Log Search' is Enabled" — feeds CloudAppEvents / activity log dependency; **CIS 6.2.1 (L1)** mailbox / Exchange auditing as compensating-evidence anchor for the mailbox-delete arm. `[VERIFY exact CIS 6.2.1 wording against current benchmark]`. Source: CIS Microsoft 365 Foundations Benchmark v5.0.0
- **Microsoft Secure Score** (Apps group — current four-group structure: Identity / Device / Apps / Data): improvement actions in the Apps group that depend on MDA being enabled and configured (notably "Detect anomalous user behaviour" / activity-policy actions). Sources: [Microsoft Learn — Microsoft Secure Score](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score); [Microsoft Learn — Microsoft Secure Score improvement actions](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions)
- BNM RMiT clause(s): [BNM RMiT cyber-resilience + incident management + business continuity](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition — particularly the SaaS / outsourcing-of-IT clauses and the cyber-resilience expectations on detection of destructive events]`. Material on regulatory clauses is illustrative — not regulatory advice
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`; A.12.3 backup; A.16.1 incident management
- NIST CSF 2.0 subcategory(ies): `DE.CM-03` (personnel activity monitoring), `DE.AE-02` (analyse event data), `RC.RP-01` (recovery plan executed), `RC.CO-03` (recovery activities communicated) `[VERIFY against CSF 2.0 final]`
- ISO 22301 — BCM / RTO mapping to the SaaS-resident-content recovery commitment

## False-positive risk

- SharePoint site archival projects (planned content removal at scale)
- Mailbox cleanup workflows (executive-assistant clearing legacy content)
- Records-retention automation that schedules large deletes per statutory disposal schedule
- Backup → delete → restore test cycles (DR rehearsals)
- Content-migration projects (move content to new tenant, delete from old — the delete side fires)
- M&A-driven content consolidation (acquiring tenant absorbs target, target-side mass-deletes)
- Project-end cleanup by individual users (especially in tech / consulting verticals)
- SharePoint-list bulk operations that fire as many delete events from a single user action

## Real-world FP experience

Typical FP-rate trajectory (**practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.**):

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 60-80% | Records-retention automation under un-excluded service account; archival jobs; mailbox-cleanup workflows; DR rehearsals |
| W4 | 25-40% | After first exclusion-list pass — service / retention / migration accounts identified; archival projects scoped |
| W8 | 10-20% | After W2 baseline-driven threshold + per-role tuning + project-closure exception process |
| W12 | 5-15% | Documented exclusion list mature; quarterly tuning cycle running; OAuth-correlation rule absorbs ambiguous-intent alerts |
| Steady-state | 3-8% | Per-user-baseline approach (if implemented) brings this further down; remaining FPs are mostly project-end cleanup bursts and ad-hoc archival |

Named FP scenarios encountered repeatedly in production deployments:

| Scenario | Mitigation |
|---|---|
| Records-retention automation account fires bulk-delete on statutory disposal schedule | Explicit UPN regex exclusion; document the disposal-policy ownership; quarterly review |
| SharePoint site-archival project (planned migration of site to read-only / cold storage) | Time-bounded exception window; named owner; signed-off scope document |
| Departing employee's manager cleaning out the employee's content as part of offboarding | Document the offboarding-cleanup actor; consider routing the alert to HR rather than SOC for this case |
| DR rehearsal day involving large bulk-delete by a test account | Pre-rehearsal coordination with SOC; temporary exception window |
| Content-migration job (e.g. tenant-to-tenant move, M&A integration) | Documented migration-account UPN list; named owner; time-bounded |
| Power Automate flow under a user account performing scheduled cleanup | Convert to service-account execution; add to exclusion list; OR allowlist the specific automation |
| Single user-action that fires many delete events (SharePoint list bulk-edit, recycle-bin empty) | Triage runbook step: check whether the action is a single API call with item-count vs per-item enumeration |
| Mass-delete by a developer in their own OneDrive (engineering scratch cleanup) | Per-role-family thresholds; OR per-user baseline approach; OR explicit engineering-team allowlist |
| Records disposal-authorised user (public sector / legal pattern) | Route alerts on this user group to records management, not SOC |

## Operational cost

- **Exception-handling load:** medium during W4-W12 ramp (5-10 exceptions per week typical); low steady-state (1-3 per week) once the exclusion list is mature
- **Triage load:** low to medium ongoing — every alert generates a SOC ticket with the "is recovery needed?" decision; typical 3-10 alerts per week per 5k users at FP rate 10%
- **End-user friction:** low if alert-only; very high if auto-suspend on a legitimate cleanup (the user is locked out plus their cleanup remains incomplete plus the alert escalates HR / Legal)

Typical staffing: 0.1 FTE platform admin (policy tuning + quarterly review); 0.1-0.3 FTE SOC triage time depending on tenant size + FP rate. IR programme commits time proportional to TP rate — usually negligible in BFSI tenants until a real event lands.

## Privacy / data-protection considerations

- Delete-event audit-log = workforce-action data; document scope under PDPA / GDPR Art. 88 / equivalent workforce-monitoring regime
- Auto-suspend on mass-delete is a high-impact governance action — document authorisation chain (typically SOC L2 → Identity admin → HR notification within 4 hours); the same hybrid-AD silent-fail mode as Policy 3 applies and must be tested for. **Architectural reminder:** the suspend decision is owned by Entra (Identity ZT pillar Objective V) — MDA only provides the activity signal
- DPIA scope: activity-log retention + correlation with HR data (where IRM integration is enabled) + restore-action audit trail
- Where the deleted content includes regulated data (PCI cardholder data, PHI, PII) the alert evidence-record itself may carry sensitive metadata (file paths, file names); apply the same access-control discipline as for DLP-evidence records

## Integration with broader programmes

- **Incident response runbook:** the mass-delete alert is the primary trigger for the SaaS-ransomware play. Runbook sequence: alert fires → SOC triage (15-min SLA typical) → confirm or escalate → IR engaged → containment (revoke tokens, suspend user with the hybrid-AD caveat and the Entra-owned-decision attribution, isolate device) → recovery (version-history restore for in-scope files, backup restore for older windows) → post-incident review. Tabletop annually at minimum
- **Business continuity / disaster recovery:** the policy is the detection leg of a BC capability for SaaS-resident regulatory and operational documentation; pair with a documented RTO/RPO for the SaaS content class; the restore path (SharePoint version history → recycle bin → third-party backup → manual reconstruction) must be game-day-tested
- **Insider Risk Management:** signal feeds Purview IRM correlation with HR signals (departure notice, performance flags), endpoint signals, anomalous-email-forward, OAuth-grant additions. Standalone the mass-delete signal has low TP rate; correlated it is materially higher (the named-actor / departing-engineer playbooks tend to chain multiple signals). Adaptive Protection then wires the resulting risk level back into DLP, DLM, and Conditional Access. Cite [Microsoft Learn — Adaptive Protection](https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection)
- **Audit cycle:** quarterly evidence pack — policy version, exclusion list, alert volume, TP/FP breakdown, escalation outcomes. Feeds the annual control-effectiveness review under SOX / NIS2 / BNM RMiT cyber-resilience expectations `[VERIFY against current edition]`. Material on regulatory clauses is illustrative — not regulatory advice. The "did your detection layer catch the tabletop scenario within RTO?" question is the auditable artefact
- **Board reporting:** quarterly metric — confirmed destructive events per quarter (typically zero in BFSI); time-to-detect for the SaaS-ransomware scenario per the most recent tabletop; trend in alert volume + FP rate
- **Vendor risk / third-party concentration:** the SaaS-ransomware exposure has a third-party-concentration dimension — the destruction path and the recovery path both live in the same Microsoft stack absent third-party backup. Document the third-party-backup decision (Veeam / Rubrik / AvePoint or accept the version-history floor) as a control-design risk acceptance with named owner. **Attribution note:** "concentration risk" is BNM RMiT / DORA Art. 28-30 / MAS TRM supervisory framing — Microsoft's own positioning is "unified XDR / end-to-end security" per the [Microsoft Cybersecurity Reference Architecture (April 2025 refresh)](https://learn.microsoft.com/en-us/security/cybersecurity-reference-architecture/mcra). Material on regulatory clauses is illustrative — not regulatory advice

## Anti-patterns specific to this policy

1. **"Enable auto-suspend on day 1"** — destroys a hybrid-AD user account silently (the same writeback issue as Policy 3); breaks SCIM-cascaded apps; legitimate-but-misclassified user has their cleanup half-completed plus their account locked. The records-retention bot under a personal UPN fires it on day one of go-live
2. **"Set the threshold at the MDA default of 500 without a baseline"** — guarantees alert flood on first records-retention run; SOC ignores the policy by month 2; the real destructive event lands in a sea of bot noise
3. **"No exclusion list"** — the records-retention automation account, the migration accounts, the DR-test accounts all fire the threshold; trust in the policy collapses; the SOC documents a "we ignore mass-delete alerts" exception
4. **"Treat the alert as the control"** — a mass-delete alert with no confirmed-working restore path is theatre. If version history is disabled site-wide (or set below the recovery RTO window) the alert tells you something irreversible happened, with no action available
5. **"Alert-only on the `FileDeleted` arm only, omit `FileModified`"** — misses in-place overwrite ransomware variants entirely. The `FileModified` arm is essential for the T1486 SaaS variant
6. **"Notify the user when alerted"** — tips off the actor in the insider-sabotage case; the actor accelerates the remaining destruction
7. **"Treat mass-delete and mass-download as the same triage"** — different threat models; different IR plays; different recovery paths. Mass-delete needs the restore-path question first; mass-download needs the exfiltrated-content question first. Co-locating them in one runbook causes triage drift
8. **"Skip the W2 baseline because 'we don't have many service accounts'"** — every tenant has more bulk-delete actors than the security team initially documents; the baseline pull is the cheap step that prevents months of FP-tuning
9. **"No tabletop of the SaaS-ransomware scenario before go-live"** — the runbook gaps surface during the first real incident, when the cost is real. Tabletop the alert → triage → restore → post-incident sequence at least once before production hand-off
10. **"Treat the M365 native version history as a sufficient backup"** — site-collection admin can disable version history; ransomware actors with admin-level access can drain version history; the 500-versions-default is a per-file ceiling not a tenant ceiling. Either accept the residual risk explicitly (documented risk acceptance) or invest in third-party backup; do not leave the question implicit

## Coverage gaps

- Slow-and-low destruction over days/weeks — sub-threshold per window. Pair with weekly-baseline Sentinel KQL (extend the [MDA playbook Appendix B.1](../../04-vendors/microsoft-defender-for-cloud-apps.md) pattern from download to delete events)
- Attacker pre-compromises an excluded account (service account, migration account, records-retention automation) — the exclusion list is the attack surface. Periodic credential rotation + workload-identity Conditional Access required
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — detection post-event; pair with version-history + recycle-bin + third-party backup for recovery; the alert tells you something happened, not that you stopped it
- Ransomware that overwrites files in place via the M365 sync client (T1486 via OneDrive sync) — produces sync events, not necessarily file-modified events at the connector level; verify event-class fidelity per variant. Independent endpoint-layer detection (Defender for Endpoint ransomware behavioural detection) is the compensating control
- Bulk-API delete (single API call deleting many items) may arrive as a single event with item-count rather than per-item enumeration — the threshold can under-count; verify per-app event-class behaviour
- Site-collection-admin actions that delete entire sites — fire as different event class than file-level deletes; separate detection layer required
- **Third-party-concentration / service-side incident gap:** the detection control and the destruction surface both live in Microsoft. A Microsoft service-side incident (the specific July 2023 Storm-0558 token-forgery class, or the Midnight Blizzard credential-exposure class affecting Microsoft corporate systems in 2024) leaves no in-tenant detection signal; the compensating control is an independent SIEM with raw log forwarding (per the playbook concentration-risk section). **Attribution note:** "concentration risk" is BNM RMiT / DORA / MAS TRM supervisory framing — not Microsoft's positioning. The threat-actor cluster originally tracked as Storm-0558 has been promoted to **Antique Typhoon** under Microsoft's current threat-actor taxonomy; Storm-0558 is retained here as the specific reference to the July 2023 Microsoft Exchange Online token-forgery incident. **Midnight Blizzard** is Title Case per Microsoft's April 2023 naming refresh (not "MIDNIGHT BLIZZARD"). Sources: [Microsoft Learn — Microsoft threat actor naming taxonomy](https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming); [Microsoft Security Response Center — Storm-0558 token-forgery analysis (July 2023)](https://msrc.microsoft.com/blog/2023/07/microsoft-mitigates-china-based-threat-actor-storm-0558-targeting-of-customer-email/) `[VERIFY URL]`; [Microsoft Security Blog — Midnight Blizzard activity on Microsoft corporate systems (2024)](https://www.microsoft.com/en-us/security/blog/2024/01/19/midnight-blizzard-guidance-for-responders-on-nation-state-attack/) `[VERIFY URL]`
- Personal-device + personal-account scenario — not in scope of the M365 connector; cannot detect deletion of content held in personal-OneDrive or personal-Google-Drive accounts

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
