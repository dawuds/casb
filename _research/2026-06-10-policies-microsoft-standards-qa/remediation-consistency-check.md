# Microsoft-standards QA remediation consistency check - 2026-06-10

Cross-cutting verification across all 19 Tier-2 policy files in `/Users/dawud/claude/casb/03-policies/` after the Microsoft-standards QA remediation pass. Checks 10 of the 32 consolidated drift items from `consolidated-qa-report.md` (`X1`, `X2`, `X4`, `X10`, `X11`, `X12`, `X18`, plus the `Depth:` header convention).

## Summary - cross-cutting items applied successfully vs files still missing the item

| Item | Description | Files applied | Files missing | Verdict |
|---|---|---|---|---|
| **X1a** | CIS Microsoft 365 Foundations Benchmark v5.0.0 citation in `Control mappings` | 19 / 19 | 0 | **CLEAN** |
| **X1b** | Microsoft Secure Score citation in `Control mappings` | 19 / 19 | 0 | **CLEAN** |
| **X2** | File-policy ceiling = 50 (not 200) in two affected DLP files | 2 / 2 | 0 | **CLEAN** |
| **X4** | 5-way GenAI sub-category split removed (recast as Microsoft-anti-pattern caveat) | 3 / 3 GenAI files | 0 | **CLEAN** |
| **X10a** | `MIDNIGHT BLIZZARD` all-caps removed from operative content | 19 / 19 | 0 | **CLEAN** |
| **X10b** | `TA453` removed from operative content (replaced with `Mint Sandstorm`) | 19 / 19 | 0 | **CLEAN** |
| **X11** | `ML Inference API` removed from operative content (replaced with `AI Inference API`) | 19 / 19 | 0 | **CLEAN** |
| **X12** | `eDiscovery (Premium)` removed from operative content (replaced with `Purview eDiscovery`) | 19 / 19 | 0 | **CLEAN** |
| **X18** | "Practitioner-observed ranges" FP-trajectory header line | 19 / 19 (semantic) | 0 | **CLEAN with stylistic drift** (2 files use lowercase `practitioner-observed ranges` inside a parenthetical instead of the canonical capital-P standalone phrase from the spec) |
| **Depth** | `Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)` line | 19 / 19 | 0 | **CLEAN** |

**Headline: 19 / 19 files clean on every cross-cutting item checked. Zero files require an additional remediation pass.**

The only residual deviation is a stylistic inconsistency in the X18 trajectory-table header on `detect/mass-delete-anomaly.md` and `detect/terminated-user-cross-saas.md` (lowercase `practitioner-observed ranges` inside a parenthetical, rather than the spec-mandated capital-P phrase `"Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline."` as a standalone preface). The semantic content (practitioner caveat + the validate-against-baseline qualifier) is present in both — this is a hygiene-only nit, not a content drift.

## Per-cross-cutting-item verification

### X1a — CIS Microsoft 365 Foundations Benchmark v5.0.0 citation in `Control mappings`

**Spec (QA report §X1):** Add a `CIS Microsoft 365 Foundations Benchmark v5.0.0` row to every file's `Control mappings` block; this is the single highest-impact audit-defensibility change because every BNM / SOC 2 / PCI / ISO auditor will probe for it and it was previously absent from all 19 files.

| File | Status |
|---|---|
| `access-control/block-download-unmanaged-device.md` | PASS |
| `access-control/block-unsanctioned-app-with-coach.md` | PASS |
| `access-control/geo-residency-block.md` | PASS |
| `access-control/tenant-restriction-corporate-only.md` | PASS |
| `dlp/api-data-at-rest-quarantine.md` | PASS |
| `dlp/auto-label-pci-data.md` | PASS |
| `dlp/external-share-link-quarantine.md` | PASS |
| `dlp/inline-upload-block-regulated-data.md` | PASS |
| `detect/b2b-partner-exfil-alert.md` | PASS |
| `detect/impossible-travel-alert.md` | PASS |
| `detect/mass-delete-anomaly.md` | PASS |
| `detect/mass-download-alert.md` | PASS |
| `detect/terminated-user-cross-saas.md` | PASS |
| `genai/discovery-and-auto-unsanction.md` | PASS |
| `genai/inline-prompt-dlp.md` | PASS |
| `genai/sanctioned-tenant-pinning.md` | PASS |
| `oauth/high-scope-grant-alert.md` | PASS |
| `oauth/post-consent-cleanup-and-ban.md` | PASS |
| `posture/sspm-tenant-misconfig-drift.md` | PASS |

