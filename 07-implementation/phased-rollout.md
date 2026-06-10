# Phased rollout

> Programme-level discipline within a CASB / SSE deployment. Vendor-agnostic; abstracted from the MDA Day 1 / Day 30 / Day 90 pattern.
> Reader audience: programme lead, technical lead.
> Most-directly governs which `03-policies/` files: ALL 19 — phased rollout sequences every policy.

## Purpose

Discovery → risk-scoring → policy enforcement → tuning. Why this order, and what breaks if reversed. The phased rollout is the discipline that gates each policy file's transition from "not enabled" to "enforcing" — without it, the policy library is a wishlist not a control set.

## What organisations use this discipline for

Almost every CASB programme ships in audit-mode initially and ships block-mode policies over months. The phased rollout discipline owns the *which-policy-at-which-phase* decision, the done-criteria gates between phases, and the rollback protocol when a phase fails. Programmes without phasing produce one of two outcomes: block-everything-on-day-one (rolled back in week two) or never-enforce (shelfware by month six). The discipline is mostly the discipline of patience under audit-cycle pressure.

### Use case 1 — Greenfield BFSI rollout, executive sponsorship in place

- **Org type:** tier-1 ASEAN universal bank, ~30k employees, M365 E5 + MDA, mature SOC, BNM RMiT supervised
- **Trigger:** board cyber strategy approved a 12-month CASB programme; programme lead given explicit phased mandate (no shipping block-mode in Q1)
- **Scope:** all 30k users; full 5-phase pattern; per-phase done-criteria reviewed by exec sponsor
- **Outcome:** Phase 1 (discovery) completed in 5 weeks; Phase 2 (audit-mode) ran 6 weeks with first-pass FP-rate baseline (~20% across 4 policies); Phase 3 (enforcement on 5 highest-confidence policies) completed by month 4; Phase 4 (expansion + BYOD) ran through months 4-9; Phase 5 (steady-state) reached month 10. Programme stayed within plan because every phase produced its named output before the next started

### Use case 2 — Recovery rollout, post-rollback

- **Org type:** mid-cap insurance carrier, ~5k employees, M365 E5, prior CASB programme rolled back at month 3 after over-aggressive block deployment
- **Trigger:** new CISO restarted the programme with explicit "phased rollout discipline" mandate; learn from prior failure
- **Scope:** the rollback had broken user trust; recovery started with longer-than-typical audit-mode phase (12 weeks, not 4-8); first block-mode policy delayed to month 5
- **Outcome:** the longer audit-mode phase rebuilt SOC and BU confidence in the policy library; block-mode rollout in month 5 met no resistance because the BU had already seen 12 weeks of audit-mode evidence; FP rate at flip-to-block was <5% (vs ~20% in prior attempt); programme delivered the same outcome as a greenfield rollout would have, three months slower but with no rollback

### Use case 3 — Federated tier-by-tier, BUs progressing independently

- **Org type:** multinational tech firm, ~10k employees across 8 BUs, M365 E5, decentralised culture (each BU operates with significant autonomy)
- **Trigger:** central security team could not get BU-level sign-off for a synchronised rollout; pivoted to per-BU phased rollout with each BU advancing at its own pace
- **Scope:** Engineering BU rolled fastest (audit → enforce in 8 weeks because engineering culture accepts policy-as-code); Marketing rolled slower (16 weeks because the team was less comfortable with policy-driven blocking); HR rolled separately due to workforce-monitoring privacy considerations
- **Outcome:** central programme reported per-BU progress; some BUs reached Phase 4 while others were still in Phase 2; central metrics dashboard normalised the view; trade-off — slower aggregate progress, lower BU-pushback risk; programme avoided the "synchronised-rollback" failure mode where one BU's friction sinks the whole programme

### Use case 4 — Post-M&A acquired-firm rollout

