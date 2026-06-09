# Compliance lens review — Skyhigh Security (Skyhigh Cloud / CASB)

Scope: hard-marker GRC / IS-audit review of the vendor draft at `_research/2026-06-10-vendor-practitioner-reference/raw/skyhigh-security.md`. Lenses applied: BNM RMiT (Malaysia, current edition), MAS TRM (Singapore), HKMA SA-2, ISO/IEC 27017:2015, ISO/IEC 27018:2019, PDPA (Malaysia 2010 + 2024 amendments), GDPR, NIST CSF 2.0. Citations to specific clauses below are illustrative of the control families the policies would be mapped against; final mapping requires the customer's authoritative control library and is **not legal/regulatory advice**.

## Findings to act on (highest impact first)

1. **Draft does not map a single policy to a control framework.** Every one of the twelve configurable policies (lines 41–135) is described in product/operational terms only. No control reference (BNM RMiT, ISO 27017, NIST CSF subcategory, MAS TRM, HKMA SA-2, PCI DSS, PDPA). For a CASB this is the load-bearing section of the document — "what control does this policy evidence, and is the evidence auditable?" — and it is missing. Auditors do not accept a screenshot of a policy; they accept a mapping from policy → control objective → evidence artefact → retention/integrity story. The draft skips all four legs.

2. **No discussion of the audit log itself.** Vendor draft repeatedly says "incident", "alert", "save evidence", "save_evidence: false" (line 164) — but never describes the policy-decision log: what fields it carries, retention, tamper-evidence, time-source, who can edit/delete, export format, SIEM connector. Without that, every "Supported" capability claim collapses to "vendor asserts the product can do X" with no audit-quality artefact. ISO 27017 CLD.12.4.1 (logging) and ISO 27001:2022 A.8.15 (logging) both require log content, protection, and review — the draft does not let a reviewer assess any of them. **The `save_evidence: false` default in the worked example (line 164) is the wrong default for a regulated FS deployment and should be flipped — or at minimum called out.**

3. **SSL inspection on personal HTTPS is a PDPA/GDPR landmine and the draft is silent on it.** Policies #2 (inline DLP, line 53), #9 (GenAI inline, line 109), #12 (tenant restrictions, line 133) all require SSL inspection in the forward-proxy path. SSL inspection on personal banking, personal webmail, healthcare portals, or any traffic from a BYOD/dual-use device implicates: PDPA (Malaysia) §6 (consent + Personal Data Notice for processing personal data), §40 (sensitive personal data — health/financial); GDPR Art. 5(1)(a) lawfulness/transparency + Art. 6 lawful basis + Art. 9 special categories where health/banking traffic is intercepted; ISO 27018 A.10.1 (additional safeguards on PII processing). The draft does not mention category-based decryption bypass (banking, healthcare, government, personal-mail), does not mention employee notice / works-council requirements, and does not flag that "monitor only" still requires decryption. This must be in any compliance write-up.

