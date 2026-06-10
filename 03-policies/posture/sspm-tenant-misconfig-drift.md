# SSPM — tenant misconfiguration drift

> Status: v0.0 — adjacent to core CASB capability; CASB-light SSPM coverage; specialist SSPM tools have deeper coverage.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [API-mode + configuration-assessment integration](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector.
> MDA playbook reference: relates to App Governance posture features; not a numbered MDA Day 1/30/90 policy. (Internal numbering — not Microsoft Learn documentation.)
> **Microsoft architectural anchor:** Zero Trust Application pillar Objective VI ("Cloud applications are configured with secure controls") — see [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications]. SaaS Security Posture Management (SSPM) capability surfaced via Defender for Cloud Apps App Governance — see [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/saas-security-posture-management]. **Practitioner inference:** the specialist-SSPM-tool depth-vs-breadth framing (Adaptive Shield / AppOmni / Obsidian / Wing) is field judgment; Microsoft does not publish a comparative depth matrix.

## Purpose

Continuously assess the security configuration posture of sanctioned SaaS tenants (Salesforce sharing settings, M365 attack-surface configuration, ServiceNow admin settings, etc.) against a documented posture-rule library. Detect drift from baseline. Maps to MITRE ATT&CK `T1098.001 Additional Cloud Credentials`, `T1098.003 Additional Cloud Roles`, `T1556 Modify Authentication Process`, and the broader cloud-misconfiguration class that adversaries (Storm-0501, Midnight Blizzard) exploit post-compromise to entrench persistence. Microsoft canonical threat-actor names per [Microsoft Learn: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming]. **Important (practitioner inference):** CASB does this lightly — for deep coverage, dedicated SSPM tools (Adaptive Shield, AppOmni, Obsidian, Wing) are the right tier. Microsoft does not publish a depth-vs-breadth comparator.

## What organisations use this for

The honest framing: most regulated FIs do not deploy SSPM because they want to. They deploy it because an auditor, a post-incident review, or a board member surfaced "we have 47 SaaS tenants and nobody knows what's configured how" as an open finding. SSPM is **the SaaS-administration equivalent of CSPM** — a control surface that the network and endpoint teams have lived with for a decade, but which the SaaS-admin community (line-of-business owners, often outside central IT) is just learning. The deployment decision is rarely about the tool itself; it is about who owns the posture remediation backlog when the scans start firing, and that question is political before it is technical.

The CASB SSPM angle is specifically about whether the existing CASB licence covers enough of the in-scope SaaS estate to defer or eliminate the spend on a dedicated SSPM (Adaptive Shield / AppOmni / Obsidian / Wing). The answer is usually "no for the deep tenants, yes for the long tail" — most organisations end up with both, the CASB doing breadth and a specialist doing depth on the two or three crown-jewel SaaS tenants (typically Salesforce, ServiceNow, GitHub Enterprise).

### Use case 1 — Tier-1 ASEAN universal bank, post-Storm-2372-class OAuth cleanup

- **Org type:** large universal bank, ~30k employees, M365 E5 + MDA App Governance, Salesforce Financial Services Cloud as customer-relationship platform, ServiceNow as ITSM + GRC platform, BNM RMiT supervised
- **Trigger:** post-incident review after a Storm-2372-class device-code-phishing campaign successfully consented a malicious OAuth app into the bank's M365 tenant. The post-mortem surfaced that admin consent had been enabled tenant-wide, that the verified-publisher restriction was off, and that the same control-design issue existed across Salesforce (Connected App allowlist) and ServiceNow (OAuth provider config). The board ask: "tell us this won't happen elsewhere in our SaaS estate"
- **Scope:** four crown-jewel tenants — M365, Salesforce, ServiceNow, GitHub Enterprise; CASB SSPM enabled in alert-only for first 60 days; baseline captured per tenant; named tenant admin per app as remediation owner
- **Outcome:** within first scan cycle, surfaced 38 posture findings across the four tenants; 11 were classified P1 (admin-consent open in two SaaS tenants, public-link sharing enabled at Salesforce site level, branch-protection-bypass in GitHub, MFA-not-enforced for Salesforce admins); remediation backlog took 4 months to clear; ongoing scan cadence weekly, exception-update cycle quarterly. CASB SSPM kept Salesforce coverage shallow (sharing settings + admin roles only) — the bank later procured Adaptive Shield to get Salesforce profile and permission-set depth, ran CASB SSPM in parallel for breadth

### Use case 2 — Merchant acquirer pre-SOC 2 Type II + ISO 27001 surveillance audit

- **Org type:** payments processor operating across MY / SG / HK / TH, ~3k employees, mixed M365 + Google Workspace, Salesforce + Okta as identity, ~25 connected SaaS apps
- **Trigger:** annual SOC 2 Type II + ISO 27001 surveillance audit; previous year's auditor finding was "no continuous configuration assessment for sanctioned SaaS tenants — relying on annual self-attestation by tenant admins". Auditor wanted continuous evidence, not point-in-time
- **Scope:** all 25 connected SaaS in CASB coverage; posture-rule library aligned to CIS Microsoft 365 Foundations Benchmark + CIS Google Workspace Benchmark + vendor-specific baselines for Salesforce / Okta / GitHub
- **Outcome:** evidence pack for SOC 2 CC6.1 / CC6.6 / CC7.2 (continuous monitoring of configuration); ISO 27001 A.5.23 (information security for cloud services) [VERIFY] — auditor accepted CASB SSPM scan reports + monthly drift report + remediation-ticket evidence as primary control evidence. ~14 hours of analyst time per month for the cadence vs ~80 hours of pre-audit "config-attestation chase" the previous year. Trade-off: continuous evidence in exchange for a permanent remediation backlog that has to be reviewed; the org chose evidence over apparent tidiness

### Use case 3 — Digital-native challenger bank — SaaS-administration sprawl across BU-managed tenants

- **Org type:** neobank, ~600 employees, organisationally federated — each product BU (Cards, Lending, Wealth, Treasury) runs its own SaaS tenants for marketing automation, customer support, and analytics; ~40 SaaS tenants tenant-tenant-overlap with no central inventory
- **Trigger:** new CISO discovered that no central team owned posture for ~30 of the 40 tenants; each tenant had a BU admin who could (and did) change sharing defaults, MFA enforcement, and external-collaboration settings without notifying security
- **Scope:** initially the top 10 tenants by user-count + the four with documented sensitive-data flows; CASB SSPM module enabled where coverage exists (~6 of 10 in CASB catalogue); rest deferred to dedicated SSPM POC
- **Outcome:** central security team gained visibility into 6 of 10 prioritised tenants within 30 days; surfaced ~70 P1/P2 findings on first scan; the *political* outcome — establishment of a SaaS-admin governance forum with named tenant owners — was the bigger win. CASB SSPM was the wedge to force the conversation, not the solution itself. Eventually procured AppOmni for the four deep tenants; CASB SSPM retained for breadth

### Use case 4 — Insurance carrier paired with dedicated SSPM tool decision

- **Org type:** mid-cap insurance carrier, ~5k employees, Microsoft-stack-heavy (M365 E5 + MDA + Defender XDR + Purview), Guidewire on cloud, Workday for HR, ServiceNow for IT + claims-ops workflow, BNM RMiT + MAS Insurance Act supervised
- **Trigger:** capital programme requested a dedicated SSPM POC (Adaptive Shield vs AppOmni vs Obsidian); CASB-team pushback was "before we buy a third tool, prove that CASB SSPM doesn't cover the use cases". Decision posture: use CASB SSPM as the null hypothesis, procure specialist SSPM only on demonstrated gap
- **Scope:** CASB SSPM enabled on all in-coverage SaaS for 90 days; gap analysis against named posture-rule classes from the SSPM-vendor POCs (Salesforce profile/permission-set drift, ServiceNow ACL drift, GitHub branch-protection coverage, Okta sign-on-policy drift)
- **Outcome:** CASB SSPM covered ~40% of the rule classes that the specialist SSPM POCs covered, but at ~5% of the depth on the crown-jewel tenants. Procurement decision: AppOmni for Salesforce + ServiceNow + Workday; CASB SSPM retained for the M365 + GitHub + Okta + the long tail. Total cost ~30% lower than going specialist-only for all tenants; coverage on crown jewels equivalent. Trade-off the carrier accepted: two scan engines, two consoles, two evidence formats — operational overhead in exchange for cost. The board ask was "show us evidence we made an informed decision" — the parallel-run delivered that

## Implementation pattern

Typical 10-week rollout sequence for a tier-2 BFSI tenant enabling CASB SSPM for the first time:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Tenant inventory — enumerate sanctioned SaaS in scope; map to CASB-SSPM-coverage list; identify named tenant admin per app | Coverage matrix signed off; out-of-coverage tenants flagged for specialist-SSPM decision |
| W2 | Posture-rule baseline design — select rule library (CIS benchmarks per app where they exist; vendor-recommended baseline elsewhere); document deviations as documented exceptions | Posture-rule library v1; exception register opened |
| W3 | Connector configuration — enable CASB SSPM connector per tenant; verify scan completes; pull first scan report | First scan report; per-tenant findings count baseline |
| W4 | Triage of first-scan findings — classify P1 / P2 / P3 by tenant admin in partnership with security; map to remediation owners | Prioritised remediation backlog; remediation SLA agreed (typical: P1 30 days, P2 90 days, P3 best-effort) |
| W5-W6 | P1 remediation sprint — close highest-impact findings (admin-consent posture, MFA enforcement, public-sharing defaults, OAuth allowlists, break-glass account hygiene) | P1 backlog burned down; documented post-remediation re-scan |
| W7-W8 | Drift-detection cadence — scan cadence set (typical daily for crown jewels, weekly for long tail); alert-routing to tenant admin + central security; exception-approval workflow operationalised | Drift cadence live; first drift alert handled end-to-end |
| W9 | Audit-evidence pull tested — quarterly evidence pack format finalised; mapping to SOC 2 / ISO / RMiT clauses documented | Reusable evidence template |
| W10 | Steady-state operating handoff — central security retains oversight; tenant admins own remediation; monthly drift report to CISO; quarterly executive metric | Steady-state ops handoff signed |
| W11+ | Quarterly posture-rule library review; tenant-add process; specialist-SSPM-gap reassessment annually | Quarterly governance review minutes |

The W2 posture-rule library design is the hardest step. Off-the-shelf benchmarks (CIS) are a starting point but not a baseline — every regulated FI has documented exceptions (e.g. external-sharing enabled for specific partner-collaboration sites; SAML-only enforcement on a different cadence per BU). The library has to encode those exceptions explicitly or the W4 triage drowns in noise.

## Action

- Primary: **alert** on configuration drift + propose remediation; route to the named tenant admin
- Secondary: **track** in posture-management runbook with named owners per tenant; exception register for documented deviations
- **Not standard:** auto-remediate. CASB SSPM remediation depth is typically alert-only; configuration changes are made by the tenant admin via the SaaS console, not by the CASB

## Scope

- **Users:** N/A (configuration-attribute scan)
- **Apps:** sanctioned SaaS with configuration-assessment connectors — typical CASB coverage: M365, Salesforce, ServiceNow, GitHub, Okta, Box, Dropbox, Zendesk (vendor-dependent)
- **Device posture:** N/A
- **Network position:** N/A
- **Traffic:** at-rest configuration scan

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → App governance → Security configuration assessment (subset of connected apps). Coverage list per [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/saas-security-posture-management]. | Apps covered = M365, Salesforce, ServiceNow, GitHub, Okta, Box, Dropbox, Zendesk (per Microsoft Learn `saas-security-posture-management`; **[VERIFY against current SSPM coverage list]** at attestation time); Posture rules = Microsoft Secure Score-derived ([Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score]; improvement actions per [https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]) + custom rules where supported; scan cadence = vendor-set, typically daily | Depth below SSPM specialists; coverage breadth limited; remediation actions limited to manual workflow | CASB SSPM is **shallow** (practitioner inference; Microsoft does not publish depth comparator) — for tenant-deep posture (Salesforce profile/permission-set configuration, GitHub branch protection, M365 attack-surface), specialist SSPM is required. Drift-detection cadence daily at best, not real-time. **Practitioner hedge (not Microsoft-documented):** Secure Score-derived rules may drift in their underlying logic without operator notification — pin the rule version in the evidence pack. Microsoft documents Secure Score improvement-action changes via release notes only |
| Netskope | `[unverified]` — Netskope SSPM module (separate licence) | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security posture | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB + posture | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security posture | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration for a tier-2 ASEAN BFSI tenant after 8 weeks of operating CASB SSPM across four crown-jewel apps:

```yaml
sspm_policy:
  name: "SSPM — crown-jewel-tenant baseline"
  scope:
    tenants:
      - app: M365
        tenant_id: "tenant-id-redacted"
        baseline: "CIS-M365-Foundations-v3.0"
        scan_cadence: daily
        owner: "central-it-security"
      - app: Salesforce
        org_id: "00D-redacted"
        baseline: "CIS-Salesforce-v1.0 + internal-banking-baseline-v1.2"
        scan_cadence: daily
        owner: "salesforce-platform-team"
      - app: ServiceNow
        instance: "prod-instance-redacted"
        baseline: "internal-itsm-baseline-v1.0"
        scan_cadence: weekly             # less frequent — config change rate lower
        owner: "itsm-platform-team"
      - app: GitHub Enterprise
        org: "org-redacted"
        baseline: "CIS-GitHub-v1.0 + internal-source-baseline-v1.1"
        scan_cadence: daily
        owner: "devsecops-team"
  posture_rules:
    enabled_rule_classes:
      - admin-consent-and-oauth-hygiene  # P1 — addresses T1528 / T1098.001
      - mfa-enforcement-and-coverage     # P1 — addresses T1078.004 / T1556
      - external-sharing-defaults        # P1 — addresses T1213 / T1567
      - guest-and-b2b-access-control     # P2 — addresses T1199
      - audit-log-configuration          # P2 — control evidence integrity
      - break-glass-account-hygiene      # P1 — recovery posture
      - api-token-and-pat-lifetime       # P2 — addresses T1098.001
    custom_rules:
      - id: "internal-banking-001"
        description: "Salesforce profile 'Standard User' must not have ExportReports permission"
        severity: P1
        applies_to: Salesforce
      - id: "internal-banking-002"
        description: "GitHub org default branch-protection must include required-reviewers >= 2"
        severity: P1
        applies_to: GitHub
  exceptions_register:
    - rule_id: "external-sharing-defaults"
      tenant: M365
      site: "/sites/partner-collaboration-XYZ"
      justification: "Documented partner-collab site; reviewed quarterly by Vendor Risk; sunset 2027-Q2"
      owner: "vendor-risk-team"
      review_cadence: quarterly
      next_review: "2026-09-01"
  alerts:
    drift_alert:
      severity_routing:
        P1: ["central-security@example.com", "<tenant-admin-list>"]
        P2: ["<tenant-admin-list>"]
        P3: log-only
      remediation_sla:
        P1_days: 30
        P2_days: 90
        P3: best-effort
    new-finding-alert:
      first-scan-suppression: true       # avoid alert-storm on first scan; rely on backlog triage instead
  evidence:
    monthly_drift_report: true
    quarterly_audit_pack: true
    retention_years: 7                   # BFSI baseline; align with audit/SOX/RMiT
```

The `first-scan-suppression: true` flag matters — without it, the first scan against a tenant with 12 months of unmanaged drift generates 200+ findings as drift events, which destroys the alert channel before steady-state operations begin. Treat the first scan as a backlog-creation event, not a drift event.

## Variants

### Industry-specific

- **BFSI:** front-and-centre clauses are BNM RMiT (cloud services + third-party risk + concentration risk), MAS TRM Annex C (cloud usage), HKMA SA-2, EU DORA Art. 28 (ICT third-party risk), all `[VERIFY]`. Crown-jewel tenants are usually Salesforce (customer + product data), ServiceNow (ITSM/GRC), Workday (HR — paired with payroll), GitHub (regulated source). Public-link sharing and admin-consent posture are the dominant P1 finding classes. Remediation SLA tighter than other verticals — 14-30 days P1 is common
- **Healthcare:** dominant tenants are Epic / Cerner (rarely in CASB SSPM coverage — usually specialist tool), workforce-management SaaS, and Salesforce Health Cloud where used. HIPAA Security Rule mapping (45 CFR 164.308/312); the patient-data-sharing posture is the dominant P1 finding class. Audit-log-integrity posture is critical for breach-notification timing
- **Tech / SaaS:** GitHub / GitLab / Bitbucket as crown jewels; the source-code-protection posture (branch protection, secret-scanning, dependency-review) is dominant. Customer-SaaS-tenant posture if the firm itself is a SaaS provider (i.e. SSPM on your *customer-facing* tenants, not just your *internal* SaaS estate)
- **Retail:** loyalty-platform tenants, marketing-automation (Salesforce Marketing Cloud / Adobe / HubSpot), e-commerce SaaS. Customer-data-sharing posture + payment-integration posture (PCI DSS scope-creep risk) are the dominant classes. High volume, lower depth — SSPM is breadth play here
- **Public sector / legal:** government-classification overlay (Official / Sensitive / Secret) on sharing settings; legal-privilege protection on document-management SaaS (iManage, NetDocuments — typically not in CASB SSPM coverage); audit-log integrity for matter-records preservation

### Maturity-based

- **Immature deployment** looks like: one or two tenants in CASB SSPM, no posture-rule library beyond vendor defaults, no exception register (so legitimate deviations fire as findings every scan), no tenant-admin remediation ownership (central security holds the entire backlog), no scan-cadence rationale, no audit-evidence pull tested. Common at 6 months post-deployment in under-resourced programmes. Findings accumulate; nobody closes them; the tool becomes shelfware
- **Mature deployment** looks like: 6-10 tenants under scan with documented posture-rule library, named tenant-admin per app, exception register with quarterly review, P1/P2/P3 severity-routing operational, remediation SLA tracked, monthly drift report to CISO, quarterly audit-evidence pack signed by tenant admins. Specialist-SSPM gap analysis documented (i.e. the org has decided what CASB SSPM cannot cover and how that gap is filled)
- **Advanced deployment** looks like: posture findings feed Purview Insider Risk Management as boosted signals (admin-role-change anomalies, OAuth-grant velocity, break-glass-account usage anomalies); SSPM signal correlated with CASB activity-policy signal (e.g. mass-download after an admin-role-change is a higher-confidence insider-risk signal than either alone); posture rules version-pinned and the evidence pack states the rule version per finding; remediation auto-orchestrated via Power Automate / ServiceNow workflow for the standard finding classes (still alert + ticket, not auto-config-change); specialist SSPM (AppOmni / Adaptive Shield / Obsidian) deployed on crown-jewel tenants with CASB SSPM doing breadth coverage in parallel

## Control mappings

**Primary Microsoft-native baselines (auditor-defensibility critical — both must appear in every evidence pack):**

- **CIS Microsoft 365 Foundations Benchmark v5.0.0 (30 April 2025)** — umbrella mapping CIS 2.4.3 (L2) "Ensure Microsoft Defender for Cloud Apps is configured" covering the SSPM capability set; per-control sub-mappings for tenant-specific rules vary by app (see file-specific posture-rule library). Microsoft Cloud App Security is the documented enforcement mechanism for CIS-specified posture controls. Source: CIS Benchmarks (downloads.cisecurity.org).
- **Microsoft Secure Score** — Secure Score Apps + Identity + Data + Device groups (four-group structure; legacy five-group structure with Infrastructure is retired). SSPM rules are Secure Score-derived plus custom rules. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score] and [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]. Secure Score is the Microsoft-native posture-metric that auditors will probe for; evidence pack must include the score trend + improvement-action status per scan cycle.
- **Microsoft Zero Trust Application pillar Objective VI** ("Cloud applications are configured with secure controls") — [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications]. Architectural anchor for SaaS posture; Defender for Cloud Apps SSPM is the Microsoft-documented enforcement mechanism.
- **Microsoft SaaS Security Posture Management (SSPM) overview** — [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/saas-security-posture-management] — supported-app coverage list and posture-rule mechanics.

