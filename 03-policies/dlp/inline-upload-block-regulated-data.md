# Inline upload block — regulated data

> Status: v0.0 — MDA column from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [Inline DLP for sanctioned SaaS](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC) or forward-proxy. Entra ID P1 required for the MDA path.
> MDA playbook reference: [Policy 5 / 9](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Block download of Confidential-labelled files; Block clipboard paste of sensitive data — same CAAC inspection mechanic on the upload leg).

## Purpose

Prevent upload of files containing regulated data (PCI / PII / source / secrets) to sanctioned SaaS before the file lands. The pre-event prevention complement to API-mode DLP (post-event detect-and-remediate). Counters MITRE ATT&CK `T1530` and `T1567` on the upload leg.

## What organisations use this for

For regulated FIs, inline upload-block is the policy with the **highest perceived deterrence value and the highest operational friction**. It is the policy that gets shown to the board ("we now prevent cardholder data leaving the firm via SaaS upload"), but it is also the policy that breaks the most user workflows, generates the most help-desk tickets, and demands the most tuning. Unlike the [auto-label PCI](auto-label-pci-data.md) policy — which is a classification precondition that runs at-rest — this policy is **active inline interception**: every upload event is content-scanned before the bytes land at the SaaS endpoint. The user sees the block in real time, and so does the help desk.

The deployment pattern that survives is **never tenant-wide-day-1**. The pattern that fails is **block-everything-then-tune**. The W1-to-W12 ramp is dominated by FP-suppression work; the W12+ steady-state is dominated by integration with the broader DLP and audit-evidence programmes.

### Use case 1 — Merchant acquirer baselining PCI DSS v4.0 controls pre-audit

- **Org type:** payments processor operating across MY / SG / HK, ~4k employees, in-scope for PCI DSS v4.0 (both issuing and acquiring rails), M365 E5, existing Purview labelling for at-rest CHD
- **Trigger:** external QSA pre-assessment ahead of PCI DSS v4.0 cycle identified "no inline prevention control on cardholder-data upload to sanctioned SaaS" as a gap finding under Req. 3.4. The at-rest auto-label control was accepted; the inline prevention control was not in place. Board-level commitment to close the gap before the formal audit window
- **Scope:** Cards + Finance + Settlements BUs (~800 users); CAAC-onboarded apps = OneDrive + SharePoint Online + Box (Box used for partner-facing CHD exchange under contract); minimum violations = 5; SIT = Credit Card Number at confidence 85
- **Outcome:** documented Req. 3.4 prevention evidence with three-record audit trail (Entra `SigninLogs` + Entra `AuditLogs` + Defender `CloudAppEvents`); ~40 blocked uploads per week in steady state; ~8% FP rate after W12 tuning, mostly internal-BIN-pattern collisions in test-card spreadsheets; QSA accepted the control as compensating for legacy file-share workflows that could not be migrated in time

### Use case 2 — Tier-1 ASEAN universal bank running a PII protection programme under PDPA MY 2024

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, PDPA MY 2024 (Act A1709) in force with statutory data-breach notification obligations
- **Trigger:** PDPA MY 2024 came into force with 72-hour breach-notification clock `[VERIFY against current Act and JPDP guidance]`. Programme review surfaced that the firm had no inline prevention on personal-data upload to SaaS — only post-event API-mode detection. Risk Committee asked Group CISO to close the gap as a category-1 risk
- **Scope:** all employees tenant-wide for OneDrive + SharePoint Online (the M365 connector); phased CAAC onboarding for Workday + ServiceNow + Salesforce over 6 months; SITs = Malaysian IC Number (custom) + Passport Number (multi-jurisdiction) + Bank Account Number (custom); minimum violations = 10 to suppress the high-volume FP class from staff-listing spreadsheets
- **Outcome:** the programme surfaced ~200 BUs uploading staff lists with IC numbers to OneDrive as a normal HR workflow — that workflow itself became the remediation target (move to a controlled HR app); ~120 blocked uploads per week steady-state; help-desk-ticket volume spiked W3-W6, settled by W10 after coaching message redesign; the policy became the headline control in the next PDPA gap-closure board paper

### Use case 3 — Pre-IPO due diligence at a digital-native challenger bank