- **Org type:** tier-2 ASEAN bank acquired a regional fintech; ~3.5k acquired-firm employees added to combined tenant; parent firm had mature MDA at Phase 5
- **Trigger:** integration phase; acquired firm had no prior CASB; needed to align with parent baseline without disrupting acquired-firm productivity in transition
- **Scope:** acquired-firm Phase 1 (discovery only) for first 90 days; parallel-track Phase 2 (audit-mode on acquired-firm-specific policies) months 4-6; Phase 3 enforcement months 7-9; full alignment with parent baseline by month 12
- **Outcome:** acquired-firm BYOD posture (initially more permissive than parent) preserved through Phase 2-3, then tightened in Phase 4; sanction-taxonomy alignment was the slowest item (acquired firm had a long tail of small-vendor apps the parent treated as Unsanctioned); programme delivered combined-firm coverage 14 months post-close

## The discipline

### The canonical sequence

```
Phase 0  Day 0      Readiness gate (see programme-readiness.md). Stop here if not green.
Phase 1  Weeks 1-4  Discovery + visibility. No enforcement. Build the baseline.
Phase 2  Weeks 4-8  Risk-scoring + sanction taxonomy. Audit-mode policies on highest-risk classes.
Phase 3  Weeks 8-16 Enforcement (block / coach) on highest-confidence policies. Tune.
Phase 4  Months 4-9 Expansion. Lower-confidence policies. BYOD and edge cases.
Phase 5  Ongoing    Steady-state — tuning, exception triage, control attestation cycle.
```

The sequence is not optional. Reversing any pair produces predictable failure modes (named below).

### Phase 1 — Discovery (weeks 1-4)

**Goal:** know what SaaS your users actually use and what data they put in it.

| Activity | Output |
|---|---|
| Stand up Shadow IT discovery feeds — SWG logs, EDR network telemetry, firewall logs | Cloud Discovery dashboard populated with ≥30 days of data |
| Connect API connectors for the top 5 sanctioned SaaS | Sanctioned-app activity visible per user |
| Identify the top-200 discovered apps and classify into Sanctioned / Tolerated / Unsanctioned | Risk-classified app inventory; sponsor sign-off on tolerated list |
| Identify the regulated-data exposure across the top 10 sanctioned SaaS | Heatmap: PII / PCI / KYC × app |
| Document the BYOD and unmanaged-device population and exposure | BYOD baseline metric |
| Document the OAuth-grant inventory for M365 / Workspace / Salesforce | Catalogue of third-party apps with delegated permissions |

**No enforcement actions in Phase 1.** Discovery alone does not reduce risk; it enables risk decisions. Communicate to users that the platform is in observation mode.

**Done-criteria for moving to Phase 2:** ≥4 weeks of discovery data; ≥80% of top-200 SaaS classified; OAuth inventory exists; BYOD baseline numeric.

### Phase 2 — Risk-scoring + audit-mode policies (weeks 4-8)

**Goal:** quantify the risk you are about to enforce against, and pilot the highest-confidence policies in audit mode.

