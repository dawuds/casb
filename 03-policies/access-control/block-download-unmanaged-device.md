# Block download to unmanaged device

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [Inline DLP for sanctioned SaaS + Risk-based session policy](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC) or forward-proxy. Entra ID P1 required for the MDA path.
> MDA playbook reference: [Policy 5](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Block download of Confidential-labelled files to unmanaged browsers) — internal numbering, not Microsoft documentation.

## Purpose

Prevent download of files labelled Confidential or higher from sanctioned SaaS to devices that are not Entra-hybrid-joined or Intune-compliant (BYOD / personal). Reduces sensitive-document exfiltration via personal device. Counters MITRE ATT&CK `T1530 Data from Cloud Storage Object` via untrusted endpoint (Prevent for the download leg only).

## What organisations use this for

For regulated FIs this policy is the **flagship BYOD control**. It is the answer to the board question "what happens if an employee opens SharePoint on their home laptop?" and the audit question "demonstrate that sensitive content cannot leave the managed estate." Almost every deployment is driven by one of three events: a BYOD-policy rollout (new flexibility decision by HR), an audit finding (typically pre-cycle pre-assessment), or an executive demanding iPad / personal-Mac access.

The policy is conceptually simple — "block download if device is not managed" — but the deployment is rarely simple. The bilateral CAAC trap (Entra CA policy AND MDA session policy required), the `*.mcas.ms` URL-rewrite regression risk against modern SaaS UIs for non-Edge browsers, the read-only fallback decision, the executive iPad coverage decision, and the contractor-laptop policy all surface in week 1. The deployment teams that succeed treat this as a **change-management exercise as much as a technical one** — comms, exception path, and the pair-policy with Intune MAM are non-negotiable.

### Use case 1 — Tier-1 ASEAN universal bank, BYOD flexibility pilot

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, Intune-managed Windows fleet, mixed iOS / Android estate; new HR policy permitting personal-device access to mail + SharePoint outside corporate working hours
- **Trigger:** board flexibility / talent-retention decision opened BYOD to staff; CISO required a sensitive-content guardrail before BYOD policy live. BNM RMiT cybersecurity-operations section (`[VERIFY against current RMiT edition]`) cited as the supervisory expectation for data-leakage-prevention controls on BYOD endpoints (illustrative — not regulatory advice)
- **Scope:** all employees; Office 365 bundle (SharePoint + OneDrive + Exchange + Teams in browser); block + read-only fallback session policy paired; native mobile apps gated separately via Intune App Protection Policies (MAM-without-enrolment)
- **Outcome:** BYOD policy went live without a P1 data-leakage incident in year 1; ~3,000 unique users hit the read-only fallback per month (legitimate BYOD usage); ~80 hard blocks per month on Confidential-labelled content (mix of TP exfil attempts and FP from mislabelled non-sensitive content)

### Use case 2 — Merchant acquirer, executive iPad coverage decision

- **Org type:** payments processor across MY / SG / HK / TH; PCI DSS + BNM RMiT + MAS TRM + HKMA scope; ~5k employees with ~50 senior executives demanding iPad access to dashboards and email
- **Trigger:** CEO escalation — "I need to read board papers on my iPad on the plane." Existing CAAC scope did not cover iPad-native apps; executives were saving board packs to personal iCloud as a workaround, creating a much larger exfil surface than the original problem
- **Scope:** dual-control design — CAAC reverse-proxy block-download for Highly-Confidential + Confidential labels for browser sessions on unmanaged devices; **paired with Intune App Protection Policies (MAM)** on Outlook Mobile / OneDrive Mobile / Word Mobile / Teams Mobile for the iPad estate, configured with no copy-to-personal-cloud and no save-as-local. Executives onboarded to Intune MAM under a documented light-touch user agreement (PDPA-MY workforce notice updated)
- **Outcome:** iCloud-board-pack workaround eliminated; ~50 iPads onboarded to MAM; CAAC block fired ~12 times in year 1 against executive personal laptops (all triaged as legitimate "I forgot my work laptop" scenarios with documented exception path). PCI DSS Req. 7 (need-to-know access restriction) evidence pack accepted by QSA

### Use case 3 — Tier-2 BFSI, pre-RMiT-audit BYOD-risk-evidence