- **Org type:** neobank, ~600 employees, fast-growth, M365 E5, founder-led engineering culture, minimal pre-existing DLP, planning IPO listing in 18 months
- **Trigger:** pre-IPO due diligence vendor flagged "no preventive control on customer-data exfiltration via SaaS upload" as a category-A finding in the IT controls audit. The auditor would not sign without a control. Investor scrutiny made the timeline non-negotiable
- **Scope:** tenant-wide from day 1 (the org was small enough that phased BU rollout offered little benefit); OneDrive + SharePoint Online; custom SIT for the firm's own internal customer-reference-number format; minimum violations = 3 (lower threshold accepted because the small employee population made volume manageable)
- **Outcome:** ~25 blocks per week; W1-W4 FP rate ~50% driven by engineers uploading test-data files containing mock customer references that matched the production format; resolution = custom SIT exclusion for the mock-data format + engineering-team coaching message variant; FP rate at audit time ~10%; IPO auditor signed the control off; the control mapped to NIST CSF 2.0 `PR.DS-05` in the SOC 2 Type II report

### Use case 4 — M&A integration aligning two DLP estates (regional bank acquires fintech)

- **Org type:** regional bank with mature MDA + Purview DLP estate acquiring a smaller fintech with Netskope inline DLP; ~5k combined employees post-merger; integration window 18 months
- **Trigger:** post-merger Day-90 IT integration review surfaced that the two estates had divergent SIT definitions, divergent block-vs-coach defaults, and divergent exception processes. The acquirer's audit committee required a single inline DLP posture across the combined entity within 12 months
- **Scope:** the acquirer's MDA estate became the target; the fintech's Netskope estate ran in parallel for 6 months; SIT alignment was the dominant work — Netskope EDM signatures had no direct Purview DCS equivalent for several custom classes; minimum violations standardised at 5 across both estates as the interim baseline
- **Outcome:** ~30% of Netskope custom signatures had no clean Purview DCS port; the firm built parallel custom SITs in Purview from scratch; FP rate diverged between the two estates during the parallel-run (Netskope ran lower at ~7%, MDA at ~15% on the same data); the integration extended from 6 to 11 months; final state was MDA for the merged tenant with three retained Netskope policies on specific custom classes that did not port — a hybrid end-state that the audit committee reluctantly accepted

## Implementation pattern

Typical 12-week rollout for a tier-2 BFSI tenant new to inline CASB DLP. The pilot phase is the work — the technical configuration takes hours, the FP tuning takes weeks.

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Day-0 prerequisites: Entra ID P1 verified; Purview labels published; documented break-glass exclusions; bilateral CAAC trap understood by the operator team. Document the data classes in scope (PCI / national ID / bank account / passport / custom) | Day-0 sign-off; SIT inventory complete |
| W2 | Per-SIT confidence + minimum-violations baseline. Run the chosen SITs in `Audit` mode across 30-day historical activity (via Purview DCS at-rest scan as proxy) to estimate FP volume | Baseline FP estimate by SIT; threshold candidates |
| W3 | Pilot scope decision: one BU (typically Finance for PCI, or HR for national-ID-class). Configure bilateral CAAC for pilot user group. Apply session policy with action = `Audit` (NOT block). Validate end-to-end with known-positive test file | Pilot live in audit mode; bilateral CAAC verified |
| W4 | Pilot SOC review of audit-mode events; classify TP / FP; identify FP patterns. Document the coaching-message wording and the exception path (which mailbox / which approver) | First FP-pattern classifier; coaching message v1 |
| W5-W6 | Flip pilot BU to `Block`. Monitor help-desk ticket volume daily. Iterate on coaching-message wording. Adjust per-SIT confidence (typically raise from default 75% to 85%) and minimum violations (typically raise from 1 to 5+) | Pilot BU blocked; ticket volume tracked; W6 review gate |
| W7-W8 | Broaden to second BU; repeat the audit→block ramp. Add exclusion patterns for FPs surfaced in the pilot (internal reference-number formats, test-data file paths) | Second BU live; cumulative exclusion list documented |
| W9-W10 | Tenant-wide rollout in `Audit` mode; SOC review again at tenant scale (legitimate volume from BUs not yet seen) | Tenant audit data; tenant-scale FP review |
| W11-W12 | Tenant-wide `Block`. Operator runbook signed off; quarterly review cadence agreed; first audit-evidence pull tested | Steady-state ops handoff |
| W13+ | Quarterly tuning + cross-correlation with at-rest auto-label evidence for audit purposes | Quarterly metric on block-rate + FP-rate |

