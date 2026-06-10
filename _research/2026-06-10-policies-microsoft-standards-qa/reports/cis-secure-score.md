# CIS Microsoft 365 Foundations Benchmark + Microsoft Secure Score QA expert - QA report

Reviewed against:

- CIS Microsoft 365 Foundations Benchmark **v5.0.0** (30 April 2025) — current edition at time of review.
- Microsoft Secure Score (Defender XDR) documentation, `ms.date: 2026-03-07`, updated 2026-06-01.
- Microsoft Secure Score Improvement Actions documentation, `ms.date: 2025-04-28`, updated 2026-05-03.

Important sourcing note: the CIS Benchmark itself is paywalled in download form on `cisecurity.org`. The v5.0.0 PDF used as the source of truth here was retrieved from a public mirror (`itsecure.hu`) — content is the official CIS document but practitioners should re-verify any control numbering against the CIS-issued copy before citing in audit evidence.

The benchmark uses a two-level profile (**L1** baseline, **L2** highly-secure). Tier-2 in the **policy library** is a different construct (deep-dive vs starter-kit) — not a CIS construct. Throughout this report, "Tier 2" refers to the policy-library depth label; "L1 / L2" refers to the CIS implementation tier.

---

## Summary

| Category | Count |
|---|---|
| Files reviewed | 19 |
| Drift requiring correction | **11** (specific CIS-control / Secure-Score citations to add or fix) |
| Stale references | 3 (rename / sub-category / category split) |
| Missing citations (CIS or Secure Score URL needed) | **19 of 19** — no policy in the library currently cites a CIS control number or a Secure Score improvement-action ID |
| Practitioner-inference flags (kept as-is, label needed) | 6 |
| Verified-clean claims | 14 |
| Policies needing significant rework | 0 (the gap is uniformly the absence of CIS / Secure-Score mapping, not factual error) |

The single biggest finding: **across the entire 19-file library, the `Control mappings` blocks cite BNM RMiT, ISO 27017/27018, NIST CSF 2.0, SOC 2, and PCI DSS, but never once cite the CIS Microsoft 365 Foundations Benchmark or Microsoft Secure Score**. That is a systematic omission — every one of these policies maps to at least one CIS L1 or L2 control, and roughly half map to a named Secure Score improvement action. The library presents itself as the practitioner reference for an MDA-anchored CASB rollout while omitting the two Microsoft-native baselines that the QSA, the BNM examiner, and the SOC 2 Type II auditor will all probe.

---

## Verified-clean claims

Claims in the policy files I confirmed match Microsoft authoritative sources (CIS v5.0.0 and Microsoft Secure Score docs):

1. **CIS 2.4.3 enables MDA as the CASB anchor of the M365 baseline at L2.** Several policies correctly describe MDA as the inspection / discovery / governance plane underneath Day-1 / Day-30 / Day-90 controls. Consistent with CIS 2.4.3 (L2, Manual): *"Microsoft Defender for Cloud Apps is a Cloud Access Security Broker (CASB)"*.

2. **`block-unsanctioned-app-with-coach.md` line 80**: MDA Cloud discovery + auto-tagging + MDE Network Protection in block mode. Consistent with CIS 2.4.3 remediation guidance (*"Cloud Discovery > Microsoft Defender for Endpoint"* + *"Enforce app access"*).

3. **`block-download-unmanaged-device.md` line 82**: bilateral CAAC trap (Entra CA policy + MDA session policy). Architecturally correct per Microsoft `defender-cloud-apps/proxy-intro-aad` documentation.

4. **`inline-upload-block-regulated-data.md` line 83**: `Always apply if data cannot be scanned` documented as a control-design trade-off rather than a tickbox. Matches Microsoft's documented session-policy semantics.

5. **`api-data-at-rest-quarantine.md` line 83**: "Only the governance action of the first triggered policy is guaranteed to be applied" — quoted accurately from MDA File Policy documentation.

6. **`api-data-at-rest-quarantine.md` line 83**: File-policy ceiling = 200 per tenant. Correct per current MDA docs (verify date of attestation).

7. **`auto-label-pci-data.md` line 78**: Purview DCS scanning + sensitivity-label-apply on file policy. Consistent with CIS 3.3.1 (L1, Manual) *"Ensure Information Protection sensitivity label policies are published"*.

8. **`auto-label-pci-data.md` line 226**: File-policy ceiling = 200 per tenant. Verified.

