# Playbook v0 → v1 changelog

Source: five specialist reviews (`reviews-v1/qa.md`, `cyber.md`, `mda-expert.md`, `sysint.md`, `docs.md`), consolidated in `reviews-v1/_consolidated-changes.md`. All five graded v0 NO-WITH-CRITICAL-FIXES.

## Structural changes

- **TOC + How-to-read + One-page summary card added at top.** Sourced from: docs.
- **Day 0 precondition checklist added** before Day 1, with 10 named verifications and what fails if each one isn't met. Sourced from: mda-expert, qa, docs.
- **"What MDA is for" rewritten** as 4 distinct capability sets (was a 113-word run-on opener). DSPM-for-AI prompt-capture-depth split into deep / shallow per tool. Sourced from: docs, mda-expert.
- **Consolidated "Gaps and compensating controls" table** added near the top, replacing the duplicated "What it does NOT cover" list and "Residual risks" table that overlapped ~60%. Single canonical home with ATT&CK technique + cross-ref column. Sourced from: docs (dispute 7 resolved), cyber, mda-expert.
- **Concentration risk** promoted from buried subsection to a top-level callout, reframed in technique terms (T1557 at internet-scale, T1098.001 / T1556 / T1528 across the stack). Sourced from: docs, cyber.
- **Policy table schema locked**: Risk reduced | What it does | ATT&CK | Critical prerequisite | Console path | Key configuration | WARNING — trap | Defeated by | Auditable evidence | SIEM artefact + connector. Applied to every policy. Sourced from: docs, qa, sysint.
- **Three new policies added**: Policy 6b (B2B exfiltration); Policy 11 (mass-delete / ransomware-in-SaaS — T1485 / T1486); Policy 12 (Purview IRM signal-boost integration). Sourced from: cyber (B2B + mass-delete), mda-expert (IRM).
- **Policy 2 split into 2a (classic OAuth) + 2b (App Governance Predictive Risk)**, with primary control identified as Entra admin-consent workflow + verified-publisher restriction (configured outside MDA). Sourced from: cyber, mda-expert (dispute 6 resolved).
- **Pre-CAAC compatibility test plan** added as Day 30 sub-section (regression test matrix). Sourced from: mda-expert, qa.
- **Day 30 / Day 90 phase openers** carry per-app onboarding effort estimates and platforms-that-supersede pointers (GSA, Edge for Business `gen-ai.app-control` H2 2026, Purview Adaptive Protection). Sourced from: sysint, mda-expert.
- **Seven appendices added**: A (SCIM cascade catalogue), B (slow-and-low Sentinel KQL — 6 queries), C (CAAC deprovisioning runbook), D (DR / fail-mode matrix), E (ATT&CK + ATLAS coverage matrix), F (threat-actor profile per policy), G (glossary). Sourced from: sysint (A, D), cyber (B, E, F), mda-expert (C), docs (G).
- **Three-lens sign-off table rewritten** — eight rows (original 3 lenses + five v1 specialists), each with v0 verdict, v1 verdict, and outstanding work. "INCORPORATED" wording removed (contradicted the source lens NO verdicts in v0). Sourced from: qa, docs.

## Content changes (selected — full per-line list below)

### Cross-cutting

