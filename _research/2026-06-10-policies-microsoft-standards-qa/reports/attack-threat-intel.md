# MITRE ATT&CK + Microsoft Threat Intelligence QA expert - QA report

## Summary

- **Files reviewed:** 19 Tier-2 policy files (4 access-control / 4 DLP / 5 detect / 2 OAuth / 3 GenAI / 1 posture).
- **Drift to correct:** **11** (technique-name drift + tactic miscitations + threat-actor naming-convention drift + outdated AML.T0024 terminology + ATLAS technique-name mismatch).
- **Stale references:** **3** (MIDNIGHT BLIZZARD all-caps deprecated → "Midnight Blizzard"; Storm-0558 has been promoted to "Antique Typhoon"; AML.T0024 renamed "ML Inference API" → "AI Inference API").
- **Missing citations:** **6** (Storm-cluster attributions cited without a Microsoft Security Blog URL or Defender TI profile ID).
- **Practitioner-inference flags:** **5** (claims plausibly true but no Microsoft / MITRE source surfaced — author must flag as inference).
- **Verified-clean claims:** **14** technique IDs + named-actor attributions confirmed against the current ATT&CK matrix v15-v17 and Microsoft Defender XDR threat-actor-naming page (last updated 2026-06-03 per the page metadata).
- **Policies needing significant rework:** **2** (`detect/impossible-travel-alert.md` and `oauth/post-consent-cleanup-and-ban.md`) — both have a TA-attribution that is not Microsoft's name (TA453) which leaks Proofpoint taxonomy into a Microsoft-aligned doc.

---

## Verified-clean claims

These technique-ID + named-actor uses match the current MITRE ATT&CK matrix or Microsoft Threat Intelligence published reporting. Author can leave as-is.

| Policy | Claim verified | Source |
|---|---|---|
| `dlp/inline-upload-block-regulated-data.md` (line 11) | `T1530 Data from Cloud Storage` exists; tactic = Collection; SaaS scope correct | https://attack.mitre.org/techniques/T1530/ |
| `dlp/inline-upload-block-regulated-data.md` (line 11) | `T1567 Exfiltration Over Web Service` exists; tactic = Exfiltration; SaaS / Office Suite platforms in scope | https://attack.mitre.org/techniques/T1567/ |
| `dlp/api-data-at-rest-quarantine.md` (line 11) | `T1213.002 SharePoint` exists as sub-technique of T1213 Data from Information Repositories | https://attack.mitre.org/techniques/T1213/ |
| `dlp/external-share-link-quarantine.md` (line 11) | `T1199 Trusted Relationship` exists; tactic = Initial Access; covers third-party-provider abuse | https://attack.mitre.org/techniques/T1199/ |
| `access-control/tenant-restriction-corporate-only.md` (line 11) | `T1078.004 Cloud Accounts` exists as sub-technique of "Valid Accounts"; cloud-only platforms in scope | https://attack.mitre.org/techniques/T1078/004/ |
| `detect/terminated-user-cross-saas.md` (line 11) | `T1136.003 Cloud Account` exists; tactic = Persistence; cloud-account-creation scope correct | https://attack.mitre.org/techniques/T1136/003/ |
| `detect/terminated-user-cross-saas.md` (line 11) | `T1098.001 Additional Cloud Credentials` exists; tactic = Persistence / Privilege Escalation | https://attack.mitre.org/techniques/T1098/001/ |
| `detect/terminated-user-cross-saas.md` (line 11) | `T1556 Modify Authentication Process` exists; tactic = Defense Impairment / Persistence / Credential Access | https://attack.mitre.org/techniques/T1556/ |
| `detect/mass-delete-anomaly.md` (line 11) | `T1485 Data Destruction` exists; tactic = Impact | https://attack.mitre.org/techniques/T1485/ |
| `detect/mass-delete-anomaly.md` (line 11) | `T1486 Data Encrypted for Impact` exists; tactic = Impact; SaaS / IaaS platforms in scope | https://attack.mitre.org/techniques/T1486/ |
| `oauth/post-consent-cleanup-and-ban.md` (line 11) | `T1528 Steal Application Access Token` exists; tactic = Credential Access | https://attack.mitre.org/techniques/T1528/ |
| `oauth/post-consent-cleanup-and-ban.md` (line 11) | `T1550.001 Application Access Token` exists as sub-technique of T1550 Use Alternate Authentication Material; tactic = Lateral Movement (NOT Defense Evasion as in some older references — verify per current entry) | https://attack.mitre.org/techniques/T1550/001/ |
| `detect/mass-delete-anomaly.md` (line 11) | **Storm-0501** is a current Microsoft Threat Intel-tracked cluster (financially motivated); Microsoft Security Blog Aug 2025 confirms hybrid-cloud ransomware + mass-delete + in-cloud encryption pattern → matches the policy's threat-model claim | https://www.microsoft.com/en-us/security/blog/2025/08/27/storm-0501s-evolving-techniques-lead-to-cloud-based-ransomware/ ; https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming |
| Multiple policies | **Storm-2372** exists as a Group in development; the device-code-phishing campaign reporting from Feb 2025 is the canonical Microsoft source — claims about consent-phishing chain are anchored | https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/ |
| `oauth/post-consent-cleanup-and-ban.md` (Use case 1, line 25) | **Storm-1283** exists as a Microsoft-tracked cluster; OAuth-app abuse + cryptomining + ~927k phishing emails July–Nov 2023 confirmed in Microsoft Security Blog 12 Dec 2023 | https://www.microsoft.com/en-us/security/blog/2023/12/12/threat-actors-misuse-oauth-applications-to-automate-financially-driven-attacks/ |