9. **`external-share-link-quarantine.md` line 81**: SharePoint multi-geo *"supported only for OneDrive"* — accurate per current M365 connector documentation.

10. **`mass-download-alert.md` line 80**: 200k/day or 100k/3h activity-policy auto-disable thresholds — verified against MDA activity-policy documentation.

11. **`mass-download-alert.md` line 80**: Hybrid-AD `onPremisesSyncEnabled=true` silent-fail of Entra "Suspend user" governance action. Verified — Microsoft does not loop the writeback when an Entra-only delete fires against a hybrid object.

12. **`impossible-travel-alert.md` line 83**: Built-in anomaly policy is *edit, do not create*. Verified per `defender-cloud-apps/anomaly-detection-policy` docs.

13. **`post-consent-cleanup-and-ban.md` line 85**: Application (client-credentials) grants are NOT in the OAuth-apps view. Verified — Microsoft docs confirm `Cloud apps → OAuth apps` covers delegated-permission grants only.

14. **`discovery-and-auto-unsanction.md` line 11 + line 76**: Sub-category split (LLM / AI Coding Assistant / Image / Video / Voice-Avatar) — verified against current MDA Cloud Discovery category taxonomy. The policy correctly warns against using the monolithic "Generative AI" category.

---

## Drift to correct

Every item below is a specific edit to apply. Format: file:line — claim → correction.

### D1. CIS 2.4.3 not cited anywhere

**Affected:** all 19 files, in the `Control mappings` block.

**Claim:** every file maps to BNM / ISO / NIST CSF but none names CIS 2.4.3.

**Microsoft source:** CIS v5.0.0 §2.4.3 (L2, Manual) *"Ensure Microsoft Defender for Cloud Apps is enabled and configured"* is the foundational benchmark control under which every MDA-anchored policy in this library logically sits.

**Correction:** add a `CIS Microsoft 365 Foundations Benchmark v5.0.0` row to the `Control mappings` block in every file, citing 2.4.3 (L2) as the umbrella control plus the file-specific sub-control(s) from D2-D11 below.

---

### D2. `block-unsanctioned-app-with-coach.md:151-154`

**Claim:** maps only to BNM RMiT / ISO 27017 CLD.6.3.1 / NIST CSF GV.SC / SOC 2.

**CIS mapping (missing):**

- **CIS 1.3.4 (L1)** *"Ensure 'User owned apps and services' is restricted"* — the Entra-side preventive control whose detective complement is the MDA discovery feed this policy describes.
- **CIS 5.1.2.2 (L2)** *"Ensure third party integrated applications are not allowed"* — adjacent control on the SaaS-app-sanction question.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** Improvement actions under **Apps** group — *"Defender for Cloud Apps: Sanction or unsanction cloud apps"* and *"Block unsanctioned cloud apps"* (App governance + Defender for Cloud Apps product). Practitioner-tier alignment: **L2** (Cloud Discovery requires Defender for Cloud Apps E5 SKU).

**Correction:** add CIS rows + Secure Score row to the mappings block.

---

### D3. `tenant-restriction-corporate-only.md:182-187`

**Claim:** maps to BNM RMiT 10.x / 11.x / MAS TRM / ISO 27017 / NIST CSF / PCI DSS.

**CIS mapping (missing):**

- **CIS 5.1.6.2 (L1)** *"Ensure that guest user access is restricted"* — Entra Cross-Tenant Access Settings + B2B-guest restriction.
- **CIS 5.1.6.1 (L2)** *"Ensure that collaboration invitations are sent to allowed domains only"* — direct CTAS allow-list mapping.
- **CIS 5.2.2.9 (L1)** *"Ensure a managed device is required for authentication"* — paired conditional access.
- **CIS 5.2.2.12 (L1)** *"Ensure the device code sign-in flow is blocked"* — the Storm-2372 countermeasure the policy references in Use case 2 / Use case 4. **The policy describes the Storm-2372 threat extensively but never cites the CIS control that addresses it directly.**

**Secure Score (missing):** **Identity** group — *"Configure Conditional Access policies"*, *"Block legacy authentication"*, *"Require multi-factor authentication for administrative roles"*, *"Configure user risk policies"*, *"Configure sign-in risk policies"*.

**Tier:** **L1** (the CIS controls are L1; the CTAS hardening is foundational).

---

### D4. `block-download-unmanaged-device.md:177-182`

