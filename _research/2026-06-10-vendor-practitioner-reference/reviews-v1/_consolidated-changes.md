# Consolidated changes - playbook v0 -> v1

Source: five specialist reviews under `reviews-v1/` (qa.md, cyber.md, mda-expert.md, sysint.md, docs.md). All five returned **NO-WITH-CRITICAL-FIXES**. This document consolidates every finding into edits the v1 must apply, deferred items, and repo-wide recommendations. Where multiple reviewers raised the same point it is marked `(consensus - N specialists)` to indicate priority.

Reviewer key: **qa** (QA / verifiability), **cyber** (threat / risk + MITRE), **mda-expert** (deep MDA product), **sysint** (system integration / platform), **docs** (technical documentation).

---

## Cross-cutting changes (apply once, affects whole document)

- **(consensus - 5 specialists) Citation discipline is below the user's standard.** Every numeric / dated / Microsoft-named claim must carry a Microsoft Learn URL + accessed date, or be tagged as practitioner-inference / vendor-claim. Either inline the URLs or create a companion `microsoft-defender-for-cloud-apps.refs.md` keyed by claim-id with URL + accessed date + quoted phrase. Sourced from: qa (#1, #2, #3, #5, fix list), mda-expert (verify section), cyber (line 60 / line 158 verify), sysint (line 230, line 105 verify), docs (verify section).
- **(consensus - 5 specialists) Re-verify every Microsoft-stated number for 2026.** Specific items: catalogue size (31,000 vs 34,000), GenAI count (1,000+), connector latencies (<15 min M365; 24-72hr Power BI / Dynamics 365), `CloudAppEvents` retention (30 vs 90+90 days with XDR data lake), file-policy ceiling (50 raised to 200 in Feb 2026), tenant primary-data regions (Japan East added 2025 H2, India H2 2026 roadmap), MDE Network Protection supported-OS floor (Win10 1709 stale), OpenAI ChatGPT SSO-by-plan, OCAS deprecation date. Sourced from: qa (#1, #2, #3, #4, #6, #7, #16, #18, #21), mda-expert (#8, #9, #15, #19 verify; line 161 polarity), docs (verify section).
- **(consensus - 3 specialists) Portal-path stale to 2026 unified Defender portal.** The "Cloud apps -> Policies -> Policy management" left-nav and the Threat-detections/Information-protection/Conditional-Access/Shadow-IT tab layout do not exist in `security.microsoft.com` in 2026 H1. Re-verify every console path against the current unified portal; cite one canonical Microsoft Learn page that documents the portal taxonomy and date-stamp it. Sourced from: mda-expert (#1, fix Policy 1 / line 68), qa (#22), docs (#14 - consistency of `security.microsoft.com` prefix).
- **(consensus - 4 specialists) Add MITRE ATT&CK + ATLAS coverage to every policy.** Every policy table grows a row "ATT&CK technique(s) countered" with TID + Prevent/Detect/Respond verdict. Add an ATT&CK coverage matrix (T1078.004, T1098.001, T1136.003, T1528, T1530, T1550.001, T1556, T1567 / T1567.002, T1199, T1213 / T1213.002, T1485, T1486, T1557, T1574, T1090.002, T1568, T1074.002, T1537, T1027) and an ATLAS table (AML.T0010, T0024, T0040, T0051, T0052). Sourced from: cyber (#1, #7, #15, fix list, ADD section), qa (#10 - NIST CSF citations must be verbatim if used), mda-expert (drop the half-mapping in policy tables), docs (#11 - audit-evidence consistency).
- **(consensus - 3 specialists) Field schema across policy tables is inconsistent.** Lock a constant row set: **Risk reduced | What it does | ATT&CK technique(s) | Critical prerequisite | Console path | Key configuration | Trap to avoid (promoted to WARNING blockquote) | Defeated by / Out of scope | Auditable evidence (Table | Retention | Field | Correlator) | SIEM artefact and connector**. Use N/A where empty. Apply to Policies 1-10 + any net-new policies. Sourced from: docs (#1, #11, fix list line "LINES 64-105"), qa (ADD - claim-classification footnote per policy; ADD - regression-test matrix; ADD - explicit out-of-scope per policy), sysint (#3 - SIEM artefact row per policy).
- **(consensus - 2 specialists) Audit-evidence rows must name the schema field, not just the table.** State the table + retention + field carrying decision / policy version / subject / object / device-tag, and whether Sentinel forwarding preserves policy-version. Sourced from: qa (ADD - "verifiable evidence" subsection per policy), docs (#11 - fixed evidence schema).
- **(consensus - 2 specialists) Add a glossary / first-use expansion.** Expand on first use: CAAC, DCS (Data Classification Service), SIT (Sensitive Information Type), SCIM, CNAPP, SSPM, DfE / DfI / DfO, DPIA, RBI, DSPM, CAE, OCAS, MIP, EDM, MAM, BYOK, CSF, RMiT, TRM. Build a glossary section at end or a `_glossary.md` companion. Sourced from: docs (#6, ADD section), sysint (implicit via SCIM detail).
- **British English + capitalisation hygiene.** Standardise "OneDrive for Business" (first occurrence in each section) -> "OneDrive" thereafter; expand "generative AI" on first use -> "GenAI" thereafter; "Day 1" in headings, "on day one" in body; sensitivity label lower-case for generic, capitalise specific label names. Sourced from: docs (#13).
- **TOC + anchor links throughout.** Add a top-of-doc TOC with anchors to each policy + section. Convert "see Day 90 below" / "see Policy 8" prose references to anchor links. Cross-link Residual-risks rows back to the Day 1/30/90 policy they complement, and the NOT-cover list to the Day 90 GenAI block. Sourced from: docs (#3, #16, fix list).
- **Version pin + changelog.** Add a `CHANGELOG.md` or top-of-doc changelog block plus an explicit "this playbook was written against MDA / Defender XDR portal as documented at `<accessed-date>`; re-validate after Defender portal redesign / Purview DCS schema change / Entra CA schema change / MDE NP update / `CloudAppEvents` schema change." Sourced from: qa (ADD - version pin), docs (ADD - changelog), mda-expert (ADD - schema-versioning practice note).
- **Reframe three-lens sign-off to "ADDRESSED IN V0 - RE-REVIEW PENDING".** The "INCORPORATED" verdicts contradict the source lens reviews' explicit NO verdicts. Replace workflow IDs with relative links to underlying lens-review files; add a version column; separate "captured" from "deferred to separate deliverable". Sourced from: qa (#24), docs (#12).

---

## Section-by-section change list

### Front matter / "Read this if" block (lines 1-9)

- [CHANGE] Add an explicit audience definition: **"Read this if you are"** (security architect / platform engineer at a regulated FI) plus **"Skip this if"** (vendor evaluation, RFP scoring, license-cost modelling). Sourced from: docs (#4). Rationale: "your firm already has MDA" is the licensing pre-test, not the audience.
- [CHANGE] Move "What to do next" practitioner checklist (currently line 221) immediately under "Read this if" as the pre-flight check. Sourced from: docs (#17, fix list). Rationale: nothing in the body is actionable until licence / P1 / MDE / region verifications pass.
- [CHANGE] Move "What this playbook does NOT cover (out of scope)" block (currently line 211) immediately under "Read this if". Sourced from: docs (#20, fix list). Rationale: it is document scope, belongs early.
- [CHANGE] Add a "How to read this playbook" 3-line sub-section. Sourced from: docs (ADD - How to read). Rationale: consumption order is non-obvious for a 240-line doc.
- [CHANGE] Add a 1-page summary card at the top: licensing required, dependencies, CAAC two-console gotcha, 10-policy index. Sourced from: docs (ADD - summary card). Rationale: 3am readers may not need to read further.
- [CHANGE] Add a TOC with anchor links to every policy and section. Sourced from: docs (#3, fix list). Rationale: 243 lines, no nav.

### "What MDA is for" (line 11) - opening paragraph

- [CHANGE] Break the 113-word opening sentence (line 13) into 4 bullets: API governance for sanctioned SaaS / Shadow IT discovery from egress logs / OAuth grant surveillance and revoke / In-session DLP via CAAC (Entra-dependent). Sourced from: docs (#7, fix list). Rationale: unparseable run-on for a 3am reader.
- [CHANGE] Line 15 - rewrite the CAAC description to name: (a) per-user first-touch discovery requirement to flip apps from `Discovered` to `Enabled` on the CAAC apps page, (b) the cert chain presented (`*.mcas.ms` on DigiCert G2 in 2026), (c) the `postMessage` / `window.location` rewrite failure modes. Sourced from: mda-expert (#3, #5, fix Intro / line 15). Rationale: the current description hides the most common 2026 onboarding gates.
- [CHANGE] Line 15 - strip "about half of what the marketing implies" or back it with a feature-count breakdown. Sourced from: docs (verify line 15). Rationale: hand-waving quantification.

### "What it covers well" (line 17)

- [CHANGE] Line 19 - strip "first-class" -> "fully supported". Sourced from: docs (#15). Rationale: vendor-y phrase.
- [CHANGE] Line 20 - "~31,000-app catalogue" -> "Microsoft asserts 31,000 apps catalogued; independent verification not available" with Microsoft Learn cite + accessed date OR strip the number. (consensus - 3 specialists qa #1, mda-expert verify, docs verify) Rationale: vendor-self-reported numeric with no source.
- [CHANGE] Line 22 - "DRM-style controls" -> "sensitivity-label-applied encryption and permissioning". Sourced from: docs (#15). Rationale: DRM is vendor framing.
- [CHANGE] Line 23 - "categorical visibility into 1,000+ generative-AI apps" -> "GenAI-category discovery for apps already in the catalogue (Microsoft asserts >1,000; not independently verified)". (consensus - 3 specialists qa #2, mda-expert verify, docs verify) Rationale: vendor count, no cite.
- [CHANGE] Line 23 - **rewrite DSPM-for-AI prompt-capture depth.** Deep capture (prompt + response + classification) is ChatGPT Enterprise / Copilot Chat / Copilot Studio only. Shallow capture (metadata only, no body) covers Gemini for Workspace, Anthropic Claude (claude.ai web), consumer Copilot, Perplexity Enterprise. State Purview IRM licensing prereq + federated-SSO-from-same-tenant prereq. Sourced from: mda-expert (#7, verify line 23). Rationale: current sentence materially overstates Gemini / Claude coverage and the BFSI buying argument hinges on this.
- [CHANGE] Line 24 - stop calling these policies "UEBA". They are mostly threshold + rule-based with a 7-day learning window. Sourced from: cyber (verify line 24). Rationale: UEBA framing implies behavioural baselining the product does not do.

### "What it does NOT cover" (line 26)

- [CHANGE] **Line 30 (item 3) - delete the dead cross-references to "Policy #11 (malware on Box/Dropbox/Google)" and "Policy #12 (terminated-user activity)".** Those policy numbers do not exist in this playbook. Re-reference correctly or remove. Sourced from: docs (#16, fix list LINE 30). Rationale: stale internal cross-reference is the worst kind of broken link.
- [CHANGE] Line 30 - "<15 min M365 connector latency" needs a Microsoft Learn cite or downgrade to "near-real-time per Microsoft; independent measurement not published". (consensus - 3 specialists qa #3, mda-expert verify, docs verify line 30) Rationale: numeric SLA-style claim with no source.
- [CHANGE] Line 30 (item 3) - "Power BI / Dynamics 365 24-72 hours" needs cite; the number has shifted in 2025 docs. Sourced from: qa (verify), mda-expert (verify line 30), docs (verify). Rationale: same as above.
- [CHANGE] Line 38 (item 11) - flag OCAS as a **sunsetting** SKU. Microsoft announced deprecation late 2025; verify end-of-support date against current `editions-cloud-app-security-o365` docs. Migration runway matters more than the trap. Cross-reference the verify-licensing checklist item. Sourced from: mda-expert (#15, fix INTRO / line 38), docs (#16 OCAS cross-ref). Rationale: 2026 reality is sunset, not just trap.
- [CHANGE] **Line 39 - tenant region claim is wrong for 2026.** Japan East was added 2025 H2; India is on roadmap H2 2026. Rewrite "no APAC primary region as of writing" -> verify against Trust Center, name Japan East. (consensus - 3 specialists qa #4, mda-expert #20, docs verify line 39) Rationale: most consequential unverified claim in the playbook because it affects buy-no-buy for the user's primary geography.
- [CHANGE] Line 32 - "Source: Microsoft Learn, session-policy docs" is the only inline cite in the body and it has no URL. Either convert to URL + accessed date (and adopt the same format throughout) or remove. Sourced from: qa (#5). Rationale: inconsistent citation hygiene.
- [CHANGE] Convert the 12-item numbered NOT-cover list to a `Gap | Why it matters | Compensating control | Cross-ref to Day-X policy` table, merging with the Residual-risks table at line 194 to eliminate the ~60% content overlap. Sourced from: docs (#8, #19, fix LINES 194-207). Rationale: practitioner reads the same gap twice in two formats today.

### "What it touches" dependencies table (line 41)

- [CHANGE] For the Sentinel / Log Analytics row, name the actual connector path: **Microsoft Defender XDR data connector** (canonical for MDA tables - `CloudAppEvents`, `AlertInfo`, `AlertEvidence`); NOT Office 365 connector, NOT Diagnostic Settings on Entra alone. State the dual-bill risk: Sentinel ingest on top of Defender XDR retention. Sourced from: sysint (#2, fix "What it touches"). Rationale: three paths and they are not equivalent; SOC team picks the wrong one and double-bills or gets incomplete data.
- [CHANGE] Update the Activities log retention line - "default 30-day" is wrong for tenants on the unified XDR data lake (90 days hot + 90 days warm = 180 days). State: "30 days standard; 90+90 days with the Defender XDR unified data lake (GA 2025); Sentinel forwarding for 1-7 year retention; subscribe to schema-change blog for query stability." (consensus - 2 specialists qa #6, mda-expert #8) Rationale: reconciles the architect-lens 180-day contradiction qa #6 surfaced and updates for 2026 reality.
- [CHANGE] Add a third row to the Conditional Access dependency: the **Entra app registration / enterprise app** is a third object that can drift independently for non-M365 CAAC'd apps. Sourced from: sysint (verify line 50). Rationale: "two consoles, two policy objects" is understated.

### Concentration risk (line 52)

- [CHANGE] Promote out of the buried H3 inside Dependencies to a top-level callout, or to a row in the Dependencies table. Sourced from: docs (#9, #18, fix LINE 52). Rationale: one of the most important architectural points in the doc, currently in a place nobody reads.
- [CHANGE] Reframe in technique terms: CAAC proxy compromise = **T1557 Adversary-in-the-Middle at internet-scale**; single Entra identity-plane compromise = **T1098.001 / T1556 / T1528** across the entire Microsoft stack simultaneously. Sourced from: cyber (#12, fix CONCENTRATION RISK). Rationale: current vendor-count framing is too soft.

### Day 1 phase opener (line 58)

- [CHANGE] Add a 3-bullet **Prerequisites checklist** at the top of Day 1: licensing required, dependencies that must be live, exclusion lists / break-glass configured. Sourced from: docs (#10, fix LINES 60). Rationale: practitioner currently only finds out at Policy 5 they need Entra P1.
- [CHANGE] Add a policy index + dependency matrix table at the top of Day 1 (and again at Day 30 / Day 90) showing per-policy: requires CAAC Y/N, Entra P1 Y/N, MDE Y/N, Purview labels Y/N, action class, blast-radius rating. Sourced from: docs (ADD - policy index + dependency matrix). Rationale: buries "all Day 30 policies require CAAC" in a sentence today.

### Policy 1 - Shadow IT discovery + GenAI category triage (line 62)

- [CHANGE] Reframe from "Risk reduced" to "Visibility precondition for". Discovery without enforcement does not reduce risk; it enables a control decision. Sourced from: cyber (#1, fix POLICY 1). Rationale: current framing is vendor marketing.
- [CHANGE] Add ATT&CK row: visibility precondition for T1567 Exfiltration Over Web Service and T1102 Web Service C2 candidates. Sourced from: cyber (#1). Rationale: per cross-cutting ATT&CK rule.
- [CHANGE] Console path - update to 2026 unified portal path (`security.microsoft.com` -> System -> Settings -> Cloud apps -> Cloud discovery, or direct route `security.microsoft.com/cloud-discovery`). Sourced from: mda-expert (#1, fix POLICY 1 / line 68). Rationale: stale path.
- [CHANGE] Update `CloudAppEvents` retention text: "30 days standard; 90+90 days with unified XDR data lake; Sentinel-forwarded for 1-7 year; subscribe to schema-change blog for query stability." Sourced from: mda-expert (fix POLICY 1 / line 71), qa (#6). Rationale: 2026 reality.
- [CHANGE] Cite the Microsoft Learn page for the "discovered subdomains capability deprecated 2025-12-31" claim. Phrase as "deprecated 2025-12-31 - already in effect as of writing" (today is 2026-06-10). (consensus - 2 specialists qa #7, docs fix LINE 70 + verify) Rationale: future-looking phrasing implies action still required.
- [CHANGE] Verify whether the "weekly discovery report" is signed / tamper-evident. If it is a CSV with no integrity claim, it is not an audit artefact - state so. Sourced from: sysint (verify line 70). Rationale: audit-evidence framing.

### Policy 2 - OAuth app discovery + ban known-malicious (line 73)

- [CHANGE] **Reposition as a "post-consent cleanup" control, not a primary defence.** State that the primary control for T1528 is Entra admin-consent workflow + verified-publisher restriction (configured in Entra, not MDA). Sourced from: cyber (#2, fix POLICY 2 reposition). Rationale: line 77 currently asserts MDA-OAuth-ban as the primary control when it is post-consent.
- [CHANGE] Add ATT&CK row: T1528 Steal Application Access Token (Detect+Respond), T1550.001 Application Access Token (Detect), post-consent T1567 / T1071.001 (Detect). Sourced from: cyber (#1).
- [CHANGE] **Split into Policy 2a (classic OAuth apps page - consent-event metadata fidelity) and Policy 2b (App Governance Predictive Risk + OAuth threshold policies - behavioural post-consent fidelity).** State explicitly: enabling 2a without 2b leaves the post-consent behavioural class uncovered. Sourced from: mda-expert (#2, fix POLICY 2). Rationale: the two surfaces diverged in 2024 and current playbook conflates them.
- [CHANGE] Line 77 - "bypasses every network-layer control" is true for the exfil leg only; false for the consent screen (browser-rendered, inspectable). Tighten. Sourced from: cyber (verify line 77), qa (cyber framing). Rationale: overstatement.
- [CHANGE] Line 82 - **promote the Application (client-credentials) gap from a footnote to top of policy table.** State T1098.001 + T1136.003 service-principal compromise is the dominant cloud-persistence pattern; Policy 2 covers ~30% of OAuth-grant abuse, service-principal abuse is the other ~70%. Sourced from: cyber (#3, fix POLICY 2). Rationale: a practitioner reading v0 thinks OAuth is solved. It is not.
- [CHANGE] Line 82 - add Power Platform service-principal sub-class as a specific under-governed surface; direct to Power Platform Admin Center DLP. (consensus - 2 specialists mda-expert #17, sysint implicit). Rationale: heavy Power Platform tenants have hundreds of these.
- [CHANGE] Line 81 - "next token refresh" specificity is uncited; reframe as "immediate, propagating on next token validation; CAE-aware apps may revoke faster" with a CAE doc link. Sourced from: qa (#8). Rationale: practitioner-inferred specificity.
- [CHANGE] Verify line 83 - the claim that Application grants are "NOT in this view" is true for the classic OAuth apps page but partly false for App Governance (which does surface some application-permission anomalies for M365 graph-scope app-only tokens). Refine. Sourced from: mda-expert (verify line 83). Rationale: precision.

### Policy 3 - Mass-download anomaly, alert not auto-suspend (line 85)

- [CHANGE] Add ATT&CK row: T1530 Data from Cloud Storage Object + T1213.002 SharePoint + T1567.002 (Detect only, post-event). Sourced from: cyber (#1).
- [CHANGE] **Line 92 - reframe "count > 5,000 in 30 minutes" from default to starting threshold for pilot.** Provide a baselining method (P95 of legitimate users over a 30-day window) and a tightening cadence. Sourced from: qa (#9), docs (verify line 132 pattern). Rationale: silently adopted as default but it is an upper-bound pilot suggestion.
- [CHANGE] Add **"Does NOT catch slow-and-low exfil"** (100 files/day x 30 days defeats this threshold). Provide a Sentinel KQL aggregating per-user weekly download volume against a 90-day baseline. Sourced from: cyber (#4, fix POLICY 3 mass-download). Rationale: defeats the dominant patient-attacker pattern.
- [CHANGE] Add **"Does NOT catch native-sync staging (T1074.002 via OneDrive sync)"** as a separate detection requirement. Sourced from: cyber (#9). Rationale: sync client + selective sync = bulk staging without download events of the expected class.
- [CHANGE] Line 93 - expand the "Suspend user" trap with REST-layer detail: (a) underlying call `POST graph.microsoft.com/v1.0/users/{id} accountEnabled=false` runs under the Defender service principal, analyst attribution requires `CorrelationId` join; (b) **silently fails** on hybrid-AD users with `onPremisesSyncEnabled=true` and no writeback. Validate `IdentityInfo` table that `AccountEnabled` actually flipped. Sourced from: mda-expert (#6, fix POLICY 3). Rationale: practitioners auto-suspend a compromised hybrid-AD user and the attacker keeps the session.
- [CHANGE] Line 93 - add the SCIM cascade table: which apps see the disable automatically (Entra-provisioned apps via Enterprise Applications, 40-min default), which need manual intervention (Okta-provisioned, non-federated, local accounts). Sourced from: sysint (#5, fix POLICY 3 / 10). Rationale: single biggest operational unknown at deployment.
- [CHANGE] Line 94 - NIST CSF citation: either quote DE.CM-03 verbatim from CSF 2.0 or strip the parenthetical paraphrase. (consensus - 2 specialists qa #10, sysint verify line 188). Rationale: paraphrase is wrong in a compliance-traceability doc.

### Policy 4 - Impossible travel + activity from infrequent country (line 96)

- [CHANGE] Add ATT&CK row: T1078.004 Cloud Accounts (Detect, low-fidelity, naive-credential-stuffing only). Sourced from: cyber (#1, #6, #14).
- [CHANGE] Add **"Defeated by"** row: T1550.001 Application Access Token replay; residential-proxy infrastructure placing attacker IP near user. Compensating controls = Entra Token Protection (verify GA status) + Continuous Access Evaluation + sign-in-risk policies. State **"MDA's impossible-travel is naive-credential-stuffing-only"**. Sourced from: cyber (#6, #14, fix POLICY 4). Rationale: Day 1 set has zero defence against the dominant 2024-2026 attack pattern.
- [CHANGE] Line 100 - "compromised-account" framing is anachronistic; modern intrusions are T1566.002 -> T1528 -> T1550.001 (token replay from attacker infra produces no impossible-travel signal). State the gap. Sourced from: cyber (#14, verify line 100). Rationale: as above.
- [CHANGE] Line 105 - VPN-detection enrichment source: verify whether the policy *signals* the SOC when its enrichment source is unavailable. Sourced from: sysint (verify line 105). Rationale: silent enrichment outage = silent degradation.

### Day 30 phase opener (line 109)

- [CHANGE] Add prerequisites checklist (per cross-cutting). Sourced from: docs (#10).
- [CHANGE] Add per-app CAAC onboarding **effort estimate**: standard M365 / Salesforce / Box / Workday on Entra = 1-3 days incl. regression test; Okta-fronted SaaS requiring re-federation = 2-4 weeks; cert-pinned native app = unsupported, drop from scope; B2B-guest CAAC = case-by-case. Sourced from: sysint (#6, fix Day 30 / Day 90 CAAC sections). Rationale: playbook tells practitioners to "budget for regression testing" but does not size it.

### Policy 5 - Block download of sensitivity-labelled files to unmanaged browsers (line 113)

- [CHANGE] Add ATT&CK row: T1530 via untrusted endpoint (Prevent for download leg; out-of-scope for token-replay or managed-endpoint exfil). Sourced from: cyber (#1, #8).
- [CHANGE] **Add Step 0**: confirm app state = `Enabled` (not `Discovered`) on **Settings -> Cloud apps -> Conditional Access App Control -> Connected apps -> Conditional Access App Control apps**. Until the row exists per-user-first-touch, the session policy literally cannot match. Sourced from: mda-expert (#3, fix POLICY 5 / line 119). Rationale: practitioners pilot 10 users, ship to 10,000, find the toggle never flipped.
- [CHANGE] Line 119 - state the **inverse trap**: Entra CA "Use Conditional Access App Control" without an MDA session policy produces a redirect to a no-op proxy that adds `*.mcas.ms` rewrite but does nothing visible. Sourced from: qa (#11). Rationale: the trap is bilateral.
- [CHANGE] Line 121 - expand the `*.mcas.ms` rewrite collision list with **`window.parent.postMessage` cross-origin failure** as the dominant 2026 CAAC breakage class. Name affected SaaS: Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview, Atlassian Compass, Confluence whiteboard. Sourced from: mda-expert (#4, fix POLICY 5 / line 121). Rationale: undocumented by Microsoft, hits silently.
- [CHANGE] Line 121 - add cert-chain warning: managed endpoints with strict cert-pinning device profiles may reject the `*.mcas.ms` DigiCert G2 chain at TLS; policy fails-closed but looks fail-open at app layer. Sourced from: mda-expert (#5, fix POLICY 5 / line 121). Rationale: pin/un-pin test required pre-rollout.
- [CHANGE] Line 122 - **BYOD note specifies a parallel policy with no parameters.** Either spec the read-only access policy (action, client app, device filter, exact session control) or state "spec'd in Policy X". Sourced from: qa (#12). Rationale: hand-wave.
- [CHANGE] Line 123 - add CAE-induced re-establishment caveat: when CAE revokes mid-session, re-established session may bypass MDA policy without re-evaluation. Sourced from: mda-expert (#14, fix POLICY 5 / line 123). Rationale: documented as expected behaviour by MS, practitioners consider it a control gap.
- [CHANGE] Add **Token Replay row**: bypassed by T1550.001 when attacker possesses a valid refresh token issued on a managed device; pair with Entra Token Protection. Sourced from: cyber (#8, fix POLICY 5). Rationale: stealthy replay bypass.
- [CHANGE] State `always_apply_if_unscannable: false` decision in ATT&CK terms: fails open on T1027 (encrypted-by-user archives); attacker 7zips payload with password = control defeated. Sourced from: cyber (fix POLICY 5). Rationale: residual must be documented.

### Policy 6 - Auto-label PCI in OneDrive / SharePoint (line 125)

- [CHANGE] **Reframe: not an exfil control.** It is a labelling precondition for downstream encryption AND for Policy 5's `Sensitivity label ∈ {Confidential}` filter to fire. Without Policy 6, Policy 5's filter matches nothing. State the dependency chain. Sourced from: cyber (#1, fix POLICY 6). Rationale: misframed in v0.
- [CHANGE] Add ATT&CK row: precondition for label-encryption controls that frustrate T1565.001 and T1486; not a direct counter. Sourced from: cyber (#1).
- [CHANGE] Add Graph API rate-budget line: file policies pull through Graph; at tenant-wide scope they saturate (~600 req/min/tenant SharePoint Online bucket); MDA backs off to partial coverage without operator notification. Scope by parent folder on rollout. Sourced from: sysint (#4, fix Policy 2 / Policy 7). Rationale: BFSI-scale failure mode.
- [CHANGE] State Purview label dependency explicitly: the Confidential - Finance label must already exist in Purview with encryption configured, label-publishing scope including affected users, and DCS-onboarded sites - otherwise "Apply label" silently fails. Cite Purview auto-labelling prereqs doc. Sourced from: qa (Specific fixes line 131). Rationale: silent-fail prerequisite.
- [CHANGE] Line 132 - "minimum violations = 5 (not 1 - too many FPs)" - state as recommended starting heuristic to tune, or back with baseline measurement. Sourced from: docs (verify line 132). Rationale: unmeasured.
- [CHANGE] Verify line 130 / 132 - the "100-character before/after window" snippet spec must be exact (it is regulated content). Cite Purview DCS docs and DLP incident-management docs. Sourced from: qa (verify line 130, 132). Rationale: regulated-content surface.
- [CHANGE] **Line 134 - file-policy ceiling raised from 50 to 200 per tenant (verify against current `data-protection-policies` docs).** Adjust consolidation guidance. (consensus - 2 specialists mda-expert #9, qa #13). Rationale: 50-cap is stale.

### Policy 7 - Quarantine externally-shared OneDrive files with no recent activity (line 138)

- [CHANGE] Add ATT&CK row: T1213 Data from Information Repositories (Detect + clean-up, long-tail). Sourced from: cyber (#1).
- [CHANGE] Add Graph API rate-budget line (same as Policy 6). Sourced from: sysint (#4).
- [CHANGE] Line 145 - cite Purview eDiscovery preservation doc + Defender for Cloud Apps governance-action interaction doc; or downgrade the "spoliation" claim to "consult Legal before policy design - Purview eDiscovery holds may override MDA quarantine actions". Sourced from: qa (#14). Rationale: Legal will demand primary source.

### Day 90 phase opener (line 150)

- [CHANGE] Add prerequisites checklist (per cross-cutting). Sourced from: docs (#10).
- [CHANGE] Add a "platforms that supersede MDA controls in 2026" pointer: Microsoft Edge for Business `gen-ai.app-control` (H2 2026 GA), GSA Internet Access native AI-category enforcement, Purview Adaptive Protection for GenAI (Q1 2026 GA). MDA controls are *one* answer of three in 2026. Sourced from: mda-expert (#13, ADD - platforms-that-supersede). Rationale: avoids practitioners building MDA-only when GSA / Edge are simpler.

### Policy 8 - Auto-unsanction GenAI above risk threshold (line 154)

- [CHANGE] Add ATT&CK row: T1567 Exfiltration Over Web Service to AI endpoint (Detect on Discovery; Prevent only via MDE NP block). Sourced from: cyber (#1).
- [CHANGE] **Add "Defeated by" row**: BYOD mobile / hotspot; personal MSA or personal Google sign-in to ChatGPT (Entra never sees session); CDN-fronted vendor domains (T1090.002, T1568.002, T1102 - SWG block script "partially fails" means technique class defeated); image / audio bypass of clipboard inspection (DCS does not OCR / transcribe). Sourced from: cyber (#5, fix POLICY 8). Rationale: thin enforcement chain dressed as control.
- [CHANGE] **Add P0 control-design requirement**: the approved-AI allowlist is a third-party concentration-risk decision, must have documented review cadence (quarterly minimum), kill-switch posture, supply-chain-compromise contingency (AML.T0010). Sourced from: cyber (#7, fix POLICY 8). Rationale: currently treated as configuration; it is governance.
- [CHANGE] Line 161 - **"Generative AI" was split into sub-categories** (`LLM`, `AI Coding Assistant`, `AI Image Generator`, `AI Video Generator`, `AI Voice/Avatar`) in 2025 H2. Use sub-category filter or accept expanded blast radius. Sourced from: mda-expert (#16, fix POLICY 8 / line 161). Rationale: matches noisier set now.
- [CHANGE] Line 161 - state risk-score polarity explicitly. MDA risk score is **1-10 where lower = higher risk**; "≤ 5" reads ambiguously. (consensus - 2 specialists mda-expert verify line 161, docs verify line 161). Rationale: ambiguity.
- [CHANGE] Line 162 - **replace "Windows 10 1709+ or supported macOS builds" with current Microsoft Learn Network Protection supported-OS list + accessed date.** (consensus - 2 specialists qa #16, docs verify line 162). Rationale: anachronistic; 1709 reached non-LTSC EoS Oct 2019.
- [CHANGE] Line 162 - add: if licensed for GSA Internet Access, prefer GSA's native Generative AI category block over MDA -> MDE NP chain (fewer dependencies, no audit-mode footgun). Sourced from: mda-expert (fix POLICY 8 / line 162). Rationale: 2026 reality.
- [CHANGE] Line 163 - add Microsoft Edge for Business `gen-ai.app-control` (H2 2026) as more reliable enforcement than MDE NP for Edge-on-Windows estates. Cite the integration is documented end-to-end in MS Learn. Sourced from: mda-expert (fix POLICY 8 / line 163), qa (verify #15 - cite Govern discovered apps + Network Protection block-mode docs). Rationale: 2026 alternative.
- [CHANGE] Line 164 - cite the catalogue page where "90+ factors" appears, or drop the count. State the alternative basis for risk acceptance (internal risk-acceptance committee + parallel approval workflow). (consensus - 2 specialists qa #17, mda-expert verify line 164). Rationale: vendor-claim + uncited factor count; guidance-with-no-out otherwise.
- [CHANGE] Line 165 - verify the event class name in `CloudAppEvents.ActionType` for "App tag changed" / "App marked as unsanctioned" - MS has renamed. Sourced from: sysint (verify line 165). Rationale: query stability.
- [CHANGE] Add SCIM-cascade-for-blocked-app note: users with pre-existing OpenAI personal accounts still auth via personal IdPs out-of-band; MDA block does not cover. Sourced from: sysint (fix POLICY 8). Rationale: already stated as "consumer ChatGPT bypasses" elsewhere but not here.

### Policy 9 - Block clipboard paste of sensitive data into ChatGPT (line 167)

- [CHANGE] Add ATT&CK row: **correct mapping** is T1567 Exfiltration Over Web Service narrowed to the clipboard sub-channel; **not** T1052 (Physical Medium). Sourced from: cyber (#1, Policy 9). Rationale: v0 implies T1052; that is the wrong mapping.
- [CHANGE] **Add "Defeated by" row**: image / screenshot / audio of the same data (DCS does not OCR / transcribe at session layer); typing instead of pasting (no paste event); desktop ChatGPT apps; personal account; non-Edge browsers without onboarding. Sourced from: cyber (#5, fix POLICY 9). Rationale: substantial residual.
- [CHANGE] Line 174 - cite OpenAI's "SSO availability by plan" page and date-stamp; phrase as "as of 2026-06". (consensus - 2 specialists qa #18, docs verify line 174). Rationale: OpenAI plan parity has moved before.
- [CHANGE] Line 176 - rewrite the DSPM-for-AI prompt-capture-depth claim consistent with the line 23 fix (deep capture = ChatGPT Enterprise / Copilot only; shallow = Gemini / Claude / Perplexity / consumer Copilot metadata only). Sourced from: mda-expert (#7, fix POLICY 9 / line 174 + 176). Rationale: cross-cutting consistency.
- [CHANGE] Line 176 - add: clipboard inspection at 5k seats inflates Sentinel ingest ~100GB/mo. Budget impact, not just privacy. Sourced from: sysint (fix POLICY 9). Rationale: cost line.
- [CHANGE] Reference Edge for Business `gen-ai.app-control` (H2 2026) as competing / complementing control. Sourced from: mda-expert (fix POLICY 9 / line 176).

### Policy 10 - Terminated-user activity in connected SaaS (line 179)

- [CHANGE] Add ATT&CK row: T1078.004 Cloud Accounts persistence; T1136 Create Account (where SaaS-local account was created); T1098.001 Additional Cloud Credentials (where user added a token before leaving). Sourced from: cyber (#1, #11).
- [CHANGE] **Add three new sub-cases** to the policy treatment: (a) service-principal secrets / PATs / API keys added before termination -> check Entra Audit `AppRoleAssignment` and `Add service principal credentials`; (b) MFA-method additions or backup-email changes (T1556) before termination -> Entra Audit `User registered alternative authentication device`; (c) accounts created by the terminated user in connected SaaS (T1136) -> per-SaaS admin audit, not in MDA. Reference a manual offboarding runbook. Sourced from: cyber (#11, fix POLICY 10). Rationale: reframe as "one of N offboarding-gap detections".
- [CHANGE] Line 187 - state the **coverage rate honestly**: "for SaaS apps where the email identifier does not match exactly, this control has 0% coverage." Sourced from: cyber (verify line 192). Rationale: practitioner needs honest framing of dependency on shared identifier.
- [CHANGE] Add SCIM cascade table (same as Policy 3 - shared offboarding mechanic). Sourced from: sysint (#5).
- [CHANGE] Line 188 - either drop the half-mapping NIST CSF / RMiT / MAS TRM hook or carry the full subcategory list. (consensus - 2 specialists qa #10, #19). Rationale: half-mapping is worse than either extreme.

### Residual risks table (line 192)

- [CHANGE] **Line 198 - replace "Microsoft Defender for Cloud (CNAPP)" with "Microsoft Entra Workload Identities + Conditional Access for Workload Identities; Defender for Cloud Apps App Governance add-on for service-principal hygiene; Microsoft Entra ID Governance for periodic application access reviews".** (consensus - 2 specialists qa #20, cyber #3 implicit). Rationale: Defender for Cloud is the CNAPP for cloud infra, not the OAuth-application-permission answer; misleads procurement.
- [CHANGE] Line 197 - rewrite cert-pinned compensating control. Intune compliance evaluates device state, not app-layer cert pinning. Real compensating control = **Intune App Protection Policies (MAM)** with copy-paste / save-to-personal-cloud restrictions inside the managed app. Sourced from: qa (#23). Rationale: conflates device compliance with app-traffic inspection.
- [CHANGE] Line 201 - "Microsoft Edge for Business with isolation profiles" - cite or strip. Application Guard is deprecated for new development; check current Edge isolation story. Sourced from: qa (Specific fixes line 201). Rationale: imprecise.
- [CHANGE] Line 202 - verify the SSPM vendor list (Adaptive Shield / AppOmni / Obsidian / Wing) for mergers / rebrands before publishing. Sourced from: qa (verify line 202). Rationale: M&A common in this space.
- [CHANGE] **Add row: Cached `mcas.ms` redirects survive policy removal.** Deprovisioning a CAAC onboarding cleanly requires four steps across two consoles + session-cookie purge; no documented Microsoft cache-TTL SLA. Sourced from: mda-expert (#19, ADD residual). Rationale: practitioners ship change tickets, users keep hitting mcas.ms for hours/days.
- [CHANGE] Add row: Power Platform service-principal consents under-surfaced in App Governance; compensating control is Power Platform Admin Center DLP. Sourced from: mda-expert (#17).
- [CHANGE] Add row: CAE-induced session re-establishment may bypass MDA session policy re-evaluation. Sourced from: mda-expert (#14).
- [CHANGE] Add row: Microsoft Edge for Business `gen-ai.app-control` (H2 2026) + GSA AI-category native block supersede the MDA -> MDE NP chain for Edge-on-Windows and GSA-licensed estates. Sourced from: mda-expert (#13).
- [CHANGE] Rewrite the entire table in ATT&CK technique terms in parallel to the current product-shape terms. Sourced from: cyber (ADD - residual-risks-in-ATT&CK-terms). Rationale: readers can map to their own coverage matrix.

### "What this playbook does NOT cover (out of scope)" (line 211)

- [CHANGE] Move to top of document under "Read this if" (per cross-cutting). Sourced from: docs (#20).
- [CHANGE] Line 213 - add version / scope qualifiers per third-party attestation (ISO 27017:**2015**, ISO 27018:**2019**, CSA STAR Level **2**, FedRAMP **Moderate** vs **High**, IRAP **PROTECTED**, MTCS Level, OSPAR report year). Sourced from: qa (#21). Rationale: practitioner pulling from Trust Center sees multiple FedRAMP entries.

### Practitioner checklist (line 221) and Three-lens sign-off (line 234)

- [CHANGE] Move the practitioner checklist to the top of the document (per cross-cutting). Sourced from: docs (#17).
- [CHANGE] Line 230 - **rewrite the SIEM-query guidance.** Replace "correlate by session correlation ID" (does not exist) with: "Join `CloudAppEvents` and `SigninLogs` on `AccountObjectId` = `UserId` within ±30s window on `TimeGenerated`, intersected on `IPAddress`. The audit trail is **three** records: Entra `SigninLogs` (auth event), Entra `AuditLogs` (CA policy result), Defender XDR `CloudAppEvents` (session-policy match). All three required for evidence." Sourced from: sysint (#1, fix item 8). Rationale: wrong-by-omission; the join the playbook tells the practitioner to write produces an empty result set.
- [CHANGE] Line 240 - rewrite "INCORPORATED" verdicts to "ADDRESSED IN V0 - RE-REVIEW PENDING"; replace workflow ID with relative links to lens-review files; add version column; separate "captured" from "deferred". (consensus - 2 specialists qa #24, docs #12). Rationale: contradicts source lens reviews' explicit NO verdicts.

---

## Net-new content the v1 must add

**Day 0 precondition checklist (separate from Day 1).** A new section before Day 1 covering: tenant primary region pulled from M365 admin centre, Entra ID P1 confirmed for CAAC scope, MDE deployment + Network Protection mode (audit vs block), Purview labels published before any auto-label policy, Conditional Access policy backup exported, break-glass excluded accounts configured (mandatory - the "Session Controls enterprise application" gotcha can lock the tenant out), `CloudAppEvents` schema version pinned in SIEM queries. Sourced from: mda-expert (ADD - Day 0 precondition checklist), qa (ADD - break-glass section), docs (#10 - prerequisites per phase).

**Day 30 - new Policy: Mass-delete / mass-overwrite anomaly on M365.** Activity policy on `File deleted` and `File modified` events at high velocity per user. Counters T1485 Data Destruction and T1486 Data Encrypted for Impact (ransomware-in-SaaS - Storm-0501, BlackCat-on-SaaS variants). Alert + manual governance review (NOT auto-suspend). Entirely absent from v0 despite being a 2024-2026 incident class. Sourced from: cyber (#10, fix DAY 30 SECTION).

**Day 30 - new Policy: B2B / partner-tenant exfiltration.** Activity policy: "external-domain guest user downloads / shares N+ files in T window" with alert. T1199 Trusted Relationship via guest accounts. Currently the playbook does not address B2B at all. Sourced from: cyber (#4, ADD - B2B / partner-tenant policy), sysint (#7 - XTAS + B2B absent).

**Day 30 - new Policy: App Governance Predictive Risk + OAuth threshold policy (notify-only -> revoke after tuning).** App Governance Predictive Risk (shipped 2025 H2) plus OAuth threshold policies (configurable count or scope thresholds per app per window). Covers the post-consent behavioural class missing from Policy 2. Notify-only for 4 weeks before enabling auto-revoke. Sourced from: mda-expert (#10, #11, fix DAY 90 / new policy - actually Day 30 in mda-expert's framing).

**Day 90 - new Policy: Wire Defender for Cloud Apps signals into Purview Insider Risk Management adaptive scoping.** For tenants licensed for IRM, MDA activity-policy hits become boosted signals into IRM Indicators (`compliance.microsoft.com -> Insider Risk Management -> Indicators -> Defender for Cloud Apps signals`). Turns Policies 3 / 7 / 10 into high-fidelity insider-risk indicators rather than standalone alerts. Sourced from: mda-expert (#12, fix DAY 90 / new policy).

**Pre-CAAC compatibility test plan per app (Day 30 sub-section).** Test matrix: SAML federation works, in-session features still work (especially `window.parent.postMessage` and `window.location.href` writes), cert-chain accepted by managed endpoints, anti-CSRF tokens survive rewrite, OIDC iframe re-auth survives rewrite, third-party-cookie flows still work, SaaS-to-SaaS embedded OIDC initiated mid-session. Sourced from: mda-expert (ADD - separate Day 30 step), qa (ADD - regression-test matrix).

**`mcas.ms` deprovisioning runbook.** Four steps across two consoles + session-cookie purge; no Microsoft cache-TTL SLA. Required to roll back a CAAC onboarding cleanly. Sourced from: mda-expert (#19, ADD).

**DR / business continuity section.** Per-policy fail mode for CAAC outage (each session policy's `Always apply if data cannot be scanned` is the per-policy fail mode; hardcoded `false` in v0 = fail-open on proxy outage); API-mode policy behaviour on regional MDA outage; governance actions in flight on outage; alerts queued for SIEM forwarding behaviour. Add the **DR / fail-mode matrix** as a table: scenario x component x behaviour (fail-open / fail-closed / queue / drop) x operator notification (yes / no / how). Sourced from: sysint (#9, ADD - DR / fail-mode matrix).

**Multi-tenant / multi-geo behaviour section.** M365 multi-geo SharePoint coverage (verify per-geo iteration); Cross-Tenant Access Settings interaction with CAAC (B2B guest device claims come from home tenant - failure mode); satellite-tenant alert routing. Sourced from: sysint (#7, #8, ADD - Multi-tenant / multi-geo).

**ATT&CK + ATLAS coverage matrices.** End-of-doc table covering at minimum the techniques listed in cross-cutting (per cyber #15 + ADD). For each: MDA verdict (Prevent / Detect / Respond / Out of scope) + compensating control where Out of scope. Sourced from: cyber (#1, #7, #15, ADD).

**Threat-actor profile per policy.** For each Day 1/30/90 control, state which named actor TTP clusters the policy meaningfully counters (e.g. Policy 2 -> consent-phishing TA453-like / MIDNIGHT BLIZZARD / Storm-0558 / Storm-2372 / Storm-1283; Policy 8 -> insider negligence + opportunistic exfil, NOT trained-adversary). Sourced from: cyber (#13, ADD).

**Slow-and-low detection layer.** Sentinel KQL queries over `CloudAppEvents` aggregated weekly / monthly per user against baseline, covering download volume, OAuth-grant additions, external-share creation, MFA-method additions. v0 is built around fast-and-loud thresholds; slow-and-low channel wide open. Sourced from: cyber (ADD - slow-and-low).

**Purple-team / atomic-test cadence.** Every named policy gets a periodic detection test via Atomic Red Team / equivalent. Currently the playbook tells the reader to monitor FP rate; no equivalent for true-positive validation. Sourced from: cyber (ADD - purple-team cadence).

**AI-agent control envelope.** As Copilot / agentic flows proliferate, "user clicked Consent" and "AI agent invoked Consent on user's behalf" must be distinguishable in audit. State the audit-event field that distinguishes them, or flag that they cannot currently be distinguished and the resulting T1528-via-agent is an undetected technique class. Indirect prompt injection (AML.T0051) causing OAuth grant escalation is a gap. Sourced from: cyber (#7, ADD).

**Defender XDR Advanced Hunting saved-query starter pack.** 5-10 KQL queries: (a) all CAAC session-policy blocks last 7 days joined to user, (b) terminated-user activity across all connected apps by UPN suffix mismatch, (c) OAuth consent events filtered to high-permission-scope last 30 days, (d) mass-download events excluding service-account UPN regex, (e) DSPM-for-AI sensitivity-data-detected events by app, (f) the three-way audit join (`SigninLogs` + `AuditLogs` + `CloudAppEvents`). Sourced from: mda-expert (ADD - KQL starter pack), sysint (ADD - regulator audit-trail join query).

**Data flow diagram page.** For each policy: signal source (M365 connector / MDE feed / Syslog from SWG) -> MDA backend -> action surface (Entra disable / Purview label apply / file move / SWG block-script export) -> audit surface (`CloudAppEvents` row / Sentinel ingest / Graph Security API alert pull). Sourced from: sysint (ADD - data flow diagram).

**Tenant integration prerequisites checklist (beyond licensing).** Entra provisioning enabled for each SaaS to be SCIM-cascaded with documented latency; Graph Security API app registration for non-Sentinel SIEM; Sentinel workspace + Defender XDR connector enabled; Power Automate environment + Premium licence if governance-action flows in scope; ServiceNow / Jira webhook receiver or Sentinel-to-ITSM connector deployed. Sourced from: sysint (ADD - tenant integration prerequisites).

**Graph throttling budget appendix.** File-policy scan rate per tenant size; Graph rate-limit headers and how MDA surfaces them (it does not, well); link to Microsoft Graph throttling docs. Sourced from: sysint (ADD - Graph throttling budget).

**SCIM cascade catalogue.** Top 10 connected SaaS in a typical BFSI estate: disable signal latency, per-app footguns (Salesforce Permission Set sticky on deactivation; Workday Worker vs Account distinction; AWS IAM Identity Center vs Organization SCIM divergence; ServiceNow Inactive vs Locked Out semantics). Sourced from: sysint (ADD - SCIM cascade catalogue).

**Okta-as-primary-IdP addendum.** Playbook is Entra-first throughout. For BFSI shops still on Okta with Entra as resource tenant: Okta -> Entra SAML federation upstream, device-trust gap (Okta Verify state does not flow into Entra), suspend-user signal flow (deactivate in Okta -> SCIM to Entra -> Entra propagates further), CAAC onboarding requiring app federation re-pointed at Entra. Sourced from: sysint (#6, ADD - Okta addendum).

**Policy-as-code workaround section.** Documented Git workflow with portal JSON exports as source of truth; `diff` review on policy changes; alerting query that detects out-of-band portal edits via `AuditLogs`. Since native PaC is absent, this is what bigger MDA customers actually do. Sourced from: sysint (ADD - policy-as-code workaround), mda-expert (#1 - policy drift), docs (implicit via Three-lens versioning).

**Policy-drift detection.** Schedule a Graph API query against `policies/conditionalAccessPolicies` and an MDA portal export comparison monthly; orphan MDA session policy (no matching CA policy) is high-severity. Sourced from: sysint (#12, fix "What this playbook does NOT cover").

**Schema-versioning + SIEM-query-stability practice note.** `CloudAppEvents` schema has changed three times in two years (`ActionType` renamed from `ActivityType` 2023; `RawEventData` JSON string -> object late 2024). Version-pin queries and retest quarterly; subscribe to schema-change blog. Sourced from: mda-expert (#8, ADD - schema-versioning practice).

**Ticketing-system integration guidance.** From Day 30, the SOC must have a connector path, not email-parse. Paths: Sentinel -> ServiceNow ITSM connector (Event Management or Security Incident Response - different tables); Defender XDR -> ServiceNow via Sentinel SOAR / Logic Apps; Power Automate -> ServiceNow custom connector; email-to-ticket (worst). Sourced from: sysint (#11).

**Power Automate governance-action hooks - footgun list.** "Send to Power Automate" governance action footguns: licensing for Power Automate Premium HTTP connectors; ~6000 actions/5min throttle; outbound IPs change weekly (use service tag `AzureCloud.{region}`); orphan flows under personal accounts on offboarding. Sourced from: sysint (#10).

**Glossary section / companion `_glossary.md`.** CAAC, DCS, SIT, SCIM, MAM, BYOK, RBI, SSPM, CNAPP, MDE, DfE / DfI / DfO, DSPM, MAS TRM, BNM RMiT, PDPA, CAE, OCAS, CSF, DPIA, MIP, EDM. Sourced from: docs (ADD - glossary).

**Regulator-decision-tree sidebar.** Even if the full control mapping is deferred to `06-compliance/...-control-mapping.md`, a 4-row table mapping each Day 1/30/90 policy to one CSF outcome and one MY/SG regulator clause family is achievable now. Sourced from: docs (ADD - regulator-decision-tree).

---

## Disputes between specialists (where reviewers disagreed)

**Dispute 1 - `CloudAppEvents` retention default.** qa flagged the architect-lens (prior workflow) stated 180-day default vs product-lens stated 30 days; v0 adopted 30 days. mda-expert resolved: **30 days standard tier; 90 days hot + 90 days warm = 180 days with the Defender XDR unified data lake (GA 2025)**. **Resolution:** state both. "30 days standard; 90+90 days with unified XDR data lake; Sentinel forwarding for 1-7 year BFSI retention." Cite Microsoft Learn Advanced Hunting retention doc + the XDR data lake GA announcement. The original architect-lens claim was correct for tenants on the data lake; the product-lens claim was correct for the free tier. Both are right with the data-lake qualifier.

**Dispute 2 - File-policy ceiling (50 vs 200).** qa #13 treated the 50-policy ceiling as a real constraint and flagged it needs a Microsoft Learn cite. mda-expert #9 stated **the ceiling was raised to 200 in February 2026** per a quiet docs change. **Resolution:** mda-expert wins on currency; the playbook must verify against the current `data-protection-policies` page accessed 2026-06-10 and update to 200 (or whatever is current). The consolidation guidance changes accordingly - practitioners who merged 30 SITs into 5 super-policies to fit 50 can now decompose. State the verified-as-of date.

**Dispute 3 - Tenant primary region "no APAC".** qa #4 called this the single most consequential unverified claim and said either pull from Trust Center or downgrade to "verify". mda-expert #20 stated **Japan East was added 2025 H2 and India is on roadmap H2 2026**. docs verify also said pull from Trust Center. **Resolution:** mda-expert wins on currency - update to "Japan East available as of 2025 H2; India on roadmap H2 2026; verify against M365 admin centre tenant detail blade and the Microsoft Cloud regions blog accessed `<date>`". The Malaysian / SG / HK residency calculus materially changes with Japan East available.

**Dispute 4 - "Generative AI" category framing.** cyber #5 treated Policy 8's category filter as a thin enforcement chain that misses bypasses. mda-expert #16 added that the category itself was **split into sub-categories** in 2025 H2 (`LLM`, `AI Coding Assistant`, `AI Image Generator`, `AI Video Generator`, `AI Voice/Avatar`). **Resolution:** both fixes apply. The policy must (a) name the bypasses (cyber's "Defeated by" row) AND (b) use the sub-category filter to avoid lumping AI image generators (Midjourney, DALL-E) with LLMs - different risk class. Use sub-category filter or document the expanded blast radius.

**Dispute 5 - NIST CSF citations.** qa #10 / #19 said either drop all framework hooks from the policy tables and defer to the `06-compliance/...-control-mapping.md` deliverable, OR carry the full subcategory mapping; the current half-mapping (PR.AA-05, DE.CM-03 only) is worse than either extreme. docs #11 also flagged inconsistency. cyber #1 is technique-mapping focused, not CSF-mapping focused. **Resolution:** drop the half-mapping from the policy tables (cleaner) and add a single "regulator-decision-tree" 4-row sidebar (per docs ADD) plus the full mapping in the dedicated compliance deliverable. ATT&CK technique rows replace the half-mapping in policy tables; CSF / RMiT / TRM mapping moves to the compliance file. This satisfies qa, docs, cyber and the file-creation discipline.

**Dispute 6 - Policy 2 framing (primary defence vs post-consent cleanup).** cyber #2 says reposition as "post-consent cleanup", not primary defence; primary control for T1528 is Entra admin-consent workflow. mda-expert #2 says split into Policy 2a (consent-event fidelity) and Policy 2b (App Governance behavioural fidelity). **Resolution:** both are compatible. Reframe Policy 2 as **"post-consent cleanup + behavioural detection"**, primary control is **Entra admin-consent workflow + verified-publisher restriction** (configured in Entra, not MDA - state explicitly). Then split the MDA piece into 2a (classic OAuth apps page anomalies) and 2b (App Governance Predictive Risk + OAuth threshold) as a Day 30 addition. The Entra control is referenced in the residual-risks / compensating-control table.

**Dispute 7 - Where the "What it does NOT cover" content lives.** docs #8 / #19 / #20 flagged ~60% content overlap between the line-26 NOT-cover list and the line-194 Residual-risks table; pick one home. cyber #15 wants a residual table in ATT&CK terms; sysint wants a DR matrix; mda-expert wants residual rows for CAAC deprovisioning / Power Platform / CAE / Edge & GSA. **Resolution:** **single consolidated "Gaps and compensating controls" table near the top** with columns `Gap | Why MDA can't | Compensating control | ATT&CK technique(s) | Cross-ref to Day-X policy`. Eliminates the duplication, satisfies cyber's technique-framing, gives mda-expert's new rows a home, and the DR matrix becomes a separate appendix.

---

## Findings deferred (NOT to apply in v1)

- **Full BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS clause mapping.** Already deferred to the dedicated `06-compliance/microsoft-defender-for-cloud-apps-control-mapping.md` per the playbook's own line 212. Reaffirmed; do not attempt in the playbook body. (Captured in cross-cutting fix to drop the half-mapping.) Sourced from: qa (#19), docs (ADD - regulator-decision-tree is a *light* version, the full mapping stays deferred).
- **Third-party attestation citations with version qualifiers fully pulled from Service Trust Portal.** v1 should add the version qualifiers per attestation (per qa #21) but the actual STP pulls + reports remain a separate deliverable. Sourced from: qa (#21), playbook itself line 213.
- **Sub-processor list URL pull.** Required for ISO 27018 A.11 but the actual URL pull is a separate Trust Center artefact, not a playbook line. Sourced from: playbook line 214.
- **DPIA / TIA scaffolding** for CAAC reverse-proxy and DSPM-for-AI prompt capture. Both are separate documents; playbook should reference. Sourced from: playbook line 215.
- **Exit / portability planning** - policy export, log export defensible format, configuration backup-restore - mentioned as out-of-scope on line 216 today; remains so. Sourced from: playbook line 216.
- **Cross-vendor synthesis / capability matrix.** Lives in `synthesis/capability-matrix.md`; not for this playbook. Sourced from: playbook line 217, cyber #15 implicit.
- **Quantitative DCS-latency benchmarks.** Vendor doesn't publish; product lens in the prior round flagged. v1 cannot fill this; document as residual. Sourced from: playbook line 239, mda-expert (verify line 30).
- **Independent FP/FN measurement on Microsoft risk score.** Not feasible without internal pilot data. v1 states the gap (per qa #17 / mda-expert verify line 161) but cannot resolve. Sourced from: cyber (verify line 164), qa #17.
- **Independent measurement of M365 connector latency.** Microsoft does not publish an SLA; practitioner-measured numbers are tenant-specific. v1 downgrades the claim (per qa #3) but cannot replace with a defensible number. Sourced from: qa #3, sysint (verify), docs (verify).

---

## Repo-wide recommendations (NOT for the playbook itself)

These go into a separate `repo-wide-recommendations.md`; do not auto-apply to the playbook.

- **`CLAUDE.md` (repo root for `~/claude/casb/`)** - add a vendor-playbook component-model section that requires every per-vendor practitioner doc to carry: (a) citation discipline (URL + accessed date on every Microsoft / vendor-named claim, or claim-classification tag), (b) the locked policy-table schema (Risk reduced | What it does | ATT&CK technique | Critical prerequisite | Console path | Key configuration | Trap | Defeated by | Auditable evidence with field-level granularity | SIEM artefact + connector), (c) a Day 0 prerequisites section before Day 1, (d) a glossary or first-use TLA expansion, (e) ATT&CK + ATLAS coverage rows, (f) DR / fail-mode and SCIM cascade sections, (g) a refs companion file. Sourced from: all five specialists' specific fixes converging on schema discipline.
- **`_research/2026-06-10-vendor-practitioner-reference/_template.md` (per-vendor template)** - if a template exists, update to match the locked schema above. If no template exists, create one at this path so the Netskope / Zscaler / Skyhigh / Palo Alto playbooks reuse the same shape and the cross-vendor capability matrix has consistent fields to pull from. Sourced from: docs #1 (consistency), cyber #15 (cross-vendor coverage matrix), qa ADD (refs companion file).
- **`_research/2026-06-10-vendor-practitioner-reference/_glossary.md` (workspace-wide glossary)** - house the TLA expansions (CAAC, DCS, SIT, SCIM, CNAPP, SSPM, DfE/DfI/DfO, DPIA, RBI, DSPM, CAE, OCAS, MIP, EDM, MAM, BYOK, CSF, RMiT, TRM) plus vendor-equivalents from other CASBs so cross-vendor docs can link consistently. Sourced from: docs (ADD - glossary), sysint (implicit via SCIM detail).
- **`synthesis/capability-matrix.md` (cross-vendor)** - add an ATT&CK coverage column per vendor + an ATLAS coverage column. The per-vendor playbooks then link to the matrix instead of restating. Sourced from: cyber #15, ADD.
- **`synthesis/dr-failmode-matrix.md` (cross-vendor)** - DR / fail-mode comparison across MDA / Netskope / Zscaler / Skyhigh / Palo Alto. Per-vendor docs link in. Sourced from: sysint #9, ADD.
- **`synthesis/scim-cascade-catalogue.md` (cross-vendor)** - top 10 BFSI SaaS, disable-signal latency and per-app footguns, vendor-agnostic. Per-vendor docs (whoever is the IdP / CASB) link to it. Sourced from: sysint ADD - SCIM cascade catalogue.
- **`06-compliance/microsoft-defender-for-cloud-apps-control-mapping.md` (separate compliance deliverable)** - the full BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF 2.0 / PCI DSS clause mapping deferred from the playbook body. Sourced from: qa #19, playbook itself line 212.
- **`_research/2026-06-10-vendor-practitioner-reference/microsoft-defender-for-cloud-apps.refs.md` (refs companion)** - keyed by claim-id with Microsoft Learn URL + accessed date + quoted phrase for every numeric / dated / portal-path / product-named claim in the playbook. Sourced from: qa ADD - refs companion file.
- **`CHANGELOG.md` (playbook-adjacent or repo-level)** - track v0 -> v1 -> v2 iterations; what was added / changed / removed in each pass. Sourced from: docs ADD - changelog.