**Regulatory and standards mappings (illustrative only; not regulatory advice):**

- BNM RMiT clause(s): [BNM RMiT cloud services + third-party risk + concentration risk](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]`
- MAS TRM Annex C (cloud usage) `[VERIFY]`
- ISO 27017 control(s): [CLD.6.3.1 shared responsibility, CLD.12.1.5 administrator's operational security, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27001:2022 A.5.23 (information security for cloud services), A.8.9 (configuration management), A.8.16 (monitoring activities) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `ID.AM-04` (external systems inventory), `PR.IP-01` (baseline configuration), `PR.IP-03` (configuration change control), `DE.CM-08` (vulnerability scans), `GV.SC-04` (third-party suppliers) `[VERIFY]`
- SOC 2 Trust Services Criteria: CC6.1 (logical access), CC6.6 (system configuration), CC7.2 (system monitoring), CC8.1 (change management) `[VERIFY]`
- PCI DSS Req. 2.2 (secure configuration), Req. 7 (access control), Req. 12.8 (third-party service-provider management) where in-scope

## False-positive risk

- Legitimate config differences across tenants in multi-tenant federation (e.g. one tenant is sandbox, the other production; posture rules applied uniformly)
- Recently-applied admin changes pending baseline update — change made, scan fires, baseline not yet updated
- Vendor-side default changes propagating before posture rule update (Microsoft / Salesforce / etc. shift a default; the firm's posture rule has not caught up)
- Tenant-design-time exceptions encoded informally but not in the exception register (e.g. "we always leave this sharing setting on for the partner-collab site")
- Documented business exceptions waiting in approval queue
- Rule-library version drift — the underlying rule logic changes silently between CASB releases; findings appear/disappear without a config change

## Real-world FP experience

Typical FP-rate trajectory in a tier-2 BFSI tenant new to CASB SSPM. **Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.**

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 60-80% | First-scan-overshoot — every documented-but-unrecorded exception fires; posture-rule library has not been tuned for the firm's exception register |
| W4 | 25-40% | After exception-register v1 populated; first round of remediation closed real findings; remaining noise = vendor-default-shift + multi-tenant-federation differences |
| W8 | 12-20% | After posture-rule library tuned for the firm's documented exceptions; per-tenant baseline customised; remediation SLA running |
| W12 | 6-12% | Mature exception register; quarterly review cycle catching new noise; tenant admins owning the remediation conversation |
| Steady-state | 4-8% | New SaaS-vendor default shifts + quarterly rule-library updates + new BU-tenant onboarding generate the residual FP rate; pattern is steady, not declining further |

Named FP scenarios encountered repeatedly across deployments:

| Scenario | Mitigation |
|---|---|
| Documented partner-collaboration site has external-sharing enabled; fires as P1 finding every scan | Exception register entry with named owner + sunset date; rule excludes the named site path |
| Sandbox / non-prod tenant flagged against production baseline | Per-tenant baseline (sandbox baseline vs prod baseline); document the difference rather than ignore it |
| Microsoft Secure Score-derived rule changes silently between CASB releases — new findings appear without a tenant-side config change | Pin posture-rule library version in the evidence pack; quarterly rule-library review documents which rules changed and why |
| Salesforce permission set drift driven by a legitimate Connected App install | Pre-change notification process between Salesforce platform team and CASB SSPM owner; post-change baseline update within 7 days |
| Break-glass account flagged for not having MFA — this is the documented design for break-glass under DORA / RMiT (no MFA on the recovery-of-last-resort) | Named exception with quarterly review; documented compensating controls (vault + dual-control + monitoring of every use) |
| New BU-tenant onboarding fires hundreds of P1s on first scan (admin-consent enabled, public sharing on, OAuth allowlist not configured) | Pre-onboarding hardening checklist; first scan suppressed; remediation backlog triaged before drift cadence activates |
| Microsoft / Salesforce / etc. default-change rollout (vendor shifts a default platform-wide) propagates as drift across all tenants on the next scan | Subscribe to vendor release notes; pre-update the posture-rule library; quarterly rule-library review catches what was missed |
| Documented "two-person admin pattern" enforced via two named accounts flagged as "shared/excessive admin count" | Custom rule replacing the generic admin-count rule; documented design exception |

## Operational cost

- **Exception-handling load:** medium — every config drift is either a baseline update (i.e. exception-register entry) or a remediation ticket. Typical 5-15 exceptions per week during W4-W12 ramp; 2-5 per week steady-state. The exception-register quarterly review is itself ~4-8 hours per quarter
- **Triage load:** low — posture alerts are not real-time SOC events; admin-team review cadence. Typical 20-50 drift events per week across 6-10 tenants steady-state, of which ~5-10 require action
- **End-user friction:** N/A directly; indirect friction when a tenant admin is asked to change a setting that a BU has been relying on (the political conversation, not the technical one)

Typical staffing: 0.5 FTE central SSPM owner during the 10-week ramp (rule-library design + exception register + remediation orchestration); 0.2 FTE steady-state. Per-tenant admin commits 0.1-0.3 FTE during ramp (varies by tenant size); negligible steady-state once drift cadence is healthy. Audit-evidence quarterly pull: 0.05 FTE for the audit-team coordinator.

## Privacy / data-protection considerations

- Configuration metadata typically not personal data; access control on the SSPM scan results still required (the scan reveals who has what privilege, which is sensitive workforce information in many jurisdictions)
- For PII-handling SaaS, drift in sharing-controls or external-access settings is a privacy-grade event — a finding of "public link sharing enabled on tenant containing PII" is itself a DPIA-relevant signal
- Cross-border-transfer consideration: the CASB SSPM scan runs in the CASB vendor's cloud region. For Malaysian / Singapore / Hong Kong tenants, document where the scan is executed vs where the tenant data resides
- The posture-rule library + scan results + remediation tickets are themselves regulated control evidence under BNM RMiT / equivalent; treat the SSPM platform itself as in-scope for vendor-risk assessment
- Workforce-monitoring overlay: if SSPM is used to monitor *admin behaviour* (e.g. admin-role-change frequency, break-glass-account usage), the GDPR Art. 88 / PDPA-MY 2024 workforce-monitoring notice scope applies

## Integration with broader programmes

- **Audit cycles:** quarterly drift report + remediation evidence feeds SOC 2 CC6.1 / CC6.6 / CC7.2 / CC8.1; ISO 27001:2022 A.5.23 / A.8.9 / A.8.16; BNM RMiT cloud-services + third-party-risk clauses [VERIFY]. Annual auditor pull is the reusable evidence pack from W9
- **Board / executive reporting:** quarterly metric — "% of crown-jewel-tenant posture rules in green state" (trend more important than absolute); "P1 findings open beyond SLA" as a single number; named P1 findings open >60 days as narrative; tenant-admin remediation-rate by BU as accountability lens
- **Incident response runbook:** SSPM drift findings feed the IR scoping signal — if an OAuth-grant or admin-role-change finding precedes an alert from another control (mass-download, impossible-travel), the SSPM finding becomes corroborating context. The incident commander's runbook for cloud-tenant-compromise should reference the SSPM scan history as a first-15-minute pull
- **Vendor / third-party risk:** SSPM-scan-derived posture status per SaaS vendor feeds the vendor-risk register's residual-risk score; vendor-risk-team uses SSPM evidence in the annual vendor reassessment
- **DPIA refresh:** annual DPIA cycle reviews the posture-rule library scope (which tenants, which rules), the cross-border-scan posture, and the workforce-monitoring overlay
- **Adaptive Protection (Purview IRM):** in advanced deployments, admin-role-change anomalies and break-glass-account-usage anomalies feed Purview IRM as boosted insider-risk signals. Adaptive Protection now wires into DLP, DLM, and Conditional Access (not just CA). Cite [Microsoft Learn: https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection]. See internal-playbook MDA Policy 12 (IRM signal-boost integration) — *internal numbering, not a Microsoft-documented policy name*
- **Specialist-SSPM-tool decision:** the annual procurement / renewal cycle reviews the CASB SSPM coverage gap; the gap analysis is the input to the decide-keep / decide-procure-specialist / decide-replace conversation

## Anti-patterns specific to this policy

1. **"Run the first scan and treat every finding as drift"** — the first scan is a *backlog-creation* event against ambient drift accumulated over months/years. Treating it as a drift event swamps the alert channel before steady-state can establish. Suppress first-scan alerts; build the backlog; then activate drift cadence
2. **"No exception register"** — the legitimate exceptions (partner-collab sites, sandbox tenants, break-glass accounts, documented business deviations) fire every scan. Without a documented exception register, the FP rate stays >40% indefinitely and the analyst team learns to ignore findings
3. **"Use vendor-default posture rules without firm-specific tuning"** — Microsoft Secure Score / CIS benchmark / vendor-defaults encode opinions that don't match the firm's documented operating model. Findings fire that the firm has explicitly chosen to accept. Rule-library tuning is the work, not the deployment
4. **"Hold the entire remediation backlog in central security"** — tenant admins (Salesforce platform team, ServiceNow team, GitHub admins) own the configurations; if central security holds the backlog, remediation never happens because the people who can fix it aren't accountable. Backlog ownership must follow tenant-admin ownership
5. **"Run SSPM in alert-only forever"** — alert-only is a 60-day posture, not a deployment model. After 60 days, the drift cadence + remediation SLA + tenant-admin accountability must be live. Otherwise the tool reduces to a dashboard nobody acts on
6. **"Buy CASB SSPM and assume it covers the specialist-SSPM gap"** — CASB SSPM is breadth, not depth. Crown-jewel tenants (typically Salesforce + ServiceNow + GitHub + Workday at depth) need specialist SSPM. Treating CASB SSPM as a substitute creates an evidence-gap audit finding within 12 months
7. **"Ignore vendor-side default-changes"** — Microsoft / Salesforce / Google shift defaults; rules that worked yesterday fail today, or vice versa. Without a vendor-release-notes subscription + quarterly rule-library review, the posture state silently drifts even when the firm's config doesn't change
8. **"Pin a snapshot baseline and never update it"** — the inverse failure mode. Baselines decay; vendor capabilities expand; new controls become available. A baseline that hasn't been reviewed in 18 months is no longer a baseline, it's an artefact
9. **"Suppress findings instead of remediating or excepting them"** — the worst posture-debt accumulator. Suppressed findings are invisible at audit time but the underlying misconfiguration persists. Force every finding to either remediate / except-with-justification / accept-with-residual-risk-acceptance — no silent suppression
10. **"Use SSPM scan results without version-pinning the rule library"** — when the auditor asks "what was the posture state on date X under what ruleset", the firm must be able to answer. Without rule-library version pinning per finding, the evidence pack is questioned and the control is treated as unreliable

## Coverage gaps

- Apps not in the CASB's SSPM coverage list — many smaller SaaS unsupported; the long-tail visibility problem persists
- Configuration-detection lag — drift may exist for hours-to-days before scan (real-time posture-change detection generally not available via CASB SSPM; specialist SSPM tools claim near-real-time on some apps)
- Remediation depth — most CASB SSPM is alert-only, not auto-remediate. Specialist SSPM tools (Adaptive Shield, AppOmni, Obsidian, Wing) have deeper remediation; verify before procuring on CASB-only coverage assumption
- Configuration-as-code / IaC drift on SaaS (Salesforce sfdx, ServiceNow update-sets, GitHub repo-as-code) — outside SSPM's scope; needs separate posture-as-code tooling
- Tenant-vendor-side admin actions (e.g. Microsoft makes a tenant-affecting change via Microsoft service principal) — typically invisible to SSPM
- Posture of sub-tenancies (Salesforce sandboxes, GitHub forks, M365 multi-geo regions) — coverage varies by app and CASB vendor
- Identity-provider posture (Okta / Ping / ADFS) — partial CASB SSPM coverage; Okta-as-primary-IdP posture often best handled by Okta's own ITP / Workforce Identity Posture or specialist IdP-posture tooling
- See also [`../../08-failure-modes/api-mode-is-not-prevention.md`](../../08-failure-modes/api-mode-is-not-prevention.md) — SSPM is detection + remediation-orchestration, not prevention of the misconfiguration itself

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