---

## Drift to correct

Each entry: **file:line — practitioner claim — what Microsoft / MITRE says — corrected text**.

### D1. `detect/impossible-travel-alert.md:11` — **TA453 mis-attribution to Microsoft taxonomy**

- **Practitioner claim (Purpose, line 11):** "the dominant 2024-2026 intrusion pattern (T1566.002 → T1528 → T1550.001 token-replay from residential-proxy infrastructure)" — and downstream the policy invokes "TA453-class consent-phishing wave" (`oauth/post-consent-cleanup-and-ban.md` line 17 and 31).
- **What Microsoft says:** **TA453 is Proofpoint's name**, not Microsoft's. Microsoft tracks the same Iran-attributed actor as **Mint Sandstorm** (formerly PHOSPHORUS, CHARMING KITTEN, APT35). The Microsoft threat-actor-naming page lists `Mint Sandstorm` (Iran) with industry aliases PHOSPHORUS, CHARMING KITTEN, Parastoo, Newscaster, APT35 — no "TA453" appears. Source: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming.
- **Corrected text:** Replace "TA453" with **"Mint Sandstorm (formerly PHOSPHORUS)"** in `oauth/post-consent-cleanup-and-ban.md` lines 17 and 31, and any other occurrence. If the author wants to retain Proofpoint's taxonomy as a cross-reference, format as "Mint Sandstorm (Microsoft) / TA453 (Proofpoint) / Charming Kitten (CrowdStrike)" — but the Microsoft name is canonical for a Microsoft-aligned doc.

### D2. `detect/impossible-travel-alert.md:11` and 6 other policies — **MIDNIGHT BLIZZARD all-caps is the old style**

- **Practitioner claim:** "MIDNIGHT BLIZZARD service-principal abuse" (e.g. `oauth/high-scope-grant-alert.md` lines 17, 151, 245).
- **What Microsoft says:** Microsoft renamed the taxonomy in April 2023; all-caps `MIDNIGHT BLIZZARD` is the legacy industry alias. The current Microsoft name is **Title Case: "Midnight Blizzard"**. Other names per the Microsoft page: NOBELIUM, COZY BEAR, UNC2452, APT29. Source: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming.
- **Corrected text:** Replace "MIDNIGHT BLIZZARD" with **"Midnight Blizzard"** in all 7 occurrences across `oauth/high-scope-grant-alert.md` and `detect/mass-delete-anomaly.md` and `posture/sspm-tenant-misconfig-drift.md`. If the author wants legacy aliasing, parenthesise: "Midnight Blizzard (formerly NOBELIUM)".