**Claim:** maps to BNM RMiT DLP / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS / MAS TRM.

**CIS mapping (missing):**

- **CIS 7.3.2 (L2)** *"Ensure OneDrive sync is restricted for unmanaged devices"* — direct preventive complement.
- **CIS 5.2.2.9 (L1)** *"Ensure a managed device is required for authentication"* — Conditional Access prerequisite for the CAAC bilateral.
- **CIS 1.3.2 (L2)** *"Ensure 'Idle session timeout' is set to '3 hours (or less)' for unmanaged devices"* — adjacent unmanaged-device hardening.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Apps** group — *"Ensure all users can complete multifactor authentication for secure access"*, *"Conditional Access App Control"*. **Data** group — *"Sensitivity labels for Office files"*.

**Tier:** **L2** (CAAC + unmanaged-device-specific blocks are L2).

---

### D5. `geo-residency-block.md:168-174`

**Claim:** maps to BNM RMiT / PDPA / ISO 27017 / ISO 27018 / GDPR / DORA / NIST CSF.

**CIS mapping (missing):** there is **no direct CIS M365 Foundations control on data-residency at the user-session layer** — the CIS Benchmark addresses data-residency through tenant-region selection at provisioning, not through session policies. Practitioner-inference label needed.

**Secure Score (missing):** no direct improvement action for geo-residency at the user-session layer. **Verified-not-applicable** for both CIS and Secure Score.

**Correction:** label this policy explicitly as *"No direct CIS / Secure Score mapping — practitioner-inference control"*. This is honest and audit-defensible — it tells the reader that the CIS baseline does not cover this control class and the firm has elected to deploy it independently.

---

### D6. `inline-upload-block-regulated-data.md:186-192`

**Claim:** maps to BNM RMiT / PDPA / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS / MAS TRM.

**CIS mapping (missing):**

- **CIS 3.2.1 (L1)** *"Ensure DLP policies are enabled"* — direct mapping at L1.
- **CIS 3.2.2 (L1)** *"Ensure DLP policies are enabled for Microsoft Teams"* — Teams-side DLP.
- **CIS 3.3.1 (L1)** *"Ensure Information Protection sensitivity label policies are published"* — labelling prerequisite.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Data** group — *"Create policies to prevent sensitive data leaks"* (under Microsoft Purview Information Protection / DLP). **Apps** group — *"Conditional Access App Control"*.

**Tier:** **L1 for the policy intent (DLP enabled); L2 for the inline-prevention mechanism (CAAC).** This split is important — the policy authoring conflates them.

---

### D7. `api-data-at-rest-quarantine.md:157-163`

**Claim:** maps to PCI DSS Req. 3 / BNM RMiT / PDPA / ISO 27017 / ISO 27018 / NIST CSF / HIPAA.

**CIS mapping (missing):**

- **CIS 3.2.1 (L1)** *"Ensure DLP policies are enabled"* — at-rest scan is a DLP policy class.
- **CIS 3.3.1 (L1)** *"Ensure Information Protection sensitivity label policies are published"* — labels are how scanned content is acted on.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Data** group — *"Create policies to prevent sensitive data leaks"*, *"Sensitivity labels for Office files"*.

**Tier:** **L1** for DLP policy existence, **L2** for the MDA file-policy implementation surface.

---

### D8. `external-share-link-quarantine.md:152-156`

**Claim:** maps to BNM RMiT / ISO 27017 / ISO 27018 / NIST CSF / SOC 2.

**CIS mapping (missing) — the strongest set in the library:**

- **CIS 7.2.3 (L1)** *"Ensure external content sharing is restricted"*.
- **CIS 7.2.4 (L2)** *"Ensure OneDrive content sharing is restricted"*.
- **CIS 7.2.5 (L2)** *"Ensure that SharePoint guest users cannot share items they don't own"*.
- **CIS 7.2.6 (L2)** *"Ensure SharePoint external sharing is managed through domain whitelist/blacklists"*.
- **CIS 7.2.7 (L1)** *"Ensure link sharing is restricted in SharePoint and OneDrive"*.
- **CIS 7.2.9 (L1)** *"Ensure guest access to a site or OneDrive will expire automatically"* — the closest CIS analogue to the stale-window concept this policy is built around.
- **CIS 7.2.11 (L1)** *"Ensure the SharePoint default sharing link permission is set"*.

