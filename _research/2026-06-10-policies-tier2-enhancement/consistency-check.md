# Tier-2 consistency check - 2026-06-10

## Summary

- **Files reviewed:** 19 (17 enhanced + 2 exemplars — `auto-label-pci-data.md`, `mass-download-alert.md`)
- **Files at full Tier-2 schema compliance:** 17 of 19
- **Files with schema gaps:** 2
  - `access-control/block-unsanctioned-app-with-coach.md` — **Tier 1 only; never enhanced.** Missing `Depth:` line and all 7 Tier-2 sections.
  - `dlp/api-data-at-rest-quarantine.md` — **Tier 1 only; never enhanced.** Missing `Depth:` line and all 7 Tier-2 sections.
  - `dlp/external-share-link-quarantine.md` — **Tier 1 only; never enhanced.** Missing `Depth:` line and all 7 Tier-2 sections.

Actually three files are skeletal. Updated count: **16 of 19 enhanced; 3 outstanding.**

- **Use-case archetype overlaps flagged:** 6 distinct overlapping archetypes across 11 files
- **Broken cross-references:** 8 distinct links (many appear in multiple files)
- **Vendor-marketing-language slips:** 3 files (minor — mostly the word "robust" / "seamless" not present; "comprehensive" appears in a few; details below)
- **FP-trajectory plausibility flags:** 2 files (look pattern-copied not policy-tuned)
- **Industry-variant thin spots:** 1 file with thinner-than-peers Public-sector / Legal variant
- **Worked-config placeholder/incomplete flags:** 1 file (high-scope-grant-alert uses `<ISV-app-id-1>` placeholders correctly; no other placeholders found)

## Schema compliance gaps

- `access-control/block-unsanctioned-app-with-coach.md` — **Tier 1 only.** No `Depth:` declaration. Missing all 7 Tier-2 sections: *What organisations use this for*, *Implementation pattern*, *Worked configuration example*, *Variants*, *Real-world FP experience*, *Integration with broader programmes*, *Anti-patterns specific to this policy*.

- `dlp/api-data-at-rest-quarantine.md` — **Tier 1 only.** No `Depth:` declaration. Missing all 7 Tier-2 sections.

- `dlp/external-share-link-quarantine.md` — **Tier 1 only.** No `Depth:` declaration. Missing all 7 Tier-2 sections.

The other 16 files (including the 2 exemplars) carry the full Tier-2 schema — Status / Depth / Required capabilities / Deployment-mode requirement / MDA playbook reference + Purpose + What organisations use this for + Implementation pattern + Action + Scope + Vendor implementation grid + Worked configuration example + Variants (Industry + Maturity) + Control mappings + False-positive risk + Real-world FP experience + Operational cost + Privacy / data-protection + Integration with broader programmes + Anti-patterns + Coverage gaps + Three-lens sign-off.

## Use-case diversity issues

The use-case archetypes are repeated heavily across files — by design, since they reflect the same archetypal organisations being looked at from different policy angles. But there are flagged overlaps where the framing is near-identical instead of differentiated.

**Archetype 1 — "Tier-1 ASEAN universal bank, ~30k employees, M365 E5, BNM RMiT supervised, post-Storm-2372 / post-incident response"**

Appears as Use case 1 in:
- `detect/mass-download-alert.md` (Use case 4 — IRM integration)
- `detect/impossible-travel-alert.md` (Use case 1 — credential-stuffing botnet)
- `detect/mass-delete-anomaly.md` (Use case 1 — Storm-0501 peer incident)
- `detect/terminated-user-cross-saas.md` (Use case 1 — Storm-2372 OAuth cleanup)
- `detect/b2b-partner-exfil-alert.md` (Use case 1 — Storm-2372 OAuth-cleanup wave)
- `oauth/post-consent-cleanup-and-ban.md` (Use case 1 — Storm-2372 mass-cleanup)
- `oauth/high-scope-grant-alert.md` (Use case 1 — Storm-2372 service-principal sprawl)
- `genai/discovery-and-auto-unsanction.md` (Use case 1 — post-Storm-2372 consumer-AI clamp)
- `genai/inline-prompt-dlp.md` (Use case 1 — ChatGPT Enterprise rollout with developer pressure)
- `genai/sanctioned-tenant-pinning.md` (Use case 1 — post-personal-account ChatGPT exfil)
- `access-control/tenant-restriction-corporate-only.md` (Use case 1 — post-personal-ChatGPT incident)
- `access-control/block-download-unmanaged-device.md` (Use case 1 — BYOD flexibility pilot)
- `access-control/geo-residency-block.md` (Use case 1 — BNM cross-border data-flow inquiry)
- `posture/sspm-tenant-misconfig-drift.md` (Use case 1 — post-Storm-2372-class OAuth cleanup)