- **Org type:** mid-size Malaysian commercial bank, ~8k employees, M365 E5; existing BYOD policy on paper, no inline enforcement; external audit (RMiT-cycle pre-assessment) due in 4 months
- **Trigger:** pre-cycle pre-assessment flagged "no inline control prevents Confidential SharePoint content from being downloaded to unmanaged endpoints" as a high-rated finding. Compliance + IT-security joint owner; board risk committee given a remediation milestone
- **Scope:** initial rollout to Treasury + Finance + Risk BUs (~1,500 users) with full tenant rollout deferred to year 2; sensitivity-label coverage prerequisite addressed via parallel Purview auto-labelling sprint (see [auto-label-pci-data](../dlp/auto-label-pci-data.md)); read-only fallback enabled tenant-wide
- **Outcome:** finding closed before the audit cycle with documented evidence package (policy config export, sample block events from `CloudAppEvents`, exception register, workforce-notice update). Auditor accepted the phased rollout plan as a documented control-improvement trajectory rather than a current-state gap. Tenant-wide rollout completed in year 2

### Use case 4 — Digital-native challenger bank with contractor-heavy delivery model

- **Org type:** neobank, ~600 employees + ~400 contractors / consultants on personal laptops; M365 E5; pre-IPO posture work in progress
- **Trigger:** pre-IPO due diligence + an internal incident — a departing contractor downloaded ~200 product / engineering documents from SharePoint via their personal MacBook in the final week. No technical control had been in path; the loss was detected via post-event `CloudAppEvents` review only
- **Scope:** contractor population gated by Entra group membership; CAAC session policy scoped to contractor group; read-only access permitted for legitimate work, block-download for any labelled content; managed-device path (full download) preserved for the employee population; quarterly access review by HR + Legal
- **Outcome:** technical control closed the contractor-laptop gap on a forward-looking basis; the in-progress contractor population had to be re-baselined (some contractors had personal laptops with downloaded content that the firm could not retrospectively wipe — legal accepted residual risk). Used as evidence in the pre-IPO control-environment letter

## Implementation pattern

Typical 10-week rollout for a tier-2 BFSI tenant turning on CAAC session policies for the first time:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Day-0 precondition check: Entra ID P1, Purview labels published and applied to a known sensitive content set, Intune compliance posture on the managed estate, CA policy backup exported, break-glass exclusion set documented | Day-0 preconditions signed off; rollback plan documented |
| W2 | Onboard target apps to CAAC (Office 365 bundle first); execute the pre-CAAC compatibility test for each app (`window.parent.postMessage`, anti-CSRF tokens, OIDC iframe re-auth, third-party-cookie flows, cert-pinned device profile compatibility); document `*.mcas.ms` deprovisioning runbook in case of urgent rollback | Connected-app state = Enabled (not Discovered) on Settings → Conditional Access App Control; compatibility test sign-off per app |
| W3 | Configure paired policies — Entra CA policy with "Use Conditional Access App Control" session control + MDA session policy with Block + custom user message. **Action = Audit (not Block) for the entire week.** Scope to pilot group only (typically 20-50 users including IT-security team) | Bilateral policy live in audit mode; pilot group seeing `*.mcas.ms` redirect (non-Edge) or in-browser protection (Edge for Business); `CloudAppEvents` showing policy match without enforcement |
| W4 | SOC + service-desk runbook: triage path for block events, user-comms for the block message (link to enrolment portal), exception-request flow. Validate `Always apply if data cannot be scanned` decision and document the trade-off | Runbook signed off; user-message wording approved by Comms / Legal; exception-process owner named |
| W5 | Flip pilot group to Block action; observe FP rate, regression issues, user-friction signals (service-desk tickets) for one week | Pilot block events triaged; W5 FP rate baseline established |
| W6 | Expand to first business unit (typically 500-1500 users); add the BYOD read-only fallback session policy as a parallel peer (Audit action; limit-downloads session control; same device filter) | BU live; read-only fallback observed working for legitimate BYOD scenarios |
| W7-W8 | Roll out to remaining in-scope BUs in tranches; refine the exception register; document the per-BU FP rate; integrate the workforce-notice / DPIA update with HR / DPO | Tenant-scoped or BU-scoped rollout complete; FP rate trending toward steady-state |
| W9 | First-month review with the three-lens panel (Architect / Product / Compliance); decision on whether to extend to additional CAAC-onboarded apps (Salesforce, Box, Workday) | Three-lens sign-off; backlog of additional-app onboarding scoped |
| W10 | Steady-state ops handoff; quarterly review cadence documented; integrate with the audit-evidence pull schedule | Production-ready; quarterly review cadence on the compliance calendar |

