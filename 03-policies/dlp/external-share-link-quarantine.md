# External share-link quarantine

> Status: v0.0 — MDA column from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [Share-link governance + API-mode DLP](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 7](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Quarantine externally-shared OneDrive files).

## Purpose

Identify files in sanctioned cloud storage that are shared externally and have not been accessed or modified within a defined window; two-stage notify-then-quarantine flow. Catches the sharing-drift class — old external shares accumulate over years, each a small leak. Counters MITRE ATT&CK `T1213 Data from Information Repositories` (long-tail cleanup), and indirectly `T1199 Trusted Relationship` when the original share recipient relationship has ended but access persists.

## What organisations use this for

Sharing drift is the most common dirty-data condition in any multi-year M365 / Workspace tenant. A user shares a contract externally for one deal in 2021; the deal closes; the share link stays. Five years later, dozens of files per active user are shared externally with parties who have no current business relationship. Auditors find this in five minutes with a query; remediation is months of work. This policy is the systematic, ongoing cleanup that prevents the auditor finding from recurring.

The hardest single decision is the **stale window** — what counts as stale (90 days? 180? 1 year?) — and who owns the remediation queue when notifications start firing en masse. The first scan typically surfaces thousands of files for a 1k-user tenant; the team running the policy must size the remediation throughput before flipping to quarantine action.

### Use case 1 — Tier-2 professional services firm, post-engagement-end sprawl cleanup

- **Org type:** management-consulting firm, ~3k consultants, M365 E5, project-based work with frequent external collaboration (clients + sub-contractors + advisors)
- **Trigger:** General Counsel review of "what client data we still hold and who has access" surfaced ~12,000 files in OneDrive shared externally with 8,000+ unique external domains, most over 12 months stale. GC asked for documented quarterly cleanup
- **Scope:** all consultants; OneDrive for Business; staged 180-day window with 14-day owner-notification grace; coaching message names firm's records-retention policy
- **Outcome:** first scan surfaced ~12,000 stale shares; 60% quarantined in stage 1, 25% owner-renewed (legitimate ongoing engagement), 15% surfaced for engagement-team review; subsequent quarters cleared <2,000 each as the steady-state cadence took effect; auditor accepted quarterly scan + remediation log as primary evidence

### Use case 2 — Manufacturing firm with active M&A pipeline, carve-out scenarios

- **Org type:** mid-cap industrial-manufacturing firm, ~8k employees, M365 E5, frequent acquisitions + divestitures
- **Trigger:** M&A divestiture surfaced that ~2,000 files in the divested business unit's OneDrive were shared externally with the carve-out target — exposure between the deal-signing and deal-closing periods. Legal asked for systematic cleanup before next deal cycle
- **Scope:** all divested-BU + due-diligence-team users; OneDrive + SharePoint Online sites tagged as carve-out scope; aggressive 90-day window for divested scopes; standard 180-day for ongoing operations
- **Outcome:** divestiture-period exposure cleared in 8 weeks; baseline policy implemented across rest of org; M&A integration playbook updated to include external-share scan as standard pre-close hygiene; ~30% reduction in pre-close external-share exposure cycle-over-cycle

### Use case 3 — Higher-education institution, federal-grant research data

- **Org type:** large research university, ~25k students + ~5k faculty/staff, M365 EDU, federal-grant-funded research with strict data-handling requirements
- **Trigger:** federal funding agency audit raised concern about research-data sharing with collaborators after grant period ended; ~800 grant projects with ~20,000 files shared with external co-investigators, ~70% of these post-grant-period
- **Scope:** all federally-funded research-data SharePoint sites + grant-PI OneDrive folders; 365-day window (longer than corporate baseline because legitimate research collaboration runs years); coaching message links to research-data-management policy
- **Outcome:** ~14,000 stale shares quarantined over 6 months; grant-PI training + records-retention process updated; federal agency satisfied with quarterly attestation cycle; surfaced ~50 cases where external collaborator had moved to a competing institution while retaining access (high-sensitivity finding)