This is **14 of 16 enhanced files using the same archetype as Use case 1.** It is reasonable that this archetype is the most common deployment target, but the framing is interchangeable: same employee count (~30k), same MDA tier (E5), same regulator (BNM RMiT). The triggers differentiate well (Storm-0501, Storm-2372, board-ask, audit-finding) but the *org type* line is nearly identical across files. Recommend rotating in some BNM/MAS-SG-anchored variants (Maybank / OCBC / DBS-class wealth) or scaling some down to "tier-2 commercial bank" framing for variety. Currently the reader cannot distinguish whether the 14 cases are the *same* tier-1 bank surfacing different incidents over 24 months (which would be one type of cohesion) or 14 *different* tier-1 banks (which is the intended diversity).

**Archetype 2 — "Merchant acquirer operating across MY / SG / HK / TH; in-scope for PCI DSS + BNM RMiT + MAS TRM + HKMA SA-2"**

Appears as Use case 2 (or 3) in:
- `detect/mass-download-alert.md` — implicit (use case 4 cards-ops)
- `detect/impossible-travel-alert.md` (Use case 3 — Storm-2372 OAuth cleanup)
- `detect/mass-delete-anomaly.md` (Use case 4 — Storm-2372)
- `detect/b2b-partner-exfil-alert.md` (Use case 2)
- `oauth/post-consent-cleanup-and-ban.md` (Use case 2 — TA453-class)
- `dlp/auto-label-pci-data.md` (Use case 2)
- `dlp/inline-upload-block-regulated-data.md` (Use case 1)
- `genai/discovery-and-auto-unsanction.md` (Use case 2 — pre-PCI DSS v4.0)
- `genai/sanctioned-tenant-pinning.md` (Use case 3 — multi-jurisdiction CHD)
- `access-control/tenant-restriction-corporate-only.md` (Use case 2 — Storm-2372)
- `access-control/block-download-unmanaged-device.md` (Use case 2 — executive iPad)
- `access-control/geo-residency-block.md` (Use case 3 — sovereign-data overlay)

12 of 16. Same comment — same employee count (~4k-6k), same regulators. Different triggers are good (QSA finding, board ask, partner-side breach, vendor-collab incident), but the org-type framing is near-identical.

**Archetype 3 — "Digital-native challenger bank / neobank, ~500-800 employees, M365 E5, pre-IPO posture"**

Appears in:
- `detect/mass-download-alert.md` — implicit
- `detect/mass-delete-anomaly.md` (Use case 2)
- `detect/impossible-travel-alert.md` (Use case 2)
- `detect/terminated-user-cross-saas.md` (Use case 2 — M&A integration)
- `detect/b2b-partner-exfil-alert.md` (Use case 4)
- `oauth/post-consent-cleanup-and-ban.md` (Use case 3)
- `oauth/high-scope-grant-alert.md` (Use case 2 — Power Platform)
- `dlp/auto-label-pci-data.md` (Use case 3)
- `dlp/inline-upload-block-regulated-data.md` (Use case 3)
- `genai/discovery-and-auto-unsanction.md` (Use case 3)
- `genai/sanctioned-tenant-pinning.md` (Use case 2)
- `access-control/tenant-restriction-corporate-only.md` (Use case 3)
- `access-control/block-download-unmanaged-device.md` (Use case 4)

13 of 16. The "neobank ~600-800 employees, fast growth, founder-led engineering, pre-IPO" framing is interchangeable. Triggers differentiate (pre-IPO DD, Series-C raise, internal-IAM-review, contractor-departing-incident, citizen-developer review). Recommend at least 2-3 files swap this archetype for **regional bank / building society / cooperative bank** archetype for diversity.