**Secure Score (missing):** **Apps** group — *"Restrict external sharing"*, *"Anyone links"* improvement actions. **Data** group — *"Sensitivity labels"*.

**Tier:** mixed L1 + L2 — the preventive controls (7.2.3, 7.2.7, 7.2.9, 7.2.11) are L1; the deeper external-sharing guard-rails (7.2.4, 7.2.5, 7.2.6) are L2; the MDA file-policy quarantine mechanism is L2 via 2.4.3.

This is the single most under-mapped policy in the library. Seven specific CIS controls map directly and none are cited.

---

### D9. `auto-label-pci-data.md:149-153`

**Claim:** maps to PCI DSS Req. 3 / BNM RMiT / ISO 27017 / ISO 27018 / NIST CSF.

**CIS mapping (missing):**

- **CIS 3.3.1 (L1)** *"Ensure Information Protection sensitivity label policies are published"* — direct, primary mapping.
- **CIS 3.2.1 (L1)** *"Ensure DLP policies are enabled"* — DLP layer that consumes the labels.
- **CIS 9.1.6 (L1)** *"Ensure 'Allow users to apply sensitivity labels for content' is 'Enabled'"* (Power BI / Fabric scope) — adjacent control where labels propagate downstream.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Data** group — *"Apply sensitivity labels to content"*, *"Publish sensitivity labels"* (under Microsoft Purview Information Protection).

**Tier:** **L1** (label publishing is L1); **L2** for the MDA file-policy auto-apply implementation.

---

### D10. `mass-download-alert.md:143-147`

**Claim:** maps to BNM RMiT / ISO 27017 / NIST CSF.

**CIS mapping (missing):**

- **CIS 2.4.3 (L2)** umbrella — explicitly: *"Some risk detection methods provided by Entra Identity Protection also require Microsoft Defender for Cloud Apps: ... **Mass access to sensitive files**"*. **This is the only place in the entire library where CIS 2.4.3 names a specific MDA detection that the library has a policy for, and the policy does not cite it.** Add immediately.
- **CIS 2.2.1 (L1)** *"Ensure emergency access account activity is monitored"* — adjacent monitoring control.

**Secure Score (missing):** **Apps** group — *"Investigate anomaly detection alerts"* / *"Mass download by a single user"* improvement action (Defender for Cloud Apps product).

**Tier:** **L1** (audit-log search is L1 via 3.1.1); **L2** for the MDA anomaly-detection layer via 2.4.3.

---

### D11. `impossible-travel-alert.md:152-159`

**Claim:** maps to BNM RMiT / MAS TRM / ISO 27001 A.5.16 / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS Req. 10.2.

**CIS mapping (missing):**

- **CIS 2.4.3 (L2)** — explicitly names *"Impossible travel detection"* as an MDA-dependent Entra Identity Protection risk detection. **As with D10, CIS 2.4.3 directly names this detection and the policy does not cite it.**
- **CIS 5.2.2.6 (L1)** *"Enable Identity Protection user risk policies"* — the Entra-side pairing the policy correctly advocates for at line 215-218.
- **CIS 5.2.2.7 (L1)** *"Enable Identity Protection sign-in risk policies"* — the *primary* control per the policy's own framing.
- **CIS 5.2.2.8 (L2)** *"Ensure 'sign-in risk' is blocked for medium and high risk"*.

**Secure Score (missing):** **Identity** group — *"Enable sign-in risk policy"*, *"Enable user risk policy"* (both reference Microsoft Entra ID Protection).

**Tier:** **L1** for the Entra-side pairing the policy explicitly recommends; **L2** for the MDA anomaly-detection layer.

---

### D12. `mass-delete-anomaly.md:163-166`

**Claim:** maps to BNM RMiT / ISO 27017 / NIST CSF / ISO 22301.

**CIS mapping (missing):**

- **CIS 2.4.3 (L2)** umbrella.
- **CIS 3.1.1 (L1)** *"Ensure Microsoft 365 audit log search is Enabled"* — the underlying audit-log dependency the policy uses.
- **CIS 6.2.1 (L1)** *"Ensure all forms of mail forwarding are blocked and/or disabled"* — adjacent containment control referenced in IR runbook.

**Secure Score (missing):** **Apps** group — *"Investigate anomaly detection alerts"* / *"Activity from suspicious IPs"* / mass-delete-class improvement actions.

**Tier:** **L1** for the audit-log dependency; **L2** for the MDA anomaly-detection layer.

