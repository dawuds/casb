# Consolidated QA report — 2026-06-10 — Microsoft-standards validation

Consolidates findings from five specialist QA reports against the 19 Tier-2 policy files in `/Users/dawud/claude/casb/03-policies/`. Specialists reviewed against: (a) Microsoft Defender for Cloud Apps configuration docs; (b) Microsoft Cybersecurity Reference Architecture + Zero Trust + CAF Secure; (c) CIS Microsoft 365 Foundations Benchmark v5.0.0 + Microsoft Secure Score; (d) Microsoft Purview + Entra integration; (e) MITRE ATT&CK + Microsoft Threat Intelligence.

## Executive summary

- **Files reviewed:** 19 of 19
- **Specialist reports:** 5 (MDA-config / MCRA-ZT / CIS-Secure-Score / Purview-Entra / ATT&CK-Threat-Intel)
- **Total drift items:** **68** (14 MDA + 11 MCRA + 11 CIS + 21 Purview/Entra + 11 ATT&CK) — with significant overlap, ~32 unique cross-cutting issues
- **Total stale-reference items:** **25** (8 MDA + 4 MCRA + 3 CIS + 7 Purview/Entra + 3 ATT&CK)
- **Total missing-citation items:** **57** (12 MDA + 9 MCRA + 19 CIS [system-wide CIS/Secure-Score gap across all 19 files] + 11 Purview/Entra + 6 ATT&CK)
- **Total practitioner-inference items:** **37** (9 MDA + 8 MCRA + 6 CIS + 9 Purview/Entra + 5 ATT&CK)
- **Policies needing significant rework (named by ≥1 specialist):** `oauth/high-scope-grant-alert.md`; `genai/discovery-and-auto-unsanction.md`; `detect/mass-delete-anomaly.md`; `detect/impossible-travel-alert.md`; `oauth/post-consent-cleanup-and-ban.md`; `dlp/api-data-at-rest-quarantine.md`; `dlp/auto-label-pci-data.md`; `genai/inline-prompt-dlp.md`; `detect/mass-download-alert.md`
- **Policies cleared (no specialist flagged for rework, only line-edits):** `access-control/block-unsanctioned-app-with-coach.md`; `access-control/tenant-restriction-corporate-only.md`; `access-control/block-download-unmanaged-device.md`; `access-control/geo-residency-block.md`; `dlp/inline-upload-block-regulated-data.md`; `dlp/external-share-link-quarantine.md`; `detect/terminated-user-cross-saas.md`; `detect/b2b-partner-exfil-alert.md`; `genai/sanctioned-tenant-pinning.md`; `posture/sspm-tenant-misconfig-drift.md`

> **Headline finding (consensus):** The corpus is architecturally and factually sound but **systematically under-cites Microsoft authoritative sources** for the controls it implements. Two structural gaps dominate: (1) the CIS Microsoft 365 Foundations Benchmark v5.0.0 and Microsoft Secure Score are cited **zero times** across all 19 files, despite being the two Microsoft-native baselines every BNM / SOC 2 / PCI / ISO auditor will probe; (2) at least 8 named MDA built-in anomaly policies were disabled or renamed in June 2025 and the affected policy files do not reflect this. Three further issues require content rework (not just line-edits): the GenAI 5-way sub-category split is not in the current Microsoft taxonomy; the App Governance "Predictive Risk" feature name is not in Microsoft Learn; and the Claude Enterprise capability claims contradict the published Microsoft capability matrix.

---

## Cross-cutting issues

Issues raised by 2+ specialists. Each lists affected files + canonical fix.

### X1. CIS Microsoft 365 Foundations Benchmark v5.0.0 + Microsoft Secure Score citations absent (consensus — 2 specialists: CIS, MDA-config implicitly)