### D3. `detect/mass-delete-anomaly.md:11` — **Storm-0501 + "BlackCat-on-SaaS" linkage unsupported**

- **Practitioner claim (line 11):** "Storm-0501 / BlackCat-on-SaaS variants delete or overwrite files at scale inside the tenant after compromise."
- **What Microsoft says:** Storm-0501 is Microsoft-confirmed for hybrid-cloud ransomware including mass-delete and in-cloud encryption against Azure Storage (Aug 2025 blog). **BlackCat** (a.k.a. ALPHV) is not on the Microsoft threat-actor-naming page in those exact words; the closest Microsoft-tracked clusters are different ransomware-class operators. Microsoft has reported on BlackCat affiliates as `Storm-XXX` clusters historically but the "BlackCat-on-SaaS" framing is not a Microsoft Threat Intel-published linkage I could locate.
- **Corrected text:** Keep Storm-0501 linkage (verified). Replace "BlackCat-on-SaaS variants" with a more cautious "ALPHV/BlackCat ransomware affiliates" and flag as `[practitioner-inference — no specific Microsoft Threat Intel report on BlackCat operating natively against SaaS surfaces; the link is to the on-prem-to-SaaS pivot pattern]`. OR cite the Storm-0501 + on-prem-to-cloud pivot reporting directly and drop BlackCat as the named comparator.

### D4. `dlp/inline-upload-block-regulated-data.md:11` — **T1567 mapping is correct; T1530 mapping needs caveat**

- **Practitioner claim (line 11):** "Counters MITRE ATT&CK `T1530` and `T1567` on the upload leg."
- **What MITRE says:** **T1530 Data from Cloud Storage is a Collection technique, NOT an Exfiltration technique.** It models the *adversary reading* the cloud storage object. An inline upload-block policy intervenes on a *user's upload to* SaaS — that maps cleanly to T1567 (Exfiltration Over Web Service) but **mapping it to T1530 inverts the threat model** because T1530 is what the adversary does after compromise, not what an insider does on the way out.
- **Corrected text:** Remove `T1530` from the upload-block policy's purpose statement. The control's primary mapping is **T1567** (and arguably the upload event being the precursor to T1213.002 SharePoint Collection by an adversary using the same channel, but that is a stretch). Keep T1530 only where the policy is about *blocking download from* cloud storage — see `access-control/block-download-unmanaged-device.md` line 11, which uses T1530 correctly.

### D5. `dlp/inline-upload-block-regulated-data.md:11` — **T1567 phrasing**

- **Practitioner claim:** Policy frames as "Counters MITRE ATT&CK `T1530` and `T1567` on the upload leg".
- **What MITRE says:** Sub-techniques exist: `T1567.001 Exfiltration to Code Repository`, `T1567.002 Exfiltration to Cloud Storage`, `T1567.003 Exfiltration to Text Storage Sites`, `T1567.004 Exfiltration Over Webhook`. The most-specific mapping for inline upload-block against SaaS upload (OneDrive / SharePoint / Box) is **T1567.002 Exfiltration to Cloud Storage**. Source: https://attack.mitre.org/techniques/T1567/.
- **Corrected text:** Replace `T1567` with **`T1567.002 Exfiltration to Cloud Storage`** for greater precision. (The parent `T1567` is acceptable but loses specificity.) Same applies to `access-control/block-unsanctioned-app-with-coach.md` line 11 (where the bare `T1567` covers the broader shadow-app exfil class and the parent ID is appropriate).

### D6. `detect/impossible-travel-alert.md:240` — **T1528 framing of "OAuth abuse runs from wherever"**

