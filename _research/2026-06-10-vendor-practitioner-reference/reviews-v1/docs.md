# Technical documentation expert - playbook v0 review

Reviewer scope: structure, terminology, accessibility, reader flow. I am not adjudicating technical correctness — the architect, product, and compliance lenses already did. I am asking whether a tired practitioner at 3am-on-call can find what they need in this document.

## Highest-impact findings (rank-ordered)

1. **Policy entries violate their own template.** The Day 1 / Day 30 / Day 90 policy entries advertise themselves as parallel "field table" structures but the field set is not stable across entries:
   - Policy 1 has rows: Risk reduced / What it does / Console path / Key configuration / Trap to avoid / Auditable evidence (6 rows).
   - Policy 2 adds "Critical limitation" (7 rows, line 82).
   - Policy 5 adds "BYOD note" (line 122).
   - Policy 6 adds "50-policy ceiling" (line 134).
   - Policy 8 adds "Prerequisites you must verify before enabling block" and "Risk-acceptance note" (lines 162, 164).
   - Policy 9 adds "Critical prerequisite" (line 174).
   A practitioner scanning at 3am cannot rely on "row 5 is always Trap." Field drift across ten policies is the single largest readability cost in this doc. **Either lock the schema to a constant row-set (with N/A for empty rows) or hoist the variable rows into named callouts above/below the table.** Right now it is structurally inconsistent.