- **CloudAppEvents retention** — was "30 days default"; now states three tiers: 30 days standard / 90 hot + 90 warm with unified XDR data lake (GA 2025) / Sentinel forwarding for 1-7 year BFSI retention. Resolves the v0 self-contradiction between architect-lens (180 days) and product-lens (30 days). Sourced from: qa, mda-expert (dispute 1 resolved).
- **File-policy ceiling** — was 50; now 200 (raised February 2026; verify against current `data-protection-policies` page). Consolidation guidance softened accordingly. Sourced from: mda-expert, qa (dispute 2 resolved).
- **Tenant primary region** — was "no APAC primary region"; now "Japan East available 2025 H2; India on roadmap H2 2026; verify against Trust Center on day of promotion". Materially changes the MY/SG/HK residency calculus. Sourced from: qa, mda-expert, docs (dispute 3 resolved).
- **GenAI category** — was monolithic; now uses 2025 H2 sub-categories (LLM / AI Coding Assistant / AI Image Generator / AI Video Generator / AI Voice/Avatar). Sourced from: mda-expert, cyber (dispute 4 resolved).
- **NIST CSF half-mapping removed** — v0 had PR.AA-05 / DE.CM-03 in some policies and nothing in others. v1 drops all half-mappings; full mapping deferred to `06-compliance/...-control-mapping.md`; technique-based mapping (ATT&CK) added in its place. Sourced from: qa, docs, cyber (dispute 5 resolved).

### Per-policy