### Use case 4 — Tier-2 commercial bank, auditor finding on long-running external shares

- **Org type:** tier-2 ASEAN commercial bank, ~6k employees, M365 E5, BNM RMiT supervised
- **Trigger:** annual BNM RMiT compliance audit `[VERIFY against current edition]` flagged "uncontrolled external sharing on customer-facing teams' OneDrive" as a Category 2 finding; remediation required within 90 days
- **Scope:** customer-facing BUs (Retail Banking, Wealth, SME, Corporate) — ~3k users; OneDrive + SharePoint; 180-day window; coaching message references the bank's customer-data-handling policy
- **Outcome:** finding closed within the 90-day remediation window; ~9,000 stale shares quarantined; ongoing quarterly attestation cycle established; subsequent BNM audit cycle cleared the finding without re-occurrence; control became a documented compensating measure for the BU's customer-data-handling baseline

## Implementation pattern

Typical 10-week rollout for a tenant with multi-year accumulated sharing drift:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Discovery query against M365 connector — count of externally-shared files by user, by last-modified band | Baseline magnitude (typically thousands for a 1k-user tenant) |
| W2 | Stale-window decision (90 / 180 / 365 days); folder-allowlist definition (templates, intentionally public content); owner-notification email template review with Legal | Stale window + allowlist signed off |
| W3 | Configure Stage 1 policy = Notify owner; Action = Alert only initially | Notification-only baseline; owner-response rate measured |
| W4-W5 | Pilot Stage 1 with one BU; refine notification message based on owner feedback; document common exception classes | Notification-FP rate baseline |
| W6 | Configure Stage 2 policy = Quarantine (separate file policy, 14 days after Stage 1); coordinate with Legal on Purview eDiscovery legal-hold interaction | Two-stage policy live in alert mode |
| W7 | Flip to enforce-mode for pilot BU; first quarantine actions; help-desk preparation for "where did my file go" tickets | First wave of quarantines with recovery path tested |
| W8-W10 | Phased rollout to remaining BUs | Tenant-wide coverage |
| W11+ | Quarterly steady-state — scan + notify + quarantine + report | Ongoing cadence; auditor-evidence package generated quarterly |

The Legal coordination at W6 is the long pole — Purview eDiscovery legal holds override quarantine actions; the operational ownership of the override needs to be clear before scaling up.

## Action

- Primary (Stage 1): **notify** file owner with 14-day grace period
- Primary (Stage 2, 14 days later): **quarantine** (move to admin-owned location) on no-response

## Scope

- **Users:** all file owners
- **Apps:** OneDrive + SharePoint primary; Google Drive, Box, Dropbox where API connectors and quarantine actions are supported
- **Device posture:** N/A (at-rest scan)
- **Network position:** N/A
- **Traffic:** at-rest

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Information protection → Create File policy (two policies, 14 days apart) | **Stage 1:** App = OneDrive for Business; Access level = External OR Public-with-a-link OR Public; Last modified = >180 days; Governance action = Notify owner; severity = Medium. **Stage 2:** same filters, Governance action = Put file in admin quarantine; severity = High. Exclude folder allowlist (`/Shared Templates`, etc.) | API-mode; SharePoint multi-geo "supported only for OneDrive" per docs; SharePoint events not fully captured in multi-geo tenants | Single-step quarantine breaks links for internal users still referencing the file; owner-notification email often lands in junk. Always run notify-then-quarantine. **A quarantined file under Purview eDiscovery legal hold may be spoliation** — Purview hold wins, but operator may not know about hold at policy-design time. Consult Legal before policy design |
| Netskope | `[unverified]` — Netskope CASB API with share-link governance | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security API | | | |
| Skyhigh | `[unverified]` — multi-mode external-sharing controls | | | |
| Zscaler | `[unverified]` — SaaS Security API external-sharing | | | |

## Worked configuration example (tier-2 BFSI baseline)