The W3-W4 pilot is the most important investment. Skipping audit-mode and going straight to `Block` produces a Week-1 ticket spike that overwhelms help desk and creates organisational resistance to inline DLP that takes years to recover from.

## Action

- Primary: **block** upload
- Coaching variant: **block** + custom user message naming the regulated-data class detected and the approved alternative path (label-first workflow, approved file-exchange channel, request-exception mailbox)
- Pre-block phase: **audit** for at least 1 week before flipping any new SIT to block

## Scope

- **Users:** all employees (excluding documented break-glass and exception groups); phased BU rollout in practice
- **Apps:** all CAAC-onboarded apps (M365 + extended onboarding); restrict to specific apps initially
- **Device posture:** any (the file content drives the decision, not the device)
- **Network position:** any (CAAC-mediated browser session)
- **Traffic:** upload (file or paste-content where supported)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create Session policy. Plus the matching Entra Conditional Access policy with "Use Conditional Access App Control" | Session control type = Control file upload (with inspection); App = CAAC-onboarded list; Inspection method = Microsoft Purview Data Classification Service; SIT = Credit Card Number / U.S. SSN / EU passport / custom; **Match accuracy ≥ 85 + minimum unique-match count ≥ 5** (not the default `minimum_violations = 1`); Action = Block + custom user message; `Always apply if data cannot be scanned` = **document the trade-off** (false = leak via user-encrypted archive; true = block all unscannable including legitimate Zip / OneNote / proprietary formats) | Browser-only via CAAC; reverse-proxy; bilateral CAAC trap (CA policy + session policy required — either alone is a silent no-op); cert-pinned device profiles reject the `*.mcas.ms` chain and fail closed at TLS but look fail-open at the app layer | DCS regex on SSN matches many 9-digit strings (employee IDs, order numbers) — high FP. The 100-character matching-content snippet is itself regulated content stored in Defender incident metadata. `window.parent.postMessage` cross-origin failure breaks Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview, Atlassian Compass after CAAC rewrite — regression-test every app before adding to scope. CAE-induced session re-establishment may bypass MDA policy re-evaluation — pair with Entra Token Protection |
| Netskope | `[unverified]` — Netskope CASB Inline with native DLP engine; supports forward-proxy + reverse-proxy modes | | Forward-proxy + reverse-proxy support; native DLP engine; EDM (exact-data-match) signatures common for custom data classes | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access inline DLP via Enterprise DLP add-on; CASB Inline tier required | | | |
| Skyhigh | `[unverified]` — Skyhigh multi-mode DLP with inline + API + endpoint paths | | | |
| Zscaler | `[unverified]` — ZIA inline DLP via Data Protection add-on; classification engine separate from EDM | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant after W12 tuning is complete. Pilot BU = Cards + Finance; tenant-wide rollout planned for W14+.