---

### D13. `terminated-user-cross-saas.md:178-183`

**Claim:** maps to BNM RMiT / MAS TRM / ISO 27017 CLD.9.x / ISO 27018 / NIST CSF / PCI DSS.

**CIS mapping (missing):**

- **CIS 5.1.6.2 (L1)** *"Ensure that guest user access is restricted"* — adjacent control for the guest-account variant.
- **CIS 5.3.2 (L1)** *"Ensure 'Access reviews' for Guest Users are configured"* — direct mapping to the quarterly orphan-reconciliation the policy advocates.
- **CIS 5.3.3 (L1)** *"Ensure 'Access reviews' for privileged roles are configured"* — privileged-account variant.
- **CIS 2.4.3 (L2)** umbrella (the built-in *"Activity performed by terminated user"* anomaly policy).

**Secure Score (missing):** **Identity** group — *"Configure Access Reviews"*, *"Remove inactive user accounts"*.

**Tier:** **L1** for the access-reviews controls; **L2** for the MDA built-in anomaly via 2.4.3.

---

### D14. `b2b-partner-exfil-alert.md:174-179`

**Claim:** maps to BNM RMiT / ISO 27017 / ISO 27018 / NIST CSF / DORA / MAS TRM.

**CIS mapping (missing):**

- **CIS 5.1.6.1 (L2)** *"Ensure that collaboration invitations are sent to allowed domains only"* — direct mapping to the XTAS allow-list discipline.
- **CIS 5.1.6.2 (L1)** *"Ensure that guest user access is restricted"*.
- **CIS 5.1.6.3 (L2)** *"Ensure guest user invitations are limited to the Guest Inviter role"*.
- **CIS 5.3.2 (L1)** *"Ensure 'Access reviews' for Guest Users are configured"* — quarterly XTAS attestation pairing.
- **CIS 7.2.5 (L2)** *"Ensure that SharePoint guest users cannot share items they don't own"* — adjacent guest-side control.

**Secure Score (missing):** **Identity** group — *"Configure Guest user access restrictions"*, *"Configure Access Reviews for guest users"*. **Apps** group — *"Detect anomalous guest activity"*.

**Tier:** mixed **L1 + L2**.

---

### D15. `post-consent-cleanup-and-ban.md:183-188`

**Claim:** maps to BNM RMiT / ISO 27017 / ISO 27018 / NIST CSF / MAS TRM / HKMA SA-2.

**CIS mapping (missing) — strongest direct map in the library:**

- **CIS 5.1.5.1 (L2)** *"Ensure user consent to apps accessing company data on their behalf is not allowed"* — direct, primary mapping.
- **CIS 5.1.5.2 (L1)** *"Ensure the admin consent workflow is enabled"* — the Entra-layer compensating preventive control the policy explicitly recommends (line 19-20).
- **CIS 5.1.2.2 (L2)** *"Ensure third party integrated applications are not allowed"*.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Identity** group — *"Use limited administrative roles"* + *"Use the admin consent workflow"*. **Apps** group — *"Apply OAuth app policies"* improvement actions (App governance + Defender for Cloud Apps).

**Tier:** **L1** for the admin-consent workflow; **L2** for the user-consent prohibition + MDA cleanup.

This policy explicitly names the CIS-controlled approach (lines 19-20: *"this policy is the compensating cleanup layer beneath a newly-enabled Entra admin-consent workflow"*) but never cites the CIS control. Add now.

---

### D16. `high-scope-grant-alert.md:170-176`

**Claim:** maps to BNM RMiT / MAS TRM / ISO 27017 / ISO 27001:2022 A.5.15-A.8.2 / NIST CSF / PCI DSS Req. 7.

**CIS mapping (missing):**

- **CIS 5.1.5.1 (L2)** *"Ensure user consent to apps accessing company data on their behalf is not allowed"*.
- **CIS 5.1.5.2 (L1)** *"Ensure the admin consent workflow is enabled"*.
- **CIS 5.1.2.2 (L2)** *"Ensure third party integrated applications are not allowed"*.
- **CIS 9.1.10 (L1)** *"Ensure access to APIs by Service Principals is restricted"* (Power BI / Fabric scope) — direct map to the Power Platform service-principal concern the policy raises explicitly (line 28-31).
- **CIS 9.1.11 (L1)** *"Ensure Service Principals cannot create and use profiles"*.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Identity** group — *"Use the admin consent workflow"*. **Apps** group — *"App governance policies"* improvement actions.

