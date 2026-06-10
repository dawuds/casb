# MDA configuration QA expert - QA report

QA review of 19 Tier-2 policy files in `/Users/dawud/claude/casb/03-policies/` against current Microsoft Learn documentation for Defender for Cloud Apps (learn.microsoft.com/en-us/defender-cloud-apps/*), the supporting Microsoft 365 connector page, and the Microsoft security-for-AI series. Date of source pull: 2026-06-10.

## Summary

- **Files reviewed:** 19 of 19
- **Drift to correct (Microsoft source contradicts the file):** 14 distinct items, ~38 file:section occurrences
- **Stale references (renamed / deprecated / disabled features still cited as live):** 8 distinct items
- **Missing citations (Microsoft-grade claim with no URL):** 12 distinct claim classes across the corpus
- **Practitioner-inference flags (legitimate but unverifiable against Microsoft Learn):** 9 distinct claim classes
- **Verified-clean (claim matches Microsoft source):** 18 distinct claim classes

The biggest single issue is the **file-policy limit**: every file that cites "file-policy ceiling = 200 per tenant" is wrong. Microsoft Learn `data-protection-policies` states **"You're limited to 50 file policies in Defender for Cloud Apps."** This appears across `api-data-at-rest-quarantine.md`, `auto-label-pci-data.md`, and others. Compliance evidence packs that include this number will fail an inspection.

The second-biggest issue is the **June 2025 anomaly-policy disablement wave**. Microsoft moved 8 named built-in policies (Suspicious inbox forwarding, Suspicious inbox manipulation rules, Suspicious email deletion activity, Activity from anonymous IP addresses, Activity from suspicious IP addresses, Ransomware activity, Unusual ISP for an OAuth App, Suspicious file access activity by user) to a dynamic-threat-detection model and **renamed several** (Ransomware activity → "Ransomware payment instruction file uploaded to {Application}"; Suspicious inbox forwarding → "Suspicious email forwarding rule created by third-party app"; etc.). None of the 19 policy files reflect this. Source: `anomaly-detection-policy` page, June 2025 transition notice.

The third issue is the claimed **2025 H2 GenAI sub-category split** (LLM / AI Coding Assistant / AI Image Generator / AI Video Generator / AI Voice-Avatar). The current `risk-score` page lists only **"Generative AI"** as a Cloud App Catalog category. There are two adjacent categories ("AI – MCP Server", "AI – Model Provider") but not the practitioner-claimed five-way split. The sub-category list in `discovery-and-auto-unsanction.md` and `inline-prompt-dlp.md` appears to be practitioner-inferred branding, not a Microsoft taxonomy.

## Verified-clean claims

Claims confirmed against the cited Microsoft Learn page on the day of QA pull:

1. **Bilateral CAAC trap** — Entra CA policy with "Use Conditional Access App Control" session control + MDA session policy both required. Verified against `session-policy-aad` ("In order for your session policy to work, you must also have a Microsoft Entra ID Conditional Access policy").
2. **`Always apply the selected action even if the data cannot be scanned`** — exact field name. Verified `session-policy-aad`.
3. **M365 multi-geo "supported only for OneDrive"** — verbatim match. Source: `protect-office-365` ("Multi-geo deployments are only supported for OneDrive").
4. **Power BI / Dynamics 365 connector latency 24-72 hours** — verbatim match. Source: `protect-office-365`.
5. **"Only the governance action of the first triggered policy is guaranteed to be applied"** — verbatim match. Source: `data-protection-policies`.
6. **Suspend user reverts for hybrid-AD users with `onPremisesSyncEnabled=true`** — verbatim concept. Source: `protect-office-365` ("If your Microsoft Entra ID is set to automatically sync... the settings in the on-premises environment override the Microsoft Entra settings and use of the Suspend user governance action is reverted").
7. **Subdomain discovery deprecated by 2025-12-31** — verbatim match. Source: `discovered-apps` ("The feature of discovered subdomains will be deprecated by Dec 31st, 2025").
8. **Default 7-day learning window for anomaly detection** — verified. Source: `anomaly-detection-policy` ("an initial learning period of seven days").
9. **Impossible-travel country-level granularity, no alerts within or between bordering countries** — verbatim match. Source: `anomaly-detection-policy` ("The locations are calculated on a country/region level... there will be no alerts for two actions originating in the same country/region or in bordering countries/regions").
10. **Sensitivity slider (Low/Medium/High) on Impossible Travel, no numeric thresholds published** — verified. Source: `anomaly-detection-policy` describes the three levels as "System, tenant, and user suppressions" without numeric thresholds.
11. **Externally-owned files in Box/Dropbox/Google Drive are unscannable; OneDrive reassigns ownership** — verbatim match. Source: `data-protection-policies` ("OneDrive assigns an internal user as the owner... Google Drive considers these files owned by the external user... Box considers externally owned files to be private information").
12. **Publicly-shared files in SharePoint/OneDrive may "incorrectly show up as private"** — verbatim match. Source: `protect-office-365`.
13. **OAuth apps page covers delegated permissions only** — verified. Source: `manage-app-permissions` ("Defender for Cloud Apps only identifies apps that request Delegated permissions").
14. **Ban app is tenant-wide and immediate** — concept verified against `manage-app-permissions` and `governance-actions` (action = "Ban app... revoke a third-party app's permissions and ban it from receiving permissions in the future").
15. **Apply Microsoft Purview sensitivity label as a file-policy governance action** — verified. Source: `governance-actions` ("Apply sensitivity label" action available for Box, OneDrive, Google Workspace, SharePoint).
16. **Put in admin quarantine action** — verified. Source: `governance-actions`. Note: only available for Microsoft 365 SharePoint, OneDrive for Business, Box.
17. **Cloud App Catalog 90+ risk factors** — verified. Source: `risk-score` ("Apps in the cloud app catalog are scored based on more than 90 risk factors").
18. **Suspend user, Notify user, Confirm user compromised, Require user to sign in again, Revoke OAuth app permission** — all available governance actions confirmed against `governance-actions`.

## Drift to correct

### 1. **File-policy limit is 50, not 200** (CRITICAL — multiple files)

- **Microsoft source:** `data-protection-policies` → "Limitations: You're limited to 50 file policies in Defender for Cloud Apps."
- **Files needing correction:**
  - `dlp/api-data-at-rest-quarantine.md` line 83 — "File-policy ceiling = 200 per tenant (verify against current docs)"
  - `dlp/auto-label-pci-data.md` line 226 — "File-policy ceiling = 200 per tenant (verify against current docs)"
- **Corrected text:** "File-policy ceiling = 50 per tenant per Microsoft Learn `data-protection-policies` (Limitations section, current as of 2026-06)."

### 2. **Built-in anomaly policies disabled June 2025 — several still cited as live**

- **Microsoft source:** `anomaly-detection-policy` "Important" notice — "Starting June 2025, Microsoft Defender for Cloud Apps began transitioning anomaly detection policies to a dynamic threat detection model. As part of these improvements... several legacy policies have been disabled: Activity from suspicious IP addresses, Suspicious inbox manipulation rules, Suspicious email deletion activity, Activity from anonymous IP addresses, Suspicious inbox forwarding, Unusual ISP for an OAuth App, Suspicious file access activity (by user), Ransomware activity."
- **Files affected:**
  - `detect/mass-delete-anomaly.md` — references "Ransomware activity" anomaly detection as if live; needs note that it has been renamed/replaced by dynamic detection ("Ransomware payment instruction file uploaded to {Application}").
  - `detect/terminated-user-cross-saas.md` — generally OK; "Activity performed by terminated user" is still live per Microsoft Learn.
  - `detect/impossible-travel-alert.md` — generally OK; "Impossible travel" still live.
  - `oauth/post-consent-cleanup-and-ban.md` line 256 — references "Misleading OAuth app name / Misleading publisher name / Malicious OAuth app consent" — these are still live (OAuth app anomaly detection policies under `app-permission-policy`). Verified-clean for this file. However, the "Unusual ISP for an OAuth App" feature has been renamed to "OAuth application activity from an unknown ISP" and migrated to dynamic detection — no file references this directly.
- **Corrected text:** add a note to every detect/* file: "Several built-in anomaly policies (Activity from suspicious IP addresses, Suspicious inbox forwarding, Suspicious inbox manipulation rules, Activity from anonymous IP addresses, Ransomware activity, Suspicious file access activity by user, Unusual ISP for an OAuth App, Suspicious email deletion activity) were transitioned to a dynamic-threat-detection model in June 2025 and renamed. Source: Microsoft Learn `anomaly-detection-policy`."

### 3. **GenAI sub-category split (LLM / AI Coding Assistant / AI Image / AI Video / AI Voice-Avatar) is not documented as a Microsoft taxonomy**

- **Microsoft source:** `risk-score` "Supported cloud app catalog categories" lists only **"Generative AI"** as the single GenAI-related category (plus the adjacent "AI – MCP Server" and "AI – Model Provider"). The five-way sub-category split is NOT in the Microsoft Learn taxonomy.
- **Files affected:**
  - `genai/discovery-and-auto-unsanction.md` lines 76, 99-101, 153, 231 — references "the 2025 H2 sub-category split (LLM / AI Coding Assistant / AI Image Generator / AI Video Generator / AI Voice-Avatar)" as if it were a Microsoft taxonomy. Anti-pattern #4 explicitly tells the practitioner to "Use the 2025 H2 sub-category split" — this guidance cannot be followed against current Microsoft documentation.
  - `genai/inline-prompt-dlp.md` line 92 — references sub-categories indirectly through Policy 8 cross-link.
- **Corrected text:** "The Microsoft Cloud App Catalog (as of June 2026) classifies generative AI under a single 'Generative AI' category. The five-way sub-category split (LLM / AI Coding Assistant / AI Image / AI Video / AI Voice-Avatar) referenced in some 2025-era industry sources is not in the published Microsoft taxonomy. Practitioners must rely on custom App tags (Sanctioned / Unsanctioned / custom) rather than Microsoft sub-categories for differentiated policy. Source: Microsoft Learn `risk-score` → Supported cloud app catalog categories."

### 4. **DSPM-for-AI deep capture extends to Google Gemini, not "ChatGPT Enterprise + Copilot only"**

- **Microsoft source:** `security/security-for-ai/discover` → "DSPM for AI provides deeper insights for Microsoft Copilots and third-party SaaS applications like ChatGPT Enterprise and Google Gemini."
- **Files affected:**
  - `genai/discovery-and-auto-unsanction.md` line 154 (Advanced maturity bullet) — "DSPM-for-AI prompt-capture deployed on the allowlisted enterprise-AI vendors (deep capture on ChatGPT Enterprise / Copilot, shallow on Claude / Gemini per the MDA playbook DSPM table)"
  - `genai/inline-prompt-dlp.md` line 224 — "shallow on Claude / Gemini per the MDA playbook DSPM table"
- **Corrected text:** "DSPM-for-AI provides deeper insights for Microsoft Copilots, ChatGPT Enterprise, and Google Gemini per Microsoft Learn (`security-for-ai/discover`). Anthropic Claude coverage depth must be verified against the current Microsoft Purview supported-sites page (`/purview/ai-microsoft-purview-supported-sites`); do not assert a shallow/deep capture tier without source verification."

### 5. **OAuth apps console path — incorrect for current portal**

- **Microsoft source:** `manage-app-permissions` → "In the Microsoft Defender Portal, under **Cloud Apps** select **OAuth apps**."
- **Files affected:**
  - `oauth/post-consent-cleanup-and-ban.md` line 85 — "Defender portal → Cloud apps → OAuth apps (or App governance if add-on enabled). Built-in anomaly policies under Cloud apps → Policies → Policy management → Threat detections"
  - `oauth/high-scope-grant-alert.md` line 83 — "Defender portal → Cloud apps → App governance → Predictive risk policies + OAuth threshold policies. (Re-verify 2026 portal path; App Governance may be a separate left-nav.)"
- **Status:** OAuth apps path is correct per Microsoft Learn. **App Governance path** (`app-governance-manage-app-governance`) confirms it as a separate surface ("App governance in Microsoft Defender for Cloud Apps") — but the policy file's hedge "Re-verify 2026 portal path" is appropriate. The drift is in claiming "Predictive risk policies" as a UI label — see Practitioner-inference #4 below.

### 6. **OAuth threshold policy / OAuth app policy nomenclature**

- **Microsoft source:** `control-cloud-apps-with-policies` lists "OAuth app policy" as a policy type with the note "These are built-in policies that come with Defender for Cloud Apps and can't be created."
- **Files affected:**
  - `oauth/high-scope-grant-alert.md` line 83 — references "OAuth threshold policy" as if it were a Microsoft-named user-creatable thing. The current Microsoft taxonomy says OAuth app policies are built-in (Misleading OAuth app name / Misleading publisher name / Malicious OAuth app consent).
- **Corrected text:** "Microsoft's named OAuth app anomaly detection policies are built-in (Misleading OAuth app name, Misleading publisher name, Malicious OAuth app consent — see Microsoft Learn `app-permission-policy`); the 'OAuth threshold policy' label used in this file is a practitioner shorthand for App Governance threshold-based detections (App Governance is a paid add-on with its own dashboard at Cloud apps → App governance)."

### 7. **"DCS" should be "Microsoft Data Classification Services" or "Microsoft Purview Information Protection"**

- **Microsoft source:** `session-policy-aad` and `data-protection-policies` use the term **"Data Classification Services"** and link to `content-inspection` / `dcs-inspection`. `data-protection-policies` says "We recommend using the Data Classification Services."
- **Files affected:** all policies use "DCS" / "Data Classification Service" — generally OK. However `inline-prompt-dlp.md` line 116 uses `method: DataClassificationService` in YAML, which is the practitioner-config-format and not a Microsoft API identifier. Acceptable as illustrative.
- **Verdict:** verified-clean; flagged here only for cross-file consistency check.

### 8. **Risk score direction — verified, but worth stating the citation**

- **Microsoft source:** `risk-score` doesn't explicitly state "lower = riskier"; however, the `discovered-apps` page filter example uses "**Risk score equals 6 or lower**" together with `SOC 2 = No`, `Data-at-rest encryption = Not supported` — which establishes lower = riskier by usage convention.
- **Files affected:**
  - `genai/discovery-and-auto-unsanction.md` line 85 — "Risk score ≤ 5 (where lower = higher risk on Microsoft's 1-10 scale)" — **correct directionally** but the "1-10" claim should be "0-10" per the risk-score page ("Each property receives a preliminary score between 0 and 10").
  - `access-control/block-unsanctioned-app-with-coach.md` line 80 — "auto-tag policy with `Risk score ≤ 5`" — correct.
- **Corrected text:** "Risk scoring uses a 0-10 scale; lower scores indicate higher risk per the Microsoft `discovered-apps` filter convention (`Risk score equals 6 or lower` is the Microsoft-published example for finding risky apps)."

### 9. **Activity-policy auto-disable thresholds (200k/day, 100k/3h)**

- **Microsoft source:** Could not verify these specific numeric thresholds in any of the Microsoft Learn pages fetched. The `user-activity-policies` and `anomaly-detection-policy` pages were not specifically queried for these numbers; verification incomplete.
- **Files affected:**
  - `detect/mass-download-alert.md` line 80 — "Activity-policy auto-disable at 200k/day or 100k/3h — silent saturation looks like normal alert decline."
  - `detect/b2b-partner-exfil-alert.md` line 84 — "Activity-policy auto-disable at 200k/day or 100k/3h — partner-tenant bulk events can saturate."
- **Status:** Flag as practitioner-inference until verified against `user-activity-policies` source. See Missing citations #2 below.

### 10. **Suspend user / SCIM cascade ~40 min default**

- **Microsoft source:** `governance-actions` describes Suspend user without naming a SCIM cascade time. Not verified.
- **Files affected:** `detect/mass-download-alert.md` line 80, `detect/mass-delete-anomaly.md` line 83.
- **Status:** Practitioner-inference; remove the specific "~40 min default" or cite a source.

### 11. **`Default 30-day activity-log retention (standard tier)` claim**

- **Microsoft source:** Could not verify in the Microsoft Learn pages fetched. The `discovered-apps` page mentions a 90-day retention for excluded-entity historical data; this is not the same as activity-log retention.
- **Files affected:** `detect/impossible-travel-alert.md` line 245.
- **Status:** Missing citation; the figure may be correct but is uncited.

### 12. **`Activity performed by terminated user` policy is built-in and Entra-as-IdP dependent** (concept verified, but file wording is sloppy)

- **Microsoft source:** `anomaly-detection-policy` → "Activity performed by terminated user... looks for users whose accounts were deleted in Microsoft Entra ID, but still perform activities in other platforms such as AWS or Salesforce." `protect-office-365` confirms "requires Microsoft Entra ID as IdP".
- **Files affected:** `detect/terminated-user-cross-saas.md` line 85 — correctly states "built-in; edit, do not create" and "requires Entra-as-IdP for the deletion signal". Verified-clean.

### 13. **"Microsoft Defender Portal" vs "Defender portal" vs "Microsoft Defender XDR" — inconsistency**

- **Microsoft source:** Current Microsoft Learn uses **"Microsoft Defender Portal"** (e.g. session-policy-aad) and also **"Microsoft Defender XDR"** (used in some procedures). The two terms refer to the same surface at security.microsoft.com but the documentation is inconsistent. The current navigation is `Microsoft Defender Portal` → `Cloud Apps` → `Policies` → `Policy management` → relevant tab.
- **Files affected:** All 19 files use "Defender portal → Cloud apps → ..." which is fine as a shorthand, but the formal nav is "Microsoft Defender Portal → Cloud Apps → Policies → Policy management → [tab]".
- **Status:** Minor stylistic drift; standardise to "Microsoft Defender Portal" + "Cloud Apps" (with capital A in some places).

### 14. **"Conditional Access App Control" tab under Policy management**

- **Microsoft source:** `session-policy-aad` step 1: "In Microsoft Defender XDR, select the **Cloud Apps → Policies → Policy management → Conditional Access** tab."
- **Files affected:**
  - `access-control/block-download-unmanaged-device.md` line 82 — "Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create Session policy" — verified-clean.
  - `dlp/inline-upload-block-regulated-data.md` line 83 — same. Verified-clean.
  - `genai/inline-prompt-dlp.md` line 83 — same. Verified-clean.

## Stale references

1. **"Activity from suspicious IP addresses"** — disabled June 2025; renamed to "Successful logon from a suspicious IP address" and "Activity from a password-spray associated IP address". No file references this directly, but the underlying concept appears in `detect/impossible-travel-alert.md` triage runbook and could be misread.
2. **"Suspicious inbox forwarding"** — disabled June 2025; renamed to "Suspicious email forwarding rule created by third-party app". Not referenced in current policies (though it should arguably be in the OAuth post-consent cleanup runbook).
3. **"Ransomware activity"** — disabled June 2025; renamed to "Ransomware payment instruction file uploaded to {Application}". Referenced as "Ransomware detection" in `protect-office-365` (still active link, but to the renamed concept). `detect/mass-delete-anomaly.md` line 8 references "T1486 Data Encrypted for Impact (SaaS variant)" and the file is otherwise OK, but a note that the built-in ransomware-anomaly has been migrated to dynamic detection would help.
4. **"Unusual ISP for an OAuth App"** — disabled June 2025; renamed to "OAuth application activity from an unknown ISP". `oauth/post-consent-cleanup-and-ban.md` does not reference this directly, but a OAuth-related triage runbook should now reflect the new policy name.
5. **"Activity from anonymous IP addresses"** — disabled; renamed to "Activity from a TOR IP address" and "Anonymous proxy activity". Not referenced.
6. **Subdomain discovery** — deprecated 2025-12-31 per `discovered-apps`. None of the 19 policy files reference subdomain discovery directly (good).
7. **File-policy limit "200"** — stale. Microsoft Learn states 50 (see Drift #1).
8. **Cloud App Catalog category "Generative AI" sub-category split** — the practitioner-claimed 5-way 2025 H2 split is stale (or never existed in Microsoft taxonomy). See Drift #3.

## Missing citations

Claims that read like Microsoft-grade facts but lack a Microsoft Learn URL anchor:

1. **"Japan East primary-data region (added 2025 H2)"** — appears in `dlp/auto-label-pci-data.md` line 199, `dlp/inline-upload-block-regulated-data.md` line 241, `access-control/geo-residency-block.md` line 230, `genai/inline-prompt-dlp.md` line 251, `detect/mass-download-alert.md` privacy section. Cite the Microsoft Cloud regions page or Microsoft Trust Center on the day of attestation.
2. **Activity-policy auto-disable threshold "200k/day or 100k/3h"** — used in `detect/mass-download-alert.md` and `detect/b2b-partner-exfil-alert.md`. Verify against `user-activity-policies` or remove.
3. **SCIM cascade "~40 min default"** — `detect/mass-download-alert.md` line 80. Need a Microsoft source.
4. **"Default 30-day activity-log retention (standard tier)" + "90+90 days with unified XDR data lake"** — `detect/impossible-travel-alert.md` coverage gaps. Cite the Microsoft Defender XDR data-retention page or a Sentinel-linked retention doc.
5. **Edge for Business `gen-ai.app-control` (H2 2026 GA)** — `genai/discovery-and-auto-unsanction.md` line 249, `genai/inline-prompt-dlp.md` line 291, `genai/sanctioned-tenant-pinning.md` line 197. Could not find Microsoft documentation for this feature name; cite Microsoft 365 roadmap or remove the explicit version claim.
6. **App Governance Predictive Risk as a Microsoft-named feature** — `oauth/high-scope-grant-alert.md` line 4, 83 and throughout. The Microsoft Learn `app-governance-manage-app-governance` page does NOT use the phrase "Predictive Risk"; the App Governance overview describes "Insights", "Governance", "Detection", "Remediation" capabilities. The "Predictive Risk" branding could not be found. Either cite a Microsoft source or flag as practitioner-inferred branding (see Practitioner-inference #4).
7. **"DCS does not OCR at session layer"** — appears across multiple files (`inline-upload-block-regulated-data.md`, `auto-label-pci-data.md`, `inline-prompt-dlp.md`). Verify against `content-inspection` or `dcs-inspection` Microsoft pages.
8. **"M365 latency near-real-time per Microsoft"** — appears across several detect/* files. The `protect-office-365` page does say "Faster near-real-time scanning speed for files in SharePoint and OneDrive" — cite this.
9. **`CloudAppEvents` field/schema references** — many files reference `CloudAppEvents` table fields (PolicyId, Verdict, ActionType, AccountObjectId, etc.). Cite the Microsoft Defender XDR advanced hunting schema page.
10. **Entra Token Protection "verify GA status"** — appears in multiple files. The hedged language is fine; a citation to the Microsoft Entra Token Protection page would be better than "verify GA status".
11. **`SecureScore-derived rule changes silently between CASB releases`** — `posture/sspm-tenant-misconfig-drift.md` line 83. Microsoft does not document this; flag as practitioner-inference or accept as a hedge.
12. **OAuth `Restrict-Access-To-Tenants` header v1 deprecation** — `access-control/tenant-restriction-corporate-only.md` line 84. Cite Microsoft Entra tenant restrictions v2 docs.

## Practitioner-inference flags

Claims for which no Microsoft Learn source was found; legitimate practitioner observations but should be labelled:

1. **"`window.parent.postMessage` cross-origin failure breaks Salesforce Lightning / ServiceNow Polaris / Workday Extend / Box file-preview / Atlassian Compass after CAAC rewrite"** — appears across `access-control/block-download-unmanaged-device.md`, `dlp/inline-upload-block-regulated-data.md`. Real-world observation; Microsoft does not publish a list of CAAC-incompatible UI patterns. Label as field observation.
2. **"`*.mcas.ms` URL rewrite breaks ..."** — same class. Microsoft documents the `*.mcas.ms` suffix for CAAC sessions in `session-policy-aad` but does not list incompatible apps. Practitioner inference.
3. **"Cert-pinned device profiles reject `*.mcas.ms` chain, fail closed at TLS but appear fail-open at app layer"** — practitioner observation; no Microsoft source.
4. **"App Governance Predictive Risk"** as a named feature with "shipped 2025 H2" claim — `oauth/high-scope-grant-alert.md` line 17 ("Predictive Risk shipped 2025 H2"). Could not be verified against Microsoft Learn. The June 2025 Microsoft change was to the **dynamic-threat-detection model** for anomaly detection, not a "Predictive Risk" branded feature in App Governance. Strongly recommend renaming to "App Governance behavioural-anomaly detection (dynamic detection model, June 2025)" or citing a specific Microsoft source.
5. **The "FP rate trajectory" tables across all policies** — these are useful empirical practitioner observations but should be explicitly labelled as illustrative practitioner experience, not Microsoft-published benchmarks.
6. **"Storm-2372 / Storm-0501 / MIDNIGHT BLIZZARD / TA453" attack-cluster naming** — these are Microsoft-published threat-actor names but the *post-incident frequency claims* and "consent-phishing wave" framing across the OAuth and access-control files are practitioner attribution. Cite Microsoft Threat Intelligence blog posts where used.
7. **Power Platform Admin Center DLP as "mandatory companion"** — `oauth/high-scope-grant-alert.md` line 247. Practitioner judgment; not a Microsoft-published requirement.
8. **"At 5k seats, clipboard inspection inflates Sentinel ingest by ~100 GB/month"** — `genai/inline-prompt-dlp.md` line 84. Explicitly labelled "sysint estimate" in the file; OK.
9. **The "concentration-risk" framing across multiple files** — practitioner / regulatory-evidence inference, not Microsoft-published. Acceptable as practitioner framing but should not read as a Microsoft control claim.

## Policies needing significant rework

1. **`oauth/high-scope-grant-alert.md`** — high-impact: relies heavily on "App Governance Predictive Risk" as a Microsoft-named feature (Practitioner-inference #4); contains claim "Predictive Risk shipped 2025 H2" without source. Also references "OAuth threshold policy" as a separately-named policy type, which is not in the Microsoft taxonomy. Recommend full rewrite of the Vendor implementation grid, the "Worked configuration example" YAML field names, and Anti-pattern #8 ("Rely on Predictive Risk alone").

2. **`genai/discovery-and-auto-unsanction.md`** — medium-to-high impact: builds the entire policy around the 5-way GenAI sub-category split (Drift #3) that is not in Microsoft taxonomy. Anti-pattern #4 instructs the practitioner to "Use the 2025 H2 sub-category split" — this guidance is not actionable against the current Microsoft Cloud App Catalog. Rewrite the category filter section, the worked-configuration YAML, and the anti-pattern.

3. **`dlp/api-data-at-rest-quarantine.md`** — medium impact: cites the wrong file-policy limit (200 vs 50), which is a Microsoft-published number that an auditor would check. Coverage-gaps section line 226 specifically calls this out as a coverage point — must be corrected.

4. **`dlp/auto-label-pci-data.md`** — medium impact: same file-policy-limit-200 issue (line 226). Plus claim "Cardholder-data scope often consumes 3-5 policies (per app × per label family)" — at 50-policy ceiling, this is much tighter than at 200.

5. **`detect/mass-delete-anomaly.md` + `detect/mass-download-alert.md`** — medium impact: cite specific numeric thresholds (200k/day, 100k/3h, SCIM 40 min) that are uncited; the underlying policy-design advice is sound, but the specific numbers must be cited or removed.

6. **`genai/inline-prompt-dlp.md`** — medium impact: DSPM-for-AI deep/shallow capture claim (Drift #4) contradicts current Microsoft framing; Anti-pattern #5 ("Treat the inline policy as the complete answer to GenAI data-loss... typing instead of pasting is a fundamental bypass") is otherwise excellent and aligned with Microsoft framing.

## Top remediation priorities

Rank-ordered ten specific corrections to apply first:

1. **(P0) Fix file-policy ceiling to 50, not 200** — `dlp/api-data-at-rest-quarantine.md` line 83; `dlp/auto-label-pci-data.md` line 226. Cite `data-protection-policies` Limitations section. Auditor-visible.

2. **(P0) Strip the "5-way GenAI sub-category split" claim from `genai/discovery-and-auto-unsanction.md`** — lines 76, 99-101, 153, 231. Replace with "Generative AI category (Cloud App Catalog) + custom App tags". Anti-pattern #4 must be rewritten so the practitioner is not instructed to use a non-existent Microsoft taxonomy.

3. **(P0) Add a "Built-in anomaly policies disabled June 2025" note to all detect/* files** — Microsoft transitioned 8 named policies to a dynamic detection model. The mass-delete-anomaly, impossible-travel-alert, terminated-user-cross-saas, and post-consent-cleanup-and-ban files cite or imply use of policies in that list and must reference the rename / migration.

4. **(P0) Verify "App Governance Predictive Risk" branding in `oauth/high-scope-grant-alert.md`** — cannot find a Microsoft Learn page for this feature name. Recommend either citing a specific Microsoft source for "Predictive Risk" or renaming to "App Governance behavioural-anomaly detection (dynamic detection model, June 2025)". Touches the entire Vendor implementation grid and worked-configuration YAML.

5. **(P1) Fix DSPM-for-AI coverage claim in `genai/discovery-and-auto-unsanction.md` line 154 and `genai/inline-prompt-dlp.md` line 224** — Microsoft `security-for-ai/discover` explicitly names ChatGPT Enterprise AND Google Gemini as having "deeper insights" from DSPM-for-AI. The "deep only for Copilot + ChatGPT Enterprise, shallow on Claude / Gemini" practitioner table contradicts the named Microsoft source.

6. **(P1) Cite Activity-policy auto-disable thresholds in `detect/mass-download-alert.md` line 80 and `detect/b2b-partner-exfil-alert.md` line 84** — verify 200k/day, 100k/3h against `user-activity-policies` page, or remove the numeric claim.

7. **(P1) Cite Activities retention claim in `detect/impossible-travel-alert.md` line 245** — "Default 30-day activity-log retention (standard tier) — historical correlation requires Sentinel forwarding or unified data lake (90+90)". Find the Microsoft data-retention source or remove.

8. **(P1) Standardise Microsoft Cloud regions claim** — multiple files reference "Japan East primary-data region (added 2025 H2)" without citation. Either link to the Microsoft Cloud Regions page or remove the version-specific claim.

9. **(P2) Add "Edge for Business `gen-ai.app-control`" source citation or remove the version-specific "(H2 2026 GA)" claim** — `genai/discovery-and-auto-unsanction.md` line 249, `genai/inline-prompt-dlp.md` line 291, `genai/sanctioned-tenant-pinning.md` line 197.

10. **(P2) Add empirical-vs-Microsoft-published labels to all FP-rate trajectory tables** — across all detect/*, dlp/*, oauth/*, genai/*, posture/* files. The tables read like Microsoft benchmarks; they are practitioner observations. A simple header line "FP rate trajectories below are illustrative practitioner observations, not Microsoft-published benchmarks" preserves the value and removes the false-precision risk.

## Sources

- [Control cloud apps with policies — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/control-cloud-apps-with-policies)
- [Create session policies — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad)
- [File policies — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies)
- [Create anomaly detection policies — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy)
- [Manage OAuth apps — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/manage-app-permissions)
- [Protect your Microsoft 365 environment — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365)
- [Governing connected apps — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/governance-actions)
- [Cloud app catalog and risk scores — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score)
- [View discovered apps — Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/discovered-apps)
- [App governance in Microsoft Defender for Cloud Apps and Microsoft Defender XDR](https://learn.microsoft.com/en-us/defender-cloud-apps/app-governance-manage-app-governance)
- [How do I discover AI apps and the sensitive data these use in my organization?](https://learn.microsoft.com/en-us/security/security-for-ai/discover)