- **Practitioner claim (line 240, Coverage gaps):** "T1528 Steal Application Access Token — same colocation pattern; the consented OAuth grant runs from wherever the attacker has access; no sign-in event for the OAuth abuse itself."
- **What MITRE says:** Accurate framing — T1528 covers token theft (and OAuth-consent abuse is one of the methods); subsequent token use is T1550.001. Source: https://attack.mitre.org/techniques/T1528/ and https://attack.mitre.org/techniques/T1550/001/. **However:** the statement that the OAuth abuse "produces no sign-in event" is wrong in the general case — the original *user-consent* event IS audit-logged in Entra (`Consent to application`), and the *app's subsequent API calls* are logged in `CloudAppEvents` / `OfficeActivity`. What is NOT logged is a *fresh interactive sign-in* for the OAuth-as-the-app calls. Tighten the wording.
- **Corrected text:** Replace "no sign-in event for the OAuth abuse itself" with **"no fresh user sign-in event for the OAuth-app-as-actor calls — the original consent grant is audit-logged in Entra; the app's subsequent API activity is logged in `CloudAppEvents` / `OfficeActivity`; the impossible-travel signal is anchored on user sign-ins, which the OAuth-token path doesn't generate"**.

### D7. `genai/discovery-and-auto-unsanction.md:11` — **AML.T0024 renaming**