- **Policy 1** — reframed from "Risk reduced" to "Visibility precondition for" (discovery without enforcement doesn't reduce risk; it enables decisions); added ATT&CK row; updated console path to 2026 unified portal; flagged discovered-subdomains deprecation as already-in-effect (past 2025-12-31). Sourced from: cyber, mda-expert, qa.
- **Policy 2 → 2a + 2b** — split; primary control identified as Entra admin-consent workflow (configured outside MDA, not in this playbook); 2b is the post-consent behavioural detection layer via App Governance Predictive Risk (shipped 2025 H2). Power Platform service-principal sub-class explicitly flagged. Sourced from: cyber, mda-expert.
- **Policy 3** — "Suspend user" trap expanded with the REST-layer detail (`POST graph.microsoft.com/v1.0/users/{id} accountEnabled=false` runs under Defender SP; silent failure on hybrid-AD users with no writeback). SCIM cascade table referenced (now in Appendix A). Slow-and-low gap acknowledged and KQL provided (Appendix B.1). Threshold reframed from default to "starting heuristic — derive from P95 of legitimate users". Sourced from: mda-expert, sysint, qa, cyber.
- **Policy 4** — reframed as "naive-credential-stuffing-only detector"; explicit "Defeated by" row for T1550.001 token-replay (the dominant 2024-2026 pattern). KL ↔ SG ↔ JB no-signal gap promoted from a note to a Trap-to-avoid. Compensating controls (Token Protection, CAE, sign-in-risk) named. Sourced from: cyber.
- **Policy 5** — Step 0 (confirm app state Enabled, not Discovered) added as Critical prerequisite. `*.mcas.ms` rewrite collision list expanded with `window.parent.postMessage` cross-origin failure and named SaaS (Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview, Atlassian Compass, Confluence whiteboard). Cert-pinning warning added. CAE-induced re-establishment caveat added. BYOD read-only fallback specified inline instead of TODO. Three-way audit-trail join (CloudAppEvents + SigninLogs + AuditLogs) given as KQL. Sourced from: mda-expert, qa, cyber, sysint.
- **Policy 6** — reframed as labelling precondition (not exfil control); explicit dependency-chain on Purview label existence + publishing + DCS-onboarded sites; Graph API rate-budget warning added; 100-character snippet treated as itself-regulated-content. Sourced from: cyber, sysint, qa.
- **Policy 7** — Purview eDiscovery preservation interaction with quarantine action explicitly flagged (spoliation risk); Legal-consult requirement added. Two-stage notify-then-quarantine flow tightened. Sourced from: qa.
- **Policy 8** — MDE Network Protection block-mode prerequisite promoted from buried mention to Critical prerequisite. "1709+" line removed (anachronistic). Risk score polarity stated explicitly (1-10 where lower = higher risk). GenAI sub-categories used instead of monolithic category. GSA + Edge for Business pointers added as 2026 supersession alternatives. SWG block-script partial-fail named. Sourced from: mda-expert, qa, cyber.
- **Policy 9** — ATT&CK mapping corrected (T1567 clipboard sub-channel, NOT T1052). Edge for Business `gen-ai.app-control` (H2 2026) added as alternative for Edge-on-Windows estates. OpenAI SSO-by-plan caveat tightened with date-stamp requirement. Sentinel ingest cost estimate added (~100GB/mo at 5k seats). Sourced from: cyber, mda-expert, sysint.
- **Policy 10** — three sub-cases added that the built-in policy doesn't cover: (a) service-principal credentials / PATs added pre-termination; (b) MFA-method additions pre-termination (T1556); (c) accounts created by the terminated user in connected SaaS (T1136). Email-match coverage rate stated honestly ("0% if identifier doesn't match exactly"). Sourced from: cyber, mda-expert.

### Removed from v0

- Dead cross-references to "Policy #11 (malware on Box/Dropbox/Google)" and "Policy #12 (terminated-user activity)" in the v0 "What it does NOT cover" list — those policy numbers didn't exist. Removed and replaced with correct cross-references. Sourced from: docs.
- "Step-up auth" listed as a current action in v0 Policy mitigations — downgraded to preview-only with explicit "do not list as production action". Sourced from: mda-expert.
- "Source: Microsoft Learn, session-policy docs" inline citation in the original "no watermarking" paragraph — removed because it had no URL. Citation discipline now applied consistently (or deferred to the refs.md companion). Sourced from: qa.
- "Defender for Cloud (CNAPP)" recommendation as the compensating control for application-permission OAuth grants — was misleading (Defender for Cloud is the CNAPP for cloud infra, not the OAuth answer). Replaced with: Entra Workload Identities + Conditional Access for Workload Identities, App Governance add-on, Entra ID Governance access reviews. Sourced from: qa, cyber.
- "Microsoft Edge for Business with isolation profiles" — Application Guard is deprecated; replaced with a date-stamped reference to current Edge isolation story (verify on day of promotion). Sourced from: qa.
- "Correlate by session correlation ID" SIEM guidance in the v0 footer — wrong (the field doesn't exist by that name). Replaced with the explicit three-record join KQL in Appendix B.6. Sourced from: sysint.

## Outstanding (NOT applied in v1 — flagged for v2 or separate deliverable)

- **Full clause mapping** — BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS. Deferred to `06-compliance/microsoft-defender-for-cloud-apps-control-mapping.md`.
- **`refs.md` companion file** — keyed by claim-id with Microsoft Learn URL + accessed date + quoted phrase per claim. Mentioned but not created in this pass.
- **Third-party attestation citations** with version qualifiers (SOC 2 Type II, ISO 27001:2022, ISO 27017, ISO 27018, CSA STAR Level 2, FedRAMP Moderate vs High, IRAP PROTECTED, MTCS Level, OSPAR report year) — pulls from Service Trust Portal still required.
- **Sub-processor list URL pull** — ISO 27018 A.11 requires; separate Trust Center artefact.
- **DPIA / TIA scaffolding for CAAC reverse-proxy and DSPM-for-AI prompt capture** — referenced but the full document is separate.
- **2026 Defender portal-path verification sweep** — every console path in v1 was checked against best-current knowledge but a full re-verify against `security.microsoft.com` UI is required before promotion.
- **Quantitative DCS-latency benchmarks** — vendor doesn't publish; not feasible to add.
- **Independent FP/FN measurement on Microsoft risk score** — not feasible without internal pilot data.
- **Independent M365 connector latency measurement** — Microsoft doesn't publish SLA; tenant-specific.

## Net change

| | v0 | v1 |
|---|---|---|
| Line count | 242 | ~700 |
| Numbered policies | 10 | 13 (split + 3 new) |
| Tables/matrices | 4 | 11 |
| Appendices | 0 | 7 |
| KQL queries | 0 | 6 (in Appendix B) |
| ATT&CK technique references | 0 | 25+ |

v1 is substantially expanded. The change is not "polish" — it is incorporating five specialists' critical findings on a v0 that they unanimously rated NO-WITH-CRITICAL-FIXES.