The W2 pre-CAAC compatibility test is the work that day-one practitioners regret skipping. `window.parent.postMessage` cross-origin failure against Salesforce Lightning / ServiceNow Polaris / Workday Extend / Box file-preview is the dominant 2026 breakage class for non-Edge browsers (Practitioner inference: field observation; not Microsoft-documented) and surfaces only under realistic in-session interaction, not at the SAML round-trip. Edge for Business users avoid the URL-rewrite breakage entirely — see CAAC framing note below.

## Action

- Primary: **block** download
- Alternative (BYOD soft-mode): **allow view + edit in-session**, block download + copy + paste + print + share

## Scope

- **Users:** all employees (excluding documented break-glass)
- **Apps:** Microsoft 365 bundle (SharePoint, OneDrive, Exchange Online, Teams host app); extend to other CAAC-onboarded sanctioned SaaS as onboarding completes
- **Device posture:** **unmanaged** (Device tag ≠ MicrosoftEntraHybridJoined AND ≠ IntuneCompliant)
- **Network position:** any (browser session)
- **Traffic:** download leg only

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → Policies → Policy management → Conditional Access → Create Session policy. **Plus** Entra CA policy with "Use Conditional Access App Control" session control on same scope | Session control type = Control file download (with inspection); App = Office 365 bundle (not individual apps); Client app = Browser; Device filter = Device tag ≠ hybrid AND ≠ compliant; File filter = Sensitivity label ∈ {Confidential, Highly Confidential}; Action = Block; `Always apply if data cannot be scanned` = false | Reverse-proxy CAAC for non-Edge browsers; Edge for Business users get direct in-browser protection (no `*.mcas.ms` redirect) per Microsoft Learn `proxy-intro-aad`. Browser sessions only — native desktop / mobile apps bypass. Requires Entra ID P1 | **Bilateral CAAC trap** — needs CA policy AND session policy (two consoles, two objects). Inverse trap: CA policy without MDA policy = silent no-op proxy redirect. For non-Edge browsers, `*.mcas.ms` URL rewrite breaks `window.parent.postMessage` apps (Salesforce Lightning, ServiceNow Polaris, Workday Extend) — regression test before scoping broadly. Edge for Business removes the URL-rewrite class of regression for that subset of users |
| Netskope | `[unverified]` — Netskope CASB Inline supports per-app session policy with device-classification | | Native forward-proxy + reverse-proxy; broader app support than MDA's CAAC | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access SaaS Security inline | | | |
| Skyhigh | `[unverified]` — Skyhigh reverse-proxy with device-classification | | | |
| Zscaler | `[unverified]` — ZIA inline CASB with device posture check via ZCC | | | |

**CAAC reverse-proxy framing — direction of travel (X8).** `*.mcas.ms` URL rewrite is the reverse-proxy path for non-Edge browsers. Edge for Business users receive direct in-browser session control per Microsoft Learn `proxy-intro-aad` ([https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad](https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad)), which removes the URL-rewrite class of regression for Edge users. Document the residual reverse-proxy dependency for non-Edge browsers (Chrome, Firefox, Safari) and track Edge for Business adoption against the in-scope user population. A complementary Microsoft pattern is SharePoint/Exchange "limited access" / app-enforced restrictions for unmanaged devices [VERIFY: locate current Microsoft Learn URL for SharePoint app-enforced restrictions + Exchange OWA limited access patterns].

**MDE Network Protection cross-reference.** Where managed endpoints are in scope, Microsoft Defender for Endpoint (MDE) Network Protection provides a complementary endpoint-side block for unsanctioned-app upload/download — relevant as a defence-in-depth pair with the CAAC browser-session block. Cite Microsoft Learn Network Protection [VERIFY: `https://learn.microsoft.com/en-us/defender-endpoint/network-protection` — confirm current URL].

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant with Office 365 bundle as the primary scope and a read-only fallback peer policy:

