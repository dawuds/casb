# Mass-download alert

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive** (exemplar of the enhanced policy-file template).
> Required capabilities: [Anomalous-activity / UEBA detection + Compromised-account detection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 3](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Mass-download anomaly with alert).

## Purpose

Detect per-user volumetric mass-download patterns against sanctioned SaaS storage; alert SOC for triage. Catches insider exfiltration via bulk download (pre-resignation patterns) and compromised-account exfiltration after credential theft. Counters MITRE ATT&CK `T1530 Data from Cloud Storage Object`, `T1213.002 SharePoint`, `T1567.002 Exfiltration to Cloud Storage` — **Detect leg only, post-event**.

## What organisations use this for

Most organisations deploy this as their **first Day-1 insider-risk detection** because it requires no CAAC, no agent, no policy infrastructure beyond an API connector to M365. It's the easiest insider-risk signal to stand up, which means it's typically the first one — and the FP rate management is what determines whether the SOC keeps paying attention to it after the first 90 days.

The hardest single decision: the threshold value. Wrong threshold = alert fatigue or missed events. Almost every deployment tunes this multiple times in year one.

### Use case 1 — Wealth-management firm, pre-departure RM exfiltration

- **Org type:** mid-size wealth management firm, ~2k employees, relationship managers handle high-value client portfolios with deep documentation in SharePoint; M365 E5
- **Trigger:** internal compliance flag — outgoing RM had been observed copying client files to OneDrive personal folders in the weeks before resignation. Historical incidents had shown the same pattern
- **Scope:** all client-facing roles (~600 users); SharePoint + OneDrive; threshold tuned aggressively (1,000 files/30 min) given low legitimate-bulk-download rate
- **Outcome:** in year 1, caught 3 of 4 confirmed pre-departure exfiltration attempts; the 4th was slow-and-low (50 files/day over 8 weeks) and bypassed the threshold; pair-policy with weekly-baseline KQL added in year 2 to close that gap

### Use case 2 — Pharma R&D, high legitimate-bulk-download environment