```yaml
stage_1_notify:
  name: "External-share staleness — Stage 1 (notify owner)"
  type: FilePolicy
  app_filter:
    - OneDrive for Business
  file_filter:
    access_level: [External, "Public with a link", Public]
    last_modified_days_ago: "> 180"
    parent_folder_exclude:
      - "/Shared Templates/*"
      - "/Sites/*-Public/*"           # known-intentionally-public spaces
      - "/Compliance Archive/*"
  governance:
    action: NotifyOwner
    email_template: "stale-share-notify-v3"
    grace_period_days: 14
  alerts:
    severity: Medium
    email_recipients: [share-cleanup@example.com]

stage_2_quarantine:
  name: "External-share staleness — Stage 2 (quarantine, 14 days post notify)"
  type: FilePolicy
  app_filter:
    - OneDrive for Business
  file_filter:
    access_level: [External, "Public with a link", Public]
    last_modified_days_ago: "> 194"     # 180 + 14 grace
    parent_folder_exclude:
      - "/Shared Templates/*"
      - "/Sites/*-Public/*"
      - "/Compliance Archive/*"
    purview_legal_hold: false           # exclude files under Purview eDiscovery hold
  governance:
    action: PutInAdminQuarantine
    notify_owner: true
  alerts:
    severity: High
    email_recipients: [share-cleanup@example.com, soc-l1@example.com]
```

The `purview_legal_hold: false` filter is critical — without it, the policy can spoliate legal-hold material. Coordinate the rule construction with Legal.

## Variants

### Industry-specific

- **BFSI:** customer-data-handling sensitivity is the driver; stale-window typically aggressive (90-120 days); regulator-driven audit cycle the trigger
- **Healthcare:** PHI external sharing requires BAA — stale-share + missing-BAA = compliance breach; quarantine action high-priority
- **Tech / SaaS:** customer-deal-data sharing during sales cycle creates legitimate long-running external collaborations; stale-window often longer (180-365 days)
- **Higher education / research:** legitimate multi-year external research collaboration — stale-window 365+ days; per-grant scoping
- **Professional services:** project-end is the natural stale-trigger; integration with engagement-closure workflow is the maturity step
- **Manufacturing / industrial:** supplier-collaboration patterns; M&A creates carve-out exposure windows

### Maturity-based