**Tier:** **L1** for admin-consent + service-principal restriction; **L2** for the App Governance behavioural-detection layer.

This policy correctly identifies Power Platform service-principal as a gap (lines 28-31, 256, 269) — it is the only policy that does — and the CIS Benchmark has a direct control (9.1.10) for it that goes uncited.

---

### D17. `discovery-and-auto-unsanction.md:158-164`

**Claim:** maps to BNM RMiT / ISO 27017 / ISO 42001 / NIST CSF / NIST AI RMF / MITRE ATLAS / EU AI Act.

**CIS mapping (missing):**

- **CIS 1.3.4 (L1)** *"Ensure 'User owned apps and services' is restricted"* — preventive control on user-driven app adoption (which is what consumer GenAI is, at the M365 level).
- **CIS 5.1.2.2 (L2)** *"Ensure third party integrated applications are not allowed"*.
- **CIS 2.4.3 (L2)** umbrella — Cloud Discovery is the named MDA capability driving this policy.

**Secure Score (missing):** **Apps** group — *"Discover and sanction cloud apps"*, *"Block unsanctioned apps"* improvement actions.

**Tier:** **L1** for the user-app preventive controls; **L2** for the MDA discovery + auto-unsanction mechanism.

---

### D18. `inline-prompt-dlp.md:191-201`

**Claim:** maps to BNM RMiT / PDPA / HKMA / MAS TRM / ISO 27017 / ISO 27018 / ISO 42001 / NIST CSF / NIST AI RMF / MITRE ATLAS / EU AI Act / PCI DSS.

**CIS mapping (missing):**

- **CIS 3.2.1 (L1)** *"Ensure DLP policies are enabled"* — direct DLP umbrella.
- **CIS 3.3.1 (L1)** *"Ensure Information Protection sensitivity label policies are published"* — labelling prerequisite for SIT-based session policies.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Data** group — *"Create policies to prevent sensitive data leaks"*. **Apps** group — *"Conditional Access App Control"* session-policy improvement actions.

**Tier:** **L1** for the DLP / labelling prerequisites; **L2** for the inline CAAC session-policy implementation.

---

### D19. `sanctioned-tenant-pinning.md:202-209`

**Claim:** maps to BNM RMiT / ISO 27017 / ISO 27018 / NIST CSF / NIST AI RMF / EU AI Act / EU DORA.

**CIS mapping (missing):**

- **CIS 5.1.6.1 (L2)** *"Ensure that collaboration invitations are sent to allowed domains only"* — CTAS allow-list discipline that tenant-pinning implements at a different layer.
- **CIS 5.2.2.12 (L1)** *"Ensure the device code sign-in flow is blocked"* — direct Storm-2372 countermeasure adjacent to tenant pinning.
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **Identity** group — *"Configure tenant restrictions"*, *"Block device code flow"*.

**Tier:** **L1** for the device-code-flow block; **L2** for the CTAS allow-list and the MDA session-policy supplement.

---

### D20. `sspm-tenant-misconfig-drift.md:182-188`

**Claim:** maps to BNM RMiT / MAS TRM / ISO 27017 / ISO 27001:2022 / NIST CSF / SOC 2 / PCI DSS.

**CIS mapping (this is the most important policy to map):**

- The entire CIS Microsoft 365 Foundations Benchmark v5.0.0 *is* a posture-rule library. **The whole document maps to this policy.**
- The CIS controls explicitly named by Microsoft as a baseline for the Secure Score posture (per Secure Score docs that note CIS alignment).
- **CIS 2.4.3 (L2)** umbrella.

**Secure Score (missing):** **all four categories — Identity / Device / Apps / Data**. The Secure Score document is the operational expression of the posture this policy describes. The policy explicitly says *"Microsoft Secure Score-derived rules"* at line 83 — and that is the ONLY mention of Secure Score in the entire library. Add it as a primary citation in the `Control mappings` block, not just as a parenthetical caveat.

**Tier:** mixed across the entire benchmark — the policy is the operational mechanism for sustaining both L1 baseline and L2 highly-secure profile compliance.

This is the most under-mapped policy in the library given the size of the gap. CIS v5.0.0 is the source of truth for SSPM rules; the policy describes the function but never names the source.

---

## Stale references

### S1. Secure Score category structure