- **Org type:** mid-cap pharma R&D, ~5k employees, M365 E5, heavy use of SharePoint as research-dataset repository
- **Trigger:** compliance ask — auditor questioned why no bulk-download monitoring existed on the R&D-data SharePoint sites
- **Scope:** R&D-restricted SharePoint sites only (not OneDrive); threshold tuned permissively (10,000 files/30 min) because researchers legitimately download dataset packs of that volume
- **Outcome:** threshold so high that real exfil signals were drowned; pivoted in year 2 to a per-user-baseline approach (each researcher's threshold derived from their 90-day P95) rather than tenant-wide fixed threshold; FP rate dropped from ~30% to ~5%

### Use case 3 — Consulting firm, "project-end download everything" pattern

- **Org type:** professional-services firm, ~10k employees, project-based work in SharePoint sites, M365 E5
- **Trigger:** new programme — insider risk identified as a category-1 risk; mass-download alert was the first deployable signal
- **Scope:** all consultants (~8k users); SharePoint + OneDrive; baseline threshold 5,000 files/30 min
- **Outcome:** ~60% of alerts in year 1 were project-end legitimate "I'm wrapping up, let me grab everything I worked on" patterns; ~20% were resignation-related (some legitimate, some flagged for HR review); ~20% were false-positive ID-format-driven; documented exception path for project closure became standard practice

### Use case 4 — Regulated FI integrating CASB into existing insider-risk programme

- **Org type:** tier-1 ASEAN bank with existing Purview IRM deployment + UEBA in SIEM; adding MDA mass-download as an additional signal
- **Trigger:** building out the Insider Risk Management programme per BNM expectations; CASB signal joins the existing endpoint + email + identity signals
- **Scope:** all employees (~30k); SharePoint + OneDrive; CASB signal feeds Purview IRM Indicators (per MDA Policy 12 — IRM signal-boost integration)
- **Outcome:** mass-download alert TP rate measured at ~3% standalone; ~12% when correlated with other IRM signals (HR ticket, endpoint exfil-pattern detection, anomalous-email-forward, etc.); CASB signal alone insufficient — correlation is the value

## Implementation pattern

Typical 8-week rollout for a tenant new to CASB activity policies:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Document service-account UPN regex; document migration-account exclusion list; define SOC alert routing | Exclusion list signed off |
| W2 | Pull 30-day baseline of download events per user from CloudAppEvents; compute P95 + P99 per role family | Baseline statistics; threshold candidate |
| W3-W4 | Configure policy with starting threshold (5,000 files / 30 min default; adjust if baseline P99 is higher); Action = Alert + email to SOC | Policy live in alert mode |
| W5 | SOC triage of week-1 alerts; classify TP / FP; identify FP patterns | First-tuning-pass classifier of FP causes |
| W6-W7 | Refine threshold; add service-account exclusions surfaced from W5; tune SOC triage runbook | FP rate <15% target |
| W8 | Document the policy as production-ready; define quarterly review cadence | Steady-state ops handoff |
| W9+ | Quarterly threshold tuning + service-account-list update + correlation with other UEBA signals | Quarterly metric on TP / FP rate |

The W2 baseline pull is the critical step. Without it, the threshold is a guess. With it, the threshold is data-driven.

## Action

- Primary: **alert + email to SOC** (do NOT auto-suspend on a single signal)
- Escalation: **auto-suspend Entra user** only after corroborated signals (Entra ID Protection sign-in risk High AND mass-download AND not on documented exclusion list) AND human approval

## Scope

- **Users:** all (exclude documented service accounts and migration accounts by UPN regex)
- **Apps:** SharePoint, OneDrive (M365 connector); extend to other connected apps where bulk download is plausible
- **Device posture:** any
- **Network position:** any
- **Traffic:** download events (file-by-file enumeration)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Create Activity policy | Activity type = Download file; Apps = SharePoint + OneDrive (+ extended); Match parameters = Repeated activity, count > 5,000 in 30 minutes by same user, single app (starting threshold — derive P95 from 30-day baseline); Action = Alert + email to SOC; **do NOT enable auto-suspend on first turn-on**; Exclude service accounts and known migration accounts by UPN regex | API-mode; near-real-time per Microsoft (no published SLA); some downloads via sync clients arrive as different event class | "Suspend user" governance disables the Entra user object — NOT a soft suspension. SCIM cascades into Entra-provisioned apps (~40 min default); Exchange mailflow breaks if user is a shared-mailbox delegate. **Silently fails on hybrid-AD users with `onPremisesSyncEnabled=true` and no writeback** — attacker keeps the session. Activity-policy auto-disable at 200k/day or 100k/3h — silent saturation looks like normal alert decline. Slow-and-low (100 files/day × 30 days) defeats sub-day threshold entirely |
| Netskope | `[unverified]` — Netskope CASB API with native UEBA | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security activity policies | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB anomaly detection | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security API | | | |

## Worked configuration example (tier-2 BFSI baseline)

```yaml
policy:
  name: "Mass-download anomaly — tenant-wide, alert-only"
  type: ActivityPolicy
  activity_filters:
    activity_type: FileDownloaded
    apps:
      - SharePoint Online
      - OneDrive for Business
  match_parameters:
    type: RepeatedActivity
    count_threshold: 5000           # tune per W2 baseline
    time_window_minutes: 30
    scope: SingleUserSingleApp
  user_filter:
    exclude_groups:
      - "Service Accounts"
      - "Migration Accounts"
      - "DR Test Accounts"
    exclude_upn_regex:
      - "^svc-.*@example\\.com$"
      - "^app-.*@example\\.com$"
  governance:
    action: Alert
    auto_suspend: false             # NEVER true on first turn-on
    notify_user: false              # do not tip off the actor
  alerts:
    severity: High
    email_recipients: [soc-l1@example.com]
    sentinel_forward: true
    power_automate_flow: "soc-ticket-create"
    daily_alert_limit: 100
  retention:
    policy_match_record_days: 90    # steady-state; Sentinel for longer
```

The `count_threshold: 5000` value should be **derived from the W2 baseline**, not used as a literal default. P95 of legitimate-bulk-download users + 50% headroom is a sensible starting point.

## Variants

### Industry-specific

- **BFSI:** lower thresholds typical (1,000-5,000 files/30 min) because legitimate-bulk-download is rare; auto-suspend on corroborated signal more common; integration with HR pre-resignation watch-list often a separate document
- **Pharma / R&D:** higher thresholds (10,000+) because dataset-download is a normal pattern; per-user-baseline approach often required; researcher-allowlist for known legitimate bulk-download patterns
- **Tech / SaaS:** content-migration patterns dominate FPs; project-end "grab everything" pattern; threshold-and-correlation matters more than threshold alone
- **Consulting / professional services:** project-end download patterns; per-engagement-end exception process; ~60% of year-1 alerts are legitimate project closure
- **Legal:** eDiscovery + matter-management workflows produce legitimate-bulk-download events; matter-coordinator UPN allowlist

### Maturity-based

- **Immature:** one tenant-wide threshold; no baseline; no exclusion list; FP rate >40%; alerts ignored by SOC by month 3
- **Mature:** per-role-family thresholds (or per-user baseline); documented exclusion list; quarterly tuning cadence; FP rate <15%; SOC has a documented triage runbook
- **Advanced:** per-user behavioural baseline (each user's P95 of last 90 days drives threshold); CASB signal correlated with Entra ID Protection sign-in risk + endpoint exfil-detection (Purview Endpoint DLP) + HR signals (in IRM); auto-suspend only on corroborated signal classes; FP rate measured monthly with named scenarios

## Control mappings

- BNM RMiT clause(s): [BNM RMiT cybersecurity operations + logging](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `DE.CM-03` (personnel activity monitoring), `DE.AE-02` (analyse event data), `RS.AN-01` (notifications investigated) `[VERIFY]`
- Insider Threat Programme cross-references where one exists

## False-positive risk

- Legitimate bulk downloads — DR rehearsals, content migrations, BI export jobs by service accounts. Exclude service accounts by UPN
- Legal eDiscovery teams, BCM rehearsals, content migrators — stratify thresholds by role
- New-hire onboarding downloads (template packs) — may breach threshold legitimately
- Project closure / handover periods — name an exception group during the project-end window
- Researchers / analysts downloading reference datasets — pair-policy with per-user-baseline

## Real-world FP experience

Typical FP-rate trajectory:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 50-70% | Service accounts hitting threshold; legitimate content migrations; researcher dataset access |
| W4 | 25-40% | After service-account exclusion list + first migration-account additions |
| W8 | 12-20% | After per-role threshold tiering + project-closure exception process |
| W12 | 5-15% | Documented exclusion list mature; quarterly tuning cycle running |
| Steady-state | 3-8% | Per-user-baseline approach (if implemented) brings this further down |

Named FP scenarios encountered repeatedly:

| Scenario | Mitigation |
|---|---|
| Service-account UPN matching a legitimate user pattern | Explicit UPN regex exclusion; quarterly service-account inventory |
| DR rehearsal day — large bulk-download by a test account | Pre-rehearsal coordination; temporary exception window |
| Content-migration projects (e.g. moving from on-prem file share to SharePoint) | Documented migration-account UPN list; named owner; time-bounded |
| Researcher legitimately downloading a dataset for analysis | Per-user-baseline approach; or named researcher allowlist |
| eDiscovery / legal-matter coordinator pulling many documents | Allowlist UPN; matter-management workflow integration |
| New-hire onboarding (template pack, training materials) | First-30-days higher threshold OR named onboarding-window allowlist |
| Project-end / engagement-closure download burst | Project-management workflow integration; documented exception window |
| Power BI export job under a service account | Service-account exclusion; alternatively, monitor as a separate policy |

## Operational cost

- **Exception-handling load:** medium during W4-W12 ramp (10-20 exceptions per week typical); low steady-state (2-5 per week)
- **Triage load:** medium ongoing — every alert generates a SOC ticket; typical 5-15 alerts per week per 1k users at FP rate 10%
- **End-user friction:** low if alert-only; high if auto-suspend (broken account, urgent recovery)

Typical staffing: 0.1 FTE platform admin (policy tuning + quarterly review); 0.2-0.5 FTE SOC triage time depending on tenant size + FP rate. Insider-risk programme analyst typically owns the corroboration step before any auto-suspend or HR escalation.

## Privacy / data-protection considerations

- Activity-log records contain user identity + file metadata + timestamps — workforce-monitoring data, document under PDPA / GDPR Art. 88 / equivalent
- Auto-suspend governance action is high-impact; documented authorisation chain required (typically: SOC L2 → Identity admin → HR notification within 4 hours)
- DPIA scope: activity-log retention + matching-content-snippet retention + cross-correlation with HR data (if integrated with IRM)
- For PDPA MY 2024: workforce notice in employee handbook + AUP must cover SaaS activity monitoring scope

## Integration with broader programmes

- **Insider Risk Management:** the CASB mass-download signal is one of many — Purview IRM correlates with HR signals (departure notice, performance flags), endpoint exfil-pattern detection, anomalous-email-forward, anomalous-print events. Standalone the signal is weak; correlated it is strong (per Use case 4)
- **HR / Legal escalation:** documented escalation path from SOC L2 to HR / Legal when a corroborated insider-risk pattern emerges; preservation hold (Purview eDiscovery) initiated; investigation under privileged process
- **Annual audit:** activity-policy coverage + measured TP/FP rate + named-incident-investigation evidence feed the annual control-effectiveness review (under SOX / NIS2 / equivalent operational-risk regimes)
- **Board reporting:** quarterly metric — confirmed insider-risk events per quarter; trend; named-incident summary (sanitised)
- **Pre-resignation watchlist:** some BFSI firms maintain an HR-driven pre-resignation watchlist; CASB activity-policy signal integrated as input — flagged users get heightened monitoring during the notice period

## Anti-patterns specific to this policy

1. **"Set the threshold without a baseline"** — guarantees either alert fatigue (too low) or missed events (too high). Run the W2 baseline pull
2. **"Enable auto-suspend on day 1"** — destroys a hybrid-AD user account silently (writeback issue); breaks SCIM-cascaded apps; legitimate-but-misclassified user has their entire work day broken
3. **"Tenant-wide single threshold"** — fundamentally misframes the problem when role-family legitimate-volumes differ by 10×+ (e.g. RM vs researcher)
4. **"No service-account exclusion"** — service accounts will hit the threshold; SOC sees 80% alerts from one UPN; trust in the policy collapses
5. **"Notify the user when alerted"** — tips off the actor; in a real exfil event, the user accelerates the remaining transfer
6. **"Treat all alerts as insider risk"** — most alerts (per Use case 3) are legitimate project-closure or content-migration; insider-risk is a small subset; treat-everything-as-insider-risk causes HR / Legal overload
7. **"Use CASB-only signal for the suspension decision"** — CASB signal is one input; corroborate with sign-in risk + endpoint signal + HR context before suspension
8. **"No quarterly tuning"** — the org's user-population changes; new role families emerge; tenant-wide threshold drifts out of relevance
9. **"Skip the SOC runbook"** — alert arrives, no documented triage flow, analyst escalates to L2, L2 doesn't know either, alert ages out

## Coverage gaps

- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — detection is post-event
- Slow-and-low exfil (100 files/day × 30 days) defeats sub-day threshold — pair with weekly-baseline Sentinel KQL (see MDA Appendix B.1)
- Native-sync staging (T1074.002 via OneDrive sync) produces sync events, not download events — separate detection required
- T1550.001 token replay — attacker on different infrastructure may not trigger the per-user-per-app threshold
- Encrypted-by-user content — see [`../../08-failure-modes/encrypted-upload-bypass.md`](../../08-failure-modes/encrypted-upload-bypass.md)
- Personal-device + personal-account scenario — not in scope of the M365 connector

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
