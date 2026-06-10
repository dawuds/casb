# Auto-label PCI data

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [API-mode DLP + sensitivity-labelling integration](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector + Purview (or equivalent labelling platform).
> MDA playbook reference: [Policy 6](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Auto-label PCI files in OneDrive/SharePoint). *("Policy 6" is internal playbook numbering, not Microsoft documentation.)*

## Purpose

Continuously scan sanctioned SaaS storage for files containing PCI cardholder data; auto-apply a Confidential-Finance sensitivity label that carries encryption + access restrictions via the labelling platform. Reduces cardholder-data sprawl. Precondition for downstream encryption controls and the inline download-block policy.

Microsoft architectural anchor: **Data Zero Trust pillar Objectives I (know your data) and II (protect your data)** — automated classification + label-driven protection of sensitive data at rest is the documented Microsoft pattern for this control class. [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/data]

## What organisations use this for

For regulated FIs, this policy is rarely a stand-alone DLP control — it is a **classification precondition** for the broader cardholder-data protection programme. Without it, downstream policies (block-download-of-Confidential, share-link-quarantine, label-aware Conditional Access) have nothing to match on. The deployment is usually driven by an upcoming PCI DSS audit cycle rather than by a specific incident.

### Use case 1 — Tier-1 ASEAN universal bank, pre-PCI-DSS-v4.0 baseline

- **Org type:** large universal bank, ~30k employees, M365 E5, in-scope for PCI DSS v4.0 (issuing + acquiring), BNM RMiT supervised
- **Trigger:** PCI DSS v4.0 cycle starting in 6 months; pre-assessment by external QSA identified "cardholder data sprawl in OneDrive / SharePoint outside the documented CHD environment" as a gap finding
- **Scope:** Finance + Cards + Collections business units (~3k users); OneDrive + SharePoint Online; phased rollout by BU starting with Cards
- **Outcome:** documented control evidence for PCI DSS Req. 3.4 + Req. 7; ~15k files labelled in first quarter; 3 BUs surfaced as having undocumented CHD storage that became scope-reduction work

### Use case 2 — Merchant acquirer with multi-jurisdiction CHD

- **Org type:** payments processor operating across MY / SG / HK / TH; in-scope for PCI DSS + BNM RMiT + MAS TRM + HKMA SA-2; cardholder data flows cross-border by design
- **Trigger:** simultaneous supervisory inquiry across BNM / MAS / HKMA on cross-border data flows; need to demonstrate per-jurisdiction CHD classification
- **Scope:** labels designed with residency metadata (Confidential-CHD-MY / -SG / -HK / -TH); geo-aware label-application policy; tenant primary-data region pulled and documented per Purview supported-AI-sites page
- **Outcome:** residency-by-data-class evidence pack; auditor accepted Purview labels as primary classification evidence; ongoing operational cost of maintaining the label taxonomy

### Use case 3 — Digital-native challenger bank, pre-IPO audit prep

- **Org type:** neobank, ~500 employees, high-velocity card-issuance workflow, M365 E5
- **Trigger:** pre-IPO due diligence surfaced cardholder data scattered across product / engineering OneDrives (test cards, sample CHD for QA, mock data files)
- **Scope:** tenant-wide; high false-positive rate on mock data was the main operational problem
- **Outcome:** surfaced ~200 files with real CHD that should not have been there; ~2,000 files with mock-data-format CHD that needed FP-tuning; custom SIT for known mock-card patterns reduced FP rate from ~60% to ~8% over 8 weeks

### Use case 4 — DLP-platform migration, Symantec → Purview

- **Org type:** existing tenant with Symantec DLP at endpoint, adding Purview-via-MDA under an M365 E5 vendor-consolidation programme
- **Trigger:** Symantec contract renewal vs M365 E5 capability — finance pushed for consolidation
- **Scope:** parallel-run for 90 days; gradual policy migration by data class
- **Outcome:** identified classifier-set divergence (Purview SIT detection ≠ Symantec EDM detection for some custom classifiers); 6-month migration plan extended to 12 months; Symantec retained for endpoint DLP, Purview adopted for SaaS DLP — hybrid end-state

## Implementation pattern

Typical 12-week rollout sequence for a tier-2 BFSI tenant new to Purview labelling:

| Week | Activity | Output / gate |
|---|---|---|
| W1-W2 | Label taxonomy design (Confidential-Finance variants); Purview label creation; label-publishing scope decision | Label published to pilot user scope; encryption + permissions configured |
| W3-W4 | DCS pilot on small folder set (1 finance folder, ~100 files), action = Alert-only | Initial FP rate baseline; per-SIT confidence threshold tuning |
| W5-W6 | Expand pilot to one BU (Cards or Finance); still alert-only | Per-BU FP rate; exclusion list (test data folders, QA sample folders, historical archives) |
| W7-W8 | Custom SITs for known mock-data patterns (e.g. 4111-1111 family, vendor-specific test cards); per-SIT confidence level + instance count tuning | FP rate stabilised at acceptable level (typically <10%) |
| W9-W10 | Flip to label-apply action for pilot BU; monitor user / downstream-recipient friction | Encryption-aware downstream consumers identified (Outlook recipients, external partners with no label-rights); exception path documented |
| W11-W12 | Broaden to remaining in-scope BUs | First quarterly audit-evidence pull tested |
| W13+ | Steady-state — monthly tuning + quarterly attestation cycle | Audit-evidence package becomes a reusable deliverable |

The pilot phase is the work. Flipping to label-apply across the whole tenant on day one without the per-SIT confidence-tuning produces a labelling cascade that the help desk cannot triage.

## Action

- Primary: **Apply Microsoft Purview sensitivity label** (Microsoft documented governance-action name) with encryption + permissions on detection
- Phase-1 alternative: **alert-only** for at least 1 week to baseline FP rate before flipping to label-apply

## Scope

- **Users:** all OneDrive / SharePoint owners (label-application acts on the file, not the user — scope decision is which app instances to scan)
- **Apps:** OneDrive for Business + SharePoint Online (primary); Google Drive where Workspace + Google labels are integrated; Box / Dropbox where API connector + labelling integration exist
- **Device posture:** N/A (at-rest scan)
- **Network position:** N/A
- **Traffic:** at-rest

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → Policies → Policy management → Information protection → Create File policy. Plus Purview label configured at Microsoft Purview portal → Information Protection → Labels | App filter = OneDrive + SharePoint; Parent folder = scoped to finance / CHD folders on rollout; Inspection method = Data Classification Service; SIT = Credit Card Number; instance count (`minCount`) = 5; confidence level = High; Governance action = **Apply Microsoft Purview sensitivity label** → Confidential-Finance; Alert + email per match | API-mode — post-upload labelling, not pre-upload prevention. Label-encryption does NOT propagate to existing co-author sessions until they reopen | The 100-character matching-content preview is itself cardholder data — lives in Defender's incident metadata, subject to PCI DSS 3.4 / 3.5 storage controls. Label silently fails if Purview label is not published to affected users OR if DCS is not onboarded to the SharePoint sites. Validate end-to-end with a known-positive test file before broad rollout |
| Netskope | `[unverified]` — Netskope CASB API + native labelling integration | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security with sensitivity-label-apply action | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB with labelling | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security with labelling integration | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant after FP-tuning is complete. Parameter names align with Microsoft Purview SIT terminology: **instance count** (`minCount` / `maxCount`) and **confidence level** (`confidenceLevel`) rather than synthetic `minimum_violations` / numeric-confidence-percent vocabulary. [Microsoft Learn: https://learn.microsoft.com/en-us/purview/sit-modify-a-custom-sensitive-information-type]

```yaml
policy:
  name: "Auto-label PCI — Cards BU OneDrive/SharePoint"
  type: FilePolicy
  app_filter:
    - OneDrive for Business
    - SharePoint Online
  parent_folder_scope:
    include:
      - /Cards/*
      - /Finance/Cards/*
      - /Sites/Cards-Operations/*
    exclude:
      - /Cards/QA/test-data/*           # known mock-card storage
      - /Cards/Training/*               # contains 4111-1111 family
      - /Sites/Cards-Operations/Archives/pre-2024/*
  inspection:
    method: DataClassificationService
    sensitive_information_types:
      - id: "credit-card-number"
        confidenceLevel: High           # Microsoft SIT confidence-level enum; reduces FP from numeric-format collisions
        minCount: 5                     # instance count — minimum unique matches required
      - id: "custom-sit:internal-bin-list"   # custom SIT for the firm's own BIN ranges
        confidenceLevel: High
        minCount: 1
  governance:
    action: "Apply Microsoft Purview sensitivity label"   # Microsoft documented governance-action name
    label: "Confidential-Finance-CHD"
    apply_to:
      - existing_files: true        # scan backlog
      - new_files: true             # continuous
  alerts:
    per_match_alert: true
    severity: High
    email_recipients: [pci-dlp@example.com]
    daily_alert_limit: 500
  matching_content_handling:
    preview_retention_days: 90      # PCI DSS 3.4 / 3.5 control on the snippet itself
    access_control: ["pci-dlp-admin-group"]
```

The High-confidence + instance-count-5 combination is the typical post-tuning baseline. Lower confidence + lower instance count = noise. Higher = misses legitimate cardholder data in formats DCS extracts imperfectly.

## Variants

### Industry-specific

- **BFSI:** PCI DSS + BNM RMiT mapping is front-and-centre; the Credit Card Number SIT is non-negotiable; the main FP fight is mock data and customer-reference numbers in 16-digit format. Custom SIT for the firm's own BIN ranges is common.
- **Healthcare:** swap Credit Card Number for PHI classifiers (US SSN, NHS number, MRN format, ICD codes). HIPAA Security Rule mapping; the patient-portal-data scope is often the dominant data flow. FP fights are around birthdate-like strings and reference-number formats.
- **Tech:** source-code classifiers (API keys, secrets, internal codebase paths), GitHub-token formats, OAuth-secret patterns. Less CHD focus; more secrets-detection.
- **Retail:** loyalty-programme PII, customer transaction histories; less classifier sophistication but higher volume; FP-tuning challenge is the sheer scale of legitimate matches.
- **Public sector:** government-ID classifiers (national ID formats per jurisdiction); often paired with classification-marking schemes (Official / Sensitive / Secret variants).

### Maturity-based

- **Immature:** one policy, tenant-wide, Alert-only, FP rate >40%, label-application disabled, no exception process, no integration with audit cycle. Common at 6 months post-deployment for under-resourced programmes.
- **Mature:** per-BU policies scoped to known CHD folder sets, FP rate <10%, label-application live with documented exception path, quarterly audit-evidence pull tested, integrated with PCI DSS Req. 3.4 / 3.5 attestation cycle.
- **Advanced:** **Purview Adaptive Protection** integration (insider-risk signal boost on label-application events; Adaptive Protection now wires into DLP, DLM, and Conditional Access — not just CA); multi-jurisdiction residency-aware labels with geo-policy enforcement; classifier confidence level tuned per-BU; per-label downstream-recipient access governance integrated with B2B partner-tenant Cross-Tenant Access Settings. [Microsoft Learn: https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection]

## Control mappings

- **CIS Microsoft 365 Foundations Benchmark v5.0.0 (30 April 2025):**
  - **CIS 3.3.1 (L1)** — *primary map*: Ensure DLP policies are enabled (the umbrella for Purview/MDA labelling and DLP enforcement).
  - **CIS 3.2.1 (L1)** — Ensure SharePoint Online Information Protection policies are set up and used.
  - **CIS 9.1.6 (L1)** — relevant compensating control on sensitivity-label coverage.
  - **CIS 2.4.3 (L2)** — Ensure Microsoft Defender for Cloud Apps is enabled (umbrella for the MDA file-policy mechanism).
- **Microsoft Secure Score** — Data group improvement actions ("Enable Microsoft Purview Information Protection sensitivity labels", "Configure DLP policies", scope-coverage actions). Current Secure Score grouping is four-group (Identity / Device / Apps / Data); the legacy five-group structure including Infrastructure is retired. [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score] [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]
- **Microsoft Zero Trust — Data pillar Objectives I (know your data) and II (protect your data).** [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/data]
- PCI DSS v4.0 Req. 3 (protect cardholder data); Req. 3.4 (PAN unreadable); Req. 3.5 (key management for stored PAN); Req. 7 (need-to-know access); Req. 12.10 (incident response — label-aware IR runbook). *Illustrative cross-reference; not a substitute for QSA assessment.*
- BNM RMiT clause(s): [BNM RMiT data classification + DLP](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]`. *Illustrative; not regulatory advice.*
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.12 (information security)
- NIST CSF 2.0 subcategory(ies): `PR.DS-01` (data-at-rest), `PR.DS-05` (data leak protection), `ID.AM-05` (data classification) `[VERIFY]`

## False-positive risk

- DCS hits on test cards (4111-1111-1111-1111 family), mock data, expired/cancelled cards
- Excel spreadsheets with numeric columns matching credit-card format (employee IDs, order numbers, reference numbers)
- PDF receipts with partially-masked PAN (e.g. last 4 digits)
- Image-of-card screenshots where OCR partially extracts the format
- Legacy archived records containing PAN that the firm has decided not to remediate (scope-acceptance decision)

## Real-world FP experience

*Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.*

Typical FP-rate trajectory in a tier-2 BFSI tenant new to Purview labelling:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 60-80% | Mock cards in QA folders + training material + numeric ID columns matching Luhn |
| W4 | 25-35% | After per-SIT confidence-level raise (default Medium → High); after folder allowlist for known QA/test paths |
| W8 | 10-15% | After custom SIT for internal BIN ranges + after exclusion of historical archive folders |
| W12 | 5-10% | Mature exclusion lists; per-BU policies; documented exception path |
| Steady-state | 2-5% | Quarterly tuning cycle catches new noise sources before they grow |

Named FP scenarios encountered repeatedly across deployments (practitioner observation):

| Scenario | Mitigation |
|---|---|
| 4111-1111-1111-1111 and friends in training material, QA test data, vendor SDK sample code | Folder-path exclusion for known training/QA paths + custom-SIT-exclusion |
| 16-digit customer reference numbers passing Luhn (e.g. national ID + check-digit combinations) | Custom-SIT-exclusion patterns specific to the reference-number format |
| Order numbers / case numbers with 16-digit numeric format | Same — pattern-specific exclusion |
| Partially-masked PAN in PDF receipts (e.g. `****-****-****-1234`) | Accept as TP (residual PAN exists) OR exclude masked-PAN format specifically |
| OCR-extracted card numbers from card-photo screenshots | Often accepted as TP (real cardholder data has leaked to OneDrive) |
| Old expired-card records in archives | Scope-acceptance decision per the firm's data-retention policy + PCI DSS Req. 3.2 |

## Operational cost

- **Exception-handling load:** medium during the W3-W12 tuning ramp (5-15 exceptions per week typical); low steady-state (1-3 per week)
- **Triage load:** high during W1-W4 alert-only baseline (50-200 alerts per day typical for a new deployment); medium W5-W8 after first tuning pass; low after FP-tuning is complete
- **End-user friction:** initially high — files become encrypted, downstream recipients without label-rights cannot open. Communicate label semantics + recipient onboarding path upfront. After 2-3 months of operating, friction drops sharply

Typical staffing: 0.5 FTE for the platform admin during the 12-week ramp; 0.2 FTE steady-state. PCI DSS programme team's compliance analyst also commits ~0.1 FTE for the quarterly audit-evidence pull.

## Privacy / data-protection considerations

- The matching-content snippet contains PCI cardholder data — itself regulated content under PCI DSS 3.4 / 3.5
- Document where the snippet is stored, who can access it, retention. The auditor will probe. Default: 90-day retention; admin-group-restricted access; explicit access-log review monthly
- For OneDrive personal-area folders, some tenants treat OneDrive as personal scope — content inspection has PDPA / GDPR implications. Workforce-notice posture must cover the scanning of personal-folder areas
- Cross-border transfer: where DCS scanning happens vs where the tenant data resides. For Malaysian / Singapore / Hong Kong tenants, Japan East as a Microsoft 365 primary-data region materially improves the cross-border story. [VERIFY against current Microsoft Cloud regions page — https://learn.microsoft.com/en-us/microsoft-365/enterprise/o365-data-locations]

## Integration with broader programmes

- **PCI DSS audit cycle:** Purview label-application audit log + labelled-file count by BU feeds Req. 3.4 / 3.5 / 7 attestation evidence. Quarterly pull; annual auditor pull
- **Purview eDiscovery:** label-application events and labelled-file inventories are discoverable through the new eDiscovery experience in the Microsoft Purview portal. (Note: classic eDiscovery experiences were retired 2025-08-31; use "Purview eDiscovery" rather than the legacy "eDiscovery (Premium)" framing.) [Microsoft Learn: https://learn.microsoft.com/en-us/purview/ediscovery-overview]
- **Board reporting:** quarterly metric — "% of in-scope cardholder data labelled vs estimated total" (driven by Discovery + this policy); trend matters more than absolute number
- **Incident response runbook:** labelled-file count provides the IR scoping signal — if a data-loss event involves labelled-files, the scope is reduced (label encryption protects); if it involves unlabelled-but-CHD-class content, the scope is larger
- **Adaptive Protection wiring:** label-application events can feed Microsoft Purview Adaptive Protection, which now signals into DLP, DLM, and Conditional Access (not Conditional Access alone). [Microsoft Learn: https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection]
- **DPIA refresh:** every annual DPIA cycle includes a review of the Purview-DCS scanning scope, the matching-content snippet retention, and the workforce-monitoring posture
- **Vendor-risk programme:** the Microsoft Purview side becomes a sub-processor relationship — sub-processor list, attestation pull (SOC 2 + ISO 27018), DPA review

## Anti-patterns specific to this policy

1. **"Apply the label to all files"** — guarantees over-classification, kills downstream usability when label encryption blocks legitimate consumers
2. **"Run tenant-wide on day 1"** — DCS quota exhaustion mid-rollout; partial coverage with no operator notification (silent under-coverage); the auditor finds gaps; the programme's credibility takes a hit
3. **"Don't bother with FP tuning, just live with the noise"** — analyst fatigue + missed real TPs (the real CHD events get lost in the FP volume)
4. **"Apply encryption on initial rollout without recipient-side preparation"** — breaks external recipients who don't have label-decryption rights; trust crisis with business partners
5. **"Treat the matching-content snippet as harmless metadata"** — it IS cardholder data; PCI DSS 3.4 / 3.5 controls apply to it. Auditor finds the snippet store, programme fails the control
6. **"Ignore mock-data exclusion until the auditor complains"** — by the time the auditor sees the noise, the analyst team has already learned to ignore alerts in general; the cultural fix is months of work
7. **"Use the vendor's default Credit Card SIT confidence level (Medium)"** — produces the W1 60-80% FP rate; raise to High confidence level for any production deployment (practitioner observation)
8. **"Skip the per-BU phasing and do tenant-wide at once"** — the exception backlog overwhelms the help desk; programme rolls back

## Coverage gaps

- Image-format card numbers (photo of a card, scan of a receipt) — DCS does not OCR at scan layer
- Files larger than DCS content-inspection limit — unscannable; tagged `Corrupt file` or skipped
- Files protected by user-applied RMS or password — see [`../../08-failure-modes/encrypted-upload-bypass.md`](../../08-failure-modes/encrypted-upload-bypass.md)
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — file existed in tenant before labelling completes
- **File-policy ceiling = 50 per tenant** per Microsoft Learn `data-protection-policies` Limitations section (current as of 2026-06; the earlier "200" figure circulated in practitioner notes is stale). Cardholder-data scope often consumes 3-5 of those 50 slots (per app × per label family) — non-trivial budget pressure when the same tenant also runs share-link-quarantine, at-rest-quarantine, and other File policies. [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies]
- Multi-geo SharePoint — supported "only for OneDrive" per the M365 connector docs

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