- **Practitioner claim (line 11):** "ATLAS `AML.T0024 Exfiltration via ML Inference API` partial."
- **What MITRE ATLAS says:** The technique is **currently titled "Exfiltration via AI Inference API"** (the rename from "ML" → "AI" reflects ATLAS's broader 2024-2025 terminology shift across the matrix). Source: confirmed by https://atlas.mitre.org/ matrix listings (also: https://www.startupdefense.io/mitre-atlas-techniques/aml-t0024-exfiltration-via-ai-inference-api-a964f).
- **Corrected text:** Replace `Exfiltration via ML Inference API` with **`Exfiltration via AI Inference API`** in all four occurrences (`genai/discovery-and-auto-unsanction.md` lines 11, 162; `genai/inline-prompt-dlp.md` lines 11, 199).

### D8. `genai/inline-prompt-dlp.md:11` — **T1052 disclaimer is correct; T1567 is the right primary**

- **Practitioner claim (line 11):** "Counters MITRE ATT&CK `T1567 Exfiltration Over Web Service` narrowed to clipboard sub-channel (Prevent — partial). MITRE ATLAS `AML.T0024 Exfiltration via ML Inference API` — partial. **Not T1052 (Physical Medium)** — the original draft's mapping was wrong."
- **What MITRE says:** Correct — T1052 Exfiltration Over Physical Medium is about removable media (USB / mobile / etc.); browser clipboard paste to a web service is not T1052. The honest practitioner self-correction is good. **However:** the most-specific ATT&CK technique for "user pastes regulated content into a web LLM" is again **T1567.002 Exfiltration to Cloud Storage** (since the LLM vendor stores the prompt). Keep T1567 as primary but consider adding the `.002` sub-technique.
- **Corrected text:** Optionally tighten to `T1567.002`; explicit acknowledgement of the T1052 self-correction is fine to keep — it shows lineage discipline.

### D9. `detect/terminated-user-cross-saas.md:11` — **T1136 → T1136.003 phrasing**

- **Practitioner claim (line 11):** "`T1136 Create Account` (where the user created a SaaS-local account before leaving)".
- **What MITRE says:** **T1136 is the parent technique**; the sub-technique for a cloud-SaaS account is `T1136.003 Cloud Account`. For a per-SaaS local account *that is not Entra-federated*, the cloud-SaaS sub-technique applies. The parent T1136 is also acceptable since on-prem accounts (T1136.001 Local / T1136.002 Domain) are not in scope of this policy.
- **Corrected text:** Replace `T1136 Create Account` with **`T1136.003 Cloud Account`** for greater precision. (Same applies in `detect/mass-download-alert.md` and `oauth/high-scope-grant-alert.md` line 11 where T1136.003 is already used correctly.)

### D10. `oauth/high-scope-grant-alert.md:11` — **T1098.001 vs T1098.003 — credential-add vs role-add**

- **Practitioner claim (line 11):** "Counters MITRE ATT&CK `T1098.001 Additional Cloud Credentials`, `T1550.001 Application Access Token`, `T1136.003 Cloud Account creation via service-principal`."
- **What MITRE says:** Both T1098.001 and T1136.003 are correct for this policy. **T1098.003 Additional Cloud Roles is missing** — when an OAuth app expands scope or an admin grants additional roles to a workload identity, that maps to T1098.003 specifically. The policy's `scope_expansion` alert class (line 124) matches T1098.003 better than T1098.001. Source: https://attack.mitre.org/techniques/T1098/003/.
- **Corrected text:** Add **`T1098.003 Additional Cloud Roles`** to the Purpose statement: "...`T1098.001 Additional Cloud Credentials`, **`T1098.003 Additional Cloud Roles`**, `T1550.001 Application Access Token`...". Note: `posture/sspm-tenant-misconfig-drift.md` line 11 already correctly references both T1098.001 and T1098.003 — that policy is the model.

### D11. `genai/inline-prompt-dlp.md:11` — **T1052 reference unnecessary even as a self-correction**

- **Practitioner claim:** Calls out historical mis-mapping to T1052 Physical Medium.
- **What MITRE says:** Fine to keep as a lineage note but consider removing from a published policy doc — it adds noise without practitioner value. Self-corrections belong in a changelog, not a Purpose statement.
- **Corrected text:** Move the "Not T1052" self-correction to the file's changelog / version notes; keep the Purpose statement focused on the current mapping.

---

## Stale references

### S1. **MIDNIGHT BLIZZARD all-caps** — current Microsoft style is "Midnight Blizzard" (Title Case)

- **Affected files:** `oauth/high-scope-grant-alert.md` (lines 17, 151, 245), `detect/mass-delete-anomaly.md` (line 250), `posture/sspm-tenant-misconfig-drift.md` (line 11).
- **Current name:** **Midnight Blizzard** (Title Case per Microsoft's threat-actor-naming taxonomy as of April 2023 rename).
- **Action:** find/replace `MIDNIGHT BLIZZARD` → `Midnight Blizzard` in all 5 occurrences.

### S2. **Storm-0558 has been promoted from "Group in development" to a named actor — "Antique Typhoon"**

- **Affected files:** `oauth/post-consent-cleanup-and-ban.md` (lines 237, 273), `detect/mass-delete-anomaly.md` (line 250), `oauth/high-scope-grant-alert.md` (line 17), `genai/inline-prompt-dlp.md` (no direct mention but cross-referenced).
- **Current name:** **Antique Typhoon** (China, nation-state) — the Storm-0558 cluster's confirmed identity post-investigation. Source: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming and https://www.microsoft.com/en-us/security/blog/2023/07/14/analysis-of-storm-0558-techniques-for-unauthorized-email-access/.
- **Action:** When the policy refers to the *July 2023 token-forgery incident* the historical name **Storm-0558** is still the canonical reference for that specific incident. When the policy generalises to "the actor cluster", use **Antique Typhoon** with `Storm-0558` parenthesised. Example: "Antique Typhoon (formerly Storm-0558) Microsoft service-side token-forgery class".

### S3. **MITRE ATLAS AML.T0024 has been renamed: "ML Inference API" → "AI Inference API"**

- **Affected files:** `genai/discovery-and-auto-unsanction.md` (lines 11, 162), `genai/inline-prompt-dlp.md` (lines 11, 199).
- **Current name:** **Exfiltration via AI Inference API** (terminology shift across ATLAS 2024-2025 to reflect broader AI scope, not just ML-specific).
- **Action:** find/replace `Exfiltration via ML Inference API` → `Exfiltration via AI Inference API` in all 4 occurrences.

---

## Missing citations

Claims that name a Microsoft-tracked threat actor but do not cite a Microsoft Security Blog URL, Defender TI profile ID, or threat-actor-naming page reference. Each should either gain a citation or be downgraded to practitioner-inference.

### M1. `detect/mass-delete-anomaly.md:21` — Storm-0501 peer incident

- "post-Storm-0501-class peer incident" — use-case 1 mentions a peer-bank incident. No Microsoft Security Blog cited for the underlying Storm-0501 cloud-ransomware reporting.
- **Cite:** https://www.microsoft.com/en-us/security/blog/2025/08/27/storm-0501s-evolving-techniques-lead-to-cloud-based-ransomware/ and https://www.microsoft.com/en-us/security/blog/2024/09/26/storm-0501-ransomware-attacks-expanding-to-hybrid-cloud-environments/.

### M2. `oauth/post-consent-cleanup-and-ban.md:25` — Storm-2372 reporting

- "post-Storm-2372 mass-cleanup" — Storm-2372 is the named trigger. No citation given.
- **Cite:** https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/.

### M3. `oauth/post-consent-cleanup-and-ban.md:17` — Storm-2372 + "TA453-class consent-phishing wave"

- The wave is named but no citation given. (Compounds with D1 — TA453 itself is the wrong taxonomy.)
- **Cite:** for Mint Sandstorm (the Microsoft name), see https://www.microsoft.com/en-us/security/blog/tag/mint-sandstorm-phosphorus/.

### M4. `detect/b2b-partner-exfil-alert.md:11` — Storm-0539 / Storm-2372 chains

- "Catches Storm-0539 / Storm-2372-style consent-phishing-into-B2B attack chains" — both named, no citations.
- **Cite:** Storm-0539 at https://www.microsoft.com/en-us/security/blog/2024/05/23/cyber-signals-inside-the-growing-risk-of-gift-card-fraud/ (note: Storm-0539 is **financially motivated, gift-card-fraud focused** per Microsoft — its linkage to "consent-phishing-into-B2B" is **practitioner inference**, the Microsoft reporting is gift-card-fraud-flavoured). Storm-2372 as above.

### M5. `detect/mass-delete-anomaly.md:250` — "Storm-0558 / Midnight Blizzard class" service-side compromise

- The "Microsoft service-side incident (the Storm-0558 / Midnight Blizzard class)" framing — two distinct named incidents (Antique Typhoon 2023 token-forgery + Midnight Blizzard 2024 corporate email compromise). Cite both.
- **Cite:** https://www.microsoft.com/en-us/security/blog/2023/07/14/analysis-of-storm-0558-techniques-for-unauthorized-email-access/ and https://www.microsoft.com/en-us/security/blog/tag/midnight-blizzard-nobelium/.

### M6. `genai/discovery-and-auto-unsanction.md:43` — Storm-2372 "OAuth-mediated AI access"

- The use case ties Storm-2372 to AI-app OAuth grants — the Microsoft reporting on Storm-2372 is about device-code phishing for token theft generally, not AI-app-specific. **Cite the underlying Microsoft report and flag the AI-extension as practitioner inference**.
- **Cite:** https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/.

---

## Practitioner-inference flags

Claims that are plausibly true based on the patterns but for which I could not locate a Microsoft Threat Intel source making the specific linkage. Each should carry a `[practitioner-inference]` annotation in the policy file rather than presenting as Microsoft-attributed.

### P1. `oauth/high-scope-grant-alert.md:151` — "compromised verified-publisher app is the MIDNIGHT BLIZZARD scenario"

- **Claim:** Verified-publisher abuse via credential-addition is the Midnight Blizzard play.
- **Microsoft reporting:** Midnight Blizzard's January 2024 corporate-email compromise (T1098.001 + service-principal abuse) is well-documented (https://www.microsoft.com/en-us/security/blog/tag/midnight-blizzard-nobelium/). The *specific* claim that "verified-publisher status was the attack vector" is **not** the framing of the Microsoft post-mortem — Midnight Blizzard's path was a legacy non-production test tenant with a residual OAuth grant, not a verified-publisher-trust abuse. Flag as practitioner-inference and reframe.
- **Suggested text:** "Verified-publisher status is a useful signal, not an authoritative trust statement — see Midnight Blizzard's January 2024 use of a legacy OAuth grant on a non-production test tenant for the pattern of legitimate-on-paper apps becoming attack-paths `[practitioner inference — Microsoft's post-mortem frames the path as legacy-tenant residual grant, not verified-publisher abuse specifically]`."

### P2. `detect/b2b-partner-exfil-alert.md:11` — Storm-0539 + consent-phishing-into-B2B

- See M4 above — Microsoft's Storm-0539 reporting is gift-card-fraud-centric. The B2B-partner-exfil framing is plausible but not Microsoft-attributed.

### P3. `detect/mass-delete-anomaly.md:11` — BlackCat-on-SaaS variants

- See D3 above — no specific Microsoft Threat Intel report on BlackCat operating natively against SaaS. Storm-0501 covers this surface; flag BlackCat as inference.

### P4. `genai/discovery-and-auto-unsanction.md:41-46` — Use case 4 Storm-2372 → "GenAI-app-via-OAuth overlap"

- The use case sequences Storm-2372's device-code phishing → OAuth-mediated AI access. The chaining is plausible but the Microsoft reporting on Storm-2372 does not explicitly call out AI-app OAuth as a target — it documents the broader device-code phishing campaign.

### P5. `oauth/post-consent-cleanup-and-ban.md:25` — "73 users had consented to a 'Microsoft Teams Connector' lookalike"

- Specific numbers (73, 18, 410) read as illustrative deployment narratives rather than Microsoft-reported counts. Already flagged via `[VERIFY ...]` and `Use case` framing throughout; consistent treatment.

---

## Policies needing significant rework

Two policies have multiple compounding issues with TA-naming and missing citations that warrant a focused rework rather than line-edits.

### R1. `detect/impossible-travel-alert.md`

- TA453 naming throughout the use cases and coverage-gap discussion (D1).
- T1528 mis-framing in Coverage gaps (D6).
- Several Microsoft-Threat-Intel claims (Storm-2372 / OAuth-grant correlation / Token Protection compensating control) made without citations — see also broader Citations gap pattern (M3).
- **Action:** Replace TA453 with Mint Sandstorm; tighten T1528 framing; add Microsoft Security Blog citations for Storm-2372 and Token Protection (verify GA at https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-token-protection).

### R2. `oauth/post-consent-cleanup-and-ban.md`

- TA453 again (lines 17, 31).
- Storm-2372 / Storm-1283 named but no Microsoft Security Blog URL anchors.
- `MIDNIGHT BLIZZARD` all-caps in anti-patterns / coverage gaps (line 273 and elsewhere — verify).
- **Action:** TA-naming pass + add citations to Storm-2372 (Feb 2025 Microsoft blog) and Storm-1283 (Dec 2023 Microsoft blog) + MIDNIGHT BLIZZARD → Midnight Blizzard.

---

## Top remediation priorities

Rank-ordered top 10 specific corrections, by combined impact (control-mapping integrity + auditor-defensibility + ease of fix).

1. **Find/replace `TA453` → `Mint Sandstorm`** across `oauth/post-consent-cleanup-and-ban.md` (lines 17, 31) and `detect/impossible-travel-alert.md` (use-case 3, around line 32). Add the Microsoft tag URL (https://www.microsoft.com/en-us/security/blog/tag/mint-sandstorm-phosphorus/) as the canonical reference. **Highest impact** — fixing taxonomy drift in a Microsoft-aligned doc.
2. **Find/replace `MIDNIGHT BLIZZARD` → `Midnight Blizzard`** across all 5+ occurrences. **Lowest cost, highest hygiene** — current Microsoft style as of Apr 2023.
3. **Find/replace `Exfiltration via ML Inference API` → `Exfiltration via AI Inference API`** for AML.T0024 in 4 occurrences (`genai/discovery-and-auto-unsanction.md` lines 11, 162; `genai/inline-prompt-dlp.md` lines 11, 199). Reflects ATLAS terminology shift.
4. **Add Microsoft Security Blog citations to Storm-cluster name-drops** — Storm-0501 (`detect/mass-delete-anomaly.md:21`), Storm-2372 (`oauth/post-consent-cleanup-and-ban.md:25`), Storm-1283 (`oauth/post-consent-cleanup-and-ban.md:25`), Storm-0539 (`detect/b2b-partner-exfil-alert.md:11`). Auditor-defensibility win.
5. **Remove T1530 from `dlp/inline-upload-block-regulated-data.md:11` Purpose** — T1530 is Collection, not Exfiltration; the upload-block policy maps to T1567/T1567.002, not T1530.
6. **Tighten `oauth/high-scope-grant-alert.md:11` to include T1098.003** — Additional Cloud Roles is the better fit for the scope-expansion alert class than T1098.001 alone.
7. **Annotate `detect/mass-delete-anomaly.md:11` "BlackCat-on-SaaS variants" as practitioner inference** — Storm-0501 is the verified Microsoft-tracked cloud-ransomware actor; BlackCat-on-SaaS as a peer label needs either a citation or a `[practitioner-inference]` flag.
8. **Reframe `oauth/high-scope-grant-alert.md:151` Midnight Blizzard verified-publisher claim** — Microsoft's post-mortem frames the path as a legacy non-production tenant with a residual OAuth grant, not verified-publisher-trust abuse specifically. Reword to match the Microsoft reporting or flag as inference.
9. **Antique Typhoon promotion** — wherever the policies generalise the Storm-0558 cluster (not the specific July 2023 incident), use **"Antique Typhoon (formerly Storm-0558)"** for current Microsoft taxonomy alignment.
10. **Tighten T1528 framing in `detect/impossible-travel-alert.md:240`** — the original `Consent to application` event IS audit-logged in Entra; the gap is the lack of *fresh interactive sign-in events* for the OAuth-app-as-actor calls, not the lack of any log at all.

---

### Notes on what is genuinely strong

- **`access-control/block-download-unmanaged-device.md`** uses T1530 correctly (download from cloud storage), T1213 correctly (in-session screenshot framing as Coverage-gap), and T1550.001 correctly as a coverage-gap acknowledgement. Good practitioner discipline.
- **`detect/terminated-user-cross-saas.md`** maps T1078.004 / T1136 / T1098.001 / T1556 all correctly — and the policy explicitly distinguishes the four offboarding sub-cases that line up with the four techniques. Strong mapping.
- **`posture/sspm-tenant-misconfig-drift.md:11`** uses both T1098.001 + T1098.003 + T1556 + the "broader cloud-misconfiguration class" framing — this is the model the other OAuth policies should follow.
- **`genai/sanctioned-tenant-pinning.md:11`** explicitly distinguishes T1078.004 (personal-account variant) from T1567 (personal-tenant variant of exfil) and references ATLAS AML.T0010 adjacency — good discipline on the SaaS-identity-layer threat model.
- Across the corpus, the `[VERIFY]` markers and explicit "this is illustrative only — not legal/regulatory advice" caveats are appropriately placed. The hygiene around uncertain regulatory clauses is good.

### Notes on Microsoft taxonomy hygiene generally

- Microsoft renamed its full threat-actor taxonomy to a weather-based system in **April 2023**. The current canonical reference is https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming (last updated 2026-06-03 per the page metadata).
- "Storm-XXXX" is a **temporary designation** for groups in development. As groups mature, they get promoted to a named actor (e.g. Storm-0558 → Antique Typhoon).
- Microsoft's policy is to maintain industry-alias mappings in the same page — so the policies *can* legitimately use legacy names (NOBELIUM, APT29, etc.) as parentheticals, but should lead with the Microsoft canonical name in a Microsoft-aligned doc.
- The Microsoft-tracked actor list is dynamic; quarterly review of cited actor names against the threat-actor-naming page is a sensible operational cadence — recommend adding to the policy-library maintenance calendar.