**Affected:** all 19 files (no current Secure Score citation, so no stale citation, but the framework is missing).

**Microsoft source:** Microsoft Secure Score uses four named groups per current documentation (2026-03-07): **Identity** (Entra), **Device** (Defender for Endpoint), **Apps** (Office 365 + Defender for Cloud Apps), **Data** (Microsoft Information Protection).

**Correction:** in any added Secure Score citation, use the current four-group structure. Do NOT use the older "Identity / Data / Device / Apps / Infrastructure" five-group structure — Microsoft removed the Infrastructure group.

### S2. CIS Benchmark version

**Affected:** none (no CIS citation in the library to update), but worth recording.

**Microsoft source:** CIS Microsoft 365 Foundations Benchmark **v5.0.0 (30 April 2025)** is current. v4.0.0 (October 2024) is the previous version. The CIS Workbench may show 7.0.0 / 6.0.1 numbering for CIS-CAT Pro tooling — that is the *tool* version, not the *benchmark* version. The benchmark itself is v5.0.0.

**Correction:** when adding CIS citations, use `CIS Microsoft 365 Foundations Benchmark v5.0.0`.

### S3. CIS 2.4.3 language

**Affected:** the description in any cite of CIS 2.4.3.

**Microsoft source:** the v5.0.0 control text is *"Ensure Microsoft Defender for Cloud Apps is enabled and configured (Manual)"* (L2, E5). Earlier versions (v1.x / v2.x) used different control numbering (1.5.x range historically). Do not use older numbering.

---

## Missing citations

Every policy in the library is missing the following citations. Listed by source authority:

### CIS Microsoft 365 Foundations Benchmark v5.0.0

Apply per-file mapping per D2-D20 above. The umbrella citation (CIS 2.4.3) applies to all 19 files. Specific sub-control citations apply per the per-file lists.

### Microsoft Secure Score Improvement Actions

Source URLs to add to every policy's `Control mappings`:

- `https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score`
- `https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions`

Per-file the specific improvement-action category (Identity / Apps / Data) per D2-D20.

### Microsoft Defender for Cloud Apps best-practices page

`https://learn.microsoft.com/en-us/defender-cloud-apps/best-practices` — explicitly cited in CIS 2.4.3 as a primary reference. None of the 19 policies cite it. Add to `block-unsanctioned-app-with-coach.md`, `block-download-unmanaged-device.md`, `inline-upload-block-regulated-data.md`, `api-data-at-rest-quarantine.md` minimum.

### Microsoft `protect-office-365` connector page

`https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365` — cited in CIS 2.4.3 as the list of MDA built-in policies for Office 365. Should be referenced by all M365-anchored policies in the library, particularly the API-mode policies (`api-data-at-rest-quarantine`, `external-share-link-quarantine`, `auto-label-pci-data`, `mass-download-alert`, `mass-delete-anomaly`).

---

## Practitioner-inference flags

Claims with no Microsoft source found — legitimate practitioner inference but the file should label them as such.

### P1. `geo-residency-block.md` — entire policy

No direct CIS or Secure Score mapping. Practitioner-inference control. Label explicitly: *"This policy implements data-residency enforcement at the user-session layer. CIS Microsoft 365 Foundations Benchmark does not include a direct control for this enforcement layer; the firm has elected to deploy it as a practitioner-inference control to satisfy regulator-driven residency obligations (BNM / PDPA / GDPR Ch. V / DORA)."*

### P2. FP-rate trajectories (all 19 files)

Every policy provides a W1-to-steady-state FP-rate trajectory table (e.g. `auto-label-pci-data.md:168-175`). **None of these numbers come from a Microsoft source.** They are practitioner observations from deployments. Microsoft does not publish FP-rate ranges for MDA DLP / activity / anomaly policies.

**Correction:** add a footnote to every FP-rate trajectory table — *"Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline."*

### P3. `discovery-and-auto-unsanction.md:85` — Microsoft risk score "90+ factors"

Cited correctly as opaque. Microsoft does not publish the factors or weights. Practitioner-flag is already in the text (*"Microsoft's risk score is opaque (uncited "90+ factors")"*) — keep as-is, it is honestly labelled.

### P4. `inline-prompt-dlp.md:83` — SIEM ingest ~100 GB/month at 5k seats

Honestly labelled in-text (*"sysint estimate"*). Keep.

### P5. `impossible-travel-alert.md:83` — Microsoft Sensitivity slider has no numeric semantics

