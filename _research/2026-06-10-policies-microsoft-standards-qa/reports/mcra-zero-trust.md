# MCRA + Zero Trust architecture QA expert - QA report

> Scope: 19 Tier-2 policy files reviewed against Microsoft Cybersecurity Reference Architecture (MCRA April 2025 release), Zero Trust deployment guidance (Identity / Applications / Data pillars), and CAF Secure methodology (Sept 2025). Cited URLs reflect what was retrieved on 2026-06-10.

## Summary

- **Files reviewed:** 19
- **Verified-clean architecture claims:** 14 high-confidence matches
- **Drift to correct:** 11 specific entries
- **Stale references:** 4 (CAAC product framing, App Governance left-nav, OWA SharePoint "limited access" link, Exchange `limited access` legacy)
- **Missing citations:** 9 (Microsoft Learn URLs absent on architectural claims that Microsoft documents explicitly)
- **Practitioner-inference flags:** 8 (claims defensible but not directly attributable to MCRA / ZT / CAF — must be labelled as practitioner inference)
- **Policies needing significant rework:** 2 (`mass-delete-anomaly.md`, `genai/discovery-and-auto-unsanction.md` — concentration-risk framing drift + missing ZT pillar anchoring)

The practitioner framing across the corpus is **architecturally sound on enforcement-layer attribution** (CAAC = enforcement layer; Entra CA = routing layer; Purview = labelling layer; Defender XDR = correlation layer) but **systematically under-cites Microsoft authoritative sources** for those attributions. The "concentration risk" framing is practitioner-inference language with no MCRA / CAF anchor — Microsoft frames the same integration as "unified XDR" or "end-to-end security" per MCRA April 2025; the practitioner framing remains defensible but must be marked as practitioner judgment, not Microsoft positioning.

## Verified-clean claims

These claims match Microsoft authoritative source materials with high confidence:

