# Compliance lens review — Netskope SSE (with CASB Inline + CASB API)

Reviewer posture: GRC lead / IS auditor. Frameworks in scope: BNM RMiT (current edition), MAS TRM, HKMA SA-2, ISO/IEC 27017:2015, ISO/IEC 27018:2019, PDPA (Malaysia 2010 + 2024 amendments), GDPR, NIST CSF 2.0. "Vendor-controlled" sources do not constitute audit evidence on their own.

Caveat — Material on regulatory clocks, supervisory expectations, and legal privilege below is illustrative only; not legal/regulatory advice. Confirm against the gazetted text of each instrument and your assurance lead before reliance.

## Findings to act on (highest impact first)

1. **No control-to-policy mapping anywhere in the draft.** The 12 configurable policies are presented as product features without a single mapping to BNM RMiT (e.g. 10.49–10.55 Data Loss Prevention; 11.x Cloud Services; 11.13/11.14 third-party risk; Appendix 5 reporting), MAS TRM (Sections 11/12/14 — third-party / cloud / cyber), ISO 27017 controls (CLD.6.3.1, CLD.8.1.5, CLD.9.5.1, CLD.9.5.2, CLD.12.1.5, CLD.12.4.5, CLD.13.1.4), ISO 27018 (A.1 consent; A.5 disclosure; A.9 transparency; A.10 deletion), or NIST CSF 2.0 (GV.SC, ID.AM-04 (SaaS inventory), PR.DS-01/02, PR.AA-05, DE.CM-03, DE.CM-09, RS.MA-01). Without that mapping the artefact is sales copy, not GRC reference material. **Mandatory before promotion.**

2. **"Supported" ≠ auditable. The draft conflates the two.** Every capability is asserted as "Supported" or "Supported with caveats" with no statement of the **log artefact** produced (event name, schema field, retention default, immutability, export integrity). An auditor will reject "the product blocked it" without a tamper-evident log record carrying timestamp, subject, object, policy ID, decision, and rule version. Examples that need a named log artefact:
   - Policy #1 Block-upload — what Skope event class (`alert`, `policy`, `incident`), what fields, what default retention, exported to where, with what integrity control?
   - Policy #2 Coach — the **justification text** the user enters is itself sensitive evidence; where is it stored, who can read it, what is the retention?
   - Policy #6 GenAI prompt block — the draft says "log full prompt-hash to SIEM (NOT full prompt — for privacy)". A hash without the cleartext is **not investigable**. Either log the cleartext under defined access control (and a DPIA), or accept the policy cannot evidence the breach. Don't pretend both.

3. **SSL inspection lit across "Employees-All" with no privacy carve-out is a PDPA/GDPR landmine.** Policies #1, #6, #8, #10, #11 require SSL inspection on the user's full traffic. Draft has zero discussion of:
   - **PDPA MY (2010 + 2024 amendments)** — notification (Principle 1) and consent in employment context; the 2024 amendments introduced a mandatory data-breach-notification obligation and a Data Protection Officer requirement. The draft is silent on whether intercepted personal-domain traffic processed by Netskope counts as a disclosure of personal data to a data processor and whether the cross-border-transfer regime (now under the 2024 amendment framework) is engaged when NewEdge PoP is outside MY. **Verify against gazetted instrument; do not state thresholds from this review.**
   - **GDPR Art. 5/6/13/35** — lawful basis for monitoring, Art. 35 DPIA almost certainly required for SSL-MITM on workforce traffic, Art. 88 / Member-State employment law.
   - **EDPB Guidelines 05/2020 on consent in employment** — consent is not a valid lawful basis where there is a power imbalance.
   - **HKPCPD PCPD employer-monitoring guidance** — disclosure-before-monitoring expected.
   - **MAS TRM and BNM RMiT** both require that monitoring tooling respect privacy obligations; neither is a free pass.
   The "Block Upload + Post; Allow Login (read-only) for personal Gmail/OneDrive (cultural concession, often pushed back by HR)" line in policy #10 telegraphs that this conflict was noticed and then waved off. HR pushback is a signal, not a nuisance.

4. **Regulator clocks are entirely absent.** None of the following are mentioned, all of which a CASB practitioner would be expected to know:
   - **BNM RMiT** — material technology / cyber incident reporting; outsourcing/third-party arrangement notifications (current edition; verify clock against gazetted document and BNM circulars before reliance).
   - **MAS TRM / Notice 644 / Notice on Technology Risk Management** — incident reporting expectations and root-cause submission windows (verify against current MAS Notice text).
   - **HKMA SA-2 and the General Principles for Technology Risk Management** — incident reporting expectations to HKMA.
   - **PDPA MY 2024 amendments** — breach-notification clock as gazetted; DPO requirement; cross-border transfer regime.
   - **GDPR Art. 33** — 72-hour notification to the supervisory authority.
   - **EU DORA** — major-ICT-incident classification and reporting under RTS (relevant if the FI has EU footprint).
   A CASB is the **detection** and often the **evidence source** for these clocks. The draft never says which Netskope event class starts which clock, nor whether Netskope's incident timestamps are reliable enough to defend a notification window. **Add a clocks section before promotion; cite primary sources only.**