- **Specialist consensus:** CIS specialist (D1 across all 19) + MDA-config specialist (Missing-citation #1 family).
- **Affected files:** all 19.
- **Canonical fix:** Add a `CIS Microsoft 365 Foundations Benchmark v5.0.0` row and a `Microsoft Secure Score` row to the `Control mappings` block in every file. CIS 2.4.3 (L2) is the umbrella for the entire library; per-file sub-controls per CIS report D2-D20. Secure Score citations should reference the current four-group structure (Identity / Device / Apps / Data — not the legacy five-group structure with Infrastructure).
- **Citations to add:** `https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score`; `https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions`; CIS v5.0.0 (30 April 2025).

### X2. File-policy ceiling = 50 per tenant, not 200 (consensus — 2 specialists: MDA-config, Purview-Entra)

- **Specialist consensus:** MDA-config Drift #1 says 50 per `data-protection-policies`; Purview-Entra Drift #6 says citation needed; CIS specialist Verified-clean #5-6 said "verify date" without finding the contradiction.
- **Resolution:** MDA-config is correct — the current Microsoft Learn page `data-protection-policies` Limitations section states "You're limited to 50 file policies in Defender for Cloud Apps." Earlier CIS-specialist clearance is stale.
- **Affected files:** `dlp/api-data-at-rest-quarantine.md` line 83; `dlp/auto-label-pci-data.md` line 226 (and recompute the "cardholder-data scope often consumes 3-5 policies" budget against the 50-ceiling).
- **Canonical fix:** "File-policy ceiling = 50 per tenant per Microsoft Learn `data-protection-policies` (Limitations section, current as of 2026-06)."

### X3. June 2025 built-in-anomaly-policy disablement / rename wave (consensus — 1 specialist with deep coverage: MDA-config; cross-referenced by ATT&CK and Purview-Entra)

- **Affected built-ins (Microsoft moved to dynamic-threat-detection model + renamed):** Activity from suspicious IP addresses; Suspicious inbox manipulation rules; Suspicious email deletion activity; Activity from anonymous IP addresses; Suspicious inbox forwarding; Unusual ISP for an OAuth App; Suspicious file access activity (by user); Ransomware activity.
- **Affected files:** `detect/mass-delete-anomaly.md` (Ransomware activity → "Ransomware payment instruction file uploaded to {Application}"); `detect/impossible-travel-alert.md` (cross-references); `oauth/post-consent-cleanup-and-ban.md` (Unusual ISP for OAuth App → "OAuth application activity from an unknown ISP"); `detect/terminated-user-cross-saas.md` (corroborating cross-reference only; "Activity performed by terminated user" itself remains live).
- **Canonical fix:** Add a single boilerplate paragraph to all detect/* files: "Several built-in anomaly policies were transitioned to a dynamic-threat-detection model in June 2025 and renamed — Activity from suspicious IP addresses, Suspicious inbox forwarding, Suspicious inbox manipulation rules, Activity from anonymous IP addresses, Ransomware activity, Suspicious file access activity by user, Unusual ISP for an OAuth App, Suspicious email deletion activity. Source: Microsoft Learn `anomaly-detection-policy` (Important notice)."

### X4. GenAI sub-category split (LLM / AI Coding Assistant / AI Image Generator / AI Video Generator / AI Voice-Avatar) is not in Microsoft taxonomy (consensus — 2 specialists: MDA-config, Purview-Entra)

- **Specialist disagreement:** CIS specialist Verified-clean #14 marks this as verified; MDA-config and Purview-Entra specialists contradict.
- **Resolution:** MDA-config is correct — the current `risk-score` page lists only "Generative AI" plus adjacent "AI – MCP Server" and "AI – Model Provider". The 5-way split is **not** in published Microsoft taxonomy. CIS specialist's "verified-clean" is stale.
- **Affected files:** `genai/discovery-and-auto-unsanction.md` lines 76, 99-101, 153, 231 (Anti-pattern #4 instructs use of the split); `genai/inline-prompt-dlp.md` line 92 (cross-link).
- **Canonical fix:** "The Microsoft Cloud App Catalog (as of June 2026) classifies generative AI under a single 'Generative AI' category. Practitioners must rely on custom App tags rather than Microsoft sub-categories. Source: `risk-score` → Supported cloud app catalog categories." Rewrite Anti-pattern #4 in `genai/discovery-and-auto-unsanction.md`.

### X5. DSPM-for-AI / Claude Enterprise capability matrix (consensus — 2 specialists: MDA-config, Purview-Entra)

- **Specialist consensus:** MDA-config Drift #4 (Microsoft `security-for-ai/discover` names Copilot + ChatGPT Enterprise + Google Gemini as deep-capture); Purview-Entra Drift #3 (Claude Enterprise capability matrix from `ai-claude-enterprise` shows DSPM + Audit only, NOT DLP / IRM / eDiscovery / labels).
- **Affected files:** `genai/discovery-and-auto-unsanction.md` line 154; `genai/inline-prompt-dlp.md` lines 224, 264; `genai/sanctioned-tenant-pinning.md` line 198.
- **Canonical fix:** Replace "deep vs shallow" framing with the explicit capability matrix per `ai-claude-enterprise` + `ai-chatgpt-enterprise` Microsoft Learn pages. ChatGPT Enterprise: DSPM + Audit + Data classification + IRM + eDiscovery + DLM supported; DLP and sensitivity labels NOT supported. Claude Enterprise (preview): DSPM + Audit only; classification, DLP, sensitivity labels, IRM, eDiscovery NOT supported. Google Gemini: third-party-AI-sites coverage via browser extension + endpoint DLP only.

### X6. Activity-policy auto-disable thresholds (200k/day, 100k/3h) — citation absent (consensus — 2 specialists: MDA-config, Purview-Entra)

- **Affected files:** `detect/mass-download-alert.md` line 80; `detect/b2b-partner-exfil-alert.md` line 84; `detect/mass-delete-anomaly.md` line 83 (cross-reference).
- **Disagreement:** CIS specialist Verified-clean #10 marks the numeric thresholds as verified. MDA-config and Purview-Entra could not locate them.
- **Resolution:** Numbers may be correct (CIS specialist's verification path is partial — likely an internal training-doc rather than Microsoft Learn). Either find an authoritative Microsoft Learn URL or mark `[VERIFY]`. Default to `[VERIFY]` if no public URL.

### X7. SCIM cascade "~40 min default" — uncited (consensus — 1 specialist: MDA-config; not flagged by others)

- **Affected files:** `detect/mass-download-alert.md` line 80; `detect/mass-delete-anomaly.md` line 83.
- **Canonical fix:** Remove specific time-figure or cite a Microsoft source.

### X8. CAAC reverse-proxy `*.mcas.ms` framing is stale for Edge-on-Windows (consensus — 2 specialists: MDA-config implicitly, MCRA-ZT explicitly)

- **Specialist consensus:** MCRA-ZT Drift #3 + Stale-reference #1 (Edge for Business in-browser protection per `proxy-intro-aad` removes the redirect chain for Edge users).
- **Affected files:** `access-control/block-download-unmanaged-device.md` line 82, 243; `dlp/inline-upload-block-regulated-data.md` line 83.
- **Canonical fix:** "`*.mcas.ms` URL rewrite is the reverse-proxy path for non-Edge browsers; Edge users get direct in-browser protection per Microsoft Learn `proxy-intro-aad`. Document residual reverse-proxy dependency for non-Edge browsers; track Edge for Business adoption."

### X9. "Concentration risk" framing is regulator language, not Microsoft (consensus — 1 specialist: MCRA-ZT; touches multiple files)

- **Specialist:** MCRA-ZT Drift #1, #2, #3 — Microsoft frames the integration as "unified XDR / end-to-end security" per MCRA April 2025; "concentration risk" is BNM RMiT / DORA / MAS TRM framing.
- **Affected files:** `genai/discovery-and-auto-unsanction.md` line 243; `detect/mass-delete-anomaly.md` lines 227, 250; `access-control/block-download-unmanaged-device.md` line 243.
- **Canonical fix:** Add explicit attribution sentence at each occurrence: "Concentration-risk framing follows BNM RMiT third-party-concentration / DORA Art. 28-30 supervisory framing — not Microsoft's own positioning, which is 'unified XDR' / 'end-to-end security' per MCRA April 2025."

### X10. Threat-actor naming hygiene (consensus — 1 specialist: ATT&CK; touches 7+ files)

- **Specialist:** ATT&CK Drift #1, #2; Stale-reference #1, #2.
- **TA453 → Mint Sandstorm** (Microsoft canonical; TA453 is Proofpoint taxonomy): `oauth/post-consent-cleanup-and-ban.md` lines 17, 31; `detect/impossible-travel-alert.md` use-case 3.
- **MIDNIGHT BLIZZARD all-caps → Midnight Blizzard** (Title Case per Microsoft Apr 2023 rename): `oauth/high-scope-grant-alert.md` lines 17, 151, 245; `detect/mass-delete-anomaly.md` line 250; `posture/sspm-tenant-misconfig-drift.md` line 11.
- **Storm-0558 → Antique Typhoon** (promoted from Group-in-development): generalised references in `oauth/post-consent-cleanup-and-ban.md` lines 237, 273; `detect/mass-delete-anomaly.md` line 250; `oauth/high-scope-grant-alert.md` line 17. Keep "Storm-0558" only for the specific July 2023 token-forgery incident.
- **Canonical fix:** Three find/replace passes. Add citation `https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming` for the taxonomy.

### X11. MITRE ATLAS AML.T0024 renamed "ML Inference API" → "AI Inference API" (consensus — 1 specialist: ATT&CK)

- **Affected files:** `genai/discovery-and-auto-unsanction.md` lines 11, 162; `genai/inline-prompt-dlp.md` lines 11, 199.
- **Canonical fix:** Find/replace `Exfiltration via ML Inference API` → `Exfiltration via AI Inference API`.

### X12. eDiscovery (Premium) classic-experience retirement 2025-08-31 (consensus — 1 specialist: Purview-Entra)

- **Affected files:** `auto-label-pci-data.md` line 224; `external-share-link-quarantine.md` line 122; `api-data-at-rest-quarantine.md` lines 105, 223; `mass-delete-anomaly.md`; `terminated-user-cross-saas.md`.
- **Canonical fix:** Replace "Purview eDiscovery (Premium)" with "Purview eDiscovery" (the new eDiscovery experience in the Purview portal); note classic experiences retired 2025-08-31. Cite `ediscovery-overview`.

### X13. SIT parameter naming + governance-action API names (consensus — 1 specialist: Purview-Entra)

- **SIT parameters:** `minimum_violations` / `minimum_unique_matches` → "instance count" (`minCount` / `maxCount`) + "confidence level" (`confidenceLevel`). Affects `inline-upload-block-regulated-data.md`, `auto-label-pci-data.md`, `api-data-at-rest-quarantine.md`, `inline-prompt-dlp.md`.
- **Governance action names:** `ApplyPurviewSensitivityLabel` → "Apply Microsoft Purview sensitivity label"; `PutInAdminQuarantine` → "Put file in admin quarantine". Affects `auto-label-pci-data.md` line 113; `external-share-link-quarantine.md` line 124.

### X14. App Governance "Predictive Risk" feature name and "shipped 2025 H2" claim unverified (consensus — 2 specialists: MDA-config, MCRA-ZT, Purview-Entra)

- **Affected file:** `oauth/high-scope-grant-alert.md` lines 4, 17, 83.
- **Canonical fix:** Either find a Microsoft Learn URL for "Predictive Risk" branding or rename to "App Governance behavioural-anomaly detection (dynamic detection model, June 2025)". Update portal path to reflect Dec 2025 unified-RBAC integration per Microsoft Learn release-notes.

### X15. OAuth-apps view covers delegated permissions only (consensus — 2 specialists: MDA-config Verified-clean #13, Purview-Entra Drift #17)

- Verified-clean by both — application (client-credentials) grants are NOT in this view. The 30%-of-OAuth-abuse coverage figure in `post-consent-cleanup-and-ban.md` line 85 is practitioner-inference (no Microsoft source).

### X16. "Suspend user" architectural ownership (consensus — 1 specialist: MCRA-ZT)

- MCRA-ZT Drift #6: the SUSPEND decision is owned by Entra (Identity ZT pillar Objective V via ID Protection + CA), with MDA as signal source. The hybrid-AD `onPremisesSyncEnabled=true` silent-fail mode is correctly documented but the architectural attribution is sloppy.
- **Affected files:** `detect/mass-download-alert.md` L66; `detect/mass-delete-anomaly.md` L67-68, L83.
- **Canonical fix:** Add sentence: "The suspend decision is owned by Entra (Identity ZT pillar Objective V); MDA provides the activity signal. Architecturally: MDA alert → Entra ID Protection user-risk boost → CA policy enforces."

### X17. Tenant Restrictions v2 architectural ownership (consensus — 1 specialist: MCRA-ZT)

- MCRA-ZT Drift #4: TRv2 is an Entra + GSA control (Identity ZT pillar Objective IV); MDA's role is the in-session CAAC supplement only.
- **Affected files:** `access-control/tenant-restriction-corporate-only.md` line 84.
- **Canonical fix:** Reframe header attribution paragraph (see MCRA-ZT report for verbatim text).

### X18. FP-rate trajectory tables are practitioner observations, not Microsoft benchmarks (consensus — 2 specialists: MDA-config, CIS)

- **Affected files:** all 19.
- **Canonical fix:** Add a single header line to every FP-rate trajectory table: "Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline."

### X19. "MDA Policy N" internal numbering presented as if Microsoft documentation (consensus — 1 specialist: MCRA-ZT)

- **Affected files:** multiple — across the entire corpus.
- **Canonical fix:** Either link to the playbook reference or annotate the numbering as internal.

### X20. Microsoft Cloud regions / "Japan East primary-data region (added 2025 H2)" citation absent (consensus — 2 specialists: MDA-config, Purview-Entra)

- **Affected files:** `dlp/auto-label-pci-data.md` line 199; `dlp/inline-upload-block-regulated-data.md` line 241; `access-control/geo-residency-block.md` line 230; `genai/inline-prompt-dlp.md` line 251; `access-control/block-download-unmanaged-device.md` line 230; `detect/mass-download-alert.md` privacy section.
- **Canonical fix:** Cite `microsoft-365/enterprise/o365-data-locations` on the day of attestation; mark specific-quarter claims as `[VERIFY against current Microsoft Cloud regions page]`.

### X21. Edge for Business `gen-ai.app-control` (H2 2026 GA) — feature name not in Microsoft docs (consensus — 2 specialists: MDA-config, Purview-Entra)

- **Affected files:** `genai/discovery-and-auto-unsanction.md` line 249; `genai/inline-prompt-dlp.md` lines 187, 291; `genai/sanctioned-tenant-pinning.md` line 197.
- **Canonical fix:** Replace specific feature name with documented "Browser DLP via Edge for Business" + cite `purview/dlp-browser-dlp-learn` and `purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz`; mark GA date as `[VERIFY against current Microsoft 365 roadmap]`.

### X22. Adaptive Protection citation absent (consensus — 1 specialist: Purview-Entra)

- **Affected files:** `block-unsanctioned-app-with-coach.md`, `auto-label-pci-data.md`, `mass-delete-anomaly.md`, `discovery-and-auto-unsanction.md`, `inline-prompt-dlp.md`.
- **Canonical fix:** Cite `purview/insider-risk-management-adaptive-protection`. Also note Adaptive Protection now wires into DLP, DLM, and Conditional Access — not just CA.

### X23. Entra admin-consent workflow primary-control citation (consensus — 2 specialists: MCRA-ZT, Purview-Entra)

- **Affected files:** `oauth/post-consent-cleanup-and-ban.md` (correct framing, missing citation); `oauth/high-scope-grant-alert.md`.
- **Canonical fix:** Cite `https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity` (Objective IV "Restrict user consent to applications") + `https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants` for the cleanup workflow.

### X24. Power Platform service-principal coverage gap (consensus — 2 specialists: CIS, Purview-Entra)

- CIS specialist D16 names CIS 9.1.10 (L1) "Ensure access to APIs by Service Principals is restricted" as a direct map. Purview-Entra Drift #18 confirms Power Platform Admin Center DLP as the documented compensating control.
- **Affected file:** `oauth/high-scope-grant-alert.md` lines 28-31, 256, 269.
- **Canonical fix:** Add CIS 9.1.10 / 9.1.11 mappings + cite `power-platform/admin/wp-data-loss-prevention`.

### X25. ATT&CK T1530 mis-mapping on upload-block policy (consensus — 1 specialist: ATT&CK)

- ATT&CK D4: T1530 is Collection (adversary reading cloud storage), not Exfiltration. The upload-block control maps to T1567/T1567.002, not T1530.
- **Affected file:** `dlp/inline-upload-block-regulated-data.md` line 11.
- **Canonical fix:** Remove T1530 from the Purpose statement; keep T1567 (or T1567.002 for greater precision).

### X26. T1098.003 Additional Cloud Roles missing (consensus — 1 specialist: ATT&CK)

- ATT&CK D10: scope-expansion alert class in `oauth/high-scope-grant-alert.md` maps to T1098.003 (roles) better than T1098.001 (credentials) alone.
- **Canonical fix:** Add T1098.003 to Purpose statement.

### X27. T1136 → T1136.003 precision (consensus — 1 specialist: ATT&CK)

- ATT&CK D9.
- **Affected file:** `detect/terminated-user-cross-saas.md` line 11.
- **Canonical fix:** Replace `T1136 Create Account` with `T1136.003 Cloud Account`.

### X28. T1567 → T1567.002 precision (consensus — 1 specialist: ATT&CK)

- ATT&CK D5: greatest precision for SaaS-upload-block is `T1567.002 Exfiltration to Cloud Storage`.
- **Affected files:** `dlp/inline-upload-block-regulated-data.md` line 11; `genai/inline-prompt-dlp.md` line 11; `access-control/block-unsanctioned-app-with-coach.md` line 11 (parent T1567 acceptable for shadow-app exfil class).
- **Canonical fix:** Where the control targets a specific cloud-storage upload (OneDrive / SharePoint / Box / ChatGPT), use T1567.002. Where it targets the broader shadow-app exfil class, keep T1567 parent.

### X29. T1528 framing of "no sign-in event" (consensus — 1 specialist: ATT&CK)

- ATT&CK D6: the original consent event IS audit-logged; the gap is the lack of fresh interactive sign-ins for OAuth-app-as-actor calls.
- **Affected file:** `detect/impossible-travel-alert.md` line 240.
- **Canonical fix:** Replace "no sign-in event for the OAuth abuse itself" with the more precise framing per ATT&CK D6.

### X30. SSPM Microsoft architectural anchor missing (consensus — 1 specialist: MCRA-ZT)

- MCRA-ZT Drift #9: Application ZT Objective VI is the Microsoft anchor for SaaS posture; the specialist-SSPM-tool framing in the policy is practitioner judgment.
- **Affected file:** `posture/sspm-tenant-misconfig-drift.md` line 11.
- **Canonical fix:** Cite `https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications` Objective VI; flag specialist-SSPM-tool depth as practitioner inference.

### X31. Data ZT pillar citation missing on DLP policies (consensus — 1 specialist: MCRA-ZT)

- MCRA-ZT Drift #10: Microsoft documents three governance actions (remove permissions, quarantine, apply labels) at `https://learn.microsoft.com/en-us/security/zero-trust/deploy/data` (Objective III).
- **Affected files:** `dlp/api-data-at-rest-quarantine.md` line 11; `dlp/external-share-link-quarantine.md` line 11.
- **Canonical fix:** Add Data ZT Objective III citation to both files.

### X32. `purview_legal_hold: false` pseudo-config filter — not a real MDA filter (consensus — 1 specialist: Purview-Entra)

- Purview-Entra Drift #5: no MDA File Policy filter named `purview_legal_hold` is documented at Microsoft Learn.
- **Affected file:** `external-share-link-quarantine.md` line 122.
- **Canonical fix:** Replace YAML pseudo-config with explicit cross-reference to the M365 retention engine + operational coordination note.

---

## File-by-file remediation

Each section lists every issue raised by any specialist for that file, merged + ranked.

### `access-control/block-unsanctioned-app-with-coach.md`

- **CIS (D2):** Add CIS 1.3.4 (L1), CIS 5.1.2.2 (L2), CIS 2.4.3 (L2); Secure Score Apps group ("Sanction or unsanction cloud apps", "Block unsanctioned cloud apps"). Tier L2.
- **MCRA-ZT (Missing-citation #1):** Cite Application ZT Initial Deployment Objective II.
- **Purview-Entra (Top remediation #5):** Add Adaptive Protection citation (`purview/insider-risk-management-adaptive-protection`).
- **MDA-config:** Risk-score scale 0-10 (line 80 reads as 1-10 in places).
- **Verdict:** **P1 — line-edits only.**

### `access-control/tenant-restriction-corporate-only.md`

- **CIS (D3):** Add CIS 5.1.6.2 (L1), CIS 5.1.6.1 (L2), CIS 5.2.2.9 (L1), CIS 5.2.2.12 (L1) — the policy describes the Storm-2372 threat extensively but never cites the CIS control that addresses it directly.
- **MCRA-ZT (Drift #4 / X17):** Reframe header attribution as Entra + GSA control, with MDA limited to in-session CAAC supplement.
- **Purview-Entra (Drift #16):** Resolve `[VERIFY]` qualifier on v1 deprecation timeline; cite `entra/external-id/tenant-restrictions-v2`.
- **MDA-config (Missing-citation #12):** OAuth `Restrict-Access-To-Tenants` header v1 deprecation citation.
- **Verdict:** **P0 — architectural reframing required.**

### `access-control/block-download-unmanaged-device.md`

- **CIS (D4):** Add CIS 7.3.2 (L2), CIS 5.2.2.9 (L1), CIS 1.3.2 (L2), CIS 2.4.3 (L2); Secure Score Apps + Data groups. Tier L2.
- **MCRA-ZT (Drift #3 / X8):** Update CAAC `*.mcas.ms` framing — Edge for Business in-browser protection direction-of-travel.
- **MCRA-ZT (Stale-reference #3):** Cite SharePoint/Exchange "limited access" Microsoft pattern.
- **Purview-Entra (Missing-citation #1):** Cite MDE Network Protection page.
- **Purview-Entra (Drift #15):** Japan East citation (X20).
- **MCRA-ZT (Drift #3 / X9):** "Concentration-risk register" reframing.
- **Verdict:** **P0 — three structural updates.**

### `access-control/geo-residency-block.md`

- **CIS (D5 / P1):** No direct CIS or Secure Score mapping. Label explicitly as practitioner-inference control to satisfy regulator-driven residency obligations (BNM / PDPA / GDPR Ch. V / DORA).
- **MCRA-ZT (Missing-citation #4):** Cite Data ZT pillar (`https://learn.microsoft.com/en-us/security/zero-trust/deploy/data`) — Purview as data-residency control layer.
- **Purview-Entra (Drift #14, #15):** Cite M365 connector page for multi-geo claim; cite Microsoft 365 data locations for Japan East.
- **MDA-config (Verified-clean #3):** M365 multi-geo "supported only for OneDrive" — already verified.
- **Verdict:** **P1 — citations + explicit practitioner-inference label.**

### `dlp/inline-upload-block-regulated-data.md`

- **CIS (D6):** Add CIS 3.2.1 (L1), CIS 3.2.2 (L1), CIS 3.3.1 (L1), CIS 2.4.3 (L2); Secure Score Data + Apps groups. Split L1 (policy intent) vs L2 (CAAC mechanism).
- **MCRA-ZT (Missing-citation #6):** Cite Data ZT Objective IV + Application ZT Objective III.
- **MCRA-ZT (Drift #3 / X8):** Update CAAC `*.mcas.ms` framing for Edge.
- **Purview-Entra (Drift #1 / X13):** SIT parameter naming (`minimum_unique_matches` → `instance count` / `minCount`).
- **Purview-Entra (Drift #4):** Add Endpoint DLP / Edge for Business cross-reference — Microsoft's primary documented control for ChatGPT paste-block.
- **ATT&CK (Drift D4 / X25):** Remove T1530 from Purpose.
- **ATT&CK (Drift D5 / X28):** Tighten T1567 → T1567.002.
- **Practitioner-inference (X18):** Mark `window.parent.postMessage` failure list as field observation.
- **Verdict:** **P0 — multiple structural items.**

### `dlp/api-data-at-rest-quarantine.md`

- **MDA-config (Drift #1 / X2 — P0):** Correct file-policy ceiling from 200 to 50.
- **CIS (D7):** Add CIS 3.2.1 (L1), CIS 3.3.1 (L1), CIS 2.4.3 (L2); Secure Score Data group. Split L1 / L2.
- **MCRA-ZT (Drift #10 / X31):** Add Data ZT Objective III citation (three governance actions).
- **Purview-Entra (Drift #6):** Cite "Only the governance action of the first triggered policy is guaranteed" (`file-filters` or file-policy reference).
- **Purview-Entra (Drift #1 / X13):** SIT parameter naming.
- **Purview-Entra (X12):** "eDiscovery (Premium)" → "Purview eDiscovery".
- **Purview-Entra (Missing-citation #6):** Cite file-policy ceiling.
- **Verdict:** **P0 — file-policy ceiling correction is auditor-visible.**

### `dlp/external-share-link-quarantine.md`

- **CIS (D8 — MOST under-mapped policy in library):** Add CIS 7.2.3 (L1), 7.2.4 (L2), 7.2.5 (L2), 7.2.6 (L2), 7.2.7 (L1), 7.2.9 (L1), 7.2.11 (L1); Secure Score Apps + Data groups.
- **MCRA-ZT (Drift #10 / X31):** Add Data ZT Objective III citation.
- **Purview-Entra (Drift #5 / X32):** Resolve `purview_legal_hold: false` pseudo-config — replace with M365 retention engine cross-reference.
- **Purview-Entra (Drift #2 / X13):** `PutInAdminQuarantine` → "Put file in admin quarantine".
- **Purview-Entra (X12):** "eDiscovery (Premium)" → "Purview eDiscovery".
- **Purview-Entra (Practitioner-inference #5):** "MDA quarantine on legal-hold = spoliation" is legal-counsel framing; label as such.
- **Verdict:** **P0 — seven specific CIS controls map directly + structural item on the pseudo-config filter.**

### `dlp/auto-label-pci-data.md`

- **MDA-config (Drift #1 / X2 — P0):** Correct file-policy ceiling from 200 to 50. Recompute "3-5 policies" budget against 50-ceiling.
- **CIS (D9):** Add CIS 3.3.1 (L1) — primary, CIS 3.2.1 (L1), CIS 9.1.6 (L1), CIS 2.4.3 (L2). Secure Score Data group.
- **MCRA-ZT (Missing-citation #5):** Cite Data ZT Objectives I + II.
- **Purview-Entra (Drift #2 / X13):** Governance-action name `ApplyPurviewSensitivityLabel` → "Apply Microsoft Purview sensitivity label".
- **Purview-Entra (Drift #1 / X13):** SIT parameter naming.
- **Purview-Entra (X12):** "eDiscovery (Premium)" → "Purview eDiscovery".
- **Purview-Entra (Drift #15 / X20):** Japan East citation.
- **Verdict:** **P0 — file-policy ceiling auditor-visible.**

### `detect/mass-download-alert.md`

- **CIS (D10):** Add CIS 2.4.3 (L2) — Microsoft explicitly names "Mass access to sensitive files" as MDA-dependent detection. Add CIS 2.2.1 (L1). Secure Score Apps group.
- **MCRA-ZT (Missing-citation #7):** Cite Application ZT Objective V (Defender for Cloud Apps UEBA).
- **MCRA-ZT (Drift #6 / X16):** Suspend-decision architectural ownership (Entra owns; MDA signal source).
- **MDA-config (Drift #9 / X6):** Activity-policy auto-disable thresholds citation needed.
- **MDA-config (Drift #10 / X7):** SCIM cascade ~40 min citation needed.
- **Purview-Entra (Missing-citation #2):** Same auto-disable thresholds gap.
- **Verdict:** **P0 — specialist named for significant rework on threshold-citation grounds.**

### `detect/impossible-travel-alert.md`

- **CIS (D11):** Add CIS 2.4.3 (L2) — Microsoft explicitly names "Impossible travel detection" as MDA-dependent. Add CIS 5.2.2.6 (L1), 5.2.2.7 (L1), 5.2.2.8 (L2). Secure Score Identity group.
- **MCRA-ZT (Missing-citation #8):** Cite Identity ZT Objective V.
- **MDA-config (Verified-clean #9):** Country-level granularity + bordering-country observation — verified.
- **MDA-config (Missing-citation #4):** "Default 30-day activity-log retention (standard tier)" citation needed.
- **MDA-config (X3):** Add June 2025 anomaly-policy disablement note.
- **Purview-Entra (Drift #9):** Cite country-level + bordering-country claim.
- **Purview-Entra (Drift #19):** Cite Sensitivity-slider opacity claim.
- **ATT&CK (Drift D1 / X10):** TA453 → Mint Sandstorm (use-case 3).
- **ATT&CK (Drift D6 / X29):** Tighten T1528 framing line 240.
- **Verdict:** **P0 — ATT&CK specialist flagged for significant rework (TA-naming + T1528 framing + uncited Microsoft Threat Intel claims).**

### `detect/mass-delete-anomaly.md`

- **CIS (D12):** Add CIS 2.4.3 (L2), CIS 3.1.1 (L1), CIS 6.2.1 (L1). Secure Score Apps group.
- **MCRA-ZT (Drift #2 / X9 — significant rework):** "Concentration risk" framing in two places (lines 227, 250) — reframe as regulator-derived, not Microsoft-derived. Anchor on CAF Secure.
- **MCRA-ZT (Drift #6 / X16):** Suspend-decision architectural ownership.
- **MDA-config (X3):** Ransomware activity → "Ransomware payment instruction file uploaded to {Application}" (June 2025 rename).
- **MDA-config (Drift #9 / X6):** Activity-policy auto-disable thresholds citation.
- **MDA-config (Drift #10 / X7):** SCIM cascade citation.
- **Purview-Entra (X12):** "eDiscovery (Premium)" → "Purview eDiscovery".
- **Purview-Entra (Practitioner-inference #6):** Cite Storm-0501 Microsoft Threat Intel blog.
- **ATT&CK (Drift D3):** Annotate "BlackCat-on-SaaS variants" as practitioner inference — Storm-0501 is the verified Microsoft cluster.
- **ATT&CK (Stale-reference #1 / X10):** MIDNIGHT BLIZZARD → Midnight Blizzard line 250.
- **ATT&CK (Missing-citation M1, M5):** Cite Microsoft Security Blog for Storm-0501 (Aug 2025), Storm-0558 / Antique Typhoon (Jul 2023), Midnight Blizzard (2024).
- **Verdict:** **P0 — flagged for significant rework by 2 specialists (MCRA-ZT + ATT&CK).**

### `detect/terminated-user-cross-saas.md`

- **CIS (D13):** Add CIS 5.1.6.2 (L1), CIS 5.3.2 (L1), CIS 5.3.3 (L1), CIS 2.4.3 (L2). Secure Score Identity group.
- **MDA-config (Verified-clean #12 / Drift #12):** Built-in policy + Entra-as-IdP dependency verified.
- **Purview-Entra (Drift #8):** Cite `defender-cloud-apps/anomaly-detection-policy` for exact-email-match claim.
- **Purview-Entra (Top remediation #10):** Add Workload Identities Premium citation for SP coverage gap line 263.
- **MDA-config (X3):** Light cross-reference to June 2025 wave.
- **ATT&CK (Drift D9 / X27):** T1136 → T1136.003.
- **Verdict:** **P1 — additions only.**

### `detect/b2b-partner-exfil-alert.md`

- **CIS (D14):** Add CIS 5.1.6.1 (L2), 5.1.6.2 (L1), 5.1.6.3 (L2), 5.3.2 (L1), 7.2.5 (L2). Secure Score Identity + Apps groups.
- **MDA-config (Drift #9 / X6):** Activity-policy auto-disable thresholds citation (line 84).
- **Purview-Entra (Drift #20):** Same.
- **ATT&CK (Missing-citation M4):** Cite Storm-0539 + Storm-2372 reporting.
- **ATT&CK (Practitioner-inference P2):** Flag Storm-0539 / consent-phishing-into-B2B linkage as inference — Microsoft Storm-0539 reporting is gift-card-fraud-centric.
- **Verdict:** **P1.**

### `oauth/post-consent-cleanup-and-ban.md`

- **CIS (D15 — STRONGEST direct map in library):** Add CIS 5.1.5.1 (L2), CIS 5.1.5.2 (L1), CIS 5.1.2.2 (L2), CIS 2.4.3 (L2). Secure Score Identity + Apps groups.
- **MCRA-ZT (Drift #7 / X23):** Add citation to ZT Identity Objective IV + `defender-office-365/detect-and-remediate-illicit-consent-grants`.
- **MDA-config (Verified-clean #13):** OAuth view = delegated permissions only (correctly framed; flag 30% figure as practitioner-inference).
- **Purview-Entra (Drift #17 / X15):** 30% figure is practitioner-inference.
- **Purview-Entra (Missing-citation #8):** Cite verified-publisher Entra control.
- **Purview-Entra (X12):** "eDiscovery (Premium)" → "Purview eDiscovery".
- **ATT&CK (Drift D1 / X10):** TA453 → Mint Sandstorm (lines 17, 31).
- **ATT&CK (Stale-reference #2 / X10):** Storm-0558 → Antique Typhoon for generalised references.
- **ATT&CK (Stale-reference #1 / X10):** MIDNIGHT BLIZZARD → Midnight Blizzard (line 273).
- **ATT&CK (Missing-citation M2, M3):** Cite Storm-2372 (Feb 2025), Storm-1283 (Dec 2023), Mint Sandstorm.
- **Verdict:** **P0 — ATT&CK specialist flagged for significant rework (TA-naming + missing citations).**

### `oauth/high-scope-grant-alert.md`

- **CIS (D16):** Add CIS 5.1.5.1 (L2), 5.1.5.2 (L1), 5.1.2.2 (L2), 9.1.10 (L1) — direct map to Power Platform SP gap, 9.1.11 (L1), 2.4.3 (L2). Secure Score Identity + Apps groups.
- **MCRA-ZT (Drift #5 / X14):** App Governance portal path update for Dec 2025 unified-RBAC; "Predictive Risk shipped 2025 H2" needs citation or rename.
- **MCRA-ZT (Stale-reference #4):** Same.
- **MDA-config (Drift #5, #6 / X14):** Same — "Predictive Risk" branding + "OAuth threshold policy" nomenclature unverified.
- **MDA-config (Practitioner-inference #4 / X14):** "Predictive Risk shipped 2025 H2" cannot be verified.
- **Purview-Entra (Drift #12, #18, X14, X24):** Same Predictive Risk citation; cite Power Platform Admin Center DLP (`power-platform/admin/wp-data-loss-prevention`).
- **Purview-Entra (Top remediation #10):** Workload Identities Premium licensing citation line 260.
- **MDA-config (Practitioner-inference #7):** Power Platform DLP "mandatory companion" is practitioner judgment.
- **ATT&CK (Drift D10 / X26):** Add T1098.003 Additional Cloud Roles to Purpose.
- **ATT&CK (Practitioner-inference P1):** Reframe Midnight Blizzard verified-publisher claim — Microsoft post-mortem frames the path as legacy-tenant residual grant, not verified-publisher abuse.
- **ATT&CK (Stale-reference #1 / X10):** MIDNIGHT BLIZZARD → Midnight Blizzard (lines 17, 151, 245).
- **Verdict:** **P0 — flagged for significant rework by 3 specialists (MDA-config + Purview-Entra + ATT&CK). Highest-impact rewrite in the library.**

### `genai/discovery-and-auto-unsanction.md`

- **CIS (D17):** Add CIS 1.3.4 (L1), 5.1.2.2 (L2), 2.4.3 (L2). Secure Score Apps group.
- **MDA-config (Drift #3 / X4 — P0):** Strip 5-way GenAI sub-category split; rewrite Anti-pattern #4.
- **MDA-config (Drift #4 / X5):** Fix DSPM-for-AI coverage claim (Google Gemini deep-capture per Microsoft Learn).
- **MDA-config (Drift #8):** Risk-score scale 0-10 (not 1-10) per `risk-score` page.
- **MCRA-ZT (Drift #1 / X9 — significant rework):** "Concentration risk" framing on AML.T0010 supply-chain decision.
- **MCRA-ZT (Drift #8):** "MDA Policy 12" Adaptive Protection attribution unsourced.
- **MCRA-ZT (Significant-rework recommendation):** Add Microsoft architectural anchor subsection (Application ZT Objectives II/III/IV/V + Nov 2025 AI Agent Protection release).
- **Purview-Entra (Drift #3 / X5):** Claude Enterprise capability matrix.
- **Purview-Entra (Drift #11 / X4):** Sub-category split citation needed.
- **Purview-Entra (Practitioner-inference #7):** Risk-score "90+ factors" citation — verify against `risk-score` page.
- **MDA-config (Missing-citation #5 / X21):** Edge for Business `gen-ai.app-control` (H2 2026 GA) citation.
- **ATT&CK (Drift D7 / X11):** AML.T0024 "ML Inference API" → "AI Inference API" (lines 11, 162).
- **ATT&CK (Missing-citation M6 / Practitioner-inference P4):** Storm-2372 OAuth-mediated AI access — cite + flag inference.
- **Verdict:** **P0 — flagged for significant rework by 2 specialists (MDA-config + MCRA-ZT). Single highest-priority rewrite alongside `high-scope-grant-alert.md`.**

### `genai/inline-prompt-dlp.md`

- **CIS (D18):** Add CIS 3.2.1 (L1), 3.3.1 (L1), 2.4.3 (L2). Secure Score Data + Apps groups.
- **MDA-config (Drift #4 / X5):** Same DSPM-for-AI coverage (Google Gemini).
- **MDA-config (Missing-citation #5 / X21):** Edge for Business `gen-ai.app-control` citation.
- **Purview-Entra (Drift #1 / X13):** SIT parameter naming.
- **Purview-Entra (Drift #3 / X5):** Claude Enterprise capability matrix line 264.
- **Purview-Entra (Drift #4):** Cross-reference Endpoint DLP / Edge for Business as Microsoft's primary documented control for ChatGPT paste-block.
- **Purview-Entra (Drift #10):** "gen-ai.app-control" feature name not in Microsoft docs.
- **ATT&CK (Drift D7 / X11):** AML.T0024 rename (lines 11, 199).
- **ATT&CK (Drift D8, D11):** T1052 self-correction is good lineage discipline; consider moving to changelog. Optionally tighten T1567 → T1567.002.
- **Verdict:** **P0 — multiple structural items.**

### `genai/sanctioned-tenant-pinning.md`

- **CIS (D19):** Add CIS 5.1.6.1 (L2), 5.2.2.12 (L1), 2.4.3 (L2). Secure Score Identity group.
- **MCRA-ZT (Drift #11):** Mark cross-vendor tenant-pinning architecture as practitioner inference at L18.
- **MDA-config (Missing-citation #5 / X21):** Edge for Business citation.
- **Purview-Entra (Drift #3 / X5):** Claude Enterprise capability matrix line 198.
- **ATT&CK (Notes — verified-clean):** T1078.004 / T1567 / ATLAS AML.T0010 framing is correct.
- **Verdict:** **P1 — additions + practitioner-inference label.**

### `posture/sspm-tenant-misconfig-drift.md`

- **CIS (D20 — MOST under-mapped given gap size):** Make CIS v5.0.0 a PRIMARY citation (not parenthetical caveat); make Microsoft Secure Score a primary citation (currently the only mention in the library is a parenthetical caveat at line 83).
- **MCRA-ZT (Drift #9 / X30):** Cite Application ZT Objective VI; flag specialist-SSPM-tool depth as practitioner inference.
- **Purview-Entra (Drift #21):** Cite `saas-security-posture-management` for app coverage list.
- **MDA-config (Missing-citation #11):** "SecureScore-derived rules silently change between releases" — Microsoft does not document; accept as practitioner hedge or flag.
- **ATT&CK (Stale-reference #1 / X10):** MIDNIGHT BLIZZARD → Midnight Blizzard (line 11).
- **ATT&CK (Notes — model file):** T1098.001 + T1098.003 + T1556 framing is the model for OAuth policies to follow.
- **Verdict:** **P0 — CIS/Secure Score primary citation is auditor-defensibility critical.**

---

## Remediation roadmap

Tiered. Within each tier, ranked by impact on practitioner-grade reliability.

### P0 — Must fix before promotion to v0.1

Audit-defensibility-critical or factually wrong against Microsoft Learn. Auditor-visible.

1. **(X1) Add CIS v5.0.0 + Microsoft Secure Score citations to every policy's `Control mappings` block.** Single highest-impact change — every BNM / SOC 2 / PCI / ISO auditor will probe for them and they are currently absent from all 19 files. Per-file sub-control list per CIS report D2-D20.
2. **(X2) Correct file-policy ceiling from 200 to 50** in `dlp/api-data-at-rest-quarantine.md` line 83 and `dlp/auto-label-pci-data.md` line 226. Cite `data-protection-policies` Limitations section. Recompute "3-5 policies" budget in auto-label against 50-ceiling.
3. **(X4) Strip the 5-way GenAI sub-category split** from `genai/discovery-and-auto-unsanction.md` (lines 76, 99-101, 153, 231 + Anti-pattern #4). Replace with "Generative AI category + custom App tags". Cite `risk-score`.
4. **(X5) Correct Claude Enterprise capability claims** across `discovery-and-auto-unsanction.md` line 154, `sanctioned-tenant-pinning.md` line 198, `inline-prompt-dlp.md` lines 224, 264. Replace "deep vs shallow" with explicit capability matrix per `ai-claude-enterprise` + `ai-chatgpt-enterprise`.
5. **(X3) Add June 2025 anomaly-policy disablement boilerplate** to all detect/* files. Rename Ransomware activity in `mass-delete-anomaly.md`. Add note to `post-consent-cleanup-and-ban.md` for Unusual ISP for OAuth App rename.
6. **(X14) Resolve App Governance "Predictive Risk" branding** in `oauth/high-scope-grant-alert.md`. Either cite Microsoft Learn URL or rename to "App Governance behavioural-anomaly detection (dynamic detection model, June 2025)". Update portal path for Dec 2025 unified-RBAC integration. Single highest-impact rewrite alongside item 3.
7. **(X10) Threat-actor naming hygiene pass:** TA453 → Mint Sandstorm; MIDNIGHT BLIZZARD → Midnight Blizzard; Storm-0558 → Antique Typhoon for generalised references. 7+ file occurrences. Find/replace + add Microsoft threat-actor-naming page citation.
8. **(X11) ATLAS AML.T0024 rename** "ML Inference API" → "AI Inference API". 4 occurrences.
9. **(X16) Correct architectural owner of SUSPEND decision** in `detect/mass-download-alert.md` and `detect/mass-delete-anomaly.md`. Entra owns; MDA is signal source. Cite Identity ZT Objective V.
10. **(X8) Update CAAC `*.mcas.ms` framing** for Edge for Business in-browser protection in `block-download-unmanaged-device.md` and `inline-upload-block-regulated-data.md`.
11. **(X12) Replace "eDiscovery (Premium)" with "Purview eDiscovery"** across 5 files. Cite `ediscovery-overview` (classic retired 2025-08-31).
12. **(X25) Remove T1530 from `inline-upload-block-regulated-data.md` Purpose** — T1530 is Collection, not Exfiltration.
13. **(X23) Add Entra admin-consent workflow citation** to `oauth/post-consent-cleanup-and-ban.md` + `oauth/high-scope-grant-alert.md`. Cite ZT Identity Objective IV + `detect-and-remediate-illicit-consent-grants`.
14. **(X9) Disentangle "concentration risk" (regulator) from "unified XDR" (Microsoft)** in `mass-delete-anomaly.md` (lines 227, 250), `genai/discovery-and-auto-unsanction.md` (line 243), `block-download-unmanaged-device.md` (line 243). Add explicit attribution sentence.
15. **(X17) Reframe `tenant-restriction-corporate-only.md` line 84** as Entra + GSA control; MDA limited to in-session CAAC supplement.
16. **(X32) Resolve `purview_legal_hold: false` pseudo-config** in `external-share-link-quarantine.md` line 122. Replace with M365 retention engine cross-reference.

### P1 — Should fix before v0.2

Citations + precision improvements.

17. **(X13) Standardise SIT parameter naming** across `inline-upload-block-regulated-data.md`, `auto-label-pci-data.md`, `api-data-at-rest-quarantine.md`, `inline-prompt-dlp.md`. `minimum_unique_matches` → `instance count` / `minCount`; `confidenceLevel` for accuracy.
18. **(X13) Replace synthetic API action names** with Microsoft documented names. `ApplyPurviewSensitivityLabel` → "Apply Microsoft Purview sensitivity label" in `auto-label-pci-data.md`; `PutInAdminQuarantine` → "Put file in admin quarantine" in `external-share-link-quarantine.md`.
19. **(X6) Cite or `[VERIFY]` activity-policy auto-disable thresholds** (200k/day, 100k/3h) across `mass-download-alert.md`, `mass-delete-anomaly.md`, `b2b-partner-exfil-alert.md`.
20. **(X22) Add Adaptive Protection citation** to `block-unsanctioned-app-with-coach.md`, `auto-label-pci-data.md`, `mass-delete-anomaly.md`, `discovery-and-auto-unsanction.md`, `inline-prompt-dlp.md`. Cite `purview/insider-risk-management-adaptive-protection`.
21. **(X20) Cite Japan East primary-data region claim** across 6+ files. Cite `microsoft-365/enterprise/o365-data-locations` or mark `[VERIFY]`.
22. **(X24) Add Power Platform SP coverage** to `oauth/high-scope-grant-alert.md`. CIS 9.1.10/9.1.11 + `power-platform/admin/wp-data-loss-prevention`.
23. **(X28) Tighten T1567 → T1567.002** in `inline-upload-block-regulated-data.md`, `inline-prompt-dlp.md`.
24. **(X26) Add T1098.003 Additional Cloud Roles** to `oauth/high-scope-grant-alert.md` Purpose.
25. **(X27) Tighten T1136 → T1136.003** in `detect/terminated-user-cross-saas.md`.
26. **(X29) Tighten T1528 framing** at `impossible-travel-alert.md` line 240.
27. **(X30) Add SSPM Microsoft architectural anchor** to `posture/sspm-tenant-misconfig-drift.md`. Cite Application ZT Objective VI.
28. **(X31) Add Data ZT pillar citation** to `dlp/api-data-at-rest-quarantine.md` + `dlp/external-share-link-quarantine.md`.
29. **Add Microsoft Security Blog citations** to all Storm-cluster name-drops — Storm-0501, Storm-2372, Storm-1283, Storm-0539, Midnight Blizzard, Antique Typhoon.
30. **Cite the impossible-travel "country-level + bordering" claim** at `impossible-travel-alert.md` line 83. Cite `defender-cloud-apps/anomaly-detection-policy`.
31. **(X18) Add FP-rate-trajectory practitioner-observation header** to all 19 files' trajectory tables.
32. **(X19) Annotate "MDA Policy N" internal numbering** across the corpus.
33. **Standardise portal-path nomenclature** to "Microsoft Defender Portal → Cloud Apps → Policies → Policy management".
34. **Cite MDE Network Protection page** at `block-download-unmanaged-device.md` line 82.
35. **Cite verified-publisher Entra control** at `post-consent-cleanup-and-ban.md` line 38.
36. **Cite App Governance + Workload Identities Premium** at `terminated-user-cross-saas.md` line 263 + `high-scope-grant-alert.md` line 260.
37. **Cite Endpoint DLP + Edge for Business browser DLP** at `inline-prompt-dlp.md` + `inline-upload-block-regulated-data.md` as Microsoft's primary documented controls for ChatGPT paste-block.
38. **Mark `geo-residency-block.md` explicitly as practitioner-inference** control class (no direct CIS / Secure Score mapping).
39. **Add (L1/L2) tier annotation** to every policy's `Control mappings` block per CIS report D2-D20.

### P2 — Nice-to-have

Hygiene + lineage.

40. **(X21) Edge for Business `gen-ai.app-control` (H2 2026 GA)** — replace with documented "Browser DLP via Edge for Business" + `[VERIFY GA]`.
41. **Cite "DCS does not OCR at session layer"** across DLP files. Cite `content-inspection` / `dcs-inspection`.
42. **Cite "M365 latency near-real-time"** across detect/* files. Cite `protect-office-365`.
43. **Cite `CloudAppEvents` schema references** across multiple files. Cite Microsoft Defender XDR advanced hunting schema page.
44. **Cite Entra Token Protection GA status** across multiple files.
45. **Annotate Tenant Restrictions v1 deprecation timeline** with citation.
46. **Move T1052 self-correction in `inline-prompt-dlp.md` to changelog** — adds noise in Purpose statement.
47. **Reframe `high-scope-grant-alert.md` Midnight Blizzard verified-publisher claim** — Microsoft post-mortem frames path as legacy-tenant residual grant, not verified-publisher abuse.
48. **Annotate `mass-delete-anomaly.md` BlackCat-on-SaaS** as practitioner inference.
49. **Annotate `b2b-partner-exfil-alert.md` Storm-0539** as practitioner inference (Microsoft Storm-0539 reporting is gift-card-fraud-centric).
50. **Annotate `discovery-and-auto-unsanction.md` Use case 4 Storm-2372 → GenAI OAuth** as practitioner inference.
51. **Cite Microsoft `protect-office-365` and `best-practices`** pages from API-mode policies (per CIS specialist Missing-citation recommendations).
52. **Standardise XTAS / CTAS terminology** — Microsoft Learn uses "cross-tenant access settings" lowercase; pick one abbreviation.

---

## Microsoft-standards alignment summary

Per-reference-family verdict.

| Reference family | Verdict | Critical gaps |
|---|---|---|
| **MDA Microsoft Learn (`defender-cloud-apps/*`)** | **Conditionally aligned** | File-policy ceiling (50 vs 200), GenAI sub-category split, App Governance Predictive Risk branding, June 2025 anomaly-policy wave all contradict current docs. |
| **MCRA + Zero Trust (Identity / Apps / Data pillars) + CAF Secure** | **Architecturally sound but under-cited** | 9 missing citations to specific ZT Objectives. "Concentration risk" framing presented as architecture rather than regulator framing. SUSPEND-decision and TRv2 ownership attribution sloppy. |
| **CIS Microsoft 365 Foundations Benchmark v5.0.0** | **Systematically uncited** | 0 of 19 files cite CIS 2.4.3 (the umbrella control). Strongest direct maps in `external-share-link-quarantine.md` (7 controls), `post-consent-cleanup-and-ban.md` (3 controls), `oauth/high-scope-grant-alert.md` (5 controls including 9.1.10 for Power Platform). |
| **Microsoft Secure Score** | **Systematically uncited** | Only mention across 19 files is a parenthetical caveat at `sspm-tenant-misconfig-drift.md` line 83. Should be a primary citation on the posture policy. |
| **Purview Information Protection / DLP / IRM / DSPM-for-AI** | **Conditionally aligned** | SIT parameter naming, governance-action API names, Claude Enterprise capability matrix, eDiscovery rename all need correction. Adaptive Protection citations missing. |
| **Entra ID Protection + Conditional Access + Workload Identities** | **Architecturally aligned** | Workload Identities Premium licensing-tier citation missing. Token Protection GA verification needed. |
| **MITRE ATT&CK (Enterprise v15-v17)** | **Mostly aligned** | T1530 mis-mapping on upload-block policy is the only structural error. Sub-technique precision (T1567.002, T1136.003, T1098.003) absent in several files. |
| **MITRE ATLAS** | **Mostly aligned** | AML.T0024 rename "ML Inference API" → "AI Inference API" not propagated. |
| **Microsoft Threat Intelligence (threat-actor naming)** | **Drifted** | TA453 (Proofpoint) leaks into 2 files. MIDNIGHT BLIZZARD all-caps style is post-Apr-2023-stale. Storm-0558 promotion to Antique Typhoon not propagated for generalised references. All Storm-cluster name-drops lack Microsoft Security Blog citations. |

---

## Outstanding gaps

Microsoft-recommended controls the policy library does not cover at all.

1. **Microsoft 365 baseline / Secure-Score-target practice as a continuous-control discipline.** The library has `posture/sspm-tenant-misconfig-drift.md` for drift detection but no policy that establishes the **baseline Secure Score target + improvement-action queue management** as an operational discipline. This is what the Microsoft Secure Score docs describe and what auditors expect to see operationalised.

2. **CAF Secure "Prepare for and respond to incidents" + "Sustain security posture" disciplines** as named anchors. Multiple policies touch on incident response and posture sustainment, but none anchors explicitly on the CAF Secure disciplines. The two flagged-for-significant-rework policies (`mass-delete-anomaly.md`, `discovery-and-auto-unsanction.md`) should adopt this anchor.

3. **Browser DLP via Microsoft Edge for Business** as a primary control class. Currently referenced as a future-state Edge feature (`gen-ai.app-control`) but Microsoft already documents Browser DLP via Edge for Business at `purview/dlp-browser-dlp-learn` as the supported control for "block paste into third-party AI sites". The library treats this as adjunct to CAAC; Microsoft documents it as primary.

4. **Endpoint DLP** as a control class. Microsoft documents Endpoint DLP at `purview/endpoint-dlp-learn-about` as the primary control for "block paste of sensitive content into third-party AI sites" on Windows-onboarded devices. The library treats CAAC as primary; Microsoft documents Endpoint DLP as primary.

5. **Microsoft Defender XDR data-retention / advanced-hunting retention policy.** The `impossible-travel-alert.md` references "Default 30-day activity-log retention (standard tier)" without a source. The library has no policy that establishes XDR data-retention as a discipline.

6. **Microsoft Entra Continuous Access Evaluation + Token Protection + Workload Identities Premium** as a named control bundle. Referenced in several policies as compensating control but no dedicated policy. Given the Midnight Blizzard / Storm-0558 / Antique Typhoon class of incidents, this is a gap.

7. **Power Platform Admin Center DLP / data-policies.** Referenced in `oauth/high-scope-grant-alert.md` line 247 as "mandatory companion" but no dedicated policy. Microsoft documents this as a distinct control surface.

8. **Microsoft Purview AI Hub / DSPM-for-AI as a posture surface** (separate from inline prompt DLP). DSPM-for-AI is the Microsoft-documented surface for AI-usage posture; the library covers inline blocking + discovery but not the posture discipline.

9. **MDA App Governance unused-app insights / behavioural-detection layer** as a named control. Dec 2025 GA per Microsoft Learn release notes; not covered as a distinct policy.

10. **Microsoft Defender unified RBAC** as a posture / change-management discipline. The Dec 2025 release notes describe the unified-RBAC migration; no policy addresses this transition.

---

## Disputes between specialists

Where 2 specialists made conflicting claims.

### Dispute 1. GenAI sub-category split (LLM / AI Coding Assistant / AI Image / AI Video / AI Voice-Avatar)

- **CIS specialist (Verified-clean #14):** "Sub-category split (LLM / AI Coding Assistant / Image / Video / Voice-Avatar) — verified against current MDA Cloud Discovery category taxonomy. The policy correctly warns against using the monolithic 'Generative AI' category."
- **MDA-config specialist (Drift #3):** "The current `risk-score` page lists only **'Generative AI'** as a Cloud App Catalog category. There are two adjacent categories ('AI – MCP Server', 'AI – Model Provider') but not the practitioner-claimed five-way split. The sub-category list ... appears to be practitioner-inferred branding, not a Microsoft taxonomy."
- **Purview-Entra specialist (Drift #11):** "The specific 5-way sub-category breakdown should be cited or marked `[VERIFY]`. The MS-documented split at the date of writing is more often described as 'AI sites' rather than per-modality categories."
- **Resolution:** **MDA-config + Purview-Entra are correct.** CIS specialist's "verified-clean" is the stale finding. The 5-way split is not in published Microsoft taxonomy. Treat as **drift to correct** per X4 above.

### Dispute 2. File-policy ceiling (50 vs 200)

- **CIS specialist (Verified-clean #5-6):** "File-policy ceiling = 200 per tenant. Correct per current MDA docs (verify date of attestation)."
- **MDA-config specialist (Drift #1):** "Microsoft source: `data-protection-policies` → 'Limitations: You're limited to 50 file policies in Defender for Cloud Apps.'"
- **Purview-Entra specialist (Missing-citation #6):** Citation needed; no specific number.
- **Resolution:** **MDA-config is correct.** The CIS specialist's clearance is stale — the current Microsoft Learn page states 50. Auditor-visible. Treat as **P0 drift** per X2 above.

### Dispute 3. Activity-policy auto-disable thresholds (200k/day, 100k/3h)

- **CIS specialist (Verified-clean #10):** "200k/day or 100k/3h activity-policy auto-disable thresholds — verified against MDA activity-policy documentation."
- **MDA-config specialist (Drift #9):** "Could not verify these specific numeric thresholds in any of the Microsoft Learn pages fetched. ... Flag as practitioner-inference until verified."
- **Purview-Entra specialist (Missing-citation #2):** "Could not locate these specific thresholds in Microsoft Learn (searched)."
- **Resolution:** **Defer to a search for an authoritative URL.** CIS specialist's "verified against MDA activity-policy documentation" did not name the URL. If a Microsoft Learn URL can be found, cite it; otherwise mark `[VERIFY]`. Treat as **P1** per X6 above.

### Dispute 4. App Governance "Predictive Risk shipped 2025 H2"

- **MCRA-ZT specialist (Verified-clean #13):** "App Governance as a paid add-on with Predictive Risk — confirmed per release-notes (Dec 2025 unused-app-insights GA noted)."
- **MDA-config specialist (Practitioner-inference #4):** "Could not be verified against Microsoft Learn. The June 2025 Microsoft change was to the dynamic-threat-detection model for anomaly detection, not a 'Predictive Risk' branded feature in App Governance."
- **Purview-Entra specialist (Drift #12):** "GA timing should be cited; the precise '2025 H2' is practitioner recall. Mark `[VERIFY]`."
- **Resolution:** **MDA-config + Purview-Entra are correct.** MCRA-ZT specialist conflated the Dec 2025 unused-app-insights GA (real per release notes) with the "Predictive Risk" branding (not in Microsoft Learn). Treat as **P0** per X14 above — either cite a Microsoft Learn URL or rename to "App Governance behavioural-anomaly detection".

### Dispute 5. "Bordering country" exclusion in impossible-travel detection

- **MDA-config specialist (Verified-clean #9):** "Impossible-travel country-level granularity, no alerts within or between bordering countries — verbatim match. Source: `anomaly-detection-policy`."
- **Purview-Entra specialist (Drift #9):** "Country-level granularity per the same anomaly-detection-policy page; 'bordering countries' exclusion is observation rather than documented rule."
- **Resolution:** **MDA-config is correct.** The `anomaly-detection-policy` page does state "there will be no alerts for two actions originating in the same country/region or in bordering countries/regions" — this is verbatim Microsoft documentation. No correction needed beyond ensuring the citation is in place. Treat as **verified-clean**.

### Dispute 6. SSPM specialist-tool depth as practitioner judgment

- **MCRA-ZT specialist (Drift #9):** "Specialist-SSPM-tool depth (Adaptive Shield, AppOmni, Obsidian, Wing) is practitioner judgment that exceeds the Microsoft-documented scope."
- **Other specialists:** Did not address the specialist-SSPM-tool framing.
- **Resolution:** **Accept MCRA-ZT specialist's framing.** No dispute; just flag specialist-SSPM-tool depth as practitioner inference. Treat as **P1** per X30 above.

### Dispute 7. Microsoft "risk score 90+ factors" attribution

- **MDA-config specialist (Verified-clean #17):** "Cloud App Catalog 90+ risk factors — verified. Source: `risk-score` ('Apps in the cloud app catalog are scored based on more than 90 risk factors')."
- **Purview-Entra specialist (Practitioner-inference #7):** "Microsoft's risk score is opaque (uncited '90+ factors') — verify and cite per `risk-score`."
- **Resolution:** **MDA-config is correct (the factor count IS published as '90+').** Purview-Entra's flag conflates "factor count is published" with "factor weights are opaque". The right framing in the policy: "Microsoft documents 90+ risk factors but does not publish the weights — the score is opaque on the weighting side, not the factor-count side." Treat as **P2** hygiene per item 51.

---

End of consolidated report.