```yaml
# Primary block policy
policy:
  name: "Block download — Confidential to unmanaged browser"
  type: SessionPolicy
  paired_entra_ca_policy: "CA-MDA-Session-Office365"   # bilateral CAAC requirement
  session_control_type: ControlFileDownloadWithInspection
  app_filter:
    - "Office 365"                       # bundle, NOT individual workloads
  client_app_filter:
    - Browser                            # omit Mobile + Desktop (native apps bypass)
  device_filter:
    expression: |
      DeviceTag != "MicrosoftEntraHybridJoined"
      AND DeviceTag != "IntuneCompliant"
  file_filter:
    sensitivity_labels:
      include:
        - "Confidential"
        - "Highly Confidential"
        - "Confidential-Finance-CHD"
        - "Confidential-Customer-Data"
  inspection:
    always_apply_if_data_cannot_be_scanned: false   # fail-open for unscannable; document trade-off
    matched_content_preview_retention_days: 90
  action: Block
  user_message: |
    For security, downloads of Confidential content are blocked on unmanaged
    devices. Request access from your managed device, or contact
    servicedesk@example.com for assistance.
  break_glass_exclusion_group: "BG-CAAC-Break-Glass"
  alerts:
    severity: Medium
    daily_alert_limit: 200
    email_recipients: [casb-ops@example.com]
    sentinel_forward: true

---
# Read-only fallback peer policy — paired with primary
policy:
  name: "Read-only fallback — unmanaged browser"
  type: SessionPolicy
  paired_entra_ca_policy: "CA-MDA-Session-Office365"   # same CA policy
  session_control_type: LimitInSessionActivities
  app_filter:
    - "Office 365"
  client_app_filter:
    - Browser
  device_filter:
    expression: |
      DeviceTag != "MicrosoftEntraHybridJoined"
      AND DeviceTag != "IntuneCompliant"
  in_session_restrictions:
    block_download: true                 # all files, not just labelled
    block_copy: true
    block_paste: true
    block_print: true
    block_share: true
  action: Audit                          # parallel monitoring; not Block
  user_message: |
    You are in a read-only session on an unmanaged device. Editing in-browser
    is permitted; downloads / printing / copy / share are disabled.
```

The two-policy pair is the standard pattern. The primary block policy is the audit-evidence-bearing record. The read-only fallback is the user-experience accommodation — without it, the W4-W6 service-desk tickets overwhelm the rollout.

## Variants

### Industry-specific

- **BFSI:** label-coverage prerequisite is the dominant constraint — the policy is meaningless without comprehensive labelling. Block is typical (not read-only fallback) for high-regulation BUs (Cards, Treasury); regulator-driven workforce-notice posture must address SaaS in-session monitoring scope explicitly under BNM RMiT / MAS TRM / HKMA / PDPA-MY 2024 (illustrative; not regulatory advice — consult counsel)
- **Healthcare:** PHI labels drive scope; HIPAA Security Rule on transmission security pushes for inline enforcement rather than detect-only; patient-portal scope often kept separate from clinician-internal scope; clinician iPad pattern very common (paired with MAM as a near-universal requirement)
- **Tech / SaaS:** source-code-classifier label drives scope; less CHD focus; engineering-team friction is the dominant operational cost (engineers expect to use personal Macs); contractor-laptop scope is the dominant FP fight
- **Retail:** loyalty / customer-PII labels; less labelling sophistication than BFSI; in-store BYOD / iPad estate sometimes in scope (with MAM); FP fight is store-staff iPad-based legitimate workflows
- **Legal / Professional services:** matter-management workflows; matter-coordinator personal-device usage common; eDiscovery + privilege-review scenarios require named exception path (downloads to personal device legitimately occur under matter-management workflow with separate controls)
- **Public sector:** classification-marking integration (Official / Sensitive / Secret) rather than Microsoft labels; clearance-level cross-check often a parallel requirement; cross-jurisdiction policy variation by agency

### Maturity-based

- **Immature:** one tenant-wide policy in Block mode, no read-only fallback, label coverage incomplete (significant unlabelled-but-sensitive content surface), no documented exception path, executive iPad / contractor laptop scenarios unaddressed. Service-desk ticket volume from week-1 user friction triggers rollback or scope-narrowing within 30 days; the policy becomes "exists on paper, scoped to almost nobody."
- **Mature:** paired primary + read-only fallback policies; documented BYOD AUP and workforce-notice posture; label coverage ≥80% on in-scope content; documented exception path including time-bounded approvals; quarterly access review; integration with Intune MAM for mobile native; FP rate <10%; service-desk runbook on the block-user-message workflow.
- **Advanced:** sensitivity-label-driven adaptive enforcement (different action by label) — Confidential = block, Highly-Confidential = block + alert with IRM signal-boost, Confidential-CHD = block + Conditional-Access-step-up reauth; per-app session policies tuned for individual SaaS UIs (Salesforce Lightning, ServiceNow); CAE (Continuous Access Evaluation) + Entra Token Protection in path to close the token-replay gap; per-app A/B testing of read-only fallback vs hard-block as a UX experiment; integration with Purview IRM for adaptive scoping where high-risk users get tighter enforcement automatically.