5. **API mode is "after-the-fact" — the draft admits it but does not draw the consequence.** Policy #4 (quarantine externally-shared sensitive files) is API-only. If a sensitive file is externally shared at T=0 and Netskope quarantines at T=45 minutes, the data has been exfiltrable to an external party for 45 minutes. An auditor will ask: (a) what is the documented MTTR (mean time to remediate) for this policy; (b) is that MTTR within the regulator's breach-clock; (c) what compensating inline control protects the window. The draft offers none of these. The DLP-owner-unknown limitation (line 234) compounds it — you cannot notify someone you cannot identify.

6. **CCI risk scoring is vendor-proprietary and not independently corroborated.** "Cloud Confidence Index" / "Cloud Confidence Level" (Poor/Low/Medium/High/Excellent) is the gating condition in policies #2 and #10. The draft says CCI uses "~100 attributes adapted from CSA CCM" but does not state: (a) the attribute list, (b) the scoring weights, (c) the refresh cadence, (d) the appeals process for a miscategorised app, (e) whether the score is auditable against a published methodology document. An auditor cannot accept a black-box risk score as the basis for a control decision. ISO 27017 CLD.6.3.1 (shared responsibility) and ISO 27036 (supplier relationships) both expect documented risk methodology, not vendor magic. **Flag CCI methodology as Unverified and require a vendor-supplied methodology paper before treating it as evidence.**

7. **UEBA / UCI thresholds are unspecified — control unenforceable.** Policy #7 (impossible travel, 800 km/h default) and policy #12 (UCI < 50 triggers RBI) condition automated enforcement on opaque scoring. The draft itself flags signal-level detail as missing (line 281). An auditor will reject a control where the threshold is "tunable" without a documented baseline, change-management process, and false-positive rate. Add: who can change the threshold, where the change is logged, what review cycle applies. NIST CSF 2.0 GV.RM (risk management strategy) and GV.OC (organisational context) expect documented thresholds.

8. **Reverse-proxy app list is feature-gated and the draft built a control matrix on inference.** Policy #5 (BYOD read-only via reverse proxy) lists M365 SharePoint / OneDrive but the supported-app list is admitted "Unverified" (line 18, line 280). Building a documented BYOD control on an inferred app list is not auditable. Pull the gated list and republish, or scope policy #5 narrowly to apps you have confirmed in writing from Netskope SE.

9. **OAuth-grant policy #3 has a silent revoke-coverage gap.** "Revoke (Entra-connector dependent)" — for Salesforce, Workspace, and other SSPM-connected platforms, the draft does not state whether revoke is supported, partial, or absent. An auditor reading "Action: Alert + Revoke" will assume revoke works everywhere. Specify per-connector; otherwise the policy evidences detection but not response — which materially changes the NIST CSF mapping (DE.CM-09 only, not RS.MI-02).

10. **No retention, no integrity, no chain-of-custody on logs.** The draft never states (a) default log retention in Netskope's tenant, (b) maximum retention available, (c) whether logs are tamper-evident at rest, (d) what hash/signing covers SIEM export, (e) whether Netskope can produce a forensically defensible log extract on subpoena. BNM RMiT (logging/audit-trail expectations in the current edition) and ISO 27017 CLD.12.4.5 (monitoring of cloud services) require this. Without it the SIEM export is a convenience feature, not evidence.

11. **Data residency for incident metadata is hand-waved (line 285).** For an MY FI under BNM RMiT, or an SG FI under MAS TRM Outsourcing/Cloud Advisory, the location of DLP-hit metadata (which contains snippets of the regulated content that triggered the hit) is a cross-border data transfer question. "Flag for vendor BNM RMiT / MAS TRM conversation" is not an answer; it is a deferral. State the question precisely: **(a)** does Netskope process incident-metadata in-region (MY, SG, HK PoPs respectively); **(b)** if not, what cross-border-transfer mechanism applies; **(c)** what contractual flow-down is required under ISO 27018 A.11.1 (sub-processors) and BNM RMiT outsourcing rules. Until answered, do not propose Netskope to a regulated MY/SG/HK FI as a data-residency-compliant tool.