**Archetype 4 — "Pharma R&D" / "Healthcare insurer" / "Mid-size healthcare provider"**

Used relatively sparingly and **well**: `detect/mass-download-alert.md` (pharma R&D), `detect/mass-delete-anomaly.md` (healthcare provider), `genai/inline-prompt-dlp.md` (healthcare insurer). No overlap concerns.

**Archetype 5 — "Consulting / professional-services firm, ~5k-15k consultants, project-based work"**

Appears in:
- `detect/mass-download-alert.md` (Use case 3)
- `detect/mass-delete-anomaly.md` (Use case 4)
- `detect/b2b-partner-exfil-alert.md` (Use case 3)
- `genai/inline-prompt-dlp.md` (Use case 4)
- `access-control/tenant-restriction-corporate-only.md` (Use case 4)

5 of 16. Triggers differentiate well (insider risk programme, OAuth incident, audit observation, codename leak, client contractual ask). Acceptable.

**Archetype 6 — "Tier-2 regional / tier-2 BFSI / mid-cap pre-audit"**

Appears in `dlp/auto-label-pci-data.md` (use case 4 M&A), `dlp/inline-upload-block-regulated-data.md` (use case 4 M&A), `oauth/post-consent-cleanup-and-ban.md` (use case 4), `oauth/high-scope-grant-alert.md` (use case 3 + 4), `posture/sspm-tenant-misconfig-drift.md` (use case 2 + 4), `access-control/block-download-unmanaged-device.md` (use case 3). 6 files using "tier-2 BFSI / mid-cap regional" — acceptable diversity.

**Recommended pruning** — at least 4 files should rotate Use case 1 away from the standard tier-1-ASEAN-30k-BNM-RMiT framing. Candidates: `detect/impossible-travel-alert.md` already does *partially* (its use case 2 is full-remote neobank); `genai/sanctioned-tenant-pinning.md` already has the tier-1-European-EU-AI-Act-DORA framing as Use case 4 — could become Use case 1 in a different file. Recommend rotating 3-4 Use case 1 slots to use the European DORA archetype or a different ASEAN regulator-led archetype.

## Cross-reference broken links

The following cross-references resolve to files that **do not exist** in `08-failure-modes/`:

| Referencing file | Broken link | Existing file actually meant |
|---|---|---|
| `genai/discovery-and-auto-unsanction.md` | `../../08-failure-modes/byod-bypass.md` | `byod-and-unmanaged-coverage-gap.md` (already exists) |
| `genai/sanctioned-tenant-pinning.md` | `../../08-failure-modes/cae-bypass.md` | _no equivalent — needs to be created_ |
| `access-control/tenant-restriction-corporate-only.md` | `../../08-failure-modes/cae-session-bypass.md` | _no equivalent — needs to be created_ |
| `access-control/tenant-restriction-corporate-only.md` | `../../08-failure-modes/oauth-application-permissions-gap.md` | `oauth-blind-spot.md` (closest existing match) |
| `detect/b2b-partner-exfil-alert.md` | `../../08-failure-modes/xtas-drift.md` | _no equivalent — needs to be created_ (the file flags `[verify path]`) |
| `detect/b2b-partner-exfil-alert.md` | `../../08-failure-modes/user-type-misclassification.md` | _no equivalent — needs to be created_ (flags `[verify path]`) |
| `detect/impossible-travel-alert.md` | `../../08-failure-modes/token-replay-bypass.md` | _no equivalent — needs to be created_ (flags `[link-target may not yet exist]`) |
| `genai/inline-prompt-dlp.md` | `shadow-ai-discovery.md` and `auto-unsanction-genai.md` (same directory) | `discovery-and-auto-unsanction.md` (single existing file covers both) |

Note: many files flag broken targets with `[VERIFY path]` / `[verify path]` / `[link-target may not yet exist]` so the broken state is at least surfaced — the writer was honest. But the links should resolve.