1. **CAAC bilateral configuration requirement** (Entra CA policy + MDA session policy) — confirmed in https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad — Microsoft documents that CA app control "delivers deep visibility and control over browser-based sessions" via the integration. Practitioner files `block-download-unmanaged-device.md` (L82), `inline-upload-block-regulated-data.md` (L83), `geo-residency-block.md` (L84), `inline-prompt-dlp.md` (L83) all correctly describe the bilateral requirement.
2. **CAAC is browser-session only; native/desktop bypass** — confirmed at https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad ("Session controls apply only to web browser–based access"; "Microsoft Teams desktop application isn't supported"). Practitioner files all correctly state this.
3. **MDA in Zero Trust Applications pillar (Initial Deployment Objective II — Discover and control Shadow IT)** — confirmed at https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications. `block-unsanctioned-app-with-coach.md` and `genai/discovery-and-auto-unsanction.md` correctly map to this objective.
4. **Conditional Access App Control is the "Initial Deployment Objective III — Protect sensitive information automatically by implementing policies"** — confirmed at the same URL (step 5 of Objective III). Policies citing CAAC for download/upload control are correctly mapped.
5. **Adaptive access + session controls = "Additional Deployment Objective IV"** — confirmed at the same URL. `block-download-unmanaged-device.md` correctly anchors to this objective.
6. **Defender for Cloud Apps + Entra ID Protection integration** is the Microsoft-documented Identity Zero Trust Objective V pattern — confirmed at https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity ("Enable Microsoft Defender for Cloud Apps integration with Microsoft Entra ID Protection"). `impossible-travel-alert.md` correctly identifies Entra ID Protection as the primary control with MDA as corroborating signal.
7. **Restrict user consent to applications** is documented Identity Objective IV — confirmed at https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity (the "Restrict user consent to applications" subsection). `oauth/post-consent-cleanup-and-ban.md` and `oauth/high-scope-grant-alert.md` correctly position Entra admin-consent workflow as the primary preventive control.
8. **Purview Information Protection as the labelling layer (Data Zero Trust Objective I-II)** — confirmed at https://learn.microsoft.com/en-us/security/zero-trust/deploy/data. `auto-label-pci-data.md`, `api-data-at-rest-quarantine.md`, `inline-upload-block-regulated-data.md` all correctly attribute labelling to Purview, not MDA.
9. **DLP for SaaS = Data Zero Trust Objective IV** ("Prevent data leakage") — confirmed at the same URL. DLP policies (`inline-upload-block-regulated-data.md`, `api-data-at-rest-quarantine.md`, `external-share-link-quarantine.md`, `auto-label-pci-data.md`) all map correctly.
10. **Insider Risk Management = Data Zero Trust Objective V** ("Manage risks") — confirmed at the same URL. References to IRM signal-boost in detect/* and dlp/* policies are correctly anchored.
11. **Defender for Cloud Apps as DLP control surface for SaaS (Box / Google Workspace integration)** — confirmed at https://learn.microsoft.com/en-us/security/zero-trust/deploy/data ("Control access to data in SaaS applications" subsection). `api-data-at-rest-quarantine.md` and `external-share-link-quarantine.md` correctly position MDA here.
12. **Continuous Access Evaluation + Token Protection as compensating control for token replay** — verified per general Microsoft Entra documentation; mention in `impossible-travel-alert.md` (L238) and `inline-upload-block-regulated-data.md` (L278) is correctly framed.
13. **App Governance as a paid add-on with Predictive Risk** — confirmed per https://learn.microsoft.com/en-us/defender-cloud-apps/release-notes (Dec 2025 unused-app-insights GA noted). `oauth/high-scope-grant-alert.md` correctly notes paid-add-on status.
14. **In-browser protection via Microsoft Edge for Business as alternative to reverse-proxy redirect** — confirmed at https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad ("Microsoft Edge users benefit from direct, in-browser protection"). References in `genai/discovery-and-auto-unsanction.md` (L249) and `genai/inline-prompt-dlp.md` (L291) are correctly framed but require updated language (see Drift §3 below).

## Drift to correct

### 1. `genai/discovery-and-auto-unsanction.md` L243 — "concentration risk" framing

**Practitioner claim:** "Approved-AI allowlist is a third-party concentration-risk decision — quarterly review + supply-chain-compromise contingency (ATLAS AML.T0010) required."

**What Microsoft says:** MCRA (https://learn.microsoft.com/en-us/security/adoption/mcra) and the Zero Trust deployment guides do not frame the Microsoft security stack integration as "concentration risk." Microsoft frames the integration as "end-to-end security" with "Microsoft Security Copilot as a broad capability." The April 2025 MCRA explicitly replaced "Secure Score" framing with "Microsoft Security Exposure Management" for the unified posture story.

**Corrected text:** "Approved-AI allowlist is a supply-chain risk surface (ATLAS AML.T0010); quarterly review + documented incident playbook required. *Note: 'concentration risk' is practitioner / regulator framing (BNM RMiT third-party concentration, DORA Art. 28-30) — Microsoft documentation positions the integration as 'unified XDR' / end-to-end security per MCRA April 2025.*"

### 2. `detect/mass-delete-anomaly.md` L227, L250 — "concentration risk" attribution

**Practitioner claim:** "the SaaS-ransomware risk is itself a concentration-risk artefact — the destruction path and the recovery path both live in the same Microsoft stack" (L227); "Concentration-risk: the detection control and the destruction surface both live in Microsoft" (L250).

**What Microsoft says:** Same as §1 — Microsoft does not use this framing. The defensible Microsoft-anchor for "independent SIEM with raw log forwarding" is CAF Secure (https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/secure/overview): "Prepare for and respond to incidents" + "Sustain security posture" sections name "telemetry ingestion and alert fidelity" as the discipline — but the term "concentration risk" itself is regulator-derived (BNM RMiT, DORA, MAS TRM), not MCRA-derived.

**Corrected text:** Add explicit attribution: "Concentration-risk framing follows BNM RMiT third-party-concentration / DORA Art. 28-30 supervisory framing — not Microsoft's own positioning, which is 'end-to-end security' / 'unified XDR' per MCRA April 2025. Compensating control of independent SIEM with raw log forwarding aligns with CAF Secure 'Sustain security posture' discipline."

### 3. `block-download-unmanaged-device.md` L243 — "concentration risk register" framing

**Practitioner claim:** "The CAAC proxy is an internet-scale T1557 target; promote as a concentration-risk item on the operational-risk register with compensating controls outside the Microsoft stack."

**What Microsoft says:** Same drift class. Additionally, Microsoft Edge for Business **in-browser protection** is the documented direction-of-travel that obviates the reverse-proxy attack surface for Edge-on-Windows estates — https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad explicitly says "Microsoft Edge users benefit from direct, in-browser protection, without redirecting to a reverse proxy."

**Corrected text:** "The CAAC reverse-proxy path is a supervisory concentration-risk consideration (BNM RMiT / DORA framing). Microsoft's roadmap response is in-browser protection via Microsoft Edge for Business (https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad), which removes the reverse-proxy chain for Edge sessions. Document the residual reverse-proxy dependency for non-Edge browsers; track Edge for Business adoption."

### 4. `access-control/tenant-restriction-corporate-only.md` L84 — header injection attribution

**Practitioner claim:** "MDA itself does NOT inject these headers — the SWG / proxy layer does."

**What Microsoft says:** Verified correct, but the architectural framing of CTAS, Tenant Restrictions v2, and the SWG injection point sits in the **Entra (Identity pillar)** documentation — not MCRA / MDA documentation. The current text positions this as an MDA-adjacent gap. Per Identity Zero Trust Objective IV (https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity), this control belongs entirely in Entra + GSA; MDA's role is only the CAAC supplement.

**Corrected text:** Reframe the header attribution paragraph to: "Per the Zero Trust Identity deployment guide, Tenant Restrictions v2 is an Entra + network-layer control. The policy decision sits in Entra Cross-Tenant Access Settings (CTAS); the routing/enforcement sits in the network proxy (GSA-native or third-party SWG). MDA provides the in-session supplementary control via CAAC for apps where post-authentication restriction adds value — but MDA is **not** the owner of tenant-restriction-v2." Add citation: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity.

### 5. `oauth/high-scope-grant-alert.md` L83 — App Governance portal path

**Practitioner claim:** "Defender portal → Cloud apps → App governance → Predictive risk policies + OAuth threshold policies. (Re-verify 2026 portal path; App Governance may be a separate left-nav.)"

**What Microsoft says:** Per Dec 2025 release note (https://learn.microsoft.com/en-us/defender-cloud-apps/release-notes), "Microsoft Defender for Cloud Apps permissions are now integrated with Microsoft Defender unified RBAC" — the portal path is being unified into the Microsoft Defender portal under the unified-RBAC model. The unused-app-insights feature has GA expansion in Dec 2025.

**Corrected text:** "Microsoft Defender portal → Cloud apps → App governance. As of Dec 2025, App Governance permissions integrate with Microsoft Defender unified RBAC (https://learn.microsoft.com/en-us/defender-cloud-apps/release-notes). Verify the workload activation status — see [Activate Microsoft Defender unified RBAC](https://learn.microsoft.com/en-us/defender-xdr/activate-defender-rbac)."

### 6. `detect/mass-download-alert.md` L66, `detect/mass-delete-anomaly.md` L67-68 — "auto-suspend Entra user" governance action description

**Practitioner claim:** "Auto-suspend Entra user only after corroborated signals" and the hybrid-AD `onPremisesSyncEnabled=true` silent-fail mode (mass-download L80, mass-delete L83).

**What Microsoft says:** The Microsoft-documented framing is that Entra's identity governance owns the suspension/disable decision — this is per Identity Zero Trust Objective II (https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity) "Conditional Access policies gate access" and Objective V "Enable Microsoft Defender for Cloud Apps integration with Microsoft Entra ID Protection." MDA is documented as the **signal source** that boosts Entra's risk score, not the enforcement layer. Practitioner files conflate the two — the SUSPEND decision is an Entra ID Protection / Conditional Access decision, with MDA providing the input signal.

**Corrected text:** Add clarifying sentence: "The suspend decision is owned by Entra (Identity ZT pillar Objective V); MDA provides the activity signal. Per https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity, the architecturally correct path is: MDA alert → Entra ID Protection user-risk boost → CA policy enforces (block / step-up / disable). Avoid framing 'MDA auto-suspend' as the architectural primitive — it is."

### 7. `oauth/post-consent-cleanup-and-ban.md` L13-14 — "primary control" attribution

**Practitioner claim:** "Primary control for T1528 is the Entra admin-consent workflow + verified-publisher restriction, configured upstream in Entra (not MDA). MDA Policy 2a inspects what already happened."

**What Microsoft says:** Verified correct in framing (Entra is primary, per https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity Objective IV "Restrict user consent to applications"). However the file is **missing the Microsoft Learn citation** for the primary-control claim and the "review prior/existing consent" guidance. The ZT identity guide explicitly links to https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants — which is the Microsoft-authoritative reference for the post-consent cleanup workflow.

**Corrected text:** Add citations: "Per Identity Zero Trust Deployment Objective IV: 'Restrict user consent and manage consent requests' is the primary preventive control (https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity); MDA Policy 2a corresponds to Microsoft's documented 'Review prior/existing consent in your organization' guidance (https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants)."

### 8. `genai/discovery-and-auto-unsanction.md` L154 — DSPM-for-AI integration with IRM Adaptive Protection

**Practitioner claim:** "DSPM-for-AI prompt-body capture on allowlisted enterprise-AI vendors feeds Purview Insider Risk Management adaptive scoping (per MDA Policy 12)."

**What Microsoft says:** Could not directly verify "Adaptive Protection" as the documented integration term in MDA documentation. Adaptive Protection is a Purview Insider Risk Management feature; the Microsoft-documented integration story for AI-usage signals + IRM is positioned under the Data ZT pillar Objective V (https://learn.microsoft.com/en-us/security/zero-trust/deploy/data). The "MDA Policy 12" framing is internal practitioner numbering, not Microsoft terminology.

**Corrected text:** "Sanctioned-AI usage telemetry can feed Purview Insider Risk Management as a signal contributing to Adaptive Protection scoping (Data ZT pillar Objective V — https://learn.microsoft.com/en-us/security/zero-trust/deploy/data). 'Policy 12' is internal playbook numbering; the architectural pattern is the MDA → IRM → Adaptive Protection signal chain documented in the Purview IRM solution overview."

### 9. `posture/sspm-tenant-misconfig-drift.md` L11 — SSPM positioning

**Practitioner claim:** "CASB does this lightly — for deep coverage, dedicated SSPM tools (Adaptive Shield, AppOmni, Obsidian, Wing) are the right tier."

**What Microsoft says:** SSPM is Application Zero Trust Deployment Objective VI ("Assess the security posture of your cloud environments" — https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications). Microsoft positions the integration as MDA-native for the SaaS estate. The practitioner framing is operationally correct (CASB SSPM is shallow vs specialist SSPM) but the **Microsoft architectural anchor is App ZT Objective VI** and that citation is missing.

**Corrected text:** Add: "Per Application Zero Trust Deployment Objective VI (https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications), MDA is the documented Microsoft tool for SaaS/PaaS posture assessment. Specialist-SSPM-tool depth (Adaptive Shield, AppOmni, Obsidian, Wing) is practitioner judgment that exceeds the Microsoft-documented scope — flag as practitioner inference."

### 10. `dlp/api-data-at-rest-quarantine.md` L11, `dlp/external-share-link-quarantine.md` L11 — Defender for Cloud Apps positioning

**Practitioner claim:** Both files describe API-mode quarantine as MDA-owned. The text does not attribute the broader architectural framing to the Microsoft Zero Trust Data pillar.

**What Microsoft says:** Per https://learn.microsoft.com/en-us/security/zero-trust/deploy/data ("Control access to data in SaaS applications"), Microsoft explicitly documents Defender for Cloud Apps as providing "Removing permissions... Quarantining files for review... Applying labels to sensitive files." These three governance actions are the exact API-mode policies in MDA. The practitioner files describe the capability but omit the Microsoft-documented architectural slot.

**Corrected text:** Add to both files: "Per Data Zero Trust Deployment Objective III (https://learn.microsoft.com/en-us/security/zero-trust/deploy/data), Microsoft documents Defender for Cloud Apps as the SaaS-DLP enforcement layer with three documented governance actions: remove permissions, quarantine files, apply labels. This policy implements the second of these."

### 11. `genai/sanctioned-tenant-pinning.md` L84, L18 — header convention attribution

**Practitioner claim:** "Five vendors, five header conventions, three sign-in surfaces" + extensive description of `Restrict-Access-To-Tenants` as Microsoft-specific.

**What Microsoft says:** Verified correct — `Restrict-Access-To-Tenants` is documented as a Microsoft auth-endpoint-only header in the Entra documentation. The practitioner framing of "tenant pinning across non-Microsoft SaaS" is **practitioner inference** — Microsoft documentation does not provide a unified architectural story for tenant pinning across OpenAI / Anthropic / Google AI vendors. This is legitimate practitioner work but should be labelled as cross-vendor inference, not Microsoft positioning.

**Corrected text:** Add at L18: "*Note: The cross-vendor tenant-pinning architecture (ChatGPT + Claude + Gemini + Copilot) is practitioner inference. Microsoft documents only `Restrict-Access-To-Tenants` / Tenant Restrictions v2 for Microsoft auth endpoints (Identity ZT pillar Objective IV — https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity). The non-Microsoft vendors' SSO / SAML conventions are per-vendor; no Microsoft-authoritative architecture covers the integration.*"

## Stale references

1. **`*.mcas.ms` URL rewrite framing** in `block-download-unmanaged-device.md` L82, `inline-upload-block-regulated-data.md` L83 — Microsoft documents this as the reverse-proxy mechanism for non-Edge browsers but explicitly notes Edge users get in-browser protection without redirect (https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad). The practitioner framing of `*.mcas.ms` as a universal failure mode is **stale for Edge-on-Windows estates** post-2025 in-browser-protection GA. Update to: "`*.mcas.ms` URL rewrite is the reverse-proxy path for non-Edge browsers; Edge users get direct in-browser protection per [Microsoft Edge for Business in-browser protection](https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad)."

2. **App Governance left-nav** in `oauth/high-scope-grant-alert.md` L83 — already flagged for re-verification by the author; per the Dec 2025 release note unified-RBAC integration, the portal path is consolidating. Update text to current Defender portal path.

3. **"Limit access to SharePoint Online / Exchange Online" link in `block-download-unmanaged-device.md`** — the file does not directly cite this but the Microsoft ZT Identity guide (https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity) explicitly documents "limited access" as the Microsoft pattern (links to /en-us/sharepoint/control-access-from-unmanaged-devices). The practitioner "read-only fallback" framing is the same control but should cite the Microsoft pattern by name.

4. **"App Governance shipped 2025 H2" in `oauth/high-scope-grant-alert.md` L83** — App Governance has been GA since 2022; "Predictive Risk shipped 2025 H2" needs a citation. The Dec 2025 release note refers to "App governance unused app insights" Preview→GA expansion but does not corroborate the 2025 H2 Predictive Risk GA claim. Mark as **practitioner-inference / needs verification**.

## Missing citations

These architectural claims need Microsoft Learn URLs:

1. **`block-unsanctioned-app-with-coach.md`** — Shadow IT discovery as Day-1 control. Citation needed: https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications (Initial Deployment Objective II).

2. **`access-control/tenant-restriction-corporate-only.md`** — CTAS hardening and partner-allowlist as Identity ZT pattern. Citation needed: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity.

3. **`access-control/block-download-unmanaged-device.md`** — "block download to unmanaged" maps to Application ZT Objective IV ("Deploy adaptive access and session controls"). Citation needed: https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications.

4. **`access-control/geo-residency-block.md`** — claim that "MDA enforces user-action-by-location, not data-by-region" is architecturally correct but missing citation to Data ZT pillar (https://learn.microsoft.com/en-us/security/zero-trust/deploy/data) which positions Purview as the data-residency control layer.

5. **`dlp/auto-label-pci-data.md`** — auto-labelling via Purview maps to Data ZT Objective I ("Classify, label and discover sensitive data") and II ("Apply encryption, access control and content markings"). Citation needed: https://learn.microsoft.com/en-us/security/zero-trust/deploy/data.

6. **`dlp/inline-upload-block-regulated-data.md`** — inline DLP maps to Data ZT Objective IV ("Prevent data leakage") + Application ZT Objective III ("Protect sensitive information automatically"). Citations needed for both.

7. **`detect/mass-download-alert.md`** — UEBA / anomaly detection maps to Application ZT Objective V ("Strengthen protection against cyber threats and rogue apps"). Citation needed: https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications (Objective V explicitly references Defender for Cloud Apps UEBA).

8. **`detect/impossible-travel-alert.md`** — already correctly identifies Entra ID Protection as primary; needs explicit citation to https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity Objective V ("User, device, location, and behavior are analyzed in real time").

9. **`oauth/post-consent-cleanup-and-ban.md`** — primary control attribution to Entra admin-consent workflow needs the citation called out in Drift §7 above.

## Practitioner-inference flags

These claims are defensible practitioner judgment but should be labelled as such:

1. **"MDA Policy 1 / 2 / 3 / ... 12" numbering** across multiple files — this is internal playbook numbering, not Microsoft documentation. Microsoft does not number policies this way. Either internalise these references (link to the playbook) or add a footnote explaining the numbering convention.

2. **"Concentration risk" framing across `mass-delete-anomaly.md`, `genai/discovery-and-auto-unsanction.md`, `block-download-unmanaged-device.md`** — practitioner / regulator framing, not Microsoft.

3. **"DSPM-for-AI deep capture only on ChatGPT Enterprise + Copilot, shallow on Claude / Gemini"** in `genai/discovery-and-auto-unsanction.md` L154 and `genai/inline-prompt-dlp.md` L264 — could not verify per Microsoft documentation in this review pass; flag as practitioner-observed coverage matrix.

4. **"Predictive Risk shipped 2025 H2"** in `oauth/high-scope-grant-alert.md` L83 — could not verify the specific GA timing; mark as practitioner inference.

5. **"100-character matching-content preview snippet"** across multiple DLP policies — could not verify the exact 100-character figure in Microsoft documentation in this pass; mark as practitioner observation.

6. **"MDE NP propagation window ~60 minutes" in `genai/discovery-and-auto-unsanction.md` L123** — explicitly flagged in the file already as "observed; not Microsoft-published SLA." Good — keep that flag.

7. **"`Restrict-Access-To-Tenants` v1 deprecation timeline" in `tenant-restriction-corporate-only.md` L84** — flagged as `[VERIFY]` in the file. Honest.

8. **"Tenant-pinning architecture across non-Microsoft AI vendors"** in `genai/sanctioned-tenant-pinning.md` — cross-vendor pattern is practitioner work; no Microsoft architecture covers this.

## Policies needing significant rework

Two files have multiple high-impact architecture-framing issues:

### 1. `detect/mass-delete-anomaly.md`

- Drift §2: "concentration risk" framing in two places (L227, L250)
- Practitioner numbering "Policy 11" / "Policy 12" without source linking
- "Microsoft service-side incident (Storm-0558 / Midnight Blizzard)" framing — correct but missing CAF Secure citation (CAF Secure "Prepare for and respond to incidents" discipline)
- "Concentration risk register" framing on the vendor side conflates regulator framing with architectural framing — needs disentangling

**Recommended rework:** Reframe the BCM / recovery / concentration-risk section to anchor on CAF Secure "Prepare for and respond to incidents" and "Sustain security posture" (https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/secure/overview), with explicit attribution that supervisory concentration-risk framing (BNM RMiT / DORA) is regulator-derived, not Microsoft-derived.

### 2. `genai/discovery-and-auto-unsanction.md`

- Drift §1: "concentration risk" framing on the AML.T0010 supply-chain decision
- Drift §8: "MDA Policy 12" Adaptive Protection attribution unsourced
- "AI governance forum" + "ISO 42001" + "NIST AI RMF" framing without anchor to Microsoft's own AI governance positioning (which lives in MCRA April 2025 AI section + the Copilot Studio agent-protection release notes Nov 2025 — https://learn.microsoft.com/en-us/defender-cloud-apps/release-notes)
- Architectural framing of "MDA Shadow IT discovery → MDE NP block → endpoint enforcement" is correct (matches Application ZT pillar Objectives II + IV) but missing explicit citations

**Recommended rework:** Add a "Microsoft architectural anchor" subsection that maps the policy to Application ZT Objectives II / III / IV / V + the Nov 2025 AI Agent Protection release (https://learn.microsoft.com/en-us/defender-cloud-apps/release-notes); separate the regulator/supervisory framing (concentration risk) from the Microsoft positioning (end-to-end security / unified XDR).

## Top remediation priorities

Ranked by impact on audit-defensibility and architectural correctness:

1. **Disentangle "concentration risk" (regulator framing) from "unified XDR / end-to-end security" (Microsoft framing)** across `mass-delete-anomaly.md`, `genai/discovery-and-auto-unsanction.md`, `block-download-unmanaged-device.md`. Add explicit attribution sentence in each file.

2. **Correct the architectural owner of the SUSPEND decision** in `detect/mass-download-alert.md` and `detect/mass-delete-anomaly.md` — clarify that Entra (via ID Protection + CA) owns the suspend decision; MDA is the signal source. Cite https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity Objective V.

3. **Reframe `access-control/tenant-restriction-corporate-only.md` L84** to clearly anchor Tenant Restrictions v2 as an Entra + GSA control (Identity ZT pillar), with MDA's role limited to the in-session CAAC supplement. Add citation.

4. **Update CAAC `*.mcas.ms` framing** in `block-download-unmanaged-device.md` L82 and `inline-upload-block-regulated-data.md` L83 to acknowledge Edge for Business in-browser protection as the Microsoft direction-of-travel (https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad). Stale framing claims `*.mcas.ms` is universal.

5. **Add Microsoft Learn citations to OAuth policies** — `oauth/post-consent-cleanup-and-ban.md` and `oauth/high-scope-grant-alert.md` correctly position Entra admin-consent workflow as primary but lack the https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity citation (Objective IV) and the https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants reference.

6. **Disambiguate MDA Policy numbering** — across all 19 files, "MDA Policy 1 / 2 / 3 / ... 12" appears as if it were Microsoft documentation. It is not. Either explicitly link the playbook reference or annotate the numbering as internal.

7. **Update `oauth/high-scope-grant-alert.md` portal path** to reflect Dec 2025 Microsoft Defender unified RBAC integration (https://learn.microsoft.com/en-us/defender-cloud-apps/release-notes).

8. **Add Data ZT pillar citation to `dlp/api-data-at-rest-quarantine.md` and `dlp/external-share-link-quarantine.md`** — Microsoft documents the three governance actions (remove permissions, quarantine, apply labels) at https://learn.microsoft.com/en-us/security/zero-trust/deploy/data.

9. **Mark cross-vendor GenAI tenant-pinning architecture as practitioner inference** in `genai/sanctioned-tenant-pinning.md` L18 — Microsoft documentation covers only the Microsoft auth-endpoint side.

10. **Add SSPM Microsoft anchor** in `posture/sspm-tenant-misconfig-drift.md` to Application ZT Objective VI (https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications) — currently the file frames SSPM entirely as practitioner judgment without crediting Microsoft's documented role for MDA in this objective.

---

> **Audit-defensibility note:** The corpus is broadly architecturally sound. The pattern of drift is **under-citation** rather than **mis-attribution** — practitioners have the right model but skip the Microsoft Learn link. For a regulated-FS audit cycle, this is fixable with citations rather than rework. The two policies flagged for significant rework (`mass-delete-anomaly.md`, `genai/discovery-and-auto-unsanction.md`) are the exception — the "concentration risk" framing in those files is regulator framing presented as if architectural, and that confusion must be untangled before a Microsoft architect (Three-lens sign-off) would clear them.

> **MCRA + ZT + CAF source coverage at 2026-06-10:** MCRA April 2025 (latest) per https://learn.microsoft.com/en-us/security/adoption/mcra; Zero Trust deployment guides (Identity 2024-08, Applications 2025-03, Data 2024-04) per the respective URLs; CAF Secure overview (Sept 2025) per https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/secure/overview; MDA release notes through May 2026 per https://learn.microsoft.com/en-us/defender-cloud-apps/release-notes. The CAF Secure-disciplines page (legacy URL in task prompt) returned 404; current canonical location is /azure/cloud-adoption-framework/secure/overview.
