# Mass-delete anomaly

> Status: v0.0 — MDA column from playbook v1 (Policy 11 — Day 30 addition); other vendors `[unverified]`.
> Required capabilities: [Anomalous-activity detection (volumetric)](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 11](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Mass-delete / mass-overwrite anomaly).

## Purpose

Detect high-velocity file deletion or modification events against sanctioned cloud storage — the ransomware-in-SaaS class. Counters MITRE ATT&CK `T1485 Data Destruction` (Detect) and `T1486 Data Encrypted for Impact` (SaaS variant). Storm-0501 / BlackCat-on-SaaS variants delete or overwrite files at scale inside the tenant after compromise.

## Action

- Primary: **alert + manual governance review** (do NOT auto-suspend on first turn-on)
- Compensating: SharePoint version history retains pre-deletion content during retention period

## Scope

- **Users:** all (exclude documented migration / archival accounts by UPN regex)
- **Apps:** SharePoint, OneDrive (M365 connector); extend to other connected apps with delete-event visibility
- **Device posture:** any
- **Network position:** any
- **Traffic:** delete + modify events

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Create Activity policy | Activity type = File deleted OR File modified; Apps = SharePoint + OneDrive; Threshold = starting heuristic 500 deletions in 30 minutes per user (derive from 30-day baseline); Action = Alert + manual governance review | API-mode; M365 near-real-time per Microsoft; some bulk operations arrive as single events with item-count, not per-item | Legitimate bulk-delete events (SharePoint site archival, content migrations, mailbox cleanup) — exclude documented migration accounts. Slow-and-low destruction (100 files/hour over days) defeats the threshold |
| Netskope | `[unverified]` — Netskope UEBA delete anomaly | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security activity policies | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB anomaly | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT incident management + business continuity](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `DE.AE-02`, `DE.CM-03`, `RC.RP-01` (recovery plan executed) `[VERIFY]`

## False-positive risk

- SharePoint site archival projects
- Mailbox cleanup workflows
- Document-retention automation that schedules large deletes
- Backup → delete → restore test cycles

## Operational cost

- **Exception-handling load:** medium — migration / archival projects need exception scoping
- **Triage load:** medium — alerts correlated with business operations
- **End-user friction:** low (alert-only); high if auto-suspend on a legitimate cleanup

## Privacy / data-protection considerations

- Delete-event audit-log = workforce-action data
- Auto-suspend on mass-delete is high-impact governance action; document authorisation chain

## Coverage gaps

- Slow-and-low destruction over days/weeks — sub-threshold per window
- Attacker pre-compromises an excluded account (service account, migration account)
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — detection post-event; pair with version-history + backup for recovery
- Ransomware that overwrites files in place (not delete then create) — depends on event-class fidelity

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