The references to existing `_failure-modes/` files (`api-mode-is-not-prevention.md`, `byod-and-unmanaged-coverage-gap.md`, `encrypted-upload-bypass.md`, `oauth-blind-spot.md`, `ssl-tls-inspection-breakage.md`, `over-blocking-and-user-circumvention.md`) all resolve correctly.

The intra-policy cross-references (e.g. `mass-download-alert.md` referencing `auto-label-pci-data.md`, `inline-upload-block-regulated-data.md` referencing `auto-label-pci-data.md`) all resolve.

## British English / tone issues

Most files maintain British English consistently (organisation, recognise, behaviour, sanction-as-noun, etc.). No "leverage" / "robust" / "best-in-class" / "scalable" / "seamless" in technical body. Some softer slips:

- **"comprehensive"** appears 2x in `genai/sanctioned-tenant-pinning.md` (advanced-maturity narrative) and 1x in `detect/terminated-user-cross-saas.md`. Acceptable in context but worth a global pass to replace with something more specific (e.g. "all-vendor coverage", "all-app coverage").

- **"continuous monitoring"** appears in `posture/sspm-tenant-misconfig-drift.md` Use case 2 outcome and Control mappings — this is the correct SOC 2 terminology so it stays.

- **"end-to-end"** appears in several files (`dlp/auto-label-pci-data.md`, `oauth/high-scope-grant-alert.md`, `access-control/block-download-unmanaged-device.md`). It is a workable phrase in deployment-pattern context; not a vendor-marketing red flag.

- No "leveraging", "robust", "scalable", "best practice", "seamless", "industry-leading", "world-class" detected.

- **Em-dashes** vs **en-dashes** — consistent use of em-dashes for inline asides. Good.

- The exemplars (`auto-label-pci-data.md`, `mass-download-alert.md`) set the tone — direct, named-trade-offs, honest about limitations. The enhanced 16 files maintain this tone; one minor exception: `oauth/high-scope-grant-alert.md` Use case 1 outcome states "Steady-state ~3-5 alerts/week on credential additions; ~80% are legitimate ISV key-rotation events" — that 80% number is unflagged for being illustrative. Consider adding `[illustrative]` or matching the exemplar pattern.

## FP-trajectory plausibility flags

The FP-rate trajectory tables are well-customised in most files. Specifically:

- **Policy-specific (good)** — `dlp/auto-label-pci-data.md` (60-80% → 25-35% → 10-15% → 5-10% → 2-5%; cause column references mock-cards, BIN lists, custom SITs); `detect/mass-download-alert.md` (50-70% → 25-40% → 12-20% → 5-15% → 3-8%; service-accounts, content-migrations); `genai/inline-prompt-dlp.md` (40-60% → 20-30% → 12-18% → 8-15% → 5-10%; vendor-default SIT confidence, custom-codename allowlist).

- **Looks pattern-copied (flag)** — `access-control/block-download-unmanaged-device.md` (35-50% → 20-30% → 10-15% → 5-10% → 3-7%) — mid-range numbers very close to `dlp/inline-upload-block-regulated-data.md` (45-65% → 20-30% → 10-18% → 6-12% → 4-8%). The triggers / causes differ (mislabel + iPad executive vs SSN regex), but the trajectory band is highly similar. Possibly because both are inline DLP-class CAAC policies — defensible — but worth a sanity check.

- **Looks pattern-copied (flag)** — `genai/discovery-and-auto-unsanction.md` (30-50% → 15-25% → 8-15% → 5-10% → 3-7%) vs `posture/sspm-tenant-misconfig-drift.md` (60-80% → 25-40% → 12-20% → 6-12% → 4-8%) vs `oauth/post-consent-cleanup-and-ban.md` (50-70% → 25-40% → 12-20% → 5-12% → 3-8%) — three different policy classes that converge to similar W4-W8 bands. The W1 spread (30-80%) does differentiate them — alert-only discovery starts low, inventory-against-stale-state starts high. The W12/steady-state band of 3-8% is a slight motif across all SaaS-management policies; this is plausible but worth confirming the underlying causes are policy-specific in each case (they are, on review).

Net: no file is overtly copy-pasted. Two clusters worth a final read for differentiation — but acceptable as is.

## Industry-variant thin spots