| Activity | Output |
|---|---|
| Compute and document risk scores for sanctioned / tolerated apps using a documented methodology (not the vendor's opaque score) | Internal risk-ranking; transparent weights |
| Stand up the OAuth-app governance baseline — review every high-permission, low-community-use, unverified-publisher app | Catalogue with named-malicious bans; manual revoke for confirmed-bad |
| Configure the **alert-only** versions of the high-confidence policies — mass download, impossible travel, OAuth anomaly, terminated-user activity | Audit logs accumulate; SOC builds triage muscle |
| Pilot one inline policy (if CAAC / proxy-mode is available) on one app, one user group, in audit action — e.g. block download of Confidential files to unmanaged devices | Per-app onboarding regression-test data |
| SOC: define alert routing, triage SLA, escalation tree | Documented runbook |

**Done-criteria for moving to Phase 3:** alert-only policies have run for ≥4 weeks; SOC can characterise FP rate per policy; pilot inline policy works on the pilot group; OAuth baseline cleanup complete.

### Phase 3 — Enforcement on highest-confidence policies (weeks 8-16)

**Goal:** flip the highest-confidence audit-mode policies to block / coach. Onboard more sanctioned apps to inline mode.

| Activity | Output |
|---|---|
| Flip to **block** action on policies with measured FP rate ≤2-3% and named exception path | Enforcement live; documented exception process |
| Onboard the next 3-5 sanctioned apps to inline / API mode (one per onboarding sprint) | Per-app regression-test sign-off |
| Stand up the **change-management communications** for end-user-visible policies (block download to BYOD, OAuth ban, etc.) | User-facing comms; help-desk training; FAQ |
| Wire CASB alerts into SIEM with documented field correlation | SIEM dashboards; SOAR playbooks for top-5 alert classes |
| Integrate the offboarding runbook — every termination triggers the cross-SaaS account check (per [`../03-policies/detect/terminated-user-cross-saas.md`](../03-policies/detect/terminated-user-cross-saas.md)) | Offboarding control attested |

**Done-criteria for moving to Phase 4:** enforcement live on ≥5 named policies; documented exception process running; SOC handling alerts within agreed SLA; first audit-evidence pull tested.

### Phase 4 — Expansion + BYOD + edge cases (months 4-9)

**Goal:** roll the working pattern to the long tail. Address BYOD, multi-IdP, GenAI, and the high-FP / low-confidence policies.

| Activity | Output |
|---|---|
| BYOD policy roll-out — read-only access, watermarking via labels, MAM-pair where mobile (per [`../03-policies/access-control/block-download-unmanaged-device.md`](../03-policies/access-control/block-download-unmanaged-device.md)) | BYOD coverage measurable |
| Multi-IdP federation — onboard the Okta-fronted SaaS apps to inline mode (high effort: 2-4 weeks per app) | Multi-IdP coverage |
| GenAI policy — discovery, sanction list, block/coach for unsanctioned, prompt-content DLP for sanctioned (per [`../03-policies/genai/discovery-and-auto-unsanction.md`](../03-policies/genai/discovery-and-auto-unsanction.md), [`../03-policies/genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md)) | GenAI policy live |
| Lower-confidence policies — clipboard inspection, share-link governance, geo-residency enforcement | Coverage breadth |
| Documented exception backlog burn-down | Exception count trending down per quarter |

**Done-criteria for moving to Phase 5:** ≥80% of named policies enforcing; BYOD position implemented; multi-IdP coverage closed; exception backlog stable or declining.

### Phase 5 — Steady-state operations

**Goal:** the platform runs. The team runs the platform.

| Activity | Cadence |
|---|---|
| Policy tuning — FP rate review, threshold adjustment | Monthly |
| Exception review — expiry, renewal, removal | Monthly |
| Discovery review — new SaaS in the tail, sanction decisions | Quarterly |
| OAuth grant review — new high-risk apps, ban decisions | Quarterly |
| Control attestation pull — audit-evidence package generation | Per audit cycle |
| Vendor capability tracking — new features (GenAI sub-categories, IRM integration, etc.) | Quarterly |
| Tabletop exercise — incident response on CASB-detected events | Annually |

### Per-policy phase assignment

Each policy in `03-policies/` has a typical phase assignment, summarised:

| Phase | Policies (typical assignment) |
|---|---|
| Phase 1 | All discovery feeds; OAuth-grant baseline |
| Phase 2 | `detect/mass-download-alert`, `detect/impossible-travel-alert`, `oauth/post-consent-cleanup-and-ban`, `access-control/block-unsanctioned-app-with-coach` (audit-mode) |
| Phase 3 | Phase 2 set flipped to enforce; `dlp/auto-label-pci-data`, `dlp/inline-upload-block-regulated-data` (where CAAC ready); `detect/terminated-user-cross-saas` |
| Phase 4 | `access-control/block-download-unmanaged-device`, `access-control/tenant-restriction-corporate-only`, `genai/discovery-and-auto-unsanction`, `genai/inline-prompt-dlp`, `genai/sanctioned-tenant-pinning`, `dlp/external-share-link-quarantine`, `detect/mass-delete-anomaly`, `detect/b2b-partner-exfil-alert`, `oauth/high-scope-grant-alert`, `access-control/geo-residency-block`, `posture/sspm-tenant-misconfig-drift` |
| Phase 5 | Ongoing tuning of all 19 |

The assignment is typical, not mandatory. Programme-specific risk appetite, audit-cycle timing, and BU-readiness can shift policies forward or backward.

## Implementation pattern

The phased-rollout discipline itself is implemented over the readiness-to-Phase-1-start window:

| Week | Activity | Output / gate |
|---|---|---|
| W-2 → W0 | Readiness sign-off (see [`programme-readiness.md`](programme-readiness.md)); locked decisions document signed | Programme charter approved |
| W-1 | Per-phase done-criteria locked with exec sponsor; rollback protocol documented (see [`change-management.md`](change-management.md)) | Phase gates defined |
| W0 | Phase 1 kickoff; SOC + BU briefings | Discovery feeds live |
| W4 | Phase 1 done-criteria review with sponsor; decision: proceed to Phase 2 OR extend Phase 1 | Phase 1 → 2 gate decision recorded |
| W8 | Phase 2 done-criteria review | Phase 2 → 3 gate decision recorded |
| W16 | Phase 3 done-criteria review | Phase 3 → 4 gate decision recorded |
| M9 | Phase 4 done-criteria review | Phase 4 → 5 gate decision recorded |
| Quarterly thereafter | Phase 5 cadence reviews | Steady-state metrics |

Each gate decision is a documented exec-sponsor sign-off, not an implicit "next phase starts". Without the gate discipline, programmes drift between phases with no defensible "we are here because X output existed at the gate" audit evidence.

## Variants

### Industry-specific

- **BFSI:** audit-cycle pressure compresses Phase 2; tendency to flip to enforce before FP-tuning is complete; counter with explicit "no flip-to-enforce without measured <5% FP rate" gate
- **Healthcare:** PHI scope drives Phase 3 priority (DLP policies first); clinical-staff change-management is slow (per [`change-management.md`](change-management.md) Use case 3); longer Phase 4
- **Tech:** engineering BU rolls faster than rest of org; per-BU phasing common; GenAI policy class often Phase 3 not Phase 4 (higher priority for source-code-leak prevention)
- **Retail:** PCI cycle drives DLP policy phase priority; high-turnover workforce creates exception-process strain in Phase 4
- **Manufacturing:** M&A frequency forces per-acquired-firm phasing; supplier-collaboration patterns make B2B-partner-exfil policy (Phase 4) higher priority
- **Higher education / research:** federal-grant compliance drives audit-evidence priority; phasing aligned to grant-cycle reporting
- **Public sector:** Approved-Software-List culture aligns with Sanction-taxonomy phase 1; longer Phase 2 due to procurement-control approval cycles

### Maturity-based

- **Immature:** phases not formally gated; programme advances by calendar rather than evidence; Phase 5 reached with known holes (BYOD-mobile coverage gap, exception backlog growing); programme drift visible by month 9
- **Mature:** documented per-phase done-criteria; exec-sponsor gate reviews; per-policy phase assignment documented; FP-rate measured per policy before flip-to-enforce
- **Advanced:** per-BU phased rollout independently tracked; per-policy phase assignment continuously refined based on Phase 5 telemetry; new-policy introduction follows the same 5-phase pattern even at year 3+; quarterly review of phase-assignment correctness

## Real-world experience

Typical patterns from production phased rollouts:

| Pattern | Frequency |
|---|---|
| Phase 2 compressed under audit-cycle pressure; FP rate at flip-to-enforce >10% | ~50% of programmes |
| Phase 3 done-criteria not met but programme advances anyway | ~40% of programmes |
| Phase 4 BYOD policy deferred indefinitely | ~30% of programmes |
| Phase 5 cadence not maintained; FP rate drifts up by year 2 | ~45% of programmes |
| Per-policy phase assignment shifts during implementation (originally Phase 4 policy pulled forward to Phase 3) | ~60% of programmes |
| Rollback at Phase 3 → 4 transition (one policy class produces broad business-unit pushback) | ~15% of programmes |
| Programme completes all 5 phases on calendar with all gates met | ~15% of programmes |

## Integration with broader programmes

- **All 19 `03-policies/` files:** the per-policy phase assignment governs when each ships
- **`programme-readiness.md`:** Phase 0 is readiness; phased-rollout depends on readiness output
- **`change-management.md`:** Phase 3 + 4 are where change-management bites; communications + exception process must be ready
- **`soc-integration.md`:** Phase 2 requires SOC triage capability; Phase 3 requires SOC SLA; Phase 5 is SOC-led
- **`success-metrics.md`:** per-phase done-criteria are metric-driven; FP rate (M5), TP rate (M4), block rate (M6), exception rate (M7) all gate phase transitions
- **`cost-of-ownership.md`:** phased rollout determines spend trajectory; Year-1 spend is Phases 1-3; Year-2 is Phase 4; Year-3+ is Phase 5
- **`../04-vendors/microsoft-defender-for-cloud-apps.md`:** the MDA Day 1 / 30 / 90 cadence inside the vendor file mirrors this phasing at the vendor-specific level
- **Board reporting:** per-phase progress + gate-decision rationale + next-phase plan; quarterly board cyber report cadence

## Failure modes from reversing the sequence

| Mistake | What goes wrong |
|---|---|
| **Block-first (skip audit mode)** | Users hit unexpected blocks within the first week, business escalates, programme rolls back, trust lost. Tuning has nothing to tune against |
| **Enforcement before discovery** | You enforce policies on the wrong SaaS scope and the policy library is sized wrong. Long-tail risk uncovered |
| **Discovery without classification** | Dashboard full of apps, nobody decides what to do about any of them. Two years later the dashboard is the deliverable |
| **Skip Phase 5 (steady-state ops)** | Policy drift; FP rate climbs; SOC alert fatigue; the platform becomes the alert noise everyone ignores |
| **No exception process** | Either the platform is bypassed by exec request without record, or business teams build shadow workarounds. Either way, the audit evidence is broken |
| **One big-bang go-live** | Per-app regression failures cascade. Rollback is across many apps simultaneously. Vendor support cannot help fast enough |

## Anti-patterns specific to phased rollout

1. **Phase 2 compressed under audit pressure** — the audit cycle is looming; programme flips to enforce before FP-tuning. The audit then catches the over-blocking; programme rolls back; the audit-cycle pressure is now worse
2. **No formal gate decision at phase transitions** — programme drifts between phases; no exec-sponsor evidence trail of "we proceeded because X criteria were met"
3. **Per-policy phase assignment fixed at programme start** — Phase 4 policy whose criticality rises (e.g. GenAI-prompt-DLP after an incident) cannot move forward; programme rigidity defeats responsiveness
4. **"All phases simultaneously"** — programme team tries to ship Phases 1-3 in parallel because of executive pressure; per-policy regression-testing fails because nothing has stabilised
5. **"We skipped Phase 1 because we already knew our SaaS estate"** — readiness inventory was wrong; programme builds on incomplete data; Phase 4 surfaces apps the team thought were not in use
6. **Skipping the rollback protocol** — Phase 3 ships, FP rate is higher than tolerable, no documented rollback; programme either lives with the noise (eroding trust) or rolls back chaotically
7. **No per-BU progress tracking in federated orgs** — central programme reports aggregate progress; individual BUs further behind than visible; aggregate metric hides the failing BU until it surfaces as escalation

## Sequencing rule of thumb

Each phase **earns** the right to the next phase by producing measurable outputs the next phase depends on. If a phase cannot produce its named outputs, do not start the next phase — instead diagnose why the current phase is stuck.

The most common failure is moving to Phase 3 (enforcement) before Phase 2 (audit-mode + measurement) has produced FP-rate data. The programme team flips to block because the audit cycle is looming. The result is firefighting, not a control.

## Cross-references

- [`programme-readiness.md`](programme-readiness.md) — Phase 0
- [`soc-integration.md`](soc-integration.md) — Phase 2-5 SOC dependency
- [`change-management.md`](change-management.md) — Phase 3-4 user-facing dependency
- [`success-metrics.md`](success-metrics.md) — per-phase done-criteria metrics
- [`cost-of-ownership.md`](cost-of-ownership.md) — phased spend trajectory
- [`../03-policies/`](../03-policies/) — all 19 policies; each has typical phase assignment
- [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md) — Day 1 / 30 / 90 mirrors the phasing
- [`../README.md`](../README.md) — repo-level navigation

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