Honestly labelled in-text (*"Sensitivity slider exposes no numeric threshold — opaque tuning"*). Keep.

### P6. `mass-download-alert.md:124` — `count_threshold: 5000` derived from W2 baseline

Honestly labelled in-text (*"should be derived from the W2 baseline, not used as a literal default"*). Keep.

---

## Policies needing significant rework

**None.** The 19 policies are factually accurate where they make Microsoft-product claims and honestly labelled where they cannot be. The systematic gap is the absence of CIS and Secure Score citations across the entire `Control mappings` block of every file. This is uniform — no single policy is materially worse than the others, and no policy needs a content rewrite. The fix is **add the two missing citation rows** to every file.

If the user's intent is to ship this library as an audit-evidence reference for an MDA-anchored programme, the absence of CIS v5.0.0 and Secure Score citations is the single highest-impact gap because **the BNM RMiT supervisor, the SOC 2 auditor, the PCI QSA, and the ISO 27001 surveillance auditor will all probe for them**. The current `Control mappings` blocks list standards the auditor will accept but omit the two Microsoft-native baselines the auditor will explicitly look for.

---

## Top remediation priorities

Rank-ordered top-10 specific corrections:

1. **Add CIS 2.4.3 (L2) and Microsoft Secure Score citations to every policy's `Control mappings` block.** Single highest-impact change. 19 files × 2 citation rows = ~30 minutes of work each. Worth more than every other item on this list combined.

2. **`mass-download-alert.md:143-147`** — add explicit citation to CIS 2.4.3 + the bullet *"Mass access to sensitive files"* it names. This is the most direct CIS-named MDA detection in the library; not citing it is the loudest gap.

3. **`impossible-travel-alert.md:152-159`** — same pattern; CIS 2.4.3 explicitly names *"Impossible travel detection"*. Add CIS 5.2.2.6, 5.2.2.7, 5.2.2.8 for the Entra ID Protection pairing the policy already advocates.

4. **`post-consent-cleanup-and-ban.md:183-188`** — add CIS 5.1.5.1 (L2) and CIS 5.1.5.2 (L1) as direct mappings. The policy describes the controls in narrative form but never cites them.

5. **`external-share-link-quarantine.md:152-156`** — add the seven CIS 7.2.x controls in D8 above. This policy maps to more named CIS controls than any other and currently cites none of them.

6. **`sspm-tenant-misconfig-drift.md:182-188`** — make CIS v5.0.0 a primary citation (not parenthetical caveat) and add Microsoft Secure Score as the operational reference for posture-rule library design. The policy describes the function the Secure Score docs implement.

7. **`high-scope-grant-alert.md:170-176`** — add CIS 9.1.10 (L1) for the Power Platform service-principal concern the policy explicitly raises. This is the only policy that names Power Platform service-principal as a gap and there is a direct CIS control for it.

8. **`auto-label-pci-data.md:149-153`** — add CIS 3.3.1 (L1) as the primary mapping. Sensitivity-label publishing is L1 and this policy is the operational manifestation of that control.

9. **`block-download-unmanaged-device.md:177-182`** — add CIS 7.3.2 (L2) *"Ensure OneDrive sync is restricted for unmanaged devices"* and CIS 5.2.2.9 (L1) *"Ensure a managed device is required for authentication"*. These two together are the closest direct CIS analogue to the policy's intent.

10. **Add the Tier (L1 / L2) annotation to every policy's `Control mappings` block.** Currently no policy states which CIS implementation tier it aligns with. The pattern in this library skews **L2** because MDA itself is L2 (CIS 2.4.3), but the underlying preventive controls are typically L1. State both per policy.

---

Sources:

- [CIS Microsoft 365 Benchmarks (overview)](https://www.cisecurity.org/benchmark/microsoft_365)
- [Microsoft Secure Score (Defender XDR)](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score)
- [Microsoft Secure Score Improvement Actions](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions)
- [CIS Microsoft 365 Foundations Benchmark v5.0.0 PDF (mirror)](https://itsecure.hu/wp-content/uploads/2025/05/CIS_Microsoft_365_Foundations_Benchmark_v5.0.0.pdf)
- [Microsoft Defender for Cloud Apps best practices](https://learn.microsoft.com/en-us/defender-cloud-apps/best-practices)
- [Defender for Cloud Apps — protect-office-365](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365)