**Strong variants (full differentiation):**
- `dlp/auto-label-pci-data.md` — BFSI / Healthcare / Tech / Retail / Public sector all named with concrete differentiation (different SIT classes per industry).
- `detect/mass-download-alert.md` — BFSI / Pharma / Tech / Consulting / Legal each with concrete threshold + workflow differentiation.
- `access-control/tenant-restriction-corporate-only.md` — BFSI / Healthcare / Tech / Retail / Public sector / Legal with named partner-allowlist patterns per vertical.
- `oauth/post-consent-cleanup-and-ban.md` — BFSI / Healthcare / Tech / Retail / Public sector with concrete scope-tier + ban-decision differentiation.
- `genai/inline-prompt-dlp.md` — BFSI / Healthcare / Tech / Retail / Legal / Public sector all carry distinct SIT class + exception-group framing.

**Adequate variants:**
- `detect/mass-delete-anomaly.md` — BFSI / Healthcare / Tech / Retail / Public sector. Public sector framing is solid (records-disposal-authorised user group). OK.
- `detect/impossible-travel-alert.md` — solid; Public sector / Legal differentiated as cleaner FP baseline + sharper auditor expectation.
- `genai/sanctioned-tenant-pinning.md` — strong; Public sector calls out sovereign-AI residency + no-personal-device discipline.
- `posture/sspm-tenant-misconfig-drift.md` — solid; Public sector / Legal differentiates on government-classification overlay + privilege-protection on iManage / NetDocuments.
- `oauth/high-scope-grant-alert.md` — solid; Healthcare names PHI/BAA, Tech notes inverted threat model.
- `detect/b2b-partner-exfil-alert.md` — solid; Healthcare names HIE/BAA, Legal names matter-coordinator workflow.
- `detect/terminated-user-cross-saas.md` — Healthcare (clinician portal access), Tech (GitHub EMU + PATs), Legal (consultant lifecycle) all concrete. OK.
- `access-control/geo-residency-block.md` — Healthcare / Public sector strong (telemedicine exception, sovereign-cloud procurement).
- `access-control/block-download-unmanaged-device.md` — Healthcare strong (clinician iPad + MAM-near-universal); Tech notes contractor-laptop FP fight; Legal eDiscovery exception path; Public sector mentions classification-marking integration.
- `dlp/inline-upload-block-regulated-data.md` — strong; Legal names matter-coordinator allowlist + DPIA expansion.
- `genai/discovery-and-auto-unsanction.md` — strong; Public sector calls out sovereignty + in-jurisdiction infrastructure.

**Thin spots:**
- None below the standard. The Tier-2 enhanced files all carry concrete per-vertical differentiation. The two main flag-worthy spots:
  - `genai/discovery-and-auto-unsanction.md` Retail variant is brief (3 lines) vs BFSI variant (5 lines + AML.T0010 callout). Acceptable but thinner.
  - `oauth/high-scope-grant-alert.md` Retail and Legal variants are each 2 lines while BFSI / Healthcare / Tech each get 3-4 lines. Minor.

## Worked-configuration quality flags

All 16 enhanced files have substantive worked-configuration blocks (YAML-ish property-list, 25-150 lines, vendor-anonymised but practitioner-recognisable). Specific quality flags:

- **All good** — `dlp/auto-label-pci-data.md`, `detect/mass-download-alert.md` (the exemplars), and all 14 other enhanced files have complete worked-configurations with realistic values, named owners, and named exception groups.

- **Placeholder convention consistent** — `<ISV-app-id-1>` / `<workspace-id>` / `<tenant-id-redacted>` / `<enterprise-org-id>` are used consistently as anonymisation tokens. No "TODO" / "TBD" / "FIXME" leakage. No `your-tenant-here` placeholders that look unfinished.

- **One minor flag** — `dlp/auto-label-pci-data.md` worked-config has `daily_alert_limit: 500` but no `daily_alert_limit` value in `detect/mass-download-alert.md` (100). Different policy classes — not a problem — but worth noting that the exemplars themselves don't establish a single convention.

