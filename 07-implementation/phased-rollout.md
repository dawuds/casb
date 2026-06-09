# Phased rollout

> Discovery → risk-scoring → policy enforcement → tuning. Why this order, and what breaks if reversed. Vendor-agnostic; abstracted from the MDA Day 1 / Day 30 / Day 90 pattern.

## The canonical sequence

```
Phase 0  Day 0      Readiness gate (see programme-readiness.md). Stop here if not green.
Phase 1  Weeks 1-4  Discovery + visibility. No enforcement. Build the baseline.
Phase 2  Weeks 4-8  Risk-scoring + sanction taxonomy. Audit-mode policies on highest-risk classes.
Phase 3  Weeks 8-16 Enforcement (block / coach) on highest-confidence policies. Tune.
Phase 4  Months 4-9 Expansion. Lower-confidence policies. BYOD and edge cases.
Phase 5  Ongoing    Steady-state — tuning, exception triage, control attestation cycle.
```

The sequence is not optional. Reversing any pair produces predictable failure modes (named below).

## Phase 1 — Discovery (weeks 1-4)

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

## Phase 2 — Risk-scoring + audit-mode policies (weeks 4-8)

**Goal:** quantify the risk you are about to enforce against, and pilot the highest-confidence policies in audit mode.

| Activity | Output |
|---|---|
| Compute and document risk scores for sanctioned / tolerated apps using a documented methodology (not the vendor's opaque score) | Internal risk-ranking; transparent weights |
| Stand up the OAuth-app governance baseline — review every high-permission, low-community-use, unverified-publisher app | Catalogue with named-malicious bans; manual revoke for confirmed-bad |
| Configure the **alert-only** versions of the high-confidence policies — mass download, impossible travel, OAuth anomaly, terminated-user activity | Audit logs accumulate; SOC builds triage muscle |
| Pilot one inline policy (if CAAC / proxy-mode is available) on one app, one user group, in audit action — e.g. block download of Confidential files to unmanaged devices | Per-app onboarding regression-test data |
| SOC: define alert routing, triage SLA, escalation tree | Documented runbook |

**Done-criteria for moving to Phase 3:** alert-only policies have run for ≥4 weeks; SOC can characterise FP rate per policy; pilot inline policy works on the pilot group; OAuth baseline cleanup complete.

## Phase 3 — Enforcement on highest-confidence policies (weeks 8-16)

**Goal:** flip the highest-confidence audit-mode policies to block / coach. Onboard more sanctioned apps to inline mode.

| Activity | Output |
|---|---|
| Flip to **block** action on policies with measured FP rate ≤2-3% and named exception path | Enforcement live; documented exception process |
| Onboard the next 3-5 sanctioned apps to inline / API mode (one per onboarding sprint) | Per-app regression-test sign-off |
| Stand up the **change-management communications** for end-user-visible policies (block download to BYOD, OAuth ban, etc.) | User-facing comms; help-desk training; FAQ |
| Wire CASB alerts into SIEM with documented field correlation | SIEM dashboards; SOAR playbooks for top-5 alert classes |
| Integrate the offboarding runbook — every termination triggers the cross-SaaS account check (per `04-vendors/microsoft-defender-for-cloud-apps.md` Policy 10) | Offboarding control attested |

**Done-criteria for moving to Phase 4:** enforcement live on ≥5 named policies; documented exception process running; SOC handling alerts within agreed SLA; first audit-evidence pull tested.

## Phase 4 — Expansion + BYOD + edge cases (months 4-9)

**Goal:** roll the working pattern to the long tail. Address BYOD, multi-IdP, GenAI, and the high-FP / low-confidence policies.

| Activity | Output |
|---|---|
| BYOD policy roll-out — read-only access, watermarking via labels, MAM-pair where mobile | BYOD coverage measurable |
| Multi-IdP federation — onboard the Okta-fronted SaaS apps to inline mode (high effort: 2-4 weeks per app) | Multi-IdP coverage |
| GenAI policy — discovery, sanction list, block/coach for unsanctioned, prompt-content DLP for sanctioned (where DSPM-for-AI available) | GenAI policy live |
| Lower-confidence policies — clipboard inspection, share-link governance, geo-residency enforcement | Coverage breadth |
| Documented exception backlog burn-down | Exception count trending down per quarter |

**Done-criteria for moving to Phase 5:** ≥80% of named policies enforcing; BYOD position implemented; multi-IdP coverage closed; exception backlog stable or declining.

## Phase 5 — Steady-state operations

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

## Failure modes from reversing the sequence

| Mistake | What goes wrong |
|---|---|
| **Block-first (skip audit mode)** | Users hit unexpected blocks within the first week, business escalates, programme rolls back, trust lost. Tuning has nothing to tune against |
| **Enforcement before discovery** | You enforce policies on the wrong SaaS scope and the policy library is sized wrong. Long-tail risk uncovered |
| **Discovery without classification** | Dashboard full of apps, nobody decides what to do about any of them. Two years later the dashboard is the deliverable |
| **Skip Phase 5 (steady-state ops)** | Policy drift; FP rate climbs; SOC alert fatigue; the platform becomes the alert noise everyone ignores |
| **No exception process** | Either the platform is bypassed by exec request without record, or business teams build shadow workarounds. Either way, the audit evidence is broken |
| **One big-bang go-live** | Per-app regression failures cascade. Rollback is across many apps simultaneously. Vendor support cannot help fast enough |

## Sequencing rule of thumb

Each phase **earns** the right to the next phase by producing measurable outputs the next phase depends on. If a phase cannot produce its named outputs, do not start the next phase — instead diagnose why the current phase is stuck.

The most common failure is moving to Phase 3 (enforcement) before Phase 2 (audit-mode + measurement) has produced FP-rate data. The programme team flips to block because the audit cycle is looming. The result is firefighting, not a control.