- **Immature:** single-stage quarantine policy; tenant-wide; no folder allowlist; no Legal coordination; broken-link complaints flood help desk; programme rolled back to alert-only
- **Mature:** two-stage notify-then-quarantine with documented 14-day grace; folder allowlist for known-intentionally-public content; Purview eDiscovery legal-hold exclusion; quarterly attestation evidence cycle; owner-notification templates tuned to common scenarios
- **Advanced:** integrated with records-retention workflow (stale-share triggers retention-decision rather than just quarantine); engagement-management-system integration (project closure auto-revokes engagement-specific shares); cross-app share-link governance (OneDrive + SharePoint + Box + Dropbox + Google Drive unified policy)

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage prevention](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring, CLD.8.1.5 removal of assets](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.5 (sub-processor disclosure where shared externally)
- NIST CSF 2.0 subcategory(ies): `PR.DS-05`, `RS.MI-02` `[VERIFY]`
- SOC 2 CC6.1 (logical access) where relevant for service-organisation tenants

## False-positive risk

- Stale-but-intentional shares (contract templates, public reference docs) — exclude via folder allowlist
- Files whose `Last modified` is touched by automated processes (BCM rehearsals, schema-update jobs) — may bypass the 180-day filter inadvertently
- Shares to known-trusted external domains that ought to persist (long-term vendor collaboration spaces) — name-based exception
- Files under Purview eDiscovery legal hold — must be excluded explicitly
- Customer-portal-facing intentional external sharing (e.g. shared workspaces for active customers) — separate folder scope

## Real-world FP experience

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 30-50% | Intentionally-public templates without folder allowlist; active engagement-team shares where notification spooks the team |
| W4 | 15-25% | After folder allowlist for known-public + intentionally-shared content |
| W8 | 8-15% | After owner-feedback-driven notification template refinement |
| W12 | 5-10% | Steady-state — quarterly review catches new mis-classifications |
| Steady-state | 3-7% | Mature exclusion lists |

Named FP scenarios:

| Scenario | Mitigation |
|---|---|
| Sales-engagement file shared with active customer 200 days ago, still in active negotiation | Owner-notify path; renewable exception with date-based expiry |
| Contract template shared widely with all law firm collaborators as standard | Folder allowlist for `/Compliance Archive/Templates/*` |
| Project-deliverable shared at engagement-end, customer still using for reference | 365-day window for sales/engagement folders; integration with engagement-closure workflow |
| Stale share to a partner-tenant that the firm later acquired (now internal) | Periodic cross-tenant federation review; remove federated-internal classification from "external" |
| File touched by an automated process (e.g. eDiscovery search) that resets last-modified | Use last-content-modified vs last-touched timestamp where the connector supports |

## Operational cost

- **Exception-handling load:** high during first 12-week ramp (50-100 exceptions per week typical for 1k-user tenant with multi-year drift); medium steady-state (10-20 per week)
- **Triage load:** low — automated through Stage 1 / 2; only escalations to legal hold review require analyst time
- **End-user friction:** medium initially (notification spike + quarantined-file-recovery requests); low steady-state once ongoing-share-hygiene becomes culturally embedded

Typical staffing: 0.2-0.3 FTE during the 12-week ramp; 0.1 FTE steady-state; ~5-10% incremental help-desk time during high-notify weeks.

## Privacy / data-protection considerations

- Owner-notification emails name files + recipients — file metadata is workforce-monitoring-adjacent data
- Files containing personal data of external recipients (customers, partners, candidates) — quarantine event may itself be a regulated data-handling action
- Purview eDiscovery interaction is the dominant Legal/Privacy coordination point
- Cross-border-transfer implications where external-recipient is in a different jurisdiction — document under PDPA / GDPR transfer regime

## Integration with broader programmes

- **Records-retention policy:** stale-share-quarantine is the operational arm of retention policy for externally-shared content; integrate with retention-decision workflow
- **M&A workflow:** pre-close hygiene scan; carve-out-period exposure tracking; post-close cleanup
- **Engagement-management:** project-closure auto-revoke; engagement-team-end-of-project review
- **Annual audit:** quarterly attestation cycle; remediation-log evidence; auditor sample-pull of quarantine actions
- **Privacy + Legal:** Purview eDiscovery legal-hold coordination; Subject Access Request handling integration
- **Board reporting:** quarterly metric — externally-shared file count by sensitivity (declining trend); stale-share count (declining trend)

## Anti-patterns specific to this policy

1. **"Single-step quarantine without owner notification"** — breaks internal users' links to the file; owner has no warning; help-desk flood + trust crisis
2. **"Tenant-wide single stale window"** — different BUs / data classes have different legitimate share lifecycles; uniform window over-quarantines some, under-covers others
3. **"Skip the Purview legal-hold exclusion"** — quarantine of legal-hold material = potential spoliation = legal exposure
4. **"Owner-notification email lacks recovery path"** — owner notified but doesn't know how to renew the share or retrieve the file; quarantines accumulate
5. **"Run aggressively without folder allowlist"** — intentionally-public content (templates, public docs) gets quarantined; users lose trust in the policy
6. **"Quarantine acts on external-share metadata without examining content sensitivity"** — high-volume + low-discrimination; missed opportunity to prioritise sensitive-content shares
7. **"No quarterly steady-state cadence"** — first cleanup massively reduces backlog, then policy languishes; new sharing drift accumulates between scans
8. **"No integration with engagement-management"** — project-end legitimate shares stay active until policy catches them as stale; better to revoke at engagement-end natively

## Coverage gaps

- [BYOD and unmanaged endpoint coverage gap](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — files shared via personal accounts on personal devices = invisible
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — share was active for the period before detection
- Files publicly shared with 'anyone' may "incorrectly show up as private" per OneDrive/SharePoint connector documentation — known fidelity gap
- External recipients who already downloaded the file before quarantine — no recall mechanism
- SharePoint multi-geo events partially captured

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