- **One observation, not a flag** — `genai/sanctioned-tenant-pinning.md` worked-config is the longest at ~100 lines and includes 5 distinct layers (Microsoft-endpoints / ChatGPT / Claude / Gemini-blocked / mobile-native). It is the most ambitious and the most useful as a reference. Could become a model for cross-vendor policy worked-configs more broadly.

- The three skeletal files (`block-unsanctioned-app-with-coach.md`, `api-data-at-rest-quarantine.md`, `external-share-link-quarantine.md`) have no worked-configuration block (they are Tier 1).

## Top 5 files requiring revision

In priority order, where "revision" means schema work or material consistency cleanup before promotion:

1. **`access-control/block-unsanctioned-app-with-coach.md`** — **Tier 1 only.** Either promote to Tier 2 (add the 7 sections) or explicitly flag in the Status block that it remains Tier 1 with "Tier 2 expansion outstanding" per the schema's `_schema.md` convention. Currently silent on depth, which mis-classes it against the other 16 enhanced files.

2. **`dlp/api-data-at-rest-quarantine.md`** — **Tier 1 only.** Same issue as above. Given it is the API-mode companion to the inline upload-block policy (one of the deepest enhanced files), the depth-asymmetry is jarring. Either promote or flag.

3. **`dlp/external-share-link-quarantine.md`** — **Tier 1 only.** Same. Two-stage notify-then-quarantine policy is a meaty topic that warrants Tier 2 — likely highest-leverage promotion candidate of the three.

4. **`access-control/tenant-restriction-corporate-only.md`** — strong file. The flag is the broken cross-references (`cae-session-bypass.md`, `oauth-application-permissions-gap.md`) — either fix the link targets or create the failure-mode pages. Also Use case 4 — minor consideration of rotating the archetype.

5. **`detect/b2b-partner-exfil-alert.md`** — broken cross-references (`xtas-drift.md`, `user-type-misclassification.md`) flagged with `[verify path]` in-line; either create the targets or remove the references. Otherwise solid.

Honourable mentions outside top 5:
- `genai/inline-prompt-dlp.md` — fix the `shadow-ai-discovery.md` + `auto-unsanction-genai.md` links to point to the single existing `discovery-and-auto-unsanction.md`.
- `genai/discovery-and-auto-unsanction.md` — fix the `byod-bypass.md` link to point to existing `byod-and-unmanaged-coverage-gap.md`.
- `detect/impossible-travel-alert.md` — create the `token-replay-bypass.md` failure-mode page, since the policy is honest about the bypass gap and the reader needs the link.

## Files cleared for promotion

In effect — schema-compliant, cross-references resolve (or are flagged honestly inline), British English consistent, FP trajectories policy-specific, industry variants concrete, worked-configs substantive:

1. `dlp/auto-label-pci-data.md` (exemplar)
2. `detect/mass-download-alert.md` (exemplar)
3. `dlp/inline-upload-block-regulated-data.md`
4. `detect/mass-delete-anomaly.md`
5. `detect/terminated-user-cross-saas.md`
6. `oauth/post-consent-cleanup-and-ban.md`
7. `oauth/high-scope-grant-alert.md`
8. `genai/sanctioned-tenant-pinning.md`
9. `access-control/block-download-unmanaged-device.md`
10. `access-control/geo-residency-block.md`
11. `posture/sspm-tenant-misconfig-drift.md`

The following are *substantively* Tier-2 complete but carry minor broken-link or use-case-archetype-overlap issues that should be addressed before promotion:

12. `detect/impossible-travel-alert.md` — one broken link (`token-replay-bypass.md`)
13. `detect/b2b-partner-exfil-alert.md` — two broken links
14. `access-control/tenant-restriction-corporate-only.md` — two broken links
15. `genai/inline-prompt-dlp.md` — two broken links (intra-genai)
16. `genai/discovery-and-auto-unsanction.md` — one broken link

The three Tier 1 holdovers (`block-unsanctioned-app-with-coach.md`, `api-data-at-rest-quarantine.md`, `external-share-link-quarantine.md`) are **not cleared** — they either need Tier-2 promotion work or an explicit Tier-1 status declaration.

**Net: 11 clean for promotion as-is; 5 substantively Tier-2 with link-fixes outstanding; 3 not enhanced (still Tier 1 at v0.0).**
