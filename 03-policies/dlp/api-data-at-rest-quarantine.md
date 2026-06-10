# API mode data-at-rest quarantine

> Status: v0.0 — MDA column from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [API-mode DLP for sanctioned SaaS](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector. No CAAC required.
> MDA playbook reference: [Policy 2](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Auto-label PCI as a related pattern; this policy is the quarantine variant). Internal numbering; not Microsoft documentation.

## Purpose

Scan files already at rest in sanctioned SaaS storage, detect regulated content via DLP signatures, and quarantine matched files to an admin-owned location pending owner review. Catches what slipped past the inline upload gate, what arrived via sync clients, and what was uploaded before policies were enforcing. Counters MITRE ATT&CK `T1530 Data from Cloud Storage Object` Detect leg + `T1213.002 SharePoint` Detect leg. The backfill control that pairs with inline DLP prevention (see [`inline-upload-block-regulated-data.md`](inline-upload-block-regulated-data.md)).

Microsoft architectural anchor: Zero Trust **Data pillar — Objective III** (apply data-classification + DLP + protection actions: remove permissions, quarantine, apply sensitivity labels). [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/data]

## What organisations use this for

This policy is the answer to "we have 5 years of M365 data; the inline DLP we just turned on covers new uploads but the historical sprawl is untouched." Almost every regulated-FI tenant accumulates regulated content (PCI / PII / KYC / health data) in OneDrive and SharePoint over years of operation without classification-aware controls. When the inline DLP programme finally ships, the historical backlog is its own multi-month project — this policy is what runs against it.

The hardest single decision is the **scan-scope ordering** — which folder hierarchies to scan first, in what cadence. A tenant-wide scan saturates the M365 API rate limits; a per-folder phased approach is the typical pattern. The second decision is what to do with hits — quarantine immediately (clean but disruptive) or label-first-quarantine-later (slower but lower friction).

### Use case 1 — Mid-cap insurance carrier, post-incident historical PII surfacing

- **Org type:** mid-cap insurance carrier, ~5k employees, M365 E5 + MDA, HIPAA covered (US ops) + PDPA (ASEAN ops), recently onboarded MDA after a Microsoft 365 E5 upgrade
- **Trigger:** ransomware-adjacent incident where the threat actor exfiltrated ~20GB from a single user's OneDrive; post-incident review surfaced that ~40% of the exfiltrated content was historical PHI / PII that should not have been in OneDrive at all. CISO asked "how much else is like this?"
- **Scope:** all employees (~5k); OneDrive + SharePoint Online; phased by BU (Claims first, then Underwriting, then Sales, then back-office); SIT set = US SSN + Medicare Beneficiary ID + ICD-10 codes + EU national-ID formats (multi-jurisdiction coverage)
- **Outcome:** first 4 weeks of scanning surfaced ~28,000 files with regulated content across ~3,000 users; ~12,000 quarantined immediately (clear policy violations — PHI in personal folders); ~16,000 routed to owner-review (legitimate work-in-progress on claims); records-retention review accelerated by 2 quarters; control evidence for the post-incident regulator response package

### Use case 2 — Pharma R&D backfilling DLP after 7 years without

- **Org type:** mid-cap pharma research firm, ~3k employees, ~7 years on M365 without DLP, M365 E5 added in 2025 as part of FDA-compliance-readiness programme
- **Trigger:** FDA 21 CFR Part 11 audit preparation `[VERIFY against current FDA guidance]`; required evidence of classification and access control on clinical-trial source documentation; documented sprawl of CTMS extracts + IRB submissions + investigator brochures in unstructured OneDrive
- **Scope:** all clinical-research staff (~1.2k users); SharePoint clinical-trial sites + OneDrive; SIT set = patient-ID formats + study-ID patterns + custom-SIT for protocol-deviation language; aggressive 90-day window for historical, 30-day cadence for ongoing
- **Outcome:** ~45,000 files surfaced across 6 months; 80% labelled-and-retained (legitimate research data with documented owner); 15% quarantined (post-study orphan content); 5% surfaced for legal-hold review (active investigator-initiated trials with retention obligations); audit-readiness improved per QSA pre-assessment; surfaced ~12 cases where investigator-initiated trial data had been shared with former-investigator personal email (highest-severity findings)

### Use case 3 — M&A integration scenario, acquired-tenant historical sprawl

- **Org type:** tier-2 ASEAN bank acquiring a regional fintech; ~3.5k acquired-firm employees added to M365 tenant; acquired firm had no prior DLP
- **Trigger:** post-close integration review surfaced that acquired-firm OneDrive contained substantial cardholder data (the firm operated a card-issuing programme), KYC files, and customer-PII that had been uploaded without classification over 4 years; pre-deal due-diligence had flagged the gap but not quantified it
- **Scope:** acquired-firm user population only (~3.5k users initially); 60-day aggressive scan window; ran in parallel with full inline DLP rollout for ongoing flows
- **Outcome:** ~67,000 files surfaced over 8 weeks; classifier-density unusually high (~25% of all scanned files had at least one SIT hit) reflecting the prior-no-DLP state; remediation surge required temporary FTE expansion (3 contractor analysts for 3 months); integration with combined-firm records-retention policy followed by Q2 of integration year; sub-processor sub-list updated to reflect Microsoft Purview as classification processor

### Use case 4 — Public-sector body responding to data-protection-authority inquiry

- **Org type:** large public-sector body, ~15k employees, M365 G3 / G5, multi-region operations
- **Trigger:** national data-protection authority Subject Access Request investigation surfaced that historical citizen-PII had been retained beyond statutory retention period in unstructured SharePoint sites; DPA inquiry demanded systematic remediation evidence
- **Scope:** all citizen-facing department SharePoint sites (~600 sites); broader scope than typical OneDrive-first deployments; aligned with statutory records-retention schedule (different retention periods per data class)
- **Outcome:** ~250,000 files surfaced over 12 weeks; ~180,000 either quarantined (beyond-retention) or transferred to formal records archive (within-retention but mis-located); DPA satisfied with attestation evidence package; control became permanent quarterly attestation cycle for the firm

## Implementation pattern

Typical 16-week rollout for a tenant with multi-year accumulated DLP debt:

| Week | Activity | Output / gate |
|---|---|---|
| W1-W2 | Discovery query — count files in OneDrive + SharePoint by folder, by user, by last-modified band; identify highest-volume scopes | Magnitude estimate; per-BU scope priority decision |
| W3 | SIT-set selection — name the regulated data classes in scope (PCI / PII / PHI / KYC / source code); custom SITs for firm-specific patterns | SIT inventory + business sign-off |
| W4 | Purview label + Quarantine-location setup; admin-group permissions on the quarantine location; matching-content-snippet retention + access controls | Quarantine infrastructure live |
| W5-W6 | Pilot scan on one BU folder (alert-only); FP-rate baseline; SIT confidence threshold tuning | Pilot FP baseline |
| W7-W8 | Flip pilot to label-apply (no quarantine yet); owner-notification message template review; help-desk preparation | Label-apply live for pilot |
| W9-W10 | Flip pilot to quarantine action with 14-day owner-notify grace; quarantine-recovery path tested | First quarantine actions complete |
| W11-W14 | Phased rollout to remaining BUs; ramp scan throughput per Graph API rate budget; track per-BU progress | Tenant-wide coverage achieved |
| W15-W16 | Steady-state cadence established (typically monthly scans + quarterly attestation); records-retention integration | Sustainable cadence + audit-evidence package |
| W17+ | Quarterly attestation cycle; per-SIT classifier refresh; per-BU FP-rate review | Ongoing operations |

The W5-W6 pilot is the FP-tuning critical path. Tenant-wide rollout before per-SIT confidence tuning produces a scan that hits classification-engine quotas and generates noise the SOC cannot triage.

## Action

- Primary: **Put file in admin quarantine** (move to admin-owned location) with owner notification
- Alternative: **alert + Apply Microsoft Purview sensitivity label** without quarantine for less-sensitive classes
- Two-stage option: **label + alert** Stage 1; **Put file in admin quarantine** Stage 2 (14 days later) on no-response

Governance-action names mirror current Microsoft documented action names. [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies]

## Scope

- **Users:** all (file-owner attribution drives notification routing)
- **Apps:** sanctioned SaaS with API connectors (M365 OneDrive/SharePoint primary; Box, Dropbox, Google Drive secondary)
- **Device posture:** N/A — the file is in SaaS; device of upload is irrelevant at scan time
- **Network position:** N/A
- **Traffic:** at-rest scan (not transit)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → Policies → Policy management → Information protection → Create File policy | App filter = OneDrive + SharePoint (+ Box / Dropbox / GWS where API connector configured); Inspection method = Data Classification Service; SIT = regulated data class; Governance action = **Put file in admin quarantine**; Alert + notify owner with 14-day grace before second-stage quarantine | API-mode — post-upload scan, not real-time prevention. M365 latency near-real-time per Microsoft; Power BI / Dynamics 24-72hr per docs. **File-policy ceiling = 50 per tenant** per Microsoft Learn `data-protection-policies` Limitations section (current as of 2026-06). [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies] | "Only the governance action of the first triggered policy is guaranteed to be applied" — policy ordering matters when multiple file policies could match. [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies] Externally-owned files in Box / Dropbox / Google Drive (where external user is the data owner under that platform's data model) are unscannable — only OneDrive reassigns ownership. Quarantine on a file under Microsoft Purview eDiscovery legal hold may be spoliation (legal-counsel framing; not a Microsoft control statement) — Purview hold wins |
| Netskope | `[unverified]` — Netskope CASB API + Native DLP | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security API (formerly Aperture) | | | |
| Skyhigh | `[unverified]` — Skyhigh API mode with multi-mode DLP | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security API | | | |

## Worked configuration example (tier-2 BFSI baseline)

SIT parameters reflect Microsoft documented parameter naming: `instance count` (expressed as `minCount` / `maxCount`) and `confidence level` (`confidenceLevel`). [Microsoft Learn: https://learn.microsoft.com/en-us/purview/sit-defn-credit-card-number] Earlier draft used `minimum_violations` — that is not Microsoft documentation.

```yaml
policy:
  name: "Historical PII / PCI quarantine — Claims OneDrive backfill"
  type: FilePolicy
  app_filter:
    - OneDrive for Business
    - SharePoint Online
  scope:
    parent_folder_include:
      - "/Sites/Claims-Ops/*"
      - "/Sites/Underwriting/*"
      - "/Users/claims-team/*"
    parent_folder_exclude:
      - "/Sites/Claims-Ops/Archives/pre-2020/*"     # explicit retain-with-restricted-access
      - "/Compliance Hold/*"                          # Purview eDiscovery hold scope
  inspection:
    method: DataClassificationService
    sensitive_information_types:
      - id: "credit-card-number"
        confidenceLevel: High        # was: confidence: 75
        minCount: 5                  # instance count — was: minimum_violations
      - id: "us-ssn"
        confidenceLevel: High        # was: confidence: 80
        minCount: 3
      - id: "custom-sit:internal-claim-id"
        confidenceLevel: High        # was: confidence: 85
        minCount: 1
  governance:
    stage_1:
      action: "Alert and notify owner"           # documented action
      grace_period_days: 14
    stage_2:
      action: "Put file in admin quarantine"     # documented action — was: PutInAdminQuarantine
      quarantine_location: "Compliance-Quarantine/Claims/{yyyy-mm}/"
      retain_for_review_days: 90
  matching_content_handling:
    preview_retention_days: 90
    access_control: ["pci-dlp-admin-group"]
  alerts:
    severity: High
    email_recipients: [dlp-ops@example.com, soc-l1@example.com]
    sentinel_forward: true
    daily_alert_limit: 1000           # tune for historical scan; lower steady-state
```

The `daily_alert_limit: 1000` is the operational ceiling — historical scans produce alert volumes orders of magnitude higher than steady-state. Cap to avoid SOC alert-channel saturation. **Practitioner inference:** specific daily-alert-limit value is field observation, not Microsoft-documented.

**File-policy budget:** with a 50-policy tenant ceiling, plan SIT consolidation early. PCI + PII + PHI + KYC + custom-SIT often consumes 3-5 file policies; reserve headroom for inline DLP, auto-label, external-share quarantine, and historical backfill scopes against the same 50-ceiling.

## Variants

### Industry-specific

- **BFSI:** PCI + KYC drive the SIT set; aggressive quarantine action for cardholder data; integration with PCI DSS Req. 3.4 / 3.5 / Req. 7 attestation cycle (illustrative — not regulatory advice)
- **Healthcare:** PHI / patient-ID classifiers; HIPAA BAA + Purview-as-processor sub-list documentation; longer retention review periods for clinical-trial data
- **Pharma / R&D:** clinical-trial protocol patterns + investigator-initiated-trial documentation; longer scan windows because legitimate-research-data lifecycles span years
- **Tech:** source-code + secrets detection; API key formats; less PII focus; integration with code-scanning toolchain
- **Public sector / government:** statutory records-retention drives quarantine triggers; per-record-class retention windows; longer attestation cycles
- **Manufacturing / industrial:** trade-secret + intellectual-property classifiers; supplier-NDA-bound content; pre-IPO disclosure-readiness scenarios

### Maturity-based

- **Immature:** tenant-wide scan on day 1; SIT confidence defaults; no folder allowlist; first scan saturates Graph API + produces alert flood; programme rolled back to alert-only
- **Mature:** per-BU phased scan with FP-tuned SITs; folder allowlist for known-archive scopes; Purview eDiscovery legal-hold integration; two-stage label-then-quarantine flow; quarterly attestation evidence
- **Advanced:** integrated with records-retention workflow (quarantine triggers retention-decision; not just admin-store); cross-app coverage (M365 + Box + GWS + Dropbox unified policy); Adaptive Protection-integrated for risk-based per-user prioritisation [Microsoft Learn: https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection]; matching-content-snippet retention managed-as-code

## Control mappings

- **Microsoft Zero Trust — Data pillar Objective III** (data classification + DLP + protection actions: remove permissions, quarantine, apply labels). Primary Microsoft architectural anchor. [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/data]
- **CIS Microsoft 365 Foundations Benchmark v5.0.0** (30 April 2025):
  - **CIS 3.2.1 (L1)** — Ensure DLP policies are enabled (primary)
  - **CIS 3.3.1 (L1)** — Ensure Microsoft 365 sensitive information types are configured (primary)
  - **CIS 2.4.3 (L2)** — Ensure MDA / Defender for Cloud Apps integration is enabled (umbrella control for the entire MDA-dependent library)
- **Microsoft Secure Score — Data group** improvement actions: DLP-policy coverage, sensitivity-label coverage, sensitive-information-type configuration. [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score | https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]
- PCI DSS v4.0 Req. 3 (protect cardholder data); Req. 3.4 (PAN unreadable); Req. 3.5 (key management); Req. 7 (need-to-know access); Req. 12.10 (incident response) where PCI in scope — illustrative; not regulatory advice
- BNM RMiT clause(s): [BNM RMiT data leakage + records retention](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]` — illustrative; not regulatory advice
- PDPA MY 2024 (Act A1709): personal-data protection; retention-period compliance — illustrative
- ISO 27017 control(s): [CLD.12.4.5 monitoring, CLD.8.1.5 removal of assets](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.11 (sub-processor disclosure)
- NIST CSF 2.0 subcategory(ies): `PR.DS-05`, `DE.CM-03`, `RS.MI-02` `[VERIFY]`
- HIPAA Security Rule §164.308(a)(1)(ii)(D) (information system activity review) where PHI in scope `[VERIFY]` — illustrative; not regulatory advice

## False-positive risk

- Test data + mock cards (4111 family) + employee-ID columns matching credit-card format
- Legacy archived records intentionally retained (scope-acceptance decision per data-retention policy)
- Sample data shared by vendor for analysis (legitimately present but classifier hits)
- Files containing regulated-data field-structure references (HR template that references SSN field) but no actual data

## Real-world FP experience

Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 50-70% | Default SIT confidence (~50%) + test data + employee-ID column collisions |
| W4 | 25-35% | After SIT confidence threshold tuning + initial folder exclusions |
| W8 | 12-18% | After custom-SIT for known mock-data patterns |
| W12 | 6-10% | Mature exclusion lists; per-BU classifier confidence tuning |
| Steady-state | 3-7% | Quarterly classifier refresh + records-retention integration |

Named FP scenarios:

| Scenario | Mitigation |
|---|---|
| 4111-1111-1111-1111 family in QA / training folders | Folder-path exclusion + custom-SIT-exclusion |
| 16-digit customer reference numbers passing Luhn | Custom-SIT-exclusion patterns specific to the reference format |
| Order / case numbers with 16-digit numeric format | Pattern-specific exclusion |
| Partially-masked PAN in legacy receipts | Accept as TP OR exclude masked-PAN format specifically |
| Pre-2020 archived records intentionally retained | Folder-path exclusion + scope-acceptance documentation per retention policy |
| Sample synthetic data used for analytics modelling | Folder-path allowlist + custom-SIT for synthetic-marker presence |

## Operational cost

- **Exception-handling load:** high during W7-W16 ramp (50-200 exceptions per week for a multi-year-debt tenant); medium during ongoing-cadence (10-30 per week); low steady-state
- **Triage load:** high during initial scan saturation; medium during phased rollout; low steady-state once classification debt is cleared
- **End-user friction:** medium-to-high during initial scan (notification surge + quarantine-recovery requests); low steady-state

Typical staffing: 0.5-1.0 FTE during the 16-week ramp; 0.2-0.3 FTE steady-state; for M&A or post-incident scenarios, temporary contractor FTE expansion (2-4 analysts for 3 months) is common. **Practitioner inference:** staffing ranges are field observations; not Microsoft-documented.

## Privacy / data-protection considerations

- Matching-content snippet stored in CASB metadata = regulated content (PCI / PII / PHI / etc.) — itself subject to storage controls under PCI DSS 3.4 / 3.5, HIPAA Security Rule, and PDPA / GDPR (illustrative; not regulatory advice)
- Cross-border-transfer: where DCS / classification engine performs the scan vs where the tenant data resides
- Sub-processor list update: Microsoft Purview becomes the classification processor — document in DPA + customer notification cycle
- Workforce notice scope: content inspection on workforce files at rest is in addition to inline inspection; document under PDPA MY 2024 / GDPR Art. 88 (illustrative; not regulatory advice)

## Integration with broader programmes

- **Records-retention policy:** scan + quarantine becomes the operational arm of retention; integrate quarantine-decision with retention-decision workflow
- **PCI DSS audit cycle:** Purview classification + label-application + quarantine action feed Req. 3.4 / 3.5 / 7 attestation
- **Incident response runbook:** post-incident scan (this policy expanded to broader scope) becomes part of IR playbook; matching-content snippet helps scoping
- **M&A integration:** pre-close scan as due-diligence input; post-close systematic backfill as integration milestone
- **DPIA cycle:** annual DPIA refresh includes scanning-scope + snippet-retention + workforce-monitoring review
- **Board reporting:** quarterly metric — classification coverage (% files scanned + labelled); legacy-debt trajectory (declining trend)
- **Purview eDiscovery integration:** ensure legal-hold scope is excluded before quarantine action runs. Classic eDiscovery experiences retired 2025-08-31; references should point at the new Purview eDiscovery experience. [Microsoft Learn: https://learn.microsoft.com/en-us/purview/ediscovery-overview]

## Anti-patterns specific to this policy

1. **"Tenant-wide scan on day 1"** — saturates Graph API rate limits; partial-coverage silently; long-running scans timeout or back off without operator notification
2. **"SIT confidence defaults"** — Microsoft's default confidence is calibrated for inline inspection; at-rest scan against 5+ years of accumulated data produces 80%+ FP rate at defaults; tune confidence level + instance count per SIT (practitioner observation)
3. **"Quarantine first; let users complain"** — sound bite is satisfying but generates trust crisis; always run two-stage notify-then-quarantine
4. **"Skip the matching-content snippet retention review"** — snippet IS regulated content; PCI DSS 3.4 / 3.5 controls apply; auditor will find the unprotected snippet store
5. **"Skip Purview eDiscovery legal-hold exclusion"** — quarantine of legal-hold material may be spoliation (legal-counsel framing; coordinate with legal before designing the exclusion list)
6. **"Run on Power BI / Dynamics 365 connectors"** — connector latency (24-72hr per docs) makes the policy effectively meaningless for those workloads
7. **"Treat externally-owned files in Box / Dropbox / Google Drive as scannable"** — they are not (only OneDrive reassigns ownership); policy silently misses them
8. **"No record-of-decision for quarantine actions"** — auditor asks "show me the quarantine decision for file X"; without per-action evidence the policy fails control-effectiveness test
9. **"Bound the policy by data-class only, not by folder scope"** — for a multi-year-debt tenant, this produces overwhelming initial volume; phase by folder hierarchy
10. **"Exceed the 50-file-policy tenant ceiling without consolidation plan"** — file policies are capped at 50 per tenant per Microsoft Learn `data-protection-policies`; new policy creation fails silently or displaces an existing one if not actively planned

## Coverage gaps

- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — exposure window between upload and scan completion
- [Encrypted-upload bypass](../../08-failure-modes/encrypted-upload-bypass.md) — encrypted-by-user content tagged `Password encrypted` / `Azure RMS encrypted` and skipped
- Externally-owned files in third-party SaaS where external user retains ownership — unscannable
- Files larger than DCS content-inspection limit — unscannable
- Pre-existing files in tenant before policy enabled — backfill scan required as separate operation
- Image-format regulated content — DCS does not OCR at scan layer `[VERIFY against current Microsoft Purview content-inspection documentation]`

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