3a. **Personal email is a specifically PDPA-sensitive case the draft ignores.** Inline DLP and Shadow-IT control (Policies #2, #8, #9) will decrypt and inspect Gmail/Outlook.com/Yahoo personal mail unless an explicit bypass category is configured. Reading personal email by a corporate proxy without specific consent is a §6 PDPA breach in Malaysia and an Art. 6/Art. 9 breach in the EU. The draft does not surface this.

4. **Regulator clocks are entirely absent.** A CASB sits squarely on the data-protection and incident pathways. The draft has no mention of:
   - **BNM RMiT** ICT-incident notification (current edition: **notify BNM within the timeframes specified by BNM; significant incidents require formal reporting** — exact clock by RMiT edition, customer to confirm against the BNM-issued copy they hold; do not quote a clock from memory).
   - **PDPA Malaysia 2024 amendments**: personal-data-breach notification obligation to the Commissioner **as soon as practicable** (statutory clock per the 2024 amendments — customer must verify against the gazetted text and any subsidiary regulations; do not assert a numeric clock without the citation).
   - **GDPR Art. 33**: 72-hour notification to supervisory authority for personal-data breach.
   - **MAS TRM** §8 (Incident Management) reporting timelines for relevant incidents (clock per the current Notice on Technology Risk Management; customer to confirm).
   - **HKMA SA-2** incident-reporting expectations for AIs.
   - **EU DORA** Art. 19 major-ICT-incident reporting (where applicable).
   A CASB will surface the *signal* (anomaly, compromised account, DLP incident); the **runbook from CASB-alert → regulator-clock-start → who-notifies-whom is the audit story**, and the draft does not even acknowledge the clocks exist.

5. **CloudTrust Ratings are presented as if they were independent assurance — they are not.** Lines 24, 97–103: the 1–9 CloudTrust scale and the "~110 risk attributes" are a vendor-asserted, vendor-maintained risk rating with no published methodology, no scope/criteria document, no SOC 2 / ISAE 3000 attestation on the rating process. Treating CloudTrust as evidence of third-party risk assessment under BNM RMiT 10.x (Third-Party Service Provider) / MAS Outsourcing Guidelines / ISO 27001 A.5.19–A.5.23 is **not defensible** without (a) the customer's own TPRM workflow consuming CloudTrust as an *input*, and (b) corroboration by an independent control attestation per service. Draft does not warn the reader.

6. **"Save Evidence" feature creates its own data-protection problem the draft hides.** Line 26 + line 44 + line 79: encrypted evidence copies stored to customer S3/Azure, max 250 MB per file. This means the CASB is **replicating regulated content** (PII, PHI, PCI, customer financial data) to a secondary bucket. Implications the draft never names:
   - PDPA §8/§9 (security of personal data) and ISO 27018 A.10 require equivalent controls on the evidence store.
   - Cross-border transfer (PDPA §129; GDPR Chapter V; HKMA cross-border data principles): if the S3 bucket is outside MY/SG/HK/EU, you have created a new transfer that needs a legal basis and the customer's transfer assessment.
   - Retention conflict: evidence retained beyond the source-document retention schedule violates data-minimisation (GDPR Art. 5(1)(c), (e)).
   - Discoverability: evidence is now subject to subject-access requests and legal hold.
   None of this is flagged.

7. **Reverse-proxy's SAML hijack is itself an architectural risk the draft does not name.** Line 16: "modifying the SAML endpoint in the customer's IdP so SAML assertions land at the Skyhigh reverse proxy first; the proxy evaluates context, then issues a new SAML assertion." That is a **token-issuance interposition** — Skyhigh is acting as a SAML attribute-asserting party in the trust chain. Compliance implications: NIST SP 800-63B identity-assertion integrity; MAS TRM §11 (Access Control) trust-chain documentation; the customer's IdP security control set now extends across the Skyhigh tenancy; key-rotation, signing-cert custody, and break-glass procedures all need to be documented as part of the IAM control narrative. Draft treats it as a deployment detail.

8. **Connected Apps (Policy #4) blind spot is much bigger than the draft admits.** Lines 27, 65–71, 175: OAuth grants invisible if user signs in with personal account. In a BYOD or dual-use environment this is the *normal* case, not the edge case. From a compliance standpoint, claiming "OAuth governance coverage" on the strength of Connected Apps without naming this blind spot in the customer-facing control description is **misleading the auditor**. The mapping line should read: "Connected Apps evidences OAuth-grant governance **for grants made within the connected M365 / Workspace tenant only**; grants made via personal account on a managed/unmanaged device, and grants to apps from outside the connected IdP, are out of scope of this control."

9. **GenAI policies (#9, #10) silent on input/output asymmetry that auditors will probe.** Inline GenAI policy (line 105) inspects **uploads** but does not retrieve **responses/conversation history** for non-integrated AI apps (line 185 admits this). For ChatGPT Enterprise and Copilot the API integration claim needs scrutiny — *what* is logged: prompts, completions, grounding sources, embeddings, file references? The compliance question is "can I prove what data the LLM saw and what it returned?" — the draft does not answer it. NIST AI RMF GOVERN-1.1 / MAP-2.3 / MEASURE-2.1 and ISO/IEC 42001 require evidence of monitoring; "DLP scanned the upload" is not the same control.

10. **PCI DSS v4.0.1 not mentioned anywhere despite obvious applicability.** Any deployment in payments, retail, or hospitality intersects PCI DSS 3.4.x (PAN protection in transit, including over public networks — interception by CASB needs key-custody discipline), 10.2 (audit log content), 10.5 (log integrity), 12.8 (third-party service provider management — Skyhigh becomes a TPSP storing CHD if "Save Evidence" catches a PAN-bearing file). Draft does not flag.

11. **No statement on Skyhigh's own assurance posture.** A CASB processing customer data must itself be auditable. The draft cites zero independent attestations on Skyhigh: SOC 2 Type II, ISO 27001 certification, ISO 27017 / 27018 / 27701 certifications, PCI DSS SAQ-D, FedRAMP, IRAP, or regional. Until this is established, every "Supported" claim is **vendor self-attestation**. The "Notes for the lens reviewers" section (line 219) admits no CVE pass was run; it should also admit no independent-assurance review was done.

12. **Time-source and log integrity not mentioned.** ISO 27001 A.8.17 (clock synchronisation), ISO 27017 CLD.12.4.4, MAS TRM §11 require synchronised, tamper-evident logs for forensic admissibility. The draft does not state Skyhigh's NTP discipline, log-signing, immutability, or retention defaults. Without that, every "incident" record is hearsay.

13. **PDPA Malaysia 2024 amendments — Data Protection Officer obligation absent.** The 2024 amendments require a Data Protection Officer in defined circumstances. A CASB rollout that processes employee personal data, browsing activity, and "evidence" copies of PII is exactly the trigger zone — the compliance write-up should reference DPO involvement and the Personal Data Notice update. Customer must verify the exact triggering criteria against the gazetted 2024 text.

## Capability claims to corroborate (do not promote without independent source)

- "1,900+ AI services / 50,000+ services / 110 risk attributes" (lines 23–24, 213). Vendor-asserted; no published methodology; no third-party benchmark. Treat as marketing.
- "~40 native API connectors" (lines 17, 213) — fluid count, vendor page only.
- "Unified DLP" across inline + API + endpoint (line 217). Documentation tree shows separate policy objects — vendor copy contradicts vendor docs. Flagged by the draft itself but needs hardening in the compliance write-up because the audit story for "one policy, consistent enforcement" is a common control-design objective.
- "Integrated anti-malware solution and inline emulation-based sandboxing" (line 32). Engine, vendor of engine, signature update cadence, sandbox detonation evasion posture — all unstated. Independent test (SE Labs / AV-TEST / ICSA / MITRE Engenuity) coverage: unknown.
- Reverse-proxy coverage list (line 16) — vendor-asserted; mobile / non-SAML break paths flagged at line 176 are the real story.
- DRM via Ionic / Seclore (lines 33, 180) — corroborated only by the partner page (line 206). Confirm Ionic still exists as a product post-Ionic-acquisition-history; FPE/tokenisation as a first-class action remains unverified (line 215).
- Cloud Connector log-source list (line 218) — not enumerated publicly. Cannot assume Cisco / Zscaler / Palo Alto / Check Point / Fortinet without a vendor-published compatibility matrix.
- Independent corroboration in draft is one DRM partner page + one distributor page (line 207, 219). That is not corroboration. CVE/NVD/CISA pass not done — explicitly admitted at line 219. Auditor would reject.
- Skyhigh's own SOC 2 / ISO 27001 / ISO 27017 / ISO 27018 / ISO 27701 / FedRAMP / IRAP posture: **draft is silent**.

## Policies to refine

- **Policy #1 (Sanctioned-app DLP, API)**: add (a) the log-event schema (what fields fire on an incident, retention, export); (b) the control mapping (e.g. ISO 27001 A.8.12 data leakage prevention, ISO 27017 CLD.6.3, NIST CSF PR.DS-05); (c) the data-handling story for "Save Evidence" (where stored, encryption at rest, retention, cross-border, DSR/legal-hold); (d) the false-positive remediation workflow (who reviews, SLA, audit trail of suppression decisions).
- **Policy #2 (Inline DLP, proxy)**: must state the SSL-inspection bypass categories (banking, healthcare, government, personal-mail) and reference the employee Personal Data Notice / works-council notification. Without that, this policy creates a PDPA/GDPR liability greater than the DLP risk it mitigates.
- **Policy #3 (Managed-device-required)**: `save_evidence: false` in the worked example (line 164) is wrong for a regulated FS deployment. Flip the default, or state the retention/integrity choice explicitly.
- **Policy #4 (Connected Apps)**: scope statement must read "**within the connected M365/Workspace tenant only**". Without that scope qualifier the policy is mis-evidenced.
- **Policy #5 (External-collaboration)**: needs retention statement for the evidence copy; needs explicit handling of joint-venture / contractually-permitted external sharing (allow-list governance). Auditor will ask "who approves the domain allow-list and where is the change log."
- **Policy #6 (Anomalous-access-location)**: false-positive risk understated. Mobile-carrier CG-NAT and TOR-exit are now common; "action: incident only" should be the default for new tenants until tuned. Map to NIST CSF DE.AE-02 / DE.CM-01 — but only after retention/tuning is settled.
- **Policy #7 (Compromised-account)**: needs alignment with the IdP's own brute-force / risky-sign-in (Entra ID Protection / Okta ThreatInsight). Two systems alerting on the same primitive without de-duplication produces noise and audit confusion. State which system is the system-of-record.
- **Policy #8 (Shadow-IT, CloudTrust threshold)**: do not present CloudTrust rating as an assurance signal. State it as **vendor risk indicator** consumed by the customer's own TPRM process. Map to ISO 27001 A.5.19/A.5.20 only with that qualifier.
- **Policy #9 (GenAI inline)**: SSL inspection on AI traffic from a personal browser session is the same PDPA/GDPR issue as Policy #2. Add bypass category for "personal AI use on BYOD" unless contractually disallowed via employee notice.
- **Policy #10 (Copilot DLP)**: must state what is logged from the Copilot API integration — prompts, completions, grounding sources, or only DLP-event metadata. Without that, the AI-governance audit story collapses.
- **Policy #11 (Malware scan)**: name the AV/sandbox engine, signature cadence, and how false-positive override is audited. Auditors treat AV as a control only when supplier and update cadence are documented.
- **Policy #12 (Tenant restrictions)**: blast radius is correctly noted ("whole org locked out of M365") — add the change-control / four-eyes requirement and the rollback plan.

## Limitations the draft missed

- **No statement on the CASB's processing of employee personal data**. Browsing, geo, login telemetry, anomaly scores, and saved evidence are themselves personal data. PDPA Personal Data Notice, GDPR Art. 13/14 transparency, and ISO 27018 A.10.13 (employee personal data processed by cloud-provider) all apply. The draft treats employees as objects of monitoring, not subjects of data-protection rights.
- **No statement on lawful basis for monitoring**. EU works councils, German BetrVG §87(1)(6), and PDPA consent-or-notice posture must be settled before deployment, not after.
- **No discussion of subject-access requests / right to erasure** against the CASB-held data (alerts, anomaly scores, evidence copies).
- **No discussion of regulatory inspection rights** — BNM / MAS / HKMA examiner access to the CASB tenant and logs; right-to-audit clauses in the Skyhigh MSA.
- **No data-localisation / regional-tenancy statement for Skyhigh's own POPs and management plane**. Where does the customer tenant live? Where do POPs decrypt? Where does the management console store config and logs? BNM RMiT 10.x / MAS Outsourcing / HKMA cross-border data and PDPA §129 transfer regime all depend on this.
- **No exit / portability plan**. ISO 27017 CLD.6.3.1 and BNM RMiT 10.x exit-strategy. CASB lock-in is real (custom classifiers, fingerprints, tuning history) — the draft does not address it.
- **No retention story for "evidence" copies and incident records** — required by ISO 27001 A.8.10 (information deletion), GDPR Art. 5(1)(e), PDPA §10 (retention principle).
- **No mention of Skyhigh's sub-processor list** — required disclosure under GDPR Art. 28 and a PDPA-derivative duty. Sub-processors include hyperscaler (AWS/Azure/GCP) for POPs, plus any third-party threat-intel feeds, plus the DRM partner (Ionic/Seclore) if invoked.
- **No mention of breach-notification obligations Skyhigh owes the customer** — Skyhigh-side incident → customer-clock for regulator notification.
- **No mention of legal hold / e-discovery posture** of the CASB-resident data.
- **No mention of accessibility / WCAG** of the admin console — increasingly required in EU public-sector contexts and emerging in MY (regulator portals).
- **No mention of FIPS 140-2/-3 module status** of any crypto component — required for some US federal-adjacent and increasingly referenced in APAC FS supervisory expectations.
- **No mention of post-quantum-crypto roadmap** — a 2026 draft on TLS-interception kit that does not state a PQ position is incomplete.
- **No mention of the EU AI Act** intersection (Art. 6 high-risk classification for workplace monitoring AI / Art. 26 deployer obligations) for the UEBA and GenAI-policy modules.
- **No mention of NIS2 / DORA** for EU-regulated deployments.

## Overall verdict

- **Promote as-is to 04-vendors/?** NO
- **One-line reason:** Draft is a competent product description but contains zero control mapping, no audit-evidence story, no regulator clock, and no acknowledgement of the SSL-inspection / personal-data conflict — every one of which is non-optional before this can be cited as compliance reference material.
