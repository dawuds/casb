# Microsoft Purview + Entra integration QA expert - QA report

Specialist: Purview + Entra Conditional Access integration QA.
Scope: 19 Tier-2 policy files at `/Users/dawud/claude/casb/03-policies/`.
Authority sources: Microsoft Learn — Purview DLP, Sensitivity labels, Data Classification, DSPM for AI, Insider Risk Management, Adaptive Protection, eDiscovery, Conditional Access (overview, session, workload identity), ID Protection, Tenant Restrictions V2, Defender for Cloud Apps session policies.

## Summary

| Category | Count |
|---|---|
| Files reviewed | 19 / 19 |
| Verified-clean structural claims | 17 |
| Drift to correct (specific file:line) | 21 |
| Stale references (renamed / moved / retired surface) | 7 |
| Missing citations (claim needs MS URL) | 11 |
| Practitioner-inference flags (no source — label as such) | 9 |
| Policies needing significant rework | 0 (rework not required; targeted edits sufficient) |

The structural integration story (CAAC bilateral pair, P1 requirement, Adaptive Protection wiring, DSPM-for-AI capture model, Workload Identity Premium + service-principal risk) is broadly correct. The drift is concentrated in: (a) SIT parameter naming (`minimum_violations` vs `minimum_unique_matches`); (b) the Per-tool DSPM-for-AI claim that Claude Enterprise supports "deep" capture — Microsoft Learn explicitly shows Claude Enterprise does NOT yet support DLP, IRM, eDiscovery, Data Lifecycle Management, Data Classification, or Communication Compliance; (c) the eDiscovery retirement (classic eDiscovery retired 31 Aug 2025 — the current product is "eDiscovery" in the Purview portal, not "Premium" except for 21Vianet); (d) the "MDA can Apply Purview sensitivity label" action language vs the documented MDA session-policy action `Protect` (which is what applies a label, only on download).

---

## Verified-clean claims

Confirmed against Microsoft Learn:

