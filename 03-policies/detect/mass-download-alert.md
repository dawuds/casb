# Mass-download alert

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Required capabilities: [Anomalous-activity / UEBA detection + Compromised-account detection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: [Policy 3](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Mass-download anomaly with alert).

## Purpose

Detect per-user volumetric mass-download patterns against sanctioned SaaS storage; alert SOC for triage. Catches insider exfiltration via bulk download (pre-resignation patterns) and compromised-account exfiltration after credential theft. Counters MITRE ATT&CK `T1530 Data from Cloud Storage Object`, `T1213.002 SharePoint`, `T1567.002 Exfiltration to Cloud Storage` — Detect leg, post-event.

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

## Control mappings

- BNM RMiT clause(s): [BNM RMiT cybersecurity operations + logging](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `DE.CM-03` (personnel activity monitoring), `DE.AE-02` (analyse event data), `RS.AN-01` (notifications investigated) `[VERIFY]`

## False-positive risk

- Legitimate bulk downloads — DR rehearsals, content migrations, BI export jobs by service accounts. Exclude service accounts by UPN
- Legal eDiscovery teams, BCM rehearsals, content migrators — stratify thresholds by role
- New-hire onboarding downloads (template packs) — may breach threshold legitimately

## Operational cost

- **Exception-handling load:** low — service-account exclusion list maintained quarterly
- **Triage load:** medium — alerts need correlation with sign-in risk + user role
- **End-user friction:** low if alert-only; high if auto-suspend (broken account, urgent recovery)

## Privacy / data-protection considerations

- Activity-log records contain user identity + file metadata + timestamps — workforce-monitoring data, document under PDPA / GDPR Art. 88
- Auto-suspend governance action is high-impact; documented authorisation chain required

## Coverage gaps

- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — detection is post-event
- Slow-and-low exfil (100 files/day × 30 days) defeats sub-day threshold — pair with weekly-baseline Sentinel KQL (see MDA Appendix B.1)
- Native-sync staging (T1074.002 via OneDrive sync) produces sync events, not download events — separate detection required
- T1550.001 token replay — attacker on different infrastructure may not trigger the per-user-per-app threshold

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