```yaml
policy:
  name: "Inline upload block — PCI/PII — Cards+Finance BU pilot"
  type: SessionPolicy
  bilateral_pair:
    entra_ca_policy: "CA-CAAC-Upload-Block-CardsFinance"   # MUST exist; bilateral trap
    entra_ca_session_control: "Use Conditional Access App Control"
  session_control:
    type: ControlFileUpload
    inspection: true
  app_filter:
    caac_onboarded_apps:
      - "Office 365 (bundle)"        # NOT individual workloads; bundle = consistent
      - "Box"                        # only after pre-CAAC compat test passed
      - "Salesforce"                 # postMessage breakage validated — workaround in place
    excluded_apps:
      - "ServiceNow"                 # postMessage breakage on Polaris UI; revisit after vendor fix
  user_filter:
    include_groups:
      - "BU-Cards-All"
      - "BU-Finance-All"
    exclude_groups:
      - "Break-Glass-Identity"       # NEVER subject to CAAC
      - "DLP-Exception-Approved"     # named, time-bounded, reviewed quarterly
      - "Service-Accounts"
  device_filter:
    apply_to: any                    # content-driven, not device-driven for this policy
  file_inspection:
    method: DataClassificationService
    sensitive_information_types:
      - id: "credit-card-number"
        match_accuracy: 85           # raised from default 75; reduces format-collision FPs
        minimum_unique_matches: 5    # NOT minimum_violations = 1
      - id: "custom-sit:malaysian-ic-number"
        match_accuracy: 90
        minimum_unique_matches: 3
      - id: "custom-sit:internal-bin-list"
        match_accuracy: 90
        minimum_unique_matches: 1
      - id: "us-ssn"
        match_accuracy: 85
        minimum_unique_matches: 10   # high threshold — SSN regex collides with employee IDs
    fail_closed_on_unscannable: false   # documented decision: legitimate Zip/OneNote not blocked
                                        # compensating control = at-rest scan via Policy 6
  governance:
    action: Block
    user_message: |
      This upload contains regulated data (cardholder data or personal identifiers).
      To proceed, label the file as Confidential-Finance first, or use the approved
      partner-exchange channel. Request exception at dlp-exception@example.com.
    notify_user: true              # legitimate to coach for upload-block; opposite default to download
  alerts:
    severity: High
    email_recipients: [dlp-soc@example.com]
    sentinel_forward: true
    per_match_alert: true
    daily_alert_limit: 200
  matching_content_handling:
    preview_retention_days: 90       # itself regulated content under PCI DSS 3.4/3.5
    access_control: ["dlp-admin-group"]
    snippet_max_chars: 100
  audit_trail:
    required_records:
      - source: "Entra SigninLogs"
        field: "ConditionalAccessStatus"
      - source: "Entra AuditLogs"
        field: "CA policy result"
      - source: "Defender CloudAppEvents"
        fields: ["PolicyId", "Verdict", "ActionType"]
```

The `match_accuracy: 85` + `minimum_unique_matches: 5` combination is the typical post-tuning baseline for the Credit Card Number SIT. The default `match_accuracy: 75` + `minimum_violations: 1` produces W1 FP rates above 50% and is not viable for production. The fail-closed-on-unscannable = false decision is the single most argued point — document the compensating control (at-rest scan via the [auto-label PCI policy](auto-label-pci-data.md)) explicitly in the control-design record.

## Variants

### Industry-specific

- **BFSI:** PCI DSS Req. 3.4 + Req. 4 mapping front-and-centre; SIT inventory dominated by Credit Card Number + custom BIN ranges + jurisdiction-specific national ID; coaching-message wording reviewed by compliance not just IT; auto-block (no coach option) common because customer-impact of CHD leak outweighs user-friction cost
- **Healthcare:** swap card SITs for PHI classifiers (MRN format, ICD codes, NHS / Medicare number); HIPAA Security Rule mapping (`[VERIFY 45 CFR 164.312]`); coaching message must accommodate the clinical-urgency exception class (legitimate clinical workflow vs DLP-block — never default to hard-block without a documented clinician exception path)
- **Tech:** SIT inventory dominated by secrets / source / API-key formats (AWS access-key signature, GitHub PAT format, OAuth client-secret patterns); engineering culture friction higher than BFSI — coaching message must explain WHY the block, not just THAT it blocked; custom SITs for internal codebase paths and project codenames
- **Retail:** loyalty-programme PII, customer transaction patterns; high upload volume from store-operations users; threshold tuning dominated by sheer volume; coaching message often replaced with self-service exception request via in-message link
- **Legal / professional services:** matter-management workflow conflicts heavily with inline DLP (matter-coordinator uploads bulk client documents legitimately); allowlist UPN approach for matter-management roles; coaching message must NOT contain client-identifying language that breaches privilege; DPIA scope expanded because client-confidential content is being inspected

### Maturity-based

- **Immature deployment** looks like: one session policy tenant-wide, default SIT confidence (~75%), minimum_violations = 1, action = Block on day one, no audit-mode pilot, no per-SIT tuning, no documented exception path, no coaching message (or generic "blocked by policy" message). W1 FP rate >50%; help-desk overwhelmed; programme rolls back to audit-only within 2 weeks; trust in inline DLP damaged for 12+ months.

- **Mature deployment** looks like: per-BU policy scoping with phased rollout completed; per-SIT confidence and minimum-unique-match counts tuned (85% / 5+ as typical baseline); documented audit-mode pilot evidence on file; coaching message reviewed quarterly; exception process named with SLA; integration with auto-label PCI policy for at-rest compensating control; integration with Sentinel for the three-record audit trail; FP rate <10% steady state; quarterly tuning cadence.