## Control mappings

- **CIS Microsoft 365 Foundations Benchmark v5.0.0 (30 April 2025):**
  - CIS 7.3.2 (L2) — device-posture-driven access restriction for SharePoint/OneDrive content
  - CIS 5.2.2.9 (L1) — Conditional Access policies for device compliance signals
  - CIS 1.3.2 (L2) — unmanaged-device access restriction baseline
  - CIS 2.4.3 (L2) — Defender for Cloud Apps as the supervisory plane for session controls
- **Microsoft Secure Score:** Apps group + Data group improvement actions (session-policy and label-driven controls). Cite [https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score) + [https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions). Current Secure Score structure is four groups (Identity / Device / Apps / Data); the legacy five-group structure (with Infrastructure) is retired.
- BNM RMiT clause(s): [BNM RMiT data leakage prevention](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current RMiT edition]` — illustrative only; not regulatory advice
- ISO 27017 control(s): [CLD.12.4.5 + ISO 27002 A.5.18 access rights](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): [A.10 (use, retention and disclosure limitation)](../../06-compliance/iso-27018.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions), `PR.DS-01` (data-at-rest), `PR.DS-02` (data-in-transit), `PR.PT-01` (audit logs) `[VERIFY]`
- PCI DSS Req. 7 (restrict access by need-to-know) where PCI data in scope; Req. 9 (physical access — interpreted as logical equivalent for SaaS-with-untrusted-endpoint) `[VERIFY against current edition]`
- MAS TRM (technology-risk management — data security on remote / BYOD endpoints) `[VERIFY]` — illustrative only

## False-positive risk

- Finance user on BYO laptop urgently needing a quarterly report — blocked. Mitigate by scoping initial rollout to pilot group; ensure exception path documented
- Senior exec on iPad-only setup — blocked; common cause of week-one rollback. Pair with read-only session policy
- Device tag empty due to CA evaluation not running on session-token-issuance branch — fails open or closed depending on `Always apply` setting
- Mislabelled non-sensitive content (e.g. an internal-comms newsletter with `Confidential` applied by error) blocked on BYOD legitimate use
- Approved-vendor / external-collaborator on tenant-issued B2B guest credentials accessing labelled content from their own managed-but-not-by-us device — appears as unmanaged from this tenant's view

## Real-world FP experience

*Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.*

Typical FP-rate trajectory in a tier-2 BFSI tenant rolling this out for the first time:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 (audit mode, pilot) | 35-50% | Mislabelled content; pilot users without managed device on day they need access; service-desk tickets from `*.mcas.ms` rewrite regressions (non-Edge browsers) |
| W4 | 20-30% | After exception register + read-only fallback live for pilot; service-desk runbook in place |
| W8 | 10-15% | After workforce comms + enrolment campaign; label-quality improvements on most-frequently-blocked content; per-app onboarding stable |
| W12 | 5-10% | Documented exception path mature; quarterly tuning cycle running; contractor-laptop scenario handled by group-scoped policy variant |
| Steady-state | 3-7% | Mature ops; per-label adaptive enforcement (Advanced maturity) brings this lower at the cost of operational complexity |

Named FP scenarios encountered repeatedly across deployments (practitioner observations; not Microsoft-documented):

| Scenario | Mitigation |
|---|---|
| Executive on iPad-only setup blocked from board-pack PDF | Intune MAM (App Protection Policies) on Outlook / OneDrive / Word Mobile; named exception for unmanaged-laptop fallback access via read-only fallback policy |
| Mislabelled internal-comms PDF (label = Confidential applied by error) blocked on BYO laptop | Label-quality remediation upstream (DLP-team workflow); time-bounded exception while remediation in flight |
| User's managed Surface in repair — accessing from home desktop for a legitimate urgent task | Documented exception-window procedure with manager + IT-security joint approval; max 24-hour exception; logged as a control deviation |
| Device tag empty / stale on first CA evaluation after device-state change | `Always apply if data cannot be scanned = false` produces fail-open (data leakage risk); document trade-off; run device-state freshness check via Intune dashboard quarterly |
| External B2B guest with managed home-tenant device, accessing labelled content in this tenant — device claim comes from home tenant (unmanaged from this tenant's view) | Cross-Tenant Access Settings (Entra) configured per partner; explicit B2B-guest exclusion or escalation path; document at partner-onboarding time |
| Contractor population legitimately needing read-only access on personal laptop | Group-scoped policy variant — read-only fallback applies, primary block does not produce a wholly-zero-access outcome |
| `window.parent.postMessage` regression on Salesforce Lightning after onboarding Salesforce to CAAC (non-Edge browsers) — entire app appears broken, not just the blocked-file scenario | Pre-CAAC compatibility test gate in W2; rollback runbook; Microsoft known-issues list for CAAC-app compatibility consulted at onboarding time; Edge for Business path avoids the URL-rewrite breakage class |
| Cert-pinned device profile on the user's own personal Mac rejecting `*.mcas.ms` chain — fails closed at TLS, looks like network outage at app layer | Triage runbook explicitly distinguishes TLS-pinning rejection vs policy block — different remediation paths |

## Operational cost

- **Exception-handling load:** medium initially (week 1-2 — 10-25 exceptions per week typical at a 1k-user pilot scope); declining as enrolment campaigns close gaps and workforce comms land; steady-state ~3-8 per week (practitioner observation; not Microsoft-documented)
- **Triage load:** low — most events are silent blocks; the alert volume from blocks is informational, not actionable per-event. The actionable triage is the regression-class events (sudden spike from a UI change to a CAAC-onboarded app)
- **End-user friction:** **high** for BYOD population unless paired with read-only session policy fallback. Service-desk ticket volume in W4-W8 is the dominant operational signal — if tickets don't decline by W8, the workforce-notice + enrolment campaign is failing and needs revisiting

Typical staffing: 0.3 FTE for the platform admin during the 10-week ramp (heavy on W2-W4 compatibility testing + W4-W6 user-comms iteration); 0.1 FTE steady-state. Service-desk has a temporary uplift in W4-W8 (~1-2 additional FTE-equivalent depending on tenant size). Insider-risk / SOC programme typically not staffed for this policy specifically — alerts feed into existing CASB queue.

## Privacy / data-protection considerations

- CAAC reverse-proxy of employee browser sessions = high-risk processing under GDPR Art. 35; DPIA mandatory in EU/UK/equivalent regimes. For PDPA-MY 2024 / HKPCPD / PDPA-SG, workforce-monitoring notice required in the employee handbook + AUP and must explicitly cover SaaS in-session monitoring + content inspection (illustrative; not legal advice)
- Audit log of the block decision lives in `CloudAppEvents` — itself regulated content (contains user identity, file metadata, decision verdict). Access governance on this evidence record matters (admin-group-restricted)
- The matched-content preview snippet (where Content Inspection is enabled) may contain regulated content (PCI cardholder data, PHI, PII). Snippet retention + access list must be documented; PCI DSS 3.4 / 3.5 controls apply where CHD is in scope; align with the [auto-label-pci-data](../dlp/auto-label-pci-data.md) snippet-handling decision
- Cross-border data flow: the CAAC proxy terminates at Microsoft's data region for the tenant. Verify against the current Microsoft Cloud regions page for the tenant primary-data region — cite Microsoft Learn `microsoft-365/enterprise/o365-data-locations` ([https://learn.microsoft.com/en-us/microsoft-365/enterprise/o365-data-locations](https://learn.microsoft.com/en-us/microsoft-365/enterprise/o365-data-locations)). Japan East as a primary-data region — `[VERIFY against current Microsoft Cloud regions page]`; material for MY / SG / HK BFSI tenants under cross-border-transfer regimes
- Personal-device session inspection has higher sensitivity than corporate-device inspection — the workforce-notice posture must distinguish; some jurisdictions require explicit opt-in for personal-device session inspection (consult local counsel)

## Integration with broader programmes

- **BNM RMiT audit cycle:** the policy config export + `CloudAppEvents` block-event sample + exception register feed the data-leakage-prevention attestation evidence pack `[VERIFY against current RMiT edition]`. Cadence: annual cycle, quarterly evidence refresh
- **PCI DSS audit cycle (where in scope):** PCI DSS Req. 7 (need-to-know access) + Req. 9 (logical-physical-access-equivalent for SaaS) — block-download-of-Confidential-CHD label is direct evidence. Quarterly pull; annual auditor pull
- **Board / executive reporting:** quarterly metric — "% of in-scope users on managed device" (driven by Intune compliance + this policy's audit log) + "block events per quarter" trend. Trend matters more than absolute number; spikes indicate either a label-quality issue or a control-evasion signal
- **Incident response runbook:** block events feed the data-exfil-attempt triage runbook. A spike in block events from a single user in a short window is a [mass-download-alert](../detect/mass-download-alert.md)-class signal — joint triage by SOC + IR
- **DPIA refresh:** every annual DPIA cycle includes a review of the CAAC scope, content-inspection scope, matched-snippet retention, and personal-device workforce-monitoring posture
- **BYOD AUP / workforce policy:** the policy is the technical enforcement of the BYOD AUP. AUP-side review (HR + Legal + DPO) and policy-side review (IT-security + Compliance) must be synchronised — drift between them produces a compliance gap
- **Intune MAM coverage decision:** native mobile bypass is the dominant coverage gap. The MAM (App Protection Policies) decision is **inseparable** from this policy — never deploy CAAC block without an explicit decision on the mobile-native pairing
- **Vendor / third-party risk:** the Microsoft CAAC proxy is a sub-processor relationship — sub-processor list, attestation pull (SOC 2 + ISO 27018), DPA review
- **Concentration risk register:** the CAAC proxy is an internet-scale T1557 target; promote as a concentration-risk item on the operational-risk register with compensating controls outside the Microsoft stack. **Attribution:** concentration-risk framing follows BNM RMiT third-party-concentration / DORA Art. 28-30 / MAS TRM supervisory framing — not Microsoft's own positioning, which is "unified XDR" / "end-to-end security" per MCRA April 2025. Surface both lenses in any board-pack derivation

## Anti-patterns specific to this policy

1. **Deploy CAAC block without the matching Entra CA policy.** Silent no-op on the MDA side; users see no behaviour change; auditor sees a "policy exists" record but no block events. The bilateral CAAC trap, item 1 of 10
2. **Deploy the Entra CA "Use Conditional Access App Control" without an MDA session policy.** The inverse silent no-op — non-Edge users see the `*.mcas.ms` rewrite (Edge users see direct in-browser path), the proxy is in path, nothing in-session enforces. Look for `CloudAppEvents` showing CAAC traffic but zero policy matches
3. **Block-mode rollout from day 1 without an audit-mode pilot week.** The W1 35-50% FP rate hits production users immediately; service-desk floods; rollback within 7 days; programme credibility damaged. Always run audit-mode for at least one week per pilot group
4. **Skip the `window.parent.postMessage` regression test on apps beyond Office 365.** For non-Edge browsers, Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview will break in non-obvious ways after CAAC onboarding. Test before scoping. Edge for Business removes the URL-rewrite breakage for the Edge subset but does not eliminate the underlying compatibility test discipline
5. **Pick individual M365 apps instead of the Office 365 bundle.** Inconsistent CAAC interception across Office workloads; user gets blocked in SharePoint but not OneDrive of the same labelled file; programme integrity collapses
6. **Set `Always apply if data cannot be scanned = true` without modelling the false-positive class.** Every legitimate Zip, OneNote, proprietary file format triggers a block; user friction reaches the unsustainable threshold within a week
7. **Set `Always apply if data cannot be scanned = false` without documenting the leak risk.** User-encrypted archives bypass the control entirely; data leakage via password-protected ZIP is the canonical exfil pattern; auditor finds it
8. **Deploy block-download with no read-only fallback.** Frames the policy as "BYOD = no access" rather than "BYOD = read-only access." Forces business workarounds (forwarding to personal email, saving to iCloud / Google Drive personal) — net data-leakage posture worse than before
9. **Forget Intune MAM for mobile native.** Native Outlook Mobile / OneDrive Mobile / Word Mobile / Teams Mobile bypass CAAC entirely. The policy advertised to the board as "blocks BYOD download" misses ~40-60% of the actual download surface (practitioner observation; not Microsoft-documented)
10. **No documented break-glass exclusion.** The "Microsoft Defender for Cloud Apps – Session Controls" Enterprise Application accidentally lands in a blanket Block-Access CA policy; every protected user locked out simultaneously; recovery requires Entra-side admin not affected by the policy — without a documented break-glass, the recovery path itself is locked
11. **Treat the executive iPad scenario as an "exception."** Executives accumulate per-user exceptions; the exception register becomes the actual policy; auditor finds it; control collapses on inspection. Either include executives in scope with MAM-paired iPad access, or document executive-cohort exclusion as a board-accepted residual risk
12. **No quarterly review.** Tenant population changes; new role families emerge (contractors, joint-venture partners, M&A absorbed staff); label taxonomy drifts; per-app CAAC compatibility regresses after vendor UI updates. Without quarterly review the policy degrades silently — the W12 5-10% FP rate climbs back to W4 levels within 12 months

## Coverage gaps

- [BYOD and unmanaged endpoint coverage gap](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — native mobile apps bypass entirely
- [SSL/TLS inspection breakage](../../08-failure-modes/ssl-tls-inspection-breakage.md) — cert-pinned managed-device profiles may reject `*.mcas.ms` chain (non-Edge path), fail closed at TLS but appear fail-open at app layer
- T1550.001 Application Access Token replay — bypasses device-tag filter entirely (token issued legitimately, replayed elsewhere). Pair with Entra Token Protection
- T1213 in-session screenshot — no technical control; visible-marking labels deter only
- Cert-pinned native desktop apps (Outlook desktop / OneDrive sync) bypass the CAAC reverse-proxy entirely — only browser sessions in scope
- CAE-induced session re-establishment may bypass MDA session-policy re-evaluation — pair with Entra Token Protection and sign-in-risk CA policies
- Encrypted-by-user archives (`T1027`) — the proxy sees `archive.zip`, cannot inspect content, cannot decide block-vs-allow on label match; either fails open (leak) or fails closed (FP-storm). See [`../../08-failure-modes/encrypted-upload-bypass.md`](../../08-failure-modes/encrypted-upload-bypass.md)
- Multi-tenant B2B guest sessions — device claim may come from the home tenant, not this tenant; Cross-Tenant Access Settings determine enforcement
- Apps not onboarded to CAAC — invisible to the policy; only API-mode (post-upload) detection available; not Prevention

## Three-lens sign-off

- **Architect:** _pending — to incorporate findings from MDA lens reviewers_
- **Product:** _pending_
- **Compliance:** _pending_

---

## Changelog

- **2026-06-10** — Microsoft-standards QA applied (verdict: P0 — three structural updates).
  - **X1 (Control mappings):** Added CIS Microsoft 365 Foundations Benchmark v5.0.0 (CIS 7.3.2, 5.2.2.9, 1.3.2, 2.4.3) + Microsoft Secure Score (Apps + Data groups) with Microsoft Learn citations.
  - **X8 (CAAC framing):** Reframed `*.mcas.ms` as the non-Edge reverse-proxy path; Edge for Business users get direct in-browser protection per Microsoft Learn `proxy-intro-aad`. Updated vendor grid, anti-patterns, coverage gaps, and W3 implementation-pattern line.
  - **X9 (Concentration risk):** Added explicit attribution sentence — concentration-risk framing is BNM RMiT / DORA Art. 28-30 / MAS TRM regulator language; Microsoft frames the same surface as "unified XDR / end-to-end security" per MCRA April 2025.
  - **X18 (FP-rate trajectory header):** Added practitioner-observation banner to the FP-rate trajectory table.
  - **X20 (Japan East primary-data region):** Replaced specific "Japan East added 2025 H2" claim with `[VERIFY against current Microsoft Cloud regions page]` marker plus Microsoft Learn `microsoft-365/enterprise/o365-data-locations` citation.
  - **File-specific — MDE Network Protection (Purview-Entra missing-citation #1):** Added cross-reference to MDE Network Protection as defence-in-depth pair for managed endpoints, with `[VERIFY]` on current Microsoft Learn URL.
  - **File-specific — SharePoint/Exchange "limited access" pattern (MCRA-ZT stale-reference #3):** Added cross-reference to the Microsoft pattern with `[VERIFY]` marker.
  - **Portal-path standardisation:** "Defender portal" → "Microsoft Defender Portal → Cloud Apps → Policies → Policy management" in vendor grid.
  - **MDA playbook reference:** Annotated "Policy 5" as internal numbering, not Microsoft documentation (X19).
  - **Practitioner-inference labelling:** Added explicit annotations on field-observation claims (W2 breakage class, mobile bypass share, exception-handling load, FP scenarios table).
  - **Regulatory citation discipline:** Added "illustrative only; not regulatory advice" caveats on BNM RMiT, PCI DSS, MAS TRM, PDPA references.
  - **Depth line:** Updated to "Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)".