12. **Sub-processor disclosure (ISO 27018 A.10/A.11) absent.** Netskope acquired Kindite (2020 — encryption), Infiot (2022 — SD-WAN), Dasera (2024 — DSPM) and operates the NewEdge PoP fabric. Each is a potential sub-processor in the data-flow. ISO 27018 A.11.1 requires the cloud-PII-processor to disclose sub-processors and notify customers of changes. The draft is silent on whether Netskope publishes a sub-processor list, how change-notification works, and whether the customer has a right to object. Add to the GRC readiness checklist.

13. **Encryption / tokenisation (Kindite, line 39) — "ask why before proposing it" is the right instinct but the compliance angle is missing.** For PDPA MY / PDPA SG / HKPCPD purposes, tokenisation at the gateway is one of the few designs that *materially* changes the data-protection posture (Netskope sees tokens, not personal data). It deserves a compliance treatment, not a dismissal. Conversely, if tokenisation is the only way the design works for residency, the implementation lift the draft mentions is itself a control-design issue (key custody, key residency, BYOK/HYOK).

14. **AI Guardrails maps to ATLAS + OWASP LLM Top 10 (line 41) — claimed, not corroborated.** Vendor-controlled claim that classifiers are "ATLAS-mapped" needs (a) the version of ATLAS, (b) the mapping table, (c) whether the mapping is reviewed when ATLAS updates. Without these the claim is marketing. Same for OWASP LLM Top 10 (version 2023? 2025? 2026?).

15. **No mention of SOC 2 / ISO 27001 / FedRAMP / CSA STAR attestations.** A GRC reference cannot promote a vendor without naming the third-party attestations the buyer will rely on for shared-responsibility evidence. State: Netskope's current SOC 2 Type II scope and report date, ISO 27001 certification scope, ISO 27017/27018 attestations (yes/no), CSA STAR level, FedRAMP authorisation level (Moderate? High?), C5 (Germany), IRAP (Australia), MTCS (Singapore). Mark each as **Unverified** if not pulled from the vendor trust portal. Without them the buyer cannot complete ISO 27017 CLD.6.3.1 (shared responsibility) or BNM RMiT third-party assurance.

## Capability claims to corroborate (do not promote without independent source)

- **"~80,000+ apps in CCI, ~100 attributes adapted from CSA CCM"** (line 29) — vendor-controlled; numbers vary across vendor marketing; pull the current methodology paper or mark Unverified.
- **"1,800+ GenAI apps catalogued"** (line 41, also flagged in draft line 283) — vendor blog claim, no independent source.
- **"Behavior Analytics produces a User Confidence Index"** (line 34) — covered scenarios are listed thematically only; specific detector logic, false-positive rates, and tuning telemetry are not public.
- **"ATLAS-mapped classifiers + OWASP LLM Top 10"** (line 41) — version, mapping table, refresh cadence all missing.
- **"SSPM 3rd-party-app discovery scoring 0–100 across five tiers"** (line 33) — methodology not public.
- **Reverse-proxy supported-app list** (line 18) — feature-gated; draft inferred.
- **Cross-region in-region scanning of DLP incident metadata** (line 36, 285) — vendor docs do not commit publicly.
- **NewEdge PoP regions and which ones host which control planes** — needed for residency / data-flow mapping; not in draft.
- **Netskope third-party attestations** (SOC 2 Type II, ISO 27001/27017/27018, CSA STAR, FedRAMP) — absent from draft; pull from Netskope trust portal and cite report date.
- **Sub-processor list and change-notification SLA** — required for ISO 27018; absent.
- **Default log-retention, export integrity, tamper-evidence** — absent; required for BNM RMiT audit-trail expectations.

## Policies to refine