2. **The header table syntax (`| | |` with blank header cells) is hostile to scanning.** Every policy uses an anonymous 2-column markdown table where the left column is a bolded label and the right column is prose. This is rendered as a table by every renderer (GitHub, Obsidian, mkdocs) but presents as a wall of text in a narrow viewport because the long-prose-cell auto-expands and the label column shrinks to wrap. **Either use a proper definition list (`Term:` / indented prose) or use H4 sub-headers per field** so the table-of-contents and the eye can land on "Trap to avoid" directly. This is also why field drift (finding #1) is invisible — the schema is implicit.

3. **No table of contents and no anchor links between policies.** Document is 243 lines, 10 numbered policies, two retrospective cross-references ("see Day 90 below" line 23, "see Policy 8 below" pattern implicit), but no `[Policy 4](#policy-4--impossible-travel--activity-from-infrequent-country-alert-only)` style links and no top-of-doc TOC. At 3am the practitioner Ctrl-F's "Policy 4" and lands on the H3 — fine for navigation, useless for cross-reference. The Residual-risks table (line 194+) does not link back to the policies it complements. The "see Day 90 below" reference on line 23 has no anchor — the reader has to scroll-hunt.

4. **The "Read this if" block is mis-shaped for the target audience.** Line 5 says "your firm already has MDA". That is the licensing pre-test, not the audience definition. The actual audience this doc is written for is: "security architect or platform engineer at a regulated FI tasked with hardening MDA configuration end-to-end." Say so. Without it, an MDR analyst, a CISO advisor, a Microsoft FastTrack engineer, and a compliance lead all open this expecting different things — and three of them bounce. Suggest two sub-bullets: **Read this if you are** (role) / **Skip this if** (what this is not — vendor evaluation, RFP scoring, license-cost modelling).

5. **Critical warnings are buried in the table cells, not flagged visually.** Every "Trap to avoid" row carries the highest-value information in the policy (the thing that breaks production). Currently they sit as the 5th or 6th row inside a 2-column table with no visual differentiation from "Console path." Compare:
   - Line 70 ("discovered subdomains capability deprecated 2025-12-31 — re-baseline before any compliance attestation cycle") — buried.
   - Line 81 ("Banning is tenant-wide and immediate... wake up to 40 broken integrations") — buried.
   - Line 93 ("Suspend user disables the Entra user object... SOC cannot un-suspend cleanly") — buried.
   - Line 121 ("treat every CAAC onboarding as a per-app regression test event") — buried.
   These are blockquote-with-icon material, not table-cell prose. **Promote every Trap row to a `> WARNING:` blockquote immediately above the policy's Key configuration row.** That is the single visual change with the biggest readability lift.

6. **Jargon is dumped without first-use definition in three places.**
   - **CAAC** is introduced on line 15 with a parenthetical "Conditional Access App Control (CAAC)" — good. But then **DCS** appears at line 47 ("via the Data Classification Service") and is not re-expanded when it next shows up at line 132 ("DCS quota + matching-content storage") — a reader who jumped in at Policy 6 will not know what DCS is.
   - **SIT** appears at line 132 with no expansion. First-use is "SIT = Credit Card Number" (line 132). A practitioner who does not know SIT = Sensitive Information Type stops reading.
   - **DfE / DfI / DfO** appears at line 46 ("Without DfE / DfI / DfO, MDA's XDR incident correlation is shallow"). Never expanded. Defender for Endpoint / Identity / Office 365. Spell out on first use.
   - **SCIM** appears at line 93 ("SCIM-deprovisioning waves into every connected app"). Not expanded.
   - **MAM** appears at line 28 ("Intune App Protection Policies (MAM)") — adequately glossed.
   - **CNAPP** appears at line 198. Not expanded.
   - **SSPM** appears at line 202. Not expanded (well — Day 90 expands as "SaaS Security Posture Management" but in the Residual table cell, after first use).
   - **MDE** appears at line 48, parenthetically expanded as Defender for Endpoint. Good.
   - **DSPM** appears at line 23 ("Microsoft Purview 'DSPM for AI' overlay") — not expanded.
   - **MAM**, **RBI** (line 201 — Remote Browser Isolation), **BYOK** (only the compliance lens uses it, never in playbook — fine), **DPIA** (line 176, 215) — DPIA needs expansion on first use.
   Build a glossary section or expand on first use. Currently the doc presumes a Microsoft-security-stack-fluent reader, which contradicts the "practitioner playbook" framing (a practitioner at a BFSI who has just been handed MDA may know SaaS but not the Microsoft TLA garden).

7. **The opening paragraph is a single 113-word sentence.** Line 13: "MDA is the Microsoft Defender layer that watches and governs your sanctioned SaaS apps via API connectors, runs Shadow IT discovery on your egress traffic, surfaces and revokes risky third-party OAuth grants on Microsoft 365 / Google Workspace / Salesforce, and — when paired with Entra ID Conditional Access — intercepts browser sessions to those apps to enforce in-session DLP and download/upload rules." This is the executive summary. A 3am reader cannot parse a 113-word run-on. Break into 4 bullets:
   - API governance for sanctioned SaaS;
   - Shadow IT discovery from egress logs;
   - OAuth grant surveillance and revoke;
   - In-session DLP via CAAC (Entra-dependent).
   Headline pattern.

8. **The "What it covers well" / "What it does NOT cover" / "What it touches" sequence is the right idea but mis-formatted.** Three differently-shaped sections in 25 lines:
   - "What it covers well" — 6 bullets, prose.
   - "What it does NOT cover" — 12 numbered items, prose-heavy, each 3–5 lines.
   - "What it touches" — a 2-column markdown table.
   Pick one shape and apply it. The 12-item NOT-cover list specifically reads as a wall of text; each item could be a row in a table with columns **Gap | Why it matters | Compensating control** (which is exactly what the Residual-risks table at line 194 does). **Merge them.** The current state has the practitioner reading the same gap twice in different formats: "BYOD mobile uncovered" at item 1 line 28, then "BYOD-mobile data exfil" at line 196 in the Residual-risks table. Pick one home for each gap.

9. **The Concentration-risk paragraph (line 52–54) is a single dense paragraph in a place no one will read it.** It sits between "What it does NOT cover" (item 12 ending line 39) and the first Day 1 policy. A practitioner skimming for "Day 1" jumps past it. This is one of the most important architectural points in the doc and the compliance lens flagged it explicitly. **Promote to an H3 callout at the top of the Dependencies block, or merge into a sidebar at the very top under "Read this if."**

10. **The Day 1 / Day 30 / Day 90 framing is good but the section openers are too thin.** Each phase has one sentence of intro:
    - Day 1 (line 60): "Configure these four within the first two weeks of go-live..."
    - Day 30 (line 111): "These three are higher-impact but riskier to misconfigure..."
    - Day 90 (line 152): "These are higher-blast-radius and require operational maturity..."
    A 3am reader needs a 3-bullet "stop and check before starting this phase" at the top of each. Currently the practitioner only finds out at Policy 5 that they need Entra ID P1 — they could have built three policies without that check. **Add a prerequisites checklist as an opener block per phase.**

11. **"Auditable evidence" rows are inconsistent in granularity.** Some name a Defender table (`CloudAppEvents`); some name a retention window (30 days); some name a CSF control (DE.CM-03); some don't. Compare:
    - Policy 1 line 71: "`CloudAppEvents` table in Advanced Hunting; default retention 30 days unless forwarded to Sentinel." — good.
    - Policy 4 line 105: "`AlertInfo` + `AlertEvidence` (Defender XDR) carries the alert; default retention 30 days unless forwarded to Sentinel." — good.
    - Policy 3 line 94: adds the CSF mapping ("NIST CSF DE.CM-03").
    - Policy 5 line 123: adds the "two signals correlated" requirement.
    - Policy 6 line 135: adds "Auditor will ask: prove the label was applied at T0, by which policy version."
    Practitioners need a **fixed evidence schema**: Table / Retention / Field carrying decision / CSF or RMiT mapping (if assigned) / Correlator requirement (if multi-source). Right now the practitioner cannot tell at a glance which evidence is volatile, which is split across products, which is sufficient for an audit.

12. **The "Three-lens sign-off" footer is the wrong shape for a `_research/` document.** Lines 236–242 record three reviews from a single workflow ID `wcr0nuuwc`. Useful for traceability but the format is opaque to anyone who didn't run the workflow. **Three improvements:**
    - Replace the cryptic workflow ID with a link to the underlying review file (e.g. `lens-reviews/microsoft-defender-for-cloud-apps-architect.md`).
    - The "Verdict" column says INCORPORATED, INCORPORATED, PARTIAL — but it does not say what version was incorporated. As iterations stack, this footer needs a version column.
    - "Outstanding" prose mixes "captured but not yet acted on" with "deferred to a separate deliverable" — separate them.

13. **British English is inconsistent.** The user instruction is British English. Mixed in this doc:
    - "sanitised" (line 3) — British. Good.
    - "Catalogues" (line 78) — British. Good.
    - "Customised" (line 90), "customised" (line 102) — British. Good.
    - "Categorised" — not present; "categorical" (line 23) is fine.
    - **"licence"** appears as a noun (line 38, 223). **"License"** in noun form would be US — needs check. Current usage is correct British noun.
    - **"organisation"** / **"organization"** — not present.
    - "labelled" (line 113) — British. Good.
    - "behaviour" — not present.
    - **"Generative AI"** / **"GenAI"** mix: line 23 "GenAI app discovery", line 154 "Generative-AI apps", line 158 "consumer-tier GenAI", line 159 "GenAI apps", line 161 "Generative AI category". Pick one capitalisation and stick to it. Recommend **"GenAI"** as shorthand after first-use expansion to "generative AI".
    - **"Sensitivity label"** vs **"Sensitivity Label"** — line 120 "Sensitivity label ∈ {Confidential, Highly Confidential}" (lower-case). Line 32 "Microsoft Purview sensitivity label" (lower-case). Line 132 "Apply Microsoft Purview sensitivity label → *Confidential — Finance*" (lower-case). Consistent — keep lower-case for the generic term, capitalise only the specific label name (`Confidential — Finance`).
    - **"OneDrive"** vs **"OneDrive for Business"** — line 19 "OneDrive sync", line 28 "OneDrive sync", line 89 "SharePoint / OneDrive", line 125 "OneDrive / SharePoint", line 132 "OneDrive for Business + SharePoint Online", line 142 "OneDrive files", line 144 "OneDrive for Business". Mixed. **Convention: "OneDrive for Business" the first time in each section, "OneDrive" thereafter.**
    - **"Day 1" / "day one"** — line 60 "Day 1 — turn-on essentials" (heading); line 80 "Day 1: review every High-risk-permission" (in body); line 103 "Sensitivity = Low or Medium on day one" (in body). Mixed. Standardise: capital-D Day in headings, "on day one" in body prose.

14. **The "Console path" rows mix conventions.** Some use `→` (Unicode right-arrow); some use `/`; some use `>`. Examples:
    - Line 68: "Microsoft Defender Portal → Cloud apps → Cloud discovery". Configure data sources at **Settings → Cloud apps → Cloud discovery → Automatic log upload**."
    - Line 79: "Cloud apps → OAuth apps".
    - Line 91: "Cloud apps → Policies → Policy management → Threat detections tab → Create policy → Activity policy".
    - Line 119: "Cloud apps → Policies → Policy management → Conditional Access tab → Create policy → Session policy".
    Convention is `→`. Good — consistent. But **the leading "Microsoft Defender Portal" prefix is dropped after the first use** (line 68 has it, line 79 onwards omit it). At 3am a practitioner Ctrl-F's for "Microsoft Defender Portal" expecting every policy to start there and gets one hit. **Either consistently prefix or call out once at the top of "Day 1" that all paths start from `security.microsoft.com`.**

15. **Vendor-y phrases survived editorial.** The intro tone is set as "no vendor marketing" (line 7) but the following phrases leaked through:
    - Line 19: "**first-class**" (M365 is first-class). Marketing word.
    - Line 20: "**~31,000-app catalogue (Microsoft's count, vendor-self-reported)**" — flagging the count as vendor-reported is good; "31,000-app catalogue" still reads as a feature claim. Compress to "Microsoft asserts 31,000 apps catalogued; independent verification not available."
    - Line 22: "**DRM-style controls**" — DRM is a vendor framing; the actual control is sensitivity-label-with-encryption. Strip "DRM-style".
    - Line 22: "**but only for apps onboarded to CAAC**" — fine.
    - Line 23: "**categorical visibility into 1,000+ generative-AI apps**" — "categorical visibility into 1,000+" is vendor-shape. Compress: "Surfaces the GenAI category for apps already in the catalogue (Microsoft asserts 1,000+; not independently verified)."
    - Line 24: "**built-in policies for impossible travel, mass download, terminated-user activity in connected apps, suspicious OAuth grants**" — fine; the words "built-in" are descriptive not marketing.
    - Line 30: "**post-upload, not prevention**" — terse and correct, keep.
    - Line 33: "**(which can carry encryption + permissions)**" — parenthetical that could be "(may include encryption and access restrictions, depending on label configuration in Purview)" for precision. Minor.

16. **Cross-references between policies do not actually exist as anchors.**
    - Line 23: "see Day 90 below" — no anchor link. Reader scrolls.
    - Line 30 (in NOT-cover list, item 3): "Policy #2 (auto-label PCI), Policy #4 (mass-download), Policy #10 (quarantine externally-shared), Policy #11 (malware on Box/Dropbox/Google), Policy #12 (terminated-user activity)" — but **there is no Policy #11 or Policy #12 in this playbook**. The playbook has Policies 1–10. The compliance lens reviewed Policies 1–12 from the vendor draft, but the playbook consolidated to 10. **This is a stale internal cross-reference and must be fixed before promotion.** A reader at line 30 looking for "Policy #11 on malware" will scroll-hunt and not find it.
    - Line 38, item 11: "OCAS — the O365 E5 subset SKU. If you're licensed O365 E5 but *not* M365 E5, you get OCAS: M365 connector only, no third-party SaaS connectors, no Conditional Access App Control beyond Office apps, no app governance, no GenAI category, no SSPM. **~80% of this playbook does not apply.**" — does not link back to the verify-licensing checklist item at line 223 ("1. **Verify licensing.** M365 E5 or MDA standalone — *not* O365 E5 alone."). Should cross-reference.

17. **The "What to do next" checklist at line 221 is the right idea but mis-placed.** Currently it sits between Residual-risks and Three-lens sign-off (lines 221–230). At 3am the practitioner who has just landed on this doc needs this checklist **first** — verify licence, verify P1, verify MDE, verify region. **Move it to the top, immediately after "Read this if".** It is the pre-flight check; nothing in the body of the doc is actionable until those 4 verifications pass.

18. **The "Concentration risk" subsection is mis-styled.** Line 52 uses `### Concentration risk` (H3) inside the Dependencies block. Two structural problems:
    - It is the only H3 inside the dependencies section — sibling to "What it covers well", "What it does NOT cover", "What it touches" which are also H3 in the prior block.
    - Hierarchy is mushy: are these H3s peers of "What MDA is for" (H2) or sub-sections of it? The leading H2 on line 11 ("## What MDA is *for*") then no H3 break before "### What it covers well" on line 17 implies they are children of "What MDA is for" — but Concentration-risk (line 52) is also at H3 and is conceptually a sibling.

19. **The Residual-risks table (line 194+) and the "What it does NOT cover" numbered list (line 26+) overlap by ~60%.** Both say BYOD-mobile is uncovered; both say cert-pinned apps fail; both say no policy-as-code. Practitioner reads the same gap twice in two formats. **Pick one home: an early "Gaps and compensating controls" table near the top, with the residual-risks block becoming a thin pointer back.** Or invert: the early NOT-cover list is the executive summary and the Residual-risks table is the detailed treatment — but then they should not repeat content.

20. **The "What this playbook does NOT cover (out of scope)" block (line 211) is in the wrong place.** It is the document scope, not a policy section. It belongs immediately under the "Read this if" header. Reading it at line 211, after 10 policies, is too late.

## Specific fixes the v1 must apply

- [LINE 30, NOT-cover item 3] Remove dead references to "Policy #11 (malware on Box/Dropbox/Google), Policy #12 (terminated-user activity)" — these policies do not exist in this playbook; the existing policy numbering is 1–10. Policy 10 in this playbook is terminated-user; the malware-on-Box/Dropbox case is in the Residual-risks table (line 206). Re-reference correctly or remove.
- [LINE 13] Break the 113-word opening sentence into 4 bullets.
- [LINES 64–105, ALL POLICY 1–4 TABLES] Lock the field schema to a constant row-set: **Risk reduced / What it does / Console path / Key configuration / Critical prerequisite (if any) / Trap to avoid / Auditable evidence**. Use "N/A" if a row is empty for a given policy. Apply consistently to Policies 1–10.
- [LINES 64–246, ALL POLICY TABLES] Promote every "Trap to avoid" row to a `> WARNING:` blockquote immediately above the "Key configuration" row in each policy. Keep the table for the rest.
- [LINE 23, 47, 93, 132, 162, 174, 176, 198, 202] Expand on first use: **DSPM** (Data Security Posture Management), **DCS** (Data Classification Service), **SIT** (Sensitive Information Type), **SCIM** (System for Cross-domain Identity Management), **CNAPP** (Cloud-Native Application Protection Platform), **SSPM** (SaaS Security Posture Management), **DfE / DfI / DfO** (Defender for Endpoint / Identity / Office 365), **DPIA** (Data Protection Impact Assessment), **RBI** (Remote Browser Isolation).
- [LINE 19, 28, 89, 125, 132, 142, 144] Standardise OneDrive references: "OneDrive for Business" on first occurrence in each section, "OneDrive" thereafter.
- [LINE 23, 154, 158, 159, 161] Standardise GenAI: expand "generative AI" on first use, use "GenAI" thereafter; do not switch between "GenAI" and "Generative-AI" mid-document.
- [LINE 68, 79, 91, 119, 131, 143, 160, 173, 185] Either prefix every Console path with "`security.microsoft.com` → " or call out once at the top of Day 1 that all paths are within the Microsoft Defender Portal.
- [TOP OF DOC, BEFORE LINE 11] Move the "What to do next" practitioner checklist (currently line 221) immediately under "Read this if". This is the pre-flight check.
- [TOP OF DOC, BEFORE LINE 11] Move "What this playbook does NOT cover (out of scope)" (currently line 211) immediately under "Read this if". This is document scope.
- [LINE 52, CONCENTRATION RISK] Promote from a buried H3 to a top-level callout or a row in the Dependencies table. Surface it earlier.
- [LINES 60, 111, 152] Each phase opener (Day 1, Day 30, Day 90) needs a 3-bullet **Prerequisites checklist** at the top: licensing required, dependencies that must be live, exclusion lists / break-glass configured.
- [LINES 64–105] Add a **TOC at top of document**: list each Day 1 / Day 30 / Day 90 policy by number and one-line title, with anchor links. Mirror the line 221 practitioner checklist.
- [LINE 19] Strip "first-class" — replace with "fully supported".
- [LINE 22] Strip "DRM-style controls" — replace with "sensitivity-label-applied encryption and permissioning".
- [LINE 23] Compress "categorical visibility into 1,000+ generative-AI apps" to "GenAI-category discovery for apps already in the catalogue (Microsoft asserts >1,000; not independently verified)".
- [LINE 80] "Day 1: review every High-risk-permission, low-community-use, unverified-publisher app." → "On day one, review every app with High-risk permissions, low community use, or an unverified publisher." Capitalisation noise.
- [LINE 38, ITEM 11] Cross-reference the OCAS trap forward to the line 223 verify-licensing checklist item.
- [LINES 194–207, RESIDUAL-RISKS TABLE] Add a third behaviour: cross-link each row back to the Day 1/30/90 policy it complements, or to the line 30+ NOT-cover item, so the reader knows whether the gap is covered downstream by a compensating control or is genuinely unaddressed.
- [LINES 236–242, THREE-LENS SIGN-OFF] Replace workflow ID `wcr0nuuwc` with relative links to the underlying lens-review files. Add a version column. Separate "captured" from "deferred to separate deliverable" within Outstanding.
- [GLOBAL] Add a glossary section at the end of the doc — or a `_glossary.md` companion file in `_research/` — covering CAAC, DCS, SIT, SCIM, MAM, BYOK, RBI, SSPM, CNAPP, MDE, DfE/DfI/DfO, DSPM, MAS TRM, BNM RMiT, PDPA, CAE, OCAS, CSF, DPIA, MIP, EDM.
- [POLICY 1, LINE 70] The "discovered subdomains capability deprecated 2025-12-31" warning currently sits inside the Trap row. Today is 2026-06-10 — the deprecation is **already past**. The phrasing should shift from "deprecated 2025-12-31" (future-looking) to "deprecated 2025-12-31 — already in effect as of writing" so the reader does not assume action is still required to migrate; they need to assume migration is already overdue.

## Claims to verify or remove

- [LINE 15] "MDA collapses to API connectors + Shadow IT discovery — **about half** of what the marketing implies." The "about half" is a hand-waving quantification with no measurement basis. Either back it with a feature-count breakdown (e.g. "of the X advertised features, Y require Entra P1") or drop the fraction.
- [LINE 20] "**~31,000-app catalogue (Microsoft's count, vendor-self-reported)**." The architect lens flagged this conflicts with a "34,000+" claim elsewhere. Pull the current authoritative figure from a dated Microsoft Learn page and cite, or mark **Unverified**.
- [LINE 23] "**1,000+ generative-AI apps**." Vendor claim; no independent benchmark; the architect lens already called this out. State "Microsoft asserts >1,000; not independently verified" or drop the count.
- [LINE 39] "**Defender for Cloud Apps tenants land in EU / US / UK / AU (paraphrased — confirm against the Microsoft Trust Center for your tenant).**" The doc itself flags this as paraphrased. The compliance and architect lenses both said: pull from the Trust Center before promotion. **Either pull the citation or remove the geo list entirely and replace with "verify your tenant's primary-data region via the Microsoft Trust Center."**
- [LINE 30] "**typical M365 connector latency is <15 minutes**." Source? Microsoft does not publish an SLA. State the basis (practitioner-measured, vendor-stated, no source).
- [LINE 31] "**Power BI / Dynamics 365 is 24–72 hours**." Source? Same problem.
- [LINE 70] "**discovered subdomains capability deprecated 2025-12-31**." Cite the Microsoft deprecation notice URL.
- [LINE 101] "**ML-based, ignores VPNs and tenant-common locations (Microsoft's claim — independent FP rate not published).**" Good — the caveat is correctly stated.
- [LINE 132] "**minimum violations = 5 (not 1 — 1 produces too many false positives on test data / mock cards)**". The "5 vs 1" decision is a practitioner heuristic without measurement basis cited. Either state it as a recommended starting heuristic to tune to your environment, or back it with a baseline measurement.
- [LINE 161] "**Risk score ≤ 5 (high-risk)**." The risk score being on a 1–10 scale should be stated explicitly (the compliance lens flagged that the scoring methodology is opaque). Add: "MDA risk score is 1–10 where 1 is highest risk; Microsoft's methodology is not disclosed."
- [LINE 162] "**Windows 10 1709+ or supported macOS builds**." Cite the current Microsoft Learn page for MDE Network Protection supported OS versions; the version floor changes.
- [LINE 174] "**Consumer ChatGPT (chat.openai.com with a free or personal Plus account) does NOT support SAML federation.**" Correct as of writing but OpenAI has moved on Team / Enterprise feature parity multiple times. State as "as of 2026-06" and cite OpenAI's docs.

## Things you (as this specialist) would ADD to the playbook

- **A 1-page "summary card" at the top.** Before the body, a single dense block covering: licensing required (M365 E5 or MDA standalone), dependencies (Entra P1 / MDE / Purview / Sentinel), prerequisites (CAAC requires both an Entra CA policy and an MDA session policy in separate consoles), and the 10-policy index by number and one-line outcome. Practitioner at 3am reads this first; if they don't need to read further, they don't.
- **A glossary section** (or companion `_glossary.md`) — see fix list above. The Microsoft Defender stack has more TLAs than any single doc can casually use.
- **A "schema for an audit-evidence row"** stated once at the top of the policy sections, so readers know what to expect in each Auditable-evidence row across all 10 policies. Currently the row is inconsistent in granularity.
- **A "policy index + dependency matrix"** — a small table at the start of Day 1 / Day 30 / Day 90 that shows for each policy: requires CAAC (Y/N), requires Entra P1 (Y/N), requires MDE (Y/N), requires Purview labels (Y/N), action class (alert / block / governance), blast-radius rating. The current "all Day 30 policies require CAAC and therefore Entra ID P1" sentence on line 111 buries this.
- **A `CHANGELOG.md` / versioning header.** This doc will iterate. Currently the only version marker is "v0 — written 2026-06-10" on line 3. After v1, v2, v3 the reader will want to know what changed. Either inline at the top or as a companion file.
- **A "How to read this playbook" sub-section** under "Read this if": tells the reader the order to consume (TOC → pre-flight check → Dependencies → policy by policy → Residual risks → Three-lens). Three lines, max.
- **Cross-link the line 30 NOT-cover list to the Day 90 GenAI policy block.** Currently the NOT-cover list says "no real-time blocking of malware in Box/Dropbox/Google" but the Day 90 GenAI policy block (Policy 8) discusses MDE Network Protection as a partial compensating control. The reader has to discover this overlap themselves.
- **A regulator-decision-tree sidebar** — even if the compliance lens's full control mapping is deferred to a separate deliverable, a 4-row table mapping each Day 1/30/90 policy to one CSF outcome and one named MY/SG regulator clause family is achievable now. Current state: zero compliance mapping in the playbook body.

## Overall verdict

- **Playbook v0 fit for practitioner use as-is?** NO-WITH-CRITICAL-FIXES
- **One-line reason:** Content is dense and honest, but field drift across policy tables, buried critical warnings inside table cells, ten unexpanded TLAs, a dead cross-reference to non-existent Policies 11 and 12, and a mis-placed pre-flight checklist mean a 3am practitioner cannot navigate this in <30 seconds — structural fixes are quick and would lift it to a defensible v1.