- **Advanced deployment** looks like: per-SIT confidence tuned per BU (Cards BU uses higher CHD threshold than Risk BU); custom SITs for the firm's own data formats (internal BIN ranges, internal reference numbers, internal product codenames); EDM-equivalent signatures where Purview supports `[VERIFY current EDM status in Purview]`; bilateral CAAC pair version-pinned in source control with documented change-management; coaching message branches by SIT class (CHD message differs from secrets message); CAE + Entra Token Protection in place; Purview IRM signal-boost on repeated block events feeding the insider-risk programme.

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage prevention + cloud risk management](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]`
- PDPA MY 2024 (Act A1709): personal-data protection at the upload boundary; the prevention control material for the 72-hour breach-notification clock `[VERIFY clock against the Act and JPDP guidance]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring of cloud services](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.12 (information security)
- NIST CSF 2.0 subcategory(ies): `PR.DS-05` (protections against data leaks), `DE.CM-06` (external service-provider activity monitored), `PR.PT-04` (communications and control networks protected) `[VERIFY]`
- PCI DSS v4.0 Req. 3.4 (PAN unreadable when stored) + Req. 4.2 (PAN protection during transmission over public networks) — applicable when CHD is in the inspection scope
- MAS TRM Annex F (data loss prevention controls in cloud) `[VERIFY]`

## False-positive risk

- DCS regex on PII SITs fires on test fixtures, employee IDs, order numbers, sample data uploaded for legitimate analysis
- Excel spreadsheets with numeric columns matching SSN format — uploads of legitimate operational data blocked
- Internal reference numbers passing Luhn check (national ID + check-digit combinations)
- Format-shifting (PDF of a spreadsheet, image of a document) where DCS extraction is imperfect or misses content entirely
- 4111-1111-family test cards in training material, QA test data, vendor SDK sample code

## Real-world FP experience

Typical FP-rate trajectory in a tier-2 BFSI tenant new to inline CASB DLP. The block-action nature of this policy concentrates FP cost in user-visible friction, unlike the alert-only mass-download policy where FPs only burden the SOC.

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 45-65% | Default SIT confidence (~75%) + `minimum_violations = 1`; SSN regex on employee IDs; CCN regex on order numbers; test-data files in engineering uploads |
| W4 | 20-30% | After per-SIT match-accuracy raise (75 → 85) + minimum-unique-match raise (1 → 5); after first-pass exclusion list for known reference-number formats |
| W8 | 10-18% | Per-BU policy split (Cards stricter than tenant); custom SIT exclusion patterns mature; test-data file path exclusions in place |
| W12 | 6-12% | Documented exception path operational; coaching-message redesign reduced repeat-block ratio; quarterly tuning cycle running |
| Steady-state | 4-8% | Mature exclusion list; per-BU thresholds; integration with IRM for repeat-pattern identification |

Named FP scenarios encountered repeatedly across deployments:

| Scenario | Mitigation |
|---|---|
| 9-digit employee ID matches U.S. SSN regex | Raise SSN match_accuracy to 90; require co-occurring context tokens ("SSN:" / "Social Security"); exclude HR-system-derived upload paths |
| 16-digit customer-reference number passing Luhn (national ID + check-digit) | Custom SIT exclusion for the reference-number format; coordinate with the customer-data-platform team to surface the pattern definitively |
| Test-card families (4111-1111, 5500-0000, etc.) in QA test data, vendor SDK samples, training material | Custom-SIT-exclusion patterns + folder-path / app-path exclusion where determinable |
| Engineering uploads of CSV test fixtures containing mock customer data matching production format | Engineering-team coaching message variant; custom mock-data pattern; QA-folder allowlist where the upload path is consistent |
| Spreadsheets with numeric ID columns matching multiple SIT regexes at once | Raise minimum_unique_matches; require context tokens; consider per-column DCS context inspection where supported |
| Image-format uploads (photo of a document, screenshot of a spreadsheet) | Accepted as coverage gap — DCS does not OCR at session layer; compensating control via endpoint Purview DLP if licensed |
| Legitimate-encrypted-file blocks (Zip with password, OneNote, proprietary format) | Set `fail_closed_on_unscannable = false`; document the trade-off; compensate with at-rest scan post-upload |
| Bulk legitimate data-class uploads at month-end (e.g. HR payroll spreadsheet) | Named time-bounded exception window with documented approver; alternative: route through dedicated approved channel that bypasses CAAC |
| `postMessage` cross-origin failure on Salesforce Lightning / ServiceNow Polaris after CAAC rewrite blocks legitimate in-app file attach | Pre-CAAC compat test required; if breakage confirmed, remove app from CAAC scope until vendor fix; document as coverage gap |

## Operational cost

- **Exception-handling load:** very high during W3-W12 ramp (20-50 exception requests per week typical at tenant scale); medium steady-state (5-10 per week). Inline block generates exception requests that the auto-label policy does not, because the user feels the block in real time.
- **Triage load:** high — every blocked upload generates a triage event with the matching-content preview. Typical 30-80 blocks per week per 1k users at FP rate 10%. SOC triage time per event is short (~3 min) but volume drives total cost.
- **End-user friction:** high initially. Coaching-message quality is the single biggest driver of friction perception. A generic "blocked by policy" message produces user anger; a specific "this upload contains cardholder data, here is the approved path" message produces user compliance. Help-desk ticket spike in W1-W6 is expected; W8+ typically settles to a long tail.

Typical staffing: 0.5 FTE platform admin during the 12-week ramp (policy tuning + exception handling + coaching-message iteration); 0.2 FTE steady-state. SOC commits ~0.3 FTE for triage at tenant scale. DLP programme analyst owns the exception approval workflow.

## Privacy / data-protection considerations

- Content inspection on upload = **high-intrusion processing** under GDPR Art. 35 / equivalent — DPIA mandatory before pilot. The scope of inspection (which SITs, which user populations, which apps) must be in the DPIA record
- The 100-character matching-content preview snippet is itself regulated content. Where matched content is cardholder data, the snippet is itself subject to PCI DSS 3.4 / 3.5 storage controls. Document snippet retention (default 90 days), admin-group-restricted access, and an explicit monthly access-log review
- Workforce-notice posture required — employee handbook + AUP must explicitly cover SaaS upload content inspection; relying on a generic "we monitor IT systems" notice is insufficient under PDPA MY 2024 / GDPR Art. 13
- Cross-border processing consideration: the inspection (DCS) happens at the tenant primary-data region; for MY / SG / HK tenants the Japan East primary-data region (added 2025 H2) materially improves the cross-border story
- The coaching message itself is user-visible processing data — it must not expose other users' content; do not include the matched-content snippet in the user-visible message under any circumstances
- For BYOD scenarios: the policy inspects personal-device uploads to corporate apps; the workforce-monitoring scope on BYOD must be in the BYOD agreement and re-acknowledged annually

## Integration with broader programmes

- **PCI DSS audit cycle:** the three-record audit trail (`SigninLogs` + `AuditLogs` + `CloudAppEvents`) feeds Req. 3.4 + Req. 4.2 evidence pulls; quarterly extraction; annual auditor pull. Cross-reference the at-rest labelled-file count from the [auto-label PCI policy](auto-label-pci-data.md) to demonstrate end-to-end prevention + classification
- **PDPA MY 2024 / GDPR / equivalent:** the policy is material to the 72-hour breach-notification clock `[VERIFY]` — blocked-upload events represent prevented breaches; the metric "personal data prevented from leaving SaaS surface per quarter" goes into the data-protection officer's annual report
- **Board / executive reporting:** quarterly metric — blocks-per-month trend; FP rate trend; top-3 SIT classes by block volume; named-exception count. The board reads the trend, not the absolute number — a rising trend signals either programme maturity (more SITs in scope, more apps onboarded) or a problem (rising user attempts to exfiltrate, rising legitimate-workflow conflict)
- **Incident response runbook:** a blocked-upload event for a high-confidence SIT triggers the insider-risk-screening runbook; repeated blocks from the same user within a window feed the [mass-download alert](../detect/mass-download-alert.md) correlation logic for pre-departure exfiltration patterns
- **Insider Risk Management:** repeated block events for the same user within 7-30 days become a Purview IRM Indicator (per MDA Policy 12 — IRM signal-boost integration); the signal correlates with HR pre-departure flags, anomalous-email-forward, anomalous-print
- **DPIA refresh:** every annual DPIA cycle includes a review of the SIT scope, the matching-content snippet retention, the workforce-notice posture, and the BYOD inspection scope
- **Vendor risk assessment:** Microsoft Purview becomes a sub-processor relationship — sub-processor list, SOC 2 + ISO 27018 attestation pull, DPA review; the Defender XDR data-lake retention also becomes a sub-processor decision under PDPA / GDPR Article 28 / equivalent
- **M&A IT integration playbook:** the SIT inventory + per-SIT confidence + minimum-unique-match config becomes the standardisation target when integrating an acquired entity's DLP estate; document the standard as a reusable artefact

## Anti-patterns specific to this policy

1. **"Block on day one, tune as we go"** — produces W1 ticket-spike that overwhelms help desk and creates organisational resistance to inline DLP that takes 12+ months to recover from. Always run audit-mode pilot for ≥1 week per SIT before flipping to Block
2. **"Use the vendor's default match_accuracy and minimum_violations"** — default 75% confidence + 1 minimum violation produces W1 FP rates above 50%. The defaults are vendor-marketing-optimised, not production-ready
3. **"One session policy tenant-wide"** — different BUs have different legitimate-upload patterns (Cards uploads CHD legitimately to internal workflows; Engineering uploads test data with mock CHD; HR uploads staff lists with national IDs). Tenant-wide single policy guarantees FP volume that cannot be tuned away without per-BU scoping
4. **"Set the user-coaching message later"** — coaching-message quality is the single biggest driver of user-friction perception. A generic "blocked by policy" message produces user anger and rage-quit-to-personal-email-attachment behaviour. Design the message before pilot, not after
5. **"Skip the bilateral CAAC verification"** — Entra CA policy without matching MDA session policy = redirect to no-op proxy; MDA session policy without matching Entra CA policy = never fires. Both are silent failures. Verify with a test user every time a new app is added to scope
6. **"Treat fail-closed-on-unscannable as a tickbox decision"** — false = silent leak via user-encrypted archive; true = block all legitimate Zip / OneNote / proprietary formats. The decision is a documented control-design trade-off, not a default. State the compensating control (at-rest scan) explicitly
7. **"Inspect everything from day one"** — running every SIT for every user on every app at policy turn-on saturates DCS and inflates the matching-content snippet store with FP material that itself becomes a regulated-content audit problem. Phase the SIT scope as carefully as the user scope
8. **"Ignore the postMessage breakage class"** — Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview, Atlassian Compass break after CAAC rewrite via `window.parent.postMessage` cross-origin failure. Pre-CAAC compat test is mandatory; rolling out the policy and then finding out an app is broken creates business-critical workflow outages
9. **"Treat the matching-content snippet as harmless triage data"** — it IS regulated content. PCI DSS 3.4 / 3.5 applies when the matched content is CHD; PDPA / GDPR applies when it's personal data. Auditor finds the snippet store, programme fails the control
10. **"Skip the coverage gap on image-format and encrypted-file uploads"** — image-format PII, password-protected archives, and certificate-pinned mobile / desktop sync clients all bypass this policy. The control documentation must name the gaps and the compensating controls (endpoint DLP, sync-client blocking, image-format DLP at upload). Selling the policy as "complete inline prevention" produces auditor pushback and false board confidence

## Coverage gaps

- [Encrypted-upload bypass](../../08-failure-modes/encrypted-upload-bypass.md) — user-encrypted archives, password-protected files, user-applied RMS labels evade content inspection
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — apps not CAAC-onboarded fall to post-upload API-mode only (or no detection)
- Image-format PII (photo of a document, screenshot of a spreadsheet) — DCS does not OCR at session layer
- Native desktop / mobile sync clients bypass — only browser sessions covered
- Cert-pinned native apps refuse the `*.mcas.ms` cert chain — fail closed at TLS but appear fail-open at app layer
- T1550.001 token-replay from managed device — token issued legitimately, replayed elsewhere, device-tag filter bypassed entirely
- `postMessage` cross-origin failure on SaaS apps using cross-frame messaging in their UI (Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview, Atlassian Compass) — breaks legitimate in-app functionality after CAAC rewrite
- CAE-induced session re-establishment may bypass MDA session-policy re-evaluation — pair with Entra Token Protection where GA

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