**Verdict:** 19 / 19 applied. CLEAN.

### X1b — Microsoft Secure Score citation in `Control mappings`

**Spec (QA report §X1):** Add a `Microsoft Secure Score` row to every file's `Control mappings` block, referencing the current four-group structure (Identity / Device / Apps / Data — not the legacy five-group structure with Infrastructure).

**Verdict:** 19 / 19 applied. CLEAN. (Same file list as X1a; both citations were applied in the same pass.)

### X2 — File-policy ceiling = 50 (not 200)

**Spec (QA report §X2):** Microsoft Learn `data-protection-policies` Limitations section states "You're limited to 50 file policies in Defender for Cloud Apps." The earlier "200" figure circulated in practitioner notes is stale.

| File | Status | Evidence |
|---|---|---|
| `dlp/api-data-at-rest-quarantine.md` | PASS | L87 (`File-policy ceiling = 50 per tenant`), L144 (`50-policy tenant ceiling`), L245 (`Exceed the 50-file-policy tenant ceiling`) |
| `dlp/auto-label-pci-data.md` | PASS | L239 (`File-policy ceiling = 50 per tenant ... the earlier '200' figure circulated in practitioner notes is stale`) — also recomputed the "3-5 policies" budget against the 50-ceiling per the canonical fix |

No remaining `200 file policies` / `200 file-policy` references found in either file.

**Verdict:** 2 / 2 applied. CLEAN.

### X4 — 5-way GenAI sub-category split REMOVED

**Spec (QA report §X4):** The 5-way practitioner split (LLM / AI Coding Assistant / AI Image Generator / AI Video Generator / AI Voice-Avatar) is not in the published Microsoft taxonomy; the `risk-score` page lists only `Generative AI` plus adjacent `AI – MCP Server` and `AI – Model Provider`. Practitioners must rely on custom App tags rather than Microsoft sub-categories. Rewrite Anti-pattern #4 in `genai/discovery-and-auto-unsanction.md`.