- **Policy #1 (Block regulated upload)** — add: (a) which Netskope event class is emitted, (b) which schema fields contain subject/object/decision/policy-version, (c) default retention and tamper-evidence claim, (d) the BNM RMiT / MAS TRM / ISO 27017 control IDs this policy evidences, (e) what the regulator expects when this fires on PII (notification clock decision tree).
- **Policy #2 (Coach low-CCI)** — clarify: justification text is itself personal data; specify storage location, access control, retention, and lawful basis (PDPA / GDPR). State whether the coach event is admissible as evidence of user acknowledgement (it usually is not without a click-wrap timestamp captured with non-repudiable signing).
- **Policy #3 (OAuth grants)** — split into "Detect" (NIST CSF DE.CM-09) and "Respond" (RS.MI-02), state per-connector whether revoke is supported, and add an explicit revocation-evidence log.
- **Policy #4 (Quarantine externally-shared)** — add documented MTTR target; map MTTR against breach-notification clocks; state compensating inline control for the pre-quarantine window; explain owner-unknown handling (line 234 limitation) including the policy for "owner could not be identified" incidents.
- **Policy #5 (BYOD read-only)** — narrow to apps with verified reverse-proxy support; declare residual risk for un-covered apps; state how device classification is evidenced (cert chain, Entra compliant-device claim, what's in the log).
- **Policy #6 (GenAI prompt block)** — decide: hash-only (not investigable) vs. cleartext (DPIA required). Don't ship both pretending privacy is preserved. Add: ISO 42001 / NIST AI RMF context; specify what counts as a "prompt-injection" detection for incident-class purposes; state the cross-border-transfer treatment if Netskope ships the prompt to an inspection function outside MY/SG/HK.
- **Policy #7 (Impossible travel)** — make the 800 km/h threshold a documented control parameter under change-management; specify the ASN / VPN exclusion list as a configuration item with named owner; state the false-positive rate baseline.
- **Policy #8 (MIP label gating)** — state the dependency on Microsoft Purview / AIP licensing explicitly as a precondition; without it the policy fails open. Add: how Netskope handles label-tampering (a user removes the label client-side before upload).
- **Policy #9 (Malware scan & quarantine)** — state sandbox jurisdiction (where the file is detonated); for regulated FI, that location matters under outsourcing rules.
- **Policy #10 (Tenant pinning)** — the "Allow Login (read-only) for personal Gmail / OneDrive" carve-out is a privacy/policy decision that needs documented sign-off from HR/Legal/DPO, not a tooling default. Make the sign-off a precondition of the policy.
- **Policy #11 (Bulk-exfil heuristic)** — state how the 100 MB / 500 MB-per-hour thresholds were chosen (baseline? regulator expectation?); add change-management.
- **Policy #12 (RBI for degraded UCI)** — clarify watermark content (user identifier? timestamp? case ID?); state evidentiary value (watermark on a leaked screenshot must trace back to the session log).

## Limitations the draft missed

- **No breach-notification-clock decision tree.** When does a DLP hit become a notifiable incident? Who decides? Where is the decision logged? CASB is upstream of this workflow; the draft pretends the workflow doesn't exist.
- **No mention of legal hold / e-discovery interaction with quarantine.** Policy #4 moves files to a quarantine folder. If those files are on legal hold, the move may itself be spoliation. Salesforce soft-delete (line 235) compounds this — a "deleted" record on hold is non-compliant.
- **No data-subject-rights interaction.** Under PDPA MY, GDPR Art. 15–22, HKPCPD DPP 6 — a data subject can request access/erasure of personal data Netskope holds about them (their browsing, their prompts, their DLP hits). How does Netskope service that request? Not mentioned.
- **No DPIA / TIA scaffolding.** SSL inspection of employee personal traffic is a high-risk processing activity under GDPR Art. 35; transfers from EU to NewEdge non-EU PoPs require a Transfer Impact Assessment post-Schrems II. Not mentioned.
- **No sub-processor change-notification window.** Netskope's M&A pattern (Kindite, Infiot, Dasera) means sub-processor change is a recurring event.
- **No statement on Netskope's own incident-response and customer-notification SLA.** If Netskope is breached, when does the customer hear? This is BNM RMiT third-party / MAS TRM outsourcing concern, not a footnote.
- **No statement on "concentration risk".** Netskope SSE consolidates SWG + CASB + ZTNA + DLP + RBI + AI Gateway. Regulators (BNM RMiT third-party, EU DORA concentration-risk recital, MAS Outsourcing) expect documented concentration-risk treatment. The draft promotes the consolidation as a feature without naming the concentration risk.
- **No statement on exit / portability.** When the FI moves off Netskope, can it export policies, DLP classifiers, UEBA models, historical logs? In a defensible format? ISO 27017 CLD.8.1.5 (removal of cloud service customer assets) and BNM RMiT outsourcing exit-strategy expectations both bite here.
- **No statement on encryption-at-rest / key management for tenant data inside Netskope** (logs, DLP incident metadata, UEBA telemetry). BYOK? HYOK? Netskope-managed only? Material to PDPA / GDPR / BNM RMiT cryptographic-control expectations.
- **No statement on availability / SLA.** A CASB inline failure mode (fail-open vs fail-closed) is a security and compliance decision. Not mentioned.
- **No statement on regulator inspection rights.** BNM RMiT and MAS Outsourcing require the regulator and the FI to retain audit and inspection rights against the cloud service provider. Does Netskope contractually accept this for MY / SG FIs?
- **NPA / ZTNA (mentioned as adjacent) reference scope is conflated with CASB.** Either keep this draft pure CASB, or expand the compliance treatment to cover ZTNA's separate control set (NIST 800-207, MAS TRM access-control).

## Overall verdict

- **Promote as-is to 04-vendors/?** NO
- **One-line reason:** Capability-rich, evidence-thin — no control-to-policy mapping, no log-artefact specification, no regulator-clock treatment, no privacy/DPIA treatment of SSL inspection, no third-party attestation citations; rewrite before promotion.