- **CAAC requires both Entra CA policy AND MDA session policy.** Documented at [defender-cloud-apps/session-policy-aad](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad) — explicit prereq: "In order for your session policy to work, you must also have a Microsoft Entra ID Conditional Access policy." The "bilateral CAAC trap" framing in `block-download-unmanaged-device.md` (lines 82, 247-248), `inline-upload-block-regulated-data.md` (line 83), `geo-residency-block.md` (lines 65, 241) is structurally correct.
- **Entra ID P1 is the licensing prereq for CAAC.** Confirmed at the same source: "A license for Microsoft Entra ID P1, either as standalone license or as part of another license." `inline-upload-block-regulated-data.md` line 6, `block-download-unmanaged-device.md` line 6 — correct.
- **MDA session-policy `Always apply the selected action even if the data cannot be scanned`** is a real per-session control. Confirmed at the same source.
- **MDA "Microsoft Defender for Cloud Apps – Session Controls" Enterprise Application** — accidental inclusion in a blocking CA policy is a documented break-glass risk. Confirmed at session-policy-aad. `block-download-unmanaged-device.md` anti-pattern #10 (line 256) is correct.
- **CAAC session policy can Apply sensitivity labels — but only via the `Protect` action on the download leg, not as a generic governance action.** Documented at session-policy-aad. See drift below — the `auto-label-pci-data.md` config example claims `ApplyPurviewSensitivityLabel` as the governance action of a *File policy* (the at-rest API path), which is supported, but the action name is "Apply sensitivity label", not `ApplyPurviewSensitivityLabel`.
- **Tenant Restrictions V2 headers** (`Restrict-Access-To-Tenants`, `Restrict-Access-Context`) — confirmed at [entra/external-id/tenant-restrictions-v2](https://learn.microsoft.com/en-us/entra/external-id/tenant-restrictions-v2). The SWG / Global Secure Access header-injection model and Microsoft's note that pure-MDA does not inject these headers is correct in `tenant-restriction-corporate-only.md` (line 84) and `sanctioned-tenant-pinning.md` (line 84).
- **Tenant Restrictions v1 is being deprecated in favour of v2** — confirmed in Microsoft docs. `tenant-restriction-corporate-only.md` line 84 (`[VERIFY:` qualifier present) correctly flags this.
- **Adaptive Protection integrates with Conditional Access via the "Insider risk" condition on CA grant controls.** Confirmed at [purview/insider-risk-management-adaptive-protection](https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection). Risk levels Minor / Moderate / Elevated. Adaptive Protection also wires into DLP and Data Lifecycle Management (not just CA). Quick setup creates a default Conditional Access policy "Block access for users with Insider Risk (Preview)" in Report-only mode. `block-unsanctioned-app-with-coach.md` line 147 (advanced variant — "Adaptive Protection-integrated for risk-based per-user scoping") is structurally correct.
- **Entra ID Protection MDA-sourced detections include Impossible travel, Activity from anonymous IP, Mass access to sensitive files, New country.** Confirmed at [entra/id-protection/overview-identity-protection](https://learn.microsoft.com/en-us/entra/id-protection/overview-identity-protection). `impossible-travel-alert.md` framing as "naive credential-stuffing-only; pair with Entra ID Protection" is structurally correct.
- **Entra ID Protection requires Entra ID P2** for risk-based Conditional Access. Confirmed at the same source. `impossible-travel-alert.md` pairing claim is correct.
- **Token Protection is a real Entra feature**, documented as a Conditional Access session control ("Require token protection for sign-in sessions") at [concept-conditional-access-session](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-session). Whether it is GA for all clients in 2026-06 is qualified in the source ("attempts to reduce attacks using token theft… token theft is rare but its impact can be significant") — the policy files' `[VERIFY GA status]` qualifiers are appropriate.
- **Continuous Access Evaluation** is enabled by default and can be Disabled per policy via Conditional Access session controls. Confirmed at concept-conditional-access-session.
- **Conditional Access for Workload Identities** requires Workload Identities Premium license; supports location-based and service-principal-risk-based block. Confirmed at [entra/identity/conditional-access/workload-identity](https://learn.microsoft.com/en-us/entra/identity/conditional-access/workload-identity). `high-scope-grant-alert.md` line 260 reference to "Entra Workload Identities P2 covers a different signal class" is structurally correct.
- **Service-principal risk detections** are owned by Entra ID Protection (Workload identities tab), not by MDA App Governance. Confirmed at id-protection overview + workload-identity Conditional Access doc.
- **Microsoft 365 Audit Log captures prompts and responses for AI apps** that are connected via DSPM-for-AI collection policies. Confirmed at [purview/ai-microsoft-purview](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview) and [purview/ai-chatgpt-enterprise](https://learn.microsoft.com/en-us/purview/ai-chatgpt-enterprise).
- **DSPM-for-AI ChatGPT Enterprise integration supports DLP for prompts via a Collection Policy** (`DSPM for AI - Capture interactions for enterprise AI apps`). `inline-prompt-dlp.md` framing of "pair with DSPM-for-AI for the typed-not-pasted gap" is structurally correct; see drift below for the claim that DLP itself is supported on ChatGPT Enterprise — Microsoft Learn says DLP = ✕ for ChatGPT Enterprise; only the Collection Policy capture path is supported, not DLP rules acting on those prompts.
- **eDiscovery legal hold preserves content and overrides retention/deletion**: the principles of retention page documents this — the longest-duration hold/retention wins. The "Purview eDiscovery hold trumps MDA quarantine action — potential spoliation" framing in `api-data-at-rest-quarantine.md` line 83 + line 223, and in `external-share-link-quarantine.md` line 81 + line 214 is structurally correct as practitioner judgement, but no Microsoft page explicitly says "MDA quarantine on hold content = spoliation". Flag as practitioner-inference — see below.
- **Sensitivity labels can be auto-applied at-rest via auto-labeling policies** in Purview Information Protection. Confirmed at [purview/sensitivity-labels](https://learn.microsoft.com/en-us/purview/sensitivity-labels) ("Apply the label automatically to files and emails, or recommend a label"). `auto-label-pci-data.md` framing is correct.

---

## Drift to correct

### 1. `inline-upload-block-regulated-data.md` line 83 + line 122-134

**Claim:** Configuration uses `minimum_unique_matches: 5` (renamed from `minimum_violations = 1` in the prose at line 83).
**What MS says:** The Microsoft DLP documentation uses **"instance count"** (`minCount` / `maxCount`) as the parameter name and **"confidence level"** (high / medium / low, mapped to numeric `confidenceLevel`) for the accuracy parameter. Source: [purview/dlp-learn-about-dlp](https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp) and the DLP policy reference. The vendor's CASB-side / MDA-side wording does say `minimum_violations` in older content. Neither `minimum_violations` nor `minimum_unique_matches` matches the current Purview DLP rule schema field name.
**Corrected text:** Replace `minimum_unique_matches` with `instance count` (or qualify as "MDA UI label — Purview rule schema uses `minCount`"). The "match accuracy 85 = confidence threshold" framing is correct; reframe to align with `confidenceLevel` (1-100 mapped from named tiers).
**Same drift recurs:** `auto-label-pci-data.md` lines 78, 105-135, 218; `api-data-at-rest-quarantine.md` lines 109-119; `inline-prompt-dlp.md` lines 118-141.

### 2. `auto-label-pci-data.md` line 113 + line 78

**Claim:** Governance action = `ApplyPurviewSensitivityLabel`.
**What MS says:** The MDA File Policy governance action is documented as **"Apply Microsoft Purview sensitivity label"** (display name). The session-policy variant uses the action **`Protect`** which applies a sensitivity label on download. Source: [defender-cloud-apps/session-policy-aad](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad) ("**Protect** — Applies a sensitivity label to the download…"). The pseudo-API name `ApplyPurviewSensitivityLabel` is the user's invention.
**Corrected text:** `action: Apply Microsoft Purview sensitivity label` (with `label_name: "Confidential-Finance-CHD"` as a sub-attribute). Same fix in `external-share-link-quarantine.md` line 124 — `PutInAdminQuarantine` is also a synthetic API name; the documented MDA File Policy action is **"Put file in admin quarantine"**.

### 3. `ai-claude-enterprise` capability mismatch — multiple files

**Claim across files:** "DSPM-for-AI prompt-body capture on allowlisted enterprise-AI vendors — deep capture on ChatGPT Enterprise + Copilot per the MDA playbook DSPM table; shallow on Claude / Gemini" — `discovery-and-auto-unsanction.md` line 154; `sanctioned-tenant-pinning.md` line 198; `inline-prompt-dlp.md` line 264.
**What MS says:** Per [purview/ai-claude-enterprise](https://learn.microsoft.com/en-us/purview/ai-claude-enterprise) (current as of 2026-05-28), Anthropic Claude Enterprise integration is **in preview** and supports **only**: DSPM, DSPM for AI (classic), Auditing. It does **NOT** support Data Classification, Sensitivity labels, DLP, IRM, Communication compliance, eDiscovery, Data Lifecycle Management, or Compliance Manager (capability table on the page). Per [purview/ai-chatgpt-enterprise](https://learn.microsoft.com/en-us/purview/ai-chatgpt-enterprise), ChatGPT Enterprise supports DSPM, Audit, Data classification, IRM, Communication compliance, eDiscovery, Data Lifecycle Management, Compliance Manager — DLP and Sensitivity labels are **NOT** supported.
**Corrected text:** The "deep vs shallow" framing should be replaced by the explicit capability matrix from Microsoft Learn. Suggest: "ChatGPT Enterprise — Auditing + IRM + eDiscovery + DLM + classification supported; DLP and sensitivity labels NOT supported. Claude Enterprise (preview) — Auditing + DSPM only; classification, DLP, sensitivity labels, IRM, eDiscovery NOT supported. Gemini — only third-party-AI-sites coverage via browser extension + endpoint DLP." Cite the per-AI-app pages.

### 4. `inline-upload-block-regulated-data.md` line 83 — "MDA → MDA session policy can ApplyPurviewSensitivityLabel as governance action"

**Claim:** Implicit in the worked example — that MDA can act as the PCI-DSS-evidence-bearing prevention layer for ChatGPT prompts via clipboard inspection.
**What MS says:** Microsoft documents two distinct paths for ChatGPT-Enterprise DLP: (1) Endpoint DLP via [purview/endpoint-dlp-learn-about](https://learn.microsoft.com/en-us/purview/endpoint-dlp-learn-about) which can warn / block paste of sensitive content into third-party AI sites in a browser; (2) DSPM-for-AI Collection Policy which **captures** prompts (audit-only) and can be inspected via Communication Compliance or IRM but does **not** itself block. Browser DLP via Microsoft Edge for Business is the third path (the Microsoft-documented one). MDA session policy via CAAC against ChatGPT Enterprise requires the AI vendor to be SAML-federated AND manually onboarded to CAAC — which `inline-prompt-dlp.md` line 83 correctly notes, but the policy text underplays that **Endpoint DLP** (not CAAC) is Microsoft's primary documented control for "block paste into ChatGPT".
**Corrected text:** In `inline-prompt-dlp.md` and `inline-upload-block-regulated-data.md` add an explicit note: "Microsoft's documented primary control for ChatGPT paste-block is **Endpoint DLP** (Windows-onboarded device) per [endpoint-dlp-learn-about](https://learn.microsoft.com/en-us/purview/endpoint-dlp-learn-about) or **Browser DLP via Edge for Business** per [dlp-create-policy-prevent-cloud-sharing-from-edge-biz](https://learn.microsoft.com/en-us/purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz). MDA CAAC against SAML-federated ChatGPT Enterprise is a secondary control."

### 5. `external-share-link-quarantine.md` line 122 — `purview_legal_hold: false` as an MDA file-filter

**Claim:** That MDA's File Policy filter exposes a `purview_legal_hold: false` boolean which can be used to exclude legal-hold material from quarantine actions.
**What MS says:** No MDA File Policy filter named `purview_legal_hold` is documented at Microsoft Learn. The actual hold-respect mechanism is at the SharePoint / OneDrive layer — the Microsoft 365 retention/hold engine resists the quarantine action; MDA does not filter on hold-state at policy-design time. The principle-of-retention page at Microsoft Learn explains that hold/retention wins over deletion. This is a synthesised filter.
**Corrected text:** Either (a) cite the source if a literal MDA file-filter for hold-state exists (the practitioner should verify and link), or (b) reword as: "Coordinate with Legal — quarantine actions on Microsoft-365-stored content under an eDiscovery hold are blocked at the M365 retention engine layer; MDA does not natively filter on hold-state in the File Policy filter expression." Practitioner-inference per the existing prose; the YAML pseudo-config overstates capability.

### 6. `api-data-at-rest-quarantine.md` line 83 — "Only the governance action of the first triggered policy is guaranteed to be applied"

**Claim:** Direct quote framed as Microsoft documentation.
**What MS says:** This wording does appear in older MDA docs around file-policy ordering. Should be cited with a Microsoft Learn URL.
**Corrected text:** Add citation. Likely target: [defender-cloud-apps/file-filters](https://learn.microsoft.com/en-us/defender-cloud-apps/file-filters) or the MDA file-policy reference.

### 7. `mass-download-alert.md` line 80, `mass-delete-anomaly.md` line 83, `terminated-user-cross-saas.md` line 85 — "Activity-policy auto-disable at 200k/day or 100k/3h"

**Claim:** Specific numeric thresholds for MDA activity-policy auto-disable.
**What MS says:** Could not locate these specific thresholds in Microsoft Learn (searched). The numbers may be accurate from internal Microsoft / playbook content but should be cited or marked as `[VERIFY]`. Flag as **missing citation** + likely practitioner-inference.

### 8. `terminated-user-cross-saas.md` line 85 — built-in "Activity performed by terminated user" anomaly policy

**Claim:** Editable, not creatable; relies on Entra-as-IdP + exact-email matching.
**What MS says:** Confirmed at [defender-cloud-apps/anomaly-detection-policy](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy) — the "Activity performed by terminated user" is one of the documented built-in anomaly detections. The exact-email-match constraint is implied (not explicitly stated as such in the docs) and is reasonable practitioner observation. **Add citation to the anomaly-detection-policy page.**

### 9. `impossible-travel-alert.md` line 83 — "Country-level only — no alerts within the same country or between bordering countries"

**Claim:** Bordering-country exclusion is a hard rule.
**What MS says:** Microsoft documents impossible-travel detection at country-level granularity per the same anomaly-detection-policy page; "bordering countries" exclusion is observation rather than documented rule. Microsoft documents the ML model uses tenant-common-location and VPN suppression. **Add citation.** The KL ↔ SG ↔ JB observation is correct practitioner-inference.

### 10. `inline-prompt-dlp.md` line 187 — "Edge for Business `gen-ai.app-control` (H2 2026 GA)"

**Claim:** A named Microsoft Edge enterprise feature gating GenAI app control.
**What MS says:** Microsoft documents Browser DLP via Edge for Business at [purview/dlp-browser-dlp-learn](https://learn.microsoft.com/en-us/purview/dlp-browser-dlp-learn) and the policy creation flow at [purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz](https://learn.microsoft.com/en-us/purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz). The specific feature name `gen-ai.app-control` is not surfaced in the documentation I could find — practitioner-inference / pre-GA naming. Treat the GA date claim as unverified.
**Corrected text:** Replace specific feature name with documented "Browser DLP via Edge for Business" + `[VERIFY GA status against current Microsoft 365 roadmap]`.

### 11. `discovery-and-auto-unsanction.md` line 85 — "GenAI sub-category split (LLM / AI Coding Assistant / AI Image Generator / AI Video Generator / AI Voice-Avatar) — 2025 H2"

**Claim:** Microsoft introduced sub-categories for the previously monolithic "Generative AI" category.
**What MS says:** The current docs at [defender-cloud-apps/risk-score](https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score) and [purview/ai-microsoft-purview-supported-sites](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview-supported-sites) confirm category-level classification of AI sites. The specific 5-way sub-category breakdown should be cited or marked `[VERIFY]`. The MS-documented split at the date of writing is more often described as "AI sites" rather than per-modality categories.
**Corrected text:** Add citation to the MDA Cloud App Catalog category-list page; mark the sub-category breakdown as `[VERIFY against current MDA catalog category list]`.

### 12. `high-scope-grant-alert.md` line 83 — App Governance "Predictive Risk shipped 2025 H2"

**Claim:** Specific GA date.
**What MS says:** Microsoft documents App Governance Predictive Risk at [defender-cloud-apps/app-governance-overview](https://learn.microsoft.com/en-us/defender-cloud-apps/app-governance-overview). GA timing should be cited; the precise "2025 H2" is practitioner recall. Mark `[VERIFY]`.

### 13. Multiple files — "eDiscovery (Premium)" naming

**Claim across files:** `auto-label-pci-data.md` line 224, `external-share-link-quarantine.md` line 122, `api-data-at-rest-quarantine.md` lines 105, 223 all reference "Purview eDiscovery legal hold" without naming version.
**What MS says:** Per [purview/ediscovery-overview](https://learn.microsoft.com/en-us/purview/ediscovery-overview): "Microsoft retired all classic eDiscovery experiences on August 31, 2025. This retirement includes classic Content Search, classic eDiscovery (Standard), and classic eDiscovery (Premium)." The current product is "eDiscovery" in the Microsoft Purview portal; the "Premium" suffix only applies to 21Vianet-hosted tenants. The "eDiscovery (Premium)" wording in policy text is now stale for most tenants.
**Corrected text:** Replace "Purview eDiscovery (Premium)" with "Purview eDiscovery" (or qualify as "the new eDiscovery experience in the Purview portal"). Mark explicitly: classic experiences retired 2025-08-31.

### 14. `geo-residency-block.md` line 84 — "M365 multi-geo 'supported only for OneDrive' per docs"

**Claim:** SharePoint multi-geo events not fully captured in MDA.
**What MS says:** This wording appears in MDA connector docs historically. Citation needed; should be linked to [defender-cloud-apps/connect-office-365](https://learn.microsoft.com/en-us/defender-cloud-apps/connect-office-365).

### 15. `block-download-unmanaged-device.md` line 230 — "Japan East was added 2025 H2"

**Claim:** Microsoft Cloud regions update.
**What MS says:** Microsoft 365 admin center documents the data location at [microsoft-365/enterprise/o365-data-locations](https://learn.microsoft.com/en-us/microsoft-365/enterprise/o365-data-locations). Japan East mention is plausible but should be cited or qualified. Recurs at `inline-upload-block-regulated-data.md` line 241, `auto-label-pci-data.md` line 199, `geo-residency-block.md` line 24, `inline-prompt-dlp.md` line 251.
**Corrected text:** Cite the Microsoft 365 data location page; mark the specific quarter as `[VERIFY against current Microsoft Cloud regions page]`.

### 16. `tenant-restriction-corporate-only.md` line 84 — "v1 was app-class-only"

**Claim:** TR v1 supported only app-class-wide restriction, while v2 supports per-tenant-ID restriction.
**What MS says:** Confirmed at [entra/external-id/tenant-restrictions-v2](https://learn.microsoft.com/en-us/entra/external-id/tenant-restrictions-v2). Correct. Already cited as `[VERIFY]` — recommend resolving the citation rather than leaving the qualifier.

### 17. `post-consent-cleanup-and-ban.md` line 85 — "Application (client-credentials) OAuth grants are NOT in this view — service-principal abuse is the dominant cloud-persistence pattern; Policy 2a covers ~30% of OAuth-grant abuse"

**Claim:** Quantitative claim about delegated vs application-permission coverage and the 30% figure.
**What MS says:** The qualitative point — that the classic OAuth-apps view covers delegated grants and not application-permission grants — is supported by [defender-cloud-apps/manage-app-permissions](https://learn.microsoft.com/en-us/defender-cloud-apps/manage-app-permissions). The "30% of OAuth-grant abuse" figure has no Microsoft source. Mark as practitioner-inference.

### 18. `high-scope-grant-alert.md` lines 83, 167 — "App Governance under-surfaces Power Platform SPs; Power Platform Admin Center DLP required as companion"

**Claim:** Coverage gap with named compensating control.
**What MS says:** Power Platform Admin Center has its own DLP / data-policies surface documented at [power-platform/admin/wp-data-loss-prevention](https://learn.microsoft.com/en-us/power-platform/admin/wp-data-loss-prevention). The practitioner observation that App Governance under-surfaces Power Platform service-principals is consistent with the Microsoft docs' framing that App Governance focuses on Microsoft Graph apps. **Add citation.**

### 19. `impossible-travel-alert.md` line 83 — "Sensitivity slider exposes no numeric threshold — opaque tuning"

**Claim:** Microsoft does not publish numeric semantics for the Sensitivity slider.
**What MS says:** Confirmed by absence — Microsoft's anomaly-detection-policy page describes Low / Medium / High without numeric semantics. Cite the page rather than leave the assertion uncited.

### 20. `b2b-partner-exfil-alert.md` line 85 — "Activity-policy auto-disable at 200k/day or 100k/3h"

**Claim:** Same numeric thresholds as `mass-download-alert.md` line 80.
**What MS says:** As above (drift item 7) — citation needed.

### 21. `sspm-tenant-misconfig-drift.md` line 83 — "App Governance → Security configuration assessment"

**Claim:** Microsoft Secure Score-derived rules are used; coverage covers M365, Salesforce, ServiceNow, GitHub, Okta, Box, Dropbox, Zendesk.
**What MS says:** Microsoft Secure Score is documented at [defender-xdr/microsoft-secure-score](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score). The specific MDA SSPM-app coverage list should be cited to [defender-cloud-apps/saas-security-posture-management](https://learn.microsoft.com/en-us/defender-cloud-apps/saas-security-posture-management) (the SaaS Security Posture Management page) — confirm current app coverage list at time of attestation. Mark `[VERIFY against current docs]`.

---

## Stale references

| Stale term in policy files | Current Microsoft naming | Files needing update |
|---|---|---|
| "Purview eDiscovery (Premium)" | "Purview eDiscovery" (new eDiscovery experience in Purview portal); classic experiences retired 2025-08-31 | `auto-label-pci-data.md`, `external-share-link-quarantine.md`, `api-data-at-rest-quarantine.md`, `mass-delete-anomaly.md`, `terminated-user-cross-saas.md` |
| "DSPM for AI" (without qualifier) | "Data Security Posture Management for AI (classic)" — Microsoft has launched a new DSPM (not for-AI specific) that subsumes the classic per-Microsoft Learn 2026-05 docs | `discovery-and-auto-unsanction.md`, `inline-prompt-dlp.md`, `sanctioned-tenant-pinning.md` |
| "App Governance is a paid add-on (verify SKU pricing per current `app-governance-licensing` doc)" | App Governance is now bundled into Microsoft 365 E5 Compliance / E5 Security depending on SKU per Microsoft Learn — verify current bundling | `high-scope-grant-alert.md` |
| "OAuth apps view" | "OAuth apps" page in the Defender portal Cloud apps area; the new App Governance left-nav has subsumed much of the classic OAuth-apps surface for tenants with the add-on | `post-consent-cleanup-and-ban.md`, `high-scope-grant-alert.md` |
| "Microsoft Cloud regions page" (referenced without URL) | [Microsoft 365 data locations](https://learn.microsoft.com/en-us/microsoft-365/enterprise/o365-data-locations) | `block-download-unmanaged-device.md`, `inline-upload-block-regulated-data.md`, `auto-label-pci-data.md`, `geo-residency-block.md`, `inline-prompt-dlp.md` |
| "Cross-Tenant Access Settings (XTAS)" — used inconsistently as "CTAS" elsewhere | Microsoft Learn uses "Cross-tenant access settings" (lowercase tax in source) — abbreviation is not standardised in Microsoft docs; the practitioner abbreviation `CTAS` is used by some Microsoft blog posts but the docs say "cross-tenant access settings" | `tenant-restriction-corporate-only.md` (uses CTAS), `b2b-partner-exfil-alert.md` (uses XTAS) — pick one |
| "Microsoft Defender XDR data-lake retention" | Microsoft documents "Defender XDR advanced hunting retention" / "Microsoft 365 Defender data retention" with specific tier-dependent windows | `inline-upload-block-regulated-data.md` |

---

## Missing citations

Claims that need a Microsoft Learn URL but currently do not have one:

1. `block-download-unmanaged-device.md` line 82 — "MDE deployed with Network Protection in block mode" — needs link to [defender-endpoint/network-protection](https://learn.microsoft.com/en-us/defender-endpoint/network-protection).
2. `mass-download-alert.md` line 80 — "Activity-policy auto-disable at 200k/day or 100k/3h" — needs MS citation or explicit `[VERIFY]`.
3. `terminated-user-cross-saas.md` line 85 — "exact-string email match" claim for the built-in anomaly policy — needs link to [defender-cloud-apps/anomaly-detection-policy](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy).
4. `impossible-travel-alert.md` line 83 — country-level + bordering-country observation — needs link to the same anomaly-detection-policy page.
5. `inline-prompt-dlp.md` line 187 — Edge for Business gen-ai.app-control GA — needs link or `[VERIFY]`.
6. `auto-label-pci-data.md` line 227 — "File-policy ceiling = 200 per tenant" — needs link to [defender-cloud-apps/file-policy-overview](https://learn.microsoft.com/en-us/defender-cloud-apps/file-policy-overview) or equivalent.
7. `external-share-link-quarantine.md` line 122 — `purview_legal_hold: false` as a file-filter — needs MS link or reframe (see drift #5).
8. `post-consent-cleanup-and-ban.md` line 38 — "verified-publisher restriction" Entra control — needs link to [entra/identity/enterprise-apps/publisher-verification-overview](https://learn.microsoft.com/en-us/entra/identity-platform/publisher-verification-overview).
9. `high-scope-grant-alert.md` line 83 — App Governance specific surface and GA date — needs link to [defender-cloud-apps/app-governance-overview](https://learn.microsoft.com/en-us/defender-cloud-apps/app-governance-overview).
10. `geo-residency-block.md` line 84 — M365 multi-geo connector limit — needs link to [defender-cloud-apps/connect-office-365](https://learn.microsoft.com/en-us/defender-cloud-apps/connect-office-365).
11. `sspm-tenant-misconfig-drift.md` line 83 — MDA SSPM coverage list + Secure Score-derived rules — needs link to [defender-cloud-apps/saas-security-posture-management](https://learn.microsoft.com/en-us/defender-cloud-apps/saas-security-posture-management).

---

## Practitioner-inference flags

Claims for which I could not find a Microsoft source — legitimate practitioner judgement but should be labelled as such, not stated as Microsoft documentation:

1. "Bilateral CAAC trap" as a named pattern — the phenomenon (CA + MDA both required) is documented; the **name** "bilateral trap" is the playbook's coinage. Across all CAAC-related files. Leave the framing but flag once: "practitioner term".
2. "Tag-as-Unsanctioned in audit-mode MDE NP is the single most common Day 90 misconfiguration" — `discovery-and-auto-unsanction.md` line 85. Plausible practitioner observation; no Microsoft source surfaces this as a named misconfiguration class.
3. "30% of OAuth abuse covered by Policy 2a (delegated grants); 70% via service-principal class" — `post-consent-cleanup-and-ban.md` lines 85, 256. Practitioner estimate; no Microsoft figure.
4. "`window.parent.postMessage` cross-origin failure breaks Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview, Atlassian Compass after CAAC rewrite" — `block-download-unmanaged-device.md` line 82, `inline-upload-block-regulated-data.md` line 83. Microsoft has a known-issues list for CAAC but does not name specific apps in this style; practitioner observation aggregated.
5. "MDA quarantine on legal-hold material = spoliation" — `external-share-link-quarantine.md` line 81, `api-data-at-rest-quarantine.md` line 223. Practitioner / legal-counsel framing; not a Microsoft assertion.
6. "Storm-0501-class actor that pivoted from on-prem to SharePoint Online, deleting and overwriting team-site content" — `mass-delete-anomaly.md` use cases. Threat-intel narrative; cite Microsoft Threat Intelligence blog post(s) rather than leaving as inferred.
7. "Microsoft's risk score is opaque (uncited '90+ factors')" — `discovery-and-auto-unsanction.md` line 85. The risk score factor count is described in MDA's app risk-score documentation at [defender-cloud-apps/risk-score](https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score) — verify and cite.
8. "Tenant-restriction-v2 deprecation timeline of v1" — `tenant-restriction-corporate-only.md` line 84 — practitioner recall, marked `[VERIFY]`. Confirmed v1 → v2 migration but specific timeline qualifier needed.
9. "CAE-induced session re-establishment may bypass MDA session-policy re-evaluation — pair with Entra Token Protection" — multiple files. Microsoft does not explicitly document CAE-vs-MDA-session-policy interaction at a single Learn page. Practitioner-inference; flag as such.

---

## Top remediation priorities

Rank-ordered. Apply these in this order before tier-1 sign-off.

1. **Correct Claude Enterprise capability claims across `discovery-and-auto-unsanction.md` (line 154), `sanctioned-tenant-pinning.md` (line 198), `inline-prompt-dlp.md` (line 264).** Replace the "deep vs shallow" capture framing with the explicit capability matrix from [purview/ai-claude-enterprise](https://learn.microsoft.com/en-us/purview/ai-claude-enterprise) and [purview/ai-chatgpt-enterprise](https://learn.microsoft.com/en-us/purview/ai-chatgpt-enterprise). Claude Enterprise currently supports only DSPM-for-AI + Audit — NOT DLP, IRM, eDiscovery. This is the highest-impact correction because it materially mis-states regulator-attestable capabilities.

2. **Replace "eDiscovery (Premium)" with "eDiscovery" across `auto-label-pci-data.md`, `external-share-link-quarantine.md`, `api-data-at-rest-quarantine.md`, `mass-delete-anomaly.md`, `terminated-user-cross-saas.md`.** Classic eDiscovery (Premium) retired 2025-08-31 per [purview/ediscovery-overview](https://learn.microsoft.com/en-us/purview/ediscovery-overview).

3. **Standardise SIT parameter naming.** Replace `minimum_violations` and `minimum_unique_matches` with the documented terminology: `instance count` (Microsoft DLP rule schema field) or qualify as "MDA UI label". Affects `inline-upload-block-regulated-data.md`, `auto-label-pci-data.md`, `api-data-at-rest-quarantine.md`, `inline-prompt-dlp.md`.

4. **Replace `ApplyPurviewSensitivityLabel` / `PutInAdminQuarantine` synthetic API-style action names with Microsoft's documented action names: "Apply Microsoft Purview sensitivity label" / "Put file in admin quarantine".** `auto-label-pci-data.md` line 113, `external-share-link-quarantine.md` line 124.

5. **Add Microsoft Learn citation to Adaptive Protection claims** — every file mentioning "Adaptive Protection-integrated" should link to [purview/insider-risk-management-adaptive-protection](https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection). Also note that Adaptive Protection now wires into DLP, DLM, and Conditional Access — not just CA. Affects `block-unsanctioned-app-with-coach.md`, `auto-label-pci-data.md`, `mass-delete-anomaly.md`, `discovery-and-auto-unsanction.md`, `inline-prompt-dlp.md`.

6. **Add citation for the "activity policy auto-disable at 200k/day or 100k/3h" thresholds** in `mass-download-alert.md`, `mass-delete-anomaly.md`, `b2b-partner-exfil-alert.md`, or replace with `[VERIFY against current Microsoft anomaly-detection-policy docs]`.

7. **Add citation for the impossible-travel "country-level only, no bordering-country alerts" claim** at `impossible-travel-alert.md` line 83 — link to [defender-cloud-apps/anomaly-detection-policy](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy).

8. **Resolve or remove the `purview_legal_hold: false` MDA file-filter pseudo-config** at `external-share-link-quarantine.md` line 122. Replace with an explicit cross-reference to the Purview hold mechanism + a documented operational coordination note ("MDA File Policy does not natively filter on hold-state; the M365 retention engine resists deletion/quarantine on held content per the principles of retention").

9. **Add ChatGPT Endpoint DLP / Edge for Business browser DLP cross-reference** to `inline-prompt-dlp.md` and `inline-upload-block-regulated-data.md`. Microsoft's primary documented control for "block paste into ChatGPT" is **Endpoint DLP**, not CAAC. Cite [purview/endpoint-dlp-learn-about](https://learn.microsoft.com/en-us/purview/endpoint-dlp-learn-about) and [purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz](https://learn.microsoft.com/en-us/purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz).

10. **Add Workload Identities Premium licensing-tier citation** to `terminated-user-cross-saas.md` line 263 (service-principal coverage gap) and `high-scope-grant-alert.md` line 260. Link to [entra/identity/conditional-access/workload-identity](https://learn.microsoft.com/en-us/entra/identity/conditional-access/workload-identity). Distinguish risk-detection scope (without WIP — limited reporting) from policy-modification scope (WIP required).

---

End of report.