| File | Status | Evidence |
|---|---|---|
| `genai/discovery-and-auto-unsanction.md` | PASS | L89 (catalog-taxonomy note: 5-way split explicitly flagged as not Microsoft-published), L269 (Anti-pattern #4 rewritten: "Assume the published category taxonomy splits Generative AI into sub-categories" + correction note at 2026-06-10 QA pass), L233-234 (residual sub-category mentions are now in fictional / illustrative content with custom-App-tag framing) |
| `genai/inline-prompt-dlp.md` | PASS | L53 (catalog-taxonomy note), L292 (Anti-pattern: "Rely on Microsoft sub-category split ... that split is not in the current Microsoft Cloud App Catalog taxonomy") |
| `genai/sanctioned-tenant-pinning.md` | PASS (n/a) | File does not depend on the sub-category split |

The remaining mentions of the five-way list in `discovery-and-auto-unsanction.md` and `inline-prompt-dlp.md` are explicit denials of the taxonomy ("**not** in the published Microsoft taxonomy") or anti-pattern callouts — not operative claims. This is the canonical-fix shape.

**Verdict:** 3 / 3 GenAI files applied. CLEAN.

### X10a — No remaining `MIDNIGHT BLIZZARD` all-caps in operative content

**Spec (QA report §X10):** Use `Midnight Blizzard` (Title Case per Microsoft's April 2023 naming refresh).

**Operative-content sweep:** Only one occurrence of `MIDNIGHT BLIZZARD` remains in the corpus — at `detect/mass-delete-anomaly.md` L254, where it appears inside a pedagogical sentence: *"**Midnight Blizzard** is Title Case per Microsoft's April 2023 naming refresh (not 'MIDNIGHT BLIZZARD')."* This is the explanation note, not stale operative content — keeping the old form here is required to make the correction comprehensible to readers.

**Verdict:** All operative MIDNIGHT BLIZZARD references replaced with Midnight Blizzard. CLEAN.

### X10b — No remaining `TA453` in operative content

**Spec (QA report §X10):** Use `Mint Sandstorm` (Microsoft canonical; TA453 is Proofpoint taxonomy).

**Operative-content sweep:** Two occurrences of `TA453` remain — at `detect/impossible-travel-alert.md` L38 and `oauth/post-consent-cleanup-and-ban.md` L33. Both are inside parenthetical pedagogical notes stating that *"'TA453' is the Proofpoint taxonomy for an overlapping cluster — per Microsoft Learn, the Microsoft canonical name should be used"* and *"related Iranian-nexus consent-phishing activity is tracked by Microsoft as **Mint Sandstorm**, not the Proofpoint label TA453"*. These are explanation notes, not stale operative content.

**Verdict:** All operative TA453 references replaced with Mint Sandstorm. CLEAN.

### X11 — No remaining `ML Inference API` in operative content

**Spec (QA report §X11):** ATLAS AML.T0024 renamed from "ML Inference API" to "AI Inference API". Find/replace across `genai/discovery-and-auto-unsanction.md` lines 11, 162; `genai/inline-prompt-dlp.md` lines 11, 199.

**Operative-content sweep:** Only one occurrence of `ML Inference API` remains in the corpus — at `genai/inline-prompt-dlp.md` L314, inside the 2026-06-10 changelog entry: *"corrected ATLAS AML.T0024 name from 'ML Inference API' to 'AI Inference API'"*. This is the lineage note explaining the rename, not stale operative content.

**Verdict:** All operative ML Inference API references replaced with AI Inference API. CLEAN.

### X12 — No remaining `eDiscovery (Premium)` in operative content

**Spec (QA report §X12):** Replace "Purview eDiscovery (Premium)" with "Purview eDiscovery" (the new eDiscovery experience in the Purview portal); note classic experiences retired 2025-08-31. Cite `ediscovery-overview`.

**Operative-content sweep:** Only two occurrences of `eDiscovery (Premium)` remain in the corpus — both in pedagogical / lineage context:
- `dlp/auto-label-pci-data.md` L215: *"(Note: classic eDiscovery experiences were retired 2025-08-31; use 'Purview eDiscovery' rather than the legacy 'eDiscovery (Premium)' framing.)"* — this is the retirement note.
- `genai/inline-prompt-dlp.md` L314: changelog entry — *"replaced 'eDiscovery (Premium)' with 'Purview eDiscovery' + retirement note"*.

Both are required to preserve the correction's audit trail.

**Verdict:** All operative eDiscovery (Premium) references replaced with Purview eDiscovery. CLEAN.

### X18 — `Practitioner-observed ranges` header on FP-rate trajectory tables

**Spec (QA report §X18):** Add a single header line to every FP-rate trajectory table: `"Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline."`

| File | Status | Header form used |
|---|---|---|
| `access-control/block-download-unmanaged-device.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `access-control/block-unsanctioned-app-with-coach.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `access-control/geo-residency-block.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `access-control/tenant-restriction-corporate-only.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `dlp/api-data-at-rest-quarantine.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `dlp/auto-label-pci-data.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `dlp/external-share-link-quarantine.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `dlp/inline-upload-block-regulated-data.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `detect/b2b-partner-exfil-alert.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `detect/impossible-travel-alert.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `detect/mass-delete-anomaly.md` | **PASS (stylistic drift)** | inline lowercase: `Typical FP-rate trajectory (**practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.**):` |
| `detect/mass-download-alert.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `detect/terminated-user-cross-saas.md` | **PASS (stylistic drift)** | inline lowercase: `Typical FP-rate trajectory (**practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.**):` |
| `genai/discovery-and-auto-unsanction.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `genai/inline-prompt-dlp.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `genai/sanctioned-tenant-pinning.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `oauth/high-scope-grant-alert.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `oauth/post-consent-cleanup-and-ban.md` | PASS (canonical) | `Practitioner-observed ranges` |
| `posture/sspm-tenant-misconfig-drift.md` | PASS (canonical) | `Practitioner-observed ranges` |

**Verdict:** 19 / 19 semantically compliant. Two files (`detect/mass-delete-anomaly.md`, `detect/terminated-user-cross-saas.md`) use the lowercase-inside-parenthetical form rather than the spec-mandated standalone capital-P preface phrase. Content carries the full practitioner-observed + validate-against-baseline qualifiers — the deviation is purely stylistic. Optional follow-up: harmonise to the standalone-capital-P preface in these two files for visual consistency across the corpus.

### Depth — `Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)`

**Spec:** Every file's Depth line should read `Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)`.

**Format observed:** `> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.` (blockquote-prefixed, used uniformly across the corpus).

**Verdict:** 19 / 19 applied. CLEAN.

## Files cleared (all P0/P1/P2 items applied)

All 19 files cleared on the 10 cross-cutting items checked in this pass:

1. `access-control/block-download-unmanaged-device.md`
2. `access-control/block-unsanctioned-app-with-coach.md`
3. `access-control/geo-residency-block.md`
4. `access-control/tenant-restriction-corporate-only.md`
5. `dlp/api-data-at-rest-quarantine.md`
6. `dlp/auto-label-pci-data.md`
7. `dlp/external-share-link-quarantine.md`
8. `dlp/inline-upload-block-regulated-data.md`
9. `detect/b2b-partner-exfil-alert.md`
10. `detect/impossible-travel-alert.md`
11. `detect/mass-delete-anomaly.md` (stylistic X18 nit — see below)
12. `detect/mass-download-alert.md`
13. `detect/terminated-user-cross-saas.md` (stylistic X18 nit — see below)
14. `genai/discovery-and-auto-unsanction.md`
15. `genai/inline-prompt-dlp.md`
16. `genai/sanctioned-tenant-pinning.md`
17. `oauth/high-scope-grant-alert.md`
18. `oauth/post-consent-cleanup-and-ban.md`
19. `posture/sspm-tenant-misconfig-drift.md`

## Files needing additional pass (some items missed)

**None on the 10 cross-cutting items checked.**

Two files (`detect/mass-delete-anomaly.md`, `detect/terminated-user-cross-saas.md`) have a stylistic — not substantive — deviation on X18 (lowercase inline form rather than capital-P standalone preface). The semantic content matches the spec on both. Not a blocking item.

## Recommendations - any leftover items requiring follow-up

### 1. Optional: harmonise X18 header style across detect/* (low-priority hygiene)

`detect/mass-delete-anomaly.md` (L185) and `detect/terminated-user-cross-saas.md` (L199) write the practitioner-observation caveat inline as a bold parenthetical inside the table preface (`Typical FP-rate trajectory (**practitioner-observed ranges; ...**):`), where the other 17 files use the standalone capital-P preface phrase from the canonical fix in QA report §X18. Semantic content identical; visual consistency would benefit from harmonising to the standalone capital-P preface used elsewhere. Not blocking.

### 2. Scope clarification: this check covered 10 of 32 consolidated cross-cutting items

This consistency check verified the items the task brief named explicitly: **X1, X2, X4, X10, X11, X12, X18, and Depth**. The remaining 22 cross-cutting items in the QA report (X3, X5–X9, X13–X17, X19–X32) require separate per-file verification — they were not in scope here. Notable items recommended for a follow-up sweep:

- **X3** (June 2025 anomaly-policy disablement boilerplate across `detect/*`) — high-impact, multi-file.
- **X5** (Claude Enterprise / ChatGPT Enterprise capability-matrix corrections across three `genai/*` files) — auditor-visible.
- **X13** (SIT parameter naming + governance-action API names across four DLP files) — auditor-visible.
- **X14** (App Governance "Predictive Risk" branding resolution in `oauth/high-scope-grant-alert.md`) — flagged for significant rework by three specialists; highest-impact single rewrite.
- **X9** (concentration-risk attribution sentences across four files).
- **X16, X17** (architectural-ownership reframings for SUSPEND decision and Tenant Restrictions v2).
- **X25** (T1530 removal from `inline-upload-block-regulated-data.md`).

Spot-checks during this pass confirmed several of these items are reflected in the corpus already — e.g., the changelog entry in `genai/inline-prompt-dlp.md` L314 enumerates Adaptive Protection, SIT-parameter rename, Claude/ChatGPT capability-matrix correction, Browser DLP / Endpoint DLP cross-reference, Japan East citation, portal-path standardisation, and the FP-trajectory header all as applied in the same 2026-06-10 pass; `oauth/post-consent-cleanup-and-ban.md` L0083 references App Governance "behavioural-anomaly detection (dynamic detection model, June 2025)" rather than the unverified "Predictive Risk" branding; `detect/mass-delete-anomaly.md` L85 marks the activity-policy auto-disable thresholds as `[VERIFY against the current activity-policies Microsoft Learn page]` per the X6 canonical fix. A targeted second-pass sweep on the remaining 22 cross-cutting items would close the loop.

### 3. No regressions detected

Pedagogical references to the old terminology (MIDNIGHT BLIZZARD, TA453, ML Inference API, eDiscovery (Premium), the 5-way GenAI split) appear only in explicit anti-pattern callouts, correction notes, or the 2026-06-10 changelog entry. These are intentional and required to preserve the audit trail of the rename — not regressions.
