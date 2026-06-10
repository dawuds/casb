# Change management

> Programme-level discipline within a CASB / SSE deployment. Vendor-agnostic.
> Reader audience: programme lead, communications lead, BU sponsors, HR / Legal liaison.
> Most-directly governs which `03-policies/` files: all user-visible policies — [`access-control/`](../03-policies/access-control/) (4 files); user-facing [`dlp/`](../03-policies/dlp/) policies; [`genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md); auto-suspend-class [`detect/`](../03-policies/detect/) policies.

## Purpose

CASB rollouts fail more often on people than on technology. This discipline owns the human side — end-user impact assessment, communications planning, exception process design, business-unit onboarding sequence, workforce-privacy posture, and roll-back protocol. Programmes that ship policies without this discipline produce the workaround culture documented in [`../08-failure-modes/over-blocking-and-user-circumvention.md`](../08-failure-modes/over-blocking-and-user-circumvention.md).

## What organisations use this discipline for

Change management is the discipline that distinguishes a tooling deployment from a programme. The technology side is mostly tractable; the human side is where most CASB programmes fall over. The discipline owns three operational artefacts: the communications plan (before / during / after every block-mode policy), the exception process (tiered, time-bounded, evidence-trail-preserving), and the roll-back protocol (when to back out a policy, who decides, on what evidence). The hardest single decision is the personal-use posture — most-deferred decision in readiness, most-political decision in implementation, and the source of the dominant rollback scenarios.

### Use case 1 — First major block-action rollout, BYOD download policy

- **Org type:** tier-2 ASEAN bank, ~12k employees, M365 E5 + MDA, post-readiness with BYOD-block-from-Confidential decision locked
- **Trigger:** Phase 3 enforcement; the first block-mode policy was [`access-control/block-download-unmanaged-device.md`](../03-policies/access-control/block-download-unmanaged-device.md); programme team knew this would be user-facing on day one
- **Scope:** 6-week change-management sprint prior to go-live — all-staff email + intranet + BU-lead briefings + help-desk training + exception-process documentation; coaching message design with named alternative ("Sign in from a managed device" with enrolment link)
- **Outcome:** go-live had ~4% of in-scope users hit the block in week 1 (mostly BYO-laptop senior executives); ~80% of week-1 blocks resolved by user enrolling their device into Intune (the named alternative worked); ~15% via exception-process for legitimate-BYOD-only scenarios; ~5% required executive-sponsor escalation; programme rolled forward with no rollback; the discipline-following programme established BU trust for subsequent Phase-3 and Phase-4 policy ships

### Use case 2 — GenAI policy rollout, most-political policy class

- **Org type:** digital-native challenger bank, ~800 employees, M365 E5 + ChatGPT Enterprise, no prior GenAI controls
- **Trigger:** post-incident; engineering team had been pasting source code into personal ChatGPT for 3+ months; board approved sanctioned ChatGPT Enterprise contingent on a control to prevent the personal-account flow continuing
- **Scope:** rollout of [`access-control/tenant-restriction-corporate-only.md`](../03-policies/access-control/tenant-restriction-corporate-only.md) + [`genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md) + AUP update; communications campaign emphasising "you can use ChatGPT, just the Enterprise variant"; DPIA refreshed; works-council consultation (EU operations)
- **Outcome:** ~95% migration to ChatGPT Enterprise in 4 weeks; ~5% persistent attempts (BYOD off-network) handled via coaching → AUP-escalation tiered process; the discipline of "name the sanctioned alternative + frame as 'still available, just different login'" was the cultural unlock; programme-team observation: 4 of 5 user complaints in week 1 were about the AUP-policy ambiguity (later resolved by AUP rewrite), not the technical block

### Use case 3 — Post-M&A integration, acquired-firm users hit new policies

- **Org type:** tier-1 ASEAN bank acquired a regional fintech; ~3.5k acquired-firm users had not previously used CASB-controlled SaaS; parent firm had mature CASB at Phase 5
- **Trigger:** integration phase; combined-tenant rollout required acquired-firm users to come under parent's policy library; previously-permissive acquired-firm posture would tighten
- **Scope:** acquired-firm-specific change-management programme — 90-day audit-mode period with weekly business-impact metrics; town-hall briefings emphasising "parent firm has lived with these policies for 3 years"; per-policy phased rollout (sanction-taxonomy alignment first, then BYOD, then GenAI)
- **Outcome:** acquired-firm cultural integration took 12+ months; the change-management discipline reduced rollback risk by sequencing policy alignment instead of forcing day-one parity; help-desk peak was at month 4 (when DLP policies hit) rather than month 1; programme survived the integration cycle

### Use case 4 — Post-rollback recovery, second-attempt rollout

- **Org type:** mid-cap insurance carrier, ~5k employees, M365 E5, prior CASB programme rolled back at month 3 after over-aggressive block deployment
- **Trigger:** new CISO restarted the programme with explicit "change-management discipline first" mandate
- **Scope:** rebuilt change-management discipline before re-enabling any block-mode policy — exception process redesigned with L1-L4 tiered authority (originally one bottleneck), communications-template library reviewed by communications team (originally CISO-drafted with no comms input), roll-back protocol documented in writing (originally implicit)
- **Outcome:** second rollout had no rollback events; the BU trust restored over 6 months; programme team observation: the discipline-building work was 8 weeks of investment that paid back in 18 months of programme stability; the discipline-first approach is now the firm's documented baseline for any IT-control rollout

## The discipline

### What end-users will actually notice

| Policy class | User experience |
|---|---|
| **Discovery only (Phase 1)** | Invisible. No user impact |
| **Audit-mode policies (Phase 2)** | Invisible to users; visible to SOC analysts |
| **Coaching messages** | User sees a banner / pop-up explaining policy intent; can continue (with audit log) |
| **Block download to unmanaged device** | User sees an error or block page when attempting download from BYOD. Major frustration if unannounced |
| **Block paste to GenAI** | User pastes into ChatGPT, paste is blocked, user sees a message. Frustration vs education |
| **OAuth app banned** | App stops working for everyone who consented. **Worst end-user impact** unless announced ahead |
| **Sensitivity-label auto-applied** | File downloads start carrying encryption — invisible until the user shares externally and the recipient cannot open |
| **Account suspended on mass-download anomaly** | User locked out, often during a legitimate task. **Highest blast radius** — break-glass support required |

The user impact rank-orders the communications priority. The bottom three rows each need a documented heads-up before the policy ships.

### Communications plan template

#### Before the policy ships

| Channel | Audience | Content |
|---|---|---|
| **All-staff email + intranet** | Everyone affected | Policy intent (the *why*, not the *how*). Effective date. What to do if blocked. Exception path |
| **Team-lead briefing** | Department heads | Specific use-cases their team will hit; pre-emptive exception requests |
| **Help-desk briefing** | Frontline IT support | Scripted FAQ; triage decision tree; escalation path |
| **Business-app-owner briefing** | Per-SaaS owners | Per-app onboarding plan; regression-test coordination |
| **Privacy / HR / Legal briefing** | Risk + people | Especially for policies that inspect personal content (clipboard, downloads); data-handling implications |

#### When the policy ships

| Channel | Content |
|---|---|
| **Block / coaching message** | The error the user sees. **Always include**: (a) why this is blocked, (b) what they can do (re-label, sign in from a managed device, use an alternative), (c) how to request an exception, (d) a link to the policy intent |
| **Help-desk tracker** | Real-time view of related tickets so the team can spot policy-misconfiguration vs user-education vs genuine block |
| **Policy state log** | Audit-trail-grade record of *when* the policy flipped from audit to block |

#### After the policy ships

| Cadence | Activity |
|---|---|
| **Week 1** | Daily review with the help desk + SOC. Tune the message text; tune the exclusion lists; identify the FP patterns |
| **Week 2-4** | Move to weekly review; close the immediate-aftermath exception spike |
| **Month 2+** | Move to monthly tuning; track the trend |

### Exception process

The exception process is **not optional**. Every block-mode policy needs one. Programmes that say "no exceptions" find that exceptions happen anyway — they just happen off-the-record.

#### Exception structure

| Field | Detail |
|---|---|
| **Exception ID** | Unique, traceable |
| **Requester** | User; manager (signs off) |
| **Policy** | Which policy is being excepted |
| **Scope** | User / group / app / data-class / time-window |
| **Justification** | Business reason in plain language |
| **Compensating control** | What replaces the bypassed control — e.g. read-only access via alternative session policy, manual review of downloaded files, time-bounded exception |
| **Approver** | Risk lead (default); CISO for high-risk classes |
| **Expiry** | All exceptions expire. 90-day default; longer requires re-approval |
| **Review** | Monthly review of active exceptions; expired exceptions auto-revoke |

#### Exception escalation

| Tier | Authority | Latency |
|---|---|---|
| **L1 — Help desk grant** | Help desk lead | Same-day, for low-risk exceptions (e.g. add a known-good app to the allowed-AI list) |
| **L2 — Risk lead grant** | Programme risk lead | ≤48h, for medium-risk exceptions |
| **L3 — CISO grant** | CISO | ≤5 days, for high-risk or precedent-setting exceptions |
| **L4 — Executive override** | Exec sponsor + CISO + Risk Committee | Per-case; minutes captured |

Programmes without tiered authorisation push every exception to a single bottleneck (typically the CISO), creating either a backlog or rubber-stamping.

### Business-unit onboarding

| Stage | Activity | Output |
|---|---|---|
| **Engage** | Business unit owner identifies their critical SaaS apps and data classes | Per-BU SaaS inventory with sponsor sign-off |
| **Pilot** | Selected user group in the BU runs with the policy in audit mode | FP-rate data per BU; identified workflows that break |
| **Tune** | Per-BU exclusion lists, per-BU exception requests, per-BU communications | Tuned policy scope for the BU |
| **Roll out** | Policy flips to block for the BU's users | BU lead signs off after week 1 stable |
| **Steady state** | Quarterly BU review of CASB-affected workflows | Continuous tuning, no surprises at audit |

BU onboarding done in parallel across many BUs at once produces firefighting. BU-by-BU sequencing produces a tunable rhythm.

### Workforce-privacy posture

CASB controls that inspect content (clipboard inspection, DLP scanning, OAuth grant surveillance, DSPM-for-AI prompt visibility) sit on the workforce-monitoring fence. The privacy posture must be documented before deployment.

| Concern | Posture must answer |
|---|---|
| **What is logged** | Per-policy: what data is captured, where it is stored, who can read it, for how long |
| **Notice to employees** | Acceptable Use Policy update; employee handbook; new-hire onboarding signature |
| **Personal use** | Whether the firm permits personal SaaS use on corporate devices/networks. If permitted, scope of inspection on personal traffic. PDPA MY 2024 / GDPR Art. 88 / HKPCPD employer-monitoring guidance all bite — illustrative, not legal advice; confirm with assurance lead |
| **Worker representation** | In jurisdictions with works councils / unions, consultation requirements |
| **Subject rights** | DSR / Subject Access Request handling for CASB-collected data |
| **DPIA / TIA** | High-risk processing (CAAC reverse-proxy of employee browser sessions, DSPM-for-AI prompt capture) requires a Data Protection Impact Assessment under GDPR Art. 35 and equivalent regimes. Transfer Impact Assessment if data crosses borders |

Programmes that skip this end up with: deploy → HR pushback → policy pulled → trust lost. Or: deploy → regulator question → mitigation under pressure → audit finding.

### Communications tone

| Avoid | Prefer |
|---|---|
| "We are deploying a new security tool to monitor your activity" | "We are adding a control that prevents accidental data sharing — here's what changes for you" |
| "Your downloads will be blocked from personal devices" | "Confidential documents now require a managed device. If you don't have one, [enrolment link] or contact [help desk]" |
| "This is required by compliance" | "This reduces the risk of [named risk]. Here's the business case [link]" |
| "Contact your manager for exceptions" | "Use [exception form link]. Most requests are decided within 48 hours" |

Plain language, named risk, named action, named timeline. Compliance-speak triggers user circumvention culture; risk-speak triggers cooperation.

### Roll-back protocol

For every policy that ships in block mode, a roll-back protocol must exist before go-live:

| Trigger | Action | Authority |
|---|---|---|
| **>5% of affected user base hits the block in week 1** | Flip policy to audit mode pending tuning | Programme lead |
| **Named business-critical workflow broken** | Targeted exclusion + tuning | Risk lead |
| **Executive escalation** | Flip to audit; review at executive sponsor level | Exec sponsor |
| **Regulator question** | Programme freeze + Legal involvement | CISO + Legal |

A documented roll-back is not weakness; it's resilience. Programmes without roll-back protocols escalate every issue to platform-removal pressure.

## Implementation pattern

Typical 4-6 week change-management sprint per major policy go-live; for the first major policy, a 6-8 week discipline-build-out is common:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Policy-impact assessment — name affected user populations, identify potentially-breaking workflows, design coaching message | Impact assessment signed by BU lead |
| W2 | Communications-plan drafting — all-staff email, BU-lead briefing deck, help-desk FAQ, business-app-owner briefing, Privacy/HR/Legal briefing | Communications draft reviewed by Communications team + Legal |
| W2-W3 | Privacy / HR / Legal review — DPIA / TIA / works-council consultation where required; AUP update drafted | Privacy posture signed by DPO; AUP update approved |
| W3 | Exception-process design — tier-1/2/3/4 authority, expiry, review cadence; exception-register tooling | Exception process operational |
| W4 | Help-desk + business-app-owner briefings; runbook review | Help-desk training complete; BU sign-off on the policy |
| W4-W5 | All-staff email + intranet announcement — typically T-7 days before go-live | Communications shipped |
| W5 | Go-live (in audit-mode for the first 1-2 weeks if not yet baselined) | Policy live; first-week alert volume baselined |
| W6 | Daily-cadence triage with help desk + SOC; exception-process throughput measured | Operational baseline; tuning needs identified |
| W7+ | Weekly cadence then monthly tuning; quarterly cross-BU review | Sustainable change-management cadence |

The W1 policy-impact assessment is the diagnostic step. Without it, communications target the wrong audiences and the coaching message names the wrong alternative.

## Variants

### Industry-specific

- **BFSI:** Privacy / Legal / works-council consultation extensive (PDPA MY 2024 + EU GDPR + jurisdiction-specific worker-rep regimes); BU-lead briefings include line-of-business risk-owners; AUP review and update is a board-attentive event
- **Healthcare:** HIPAA Privacy Officer + clinical-staff-rep consultation; clinical workflows resist disruption (lives-at-stake framing); change-management slower (12-16 weeks for major policies); PHI-aware coaching messages
- **Tech:** lighter change-management overhead (engineering-culture tolerance for policy-as-code); developer-facing communications channels (Slack / Discord rather than all-staff email); faster cadence (4-week sprints)
- **Retail:** high-turnover workforce means more onboarding-burden on coaching messages; communications channels include front-line training programmes; PCI cycle drives the priority
- **Manufacturing:** OT/IT boundary creates dual-track change-management (different stakeholder audiences); M&A frequency creates rolling change-management programmes
- **Higher education / research:** academic-freedom framing creates resistance to GenAI policies; communications channels include faculty governance bodies; longer pilot phases
- **Public sector:** union consultation rigorous; AUP changes go through formal review processes; longer cadence (6-12 months for AUP refresh)

### Maturity-based

- **Immature:** policies ship without communications plan; users hit blocks unannounced; help-desk floods; rollback within weeks; trust lost across BU; programme reputational damage
- **Mature:** documented communications-template library; tiered exception process with named authorities; quarterly tuning cadence; DPIA / TIA refreshed annually; roll-back protocol documented in writing
- **Advanced:** change-management discipline institutionalised as a Programme function (not ad-hoc); per-policy change-impact assessments stored as audit artefacts; BU-lead change-management champions in every business unit; privacy-by-design integrated into policy-design rather than reactive; communications-effectiveness measured (post-rollout survey + help-desk-ticket-rate trend)

## Real-world experience

Typical patterns from production change-management programmes:

| Pattern | Frequency |
|---|---|
| First major policy ships without communications plan; rolled back in week 1-2 | ~20% of programmes |
| Help-desk training scripted FAQ provides "no answer" for one specific common scenario; help-desk tickets stack | ~50% of programmes (W1 phenomenon, usually fixed in W2) |
| Exception process collapses to single-bottleneck (typically CISO); exception backlog builds within 4 weeks | ~40% of programmes lacking tiered authority |
| AUP update not synchronised with policy go-live; users hit block before AUP supports it; Legal / HR conflict | ~30% of programmes |
| Communications team not engaged before policy go-live; messaging is compliance-speak; user resistance | ~45% of programmes |
| Roll-back protocol not documented; first roll-back is chaotic; subsequent policies delayed by trust deficit | ~25% of programmes (~80% of first-rollback events become trust-deficit events) |
| Privacy / DPO sign-off treated as form-filling; first user complaint becomes regulator-notice scenario | ~10% of programmes; 100% of those become programme-pause events |

## Integration with broader programmes

- **`programme-readiness.md`:** locked-decisions document is the change-management input (BYOD position, personal-use position, exception process)
- **`phased-rollout.md`:** change-management runs alongside Phase 3 + 4; gates per-policy flip-to-enforce
- **`soc-integration.md`:** SOC triage of week-1 alerts feeds change-management tuning; exception-process triage requires SOC + change-mgmt coordination
- **`success-metrics.md`:** M6 (block decision rate), M7 (exception rate) are change-management-managed; trends drive cadence decisions
- **`cost-of-ownership.md`:** change-management staffing (Communications team time, exception-process L1-L4 capacity) is an operational-headcount line item
- **All user-visible `03-policies/` files:** every block-action policy has a change-management dependency; programme-team workflow is policy-design → change-mgmt review → readiness gate → ship
- **AUP / employee handbook / works-council:** change-management owns the AUP update cycle; new policy = AUP review trigger
- **DPIA / TIA:** change-management owns the privacy-posture-refresh cycle; new content-inspection policy = DPIA refresh
- **HR escalation runbook:** repeated personal-AI / personal-account / pre-resignation patterns from CASB feed HR escalation; change-mgmt owns the discipline boundary between policy-tuning and HR-action
- **Board cyber report:** quarterly user-experience narrative (block-rate trend, exception-rate trend, help-desk-impact trend); board sensitivity to user-friction signals

## Anti-patterns specific to change management

1. **Surprise go-live** — users hit blocks with no context; first call is to the CEO's office
2. **"No exceptions" doctrine** — exceptions happen off-record; audit trail breaks; control evidence breaks
3. **One-size-fits-all policy** — Finance, Engineering, HR have different legitimate workflows; uniform policies break all three
4. **Help desk untrained** — first-line support tells users to "submit a ticket" — adversarial relationship between IT and users
5. **No metrics on user friction** — block-rate trends are invisible; programme can't tell coaching from over-blocking
6. **Comms in compliance-speak** — users tune out; pushback at first error
7. **AUP update delayed** — policy lives in CASB before AUP supports it; user complaints are legitimate
8. **Single-bottleneck exception process** — every exception escalates to CISO; backlog accumulates; users circumvent
9. **Privacy posture deferred** — DPO sign-off treated as form-filling; first user complaint becomes regulator-notice event
10. **No measurement of communications effectiveness** — post-go-live survey not run; help-desk-ticket-rate not analysed; change-mgmt iterates without data
11. **Coaching message lacks named alternative** — "use an approved tool" instead of "use ChatGPT Enterprise at [link]"; users have no path forward; circumvent to consumer alternative
12. **Roll-back protocol verbal not written** — when the moment comes, programme team disagrees on what triggers rollback; chaotic backout damages trust

## Cross-references

- [`programme-readiness.md`](programme-readiness.md) — locked decisions feed change-management
- [`phased-rollout.md`](phased-rollout.md) — change-mgmt gates flip-to-enforce
- [`soc-integration.md`](soc-integration.md) — SOC alert triage feeds change-mgmt tuning
- [`success-metrics.md`](success-metrics.md) — M6 / M7 are change-mgmt-managed
- [`cost-of-ownership.md`](cost-of-ownership.md) — change-mgmt staffing cost
- [`../03-policies/access-control/`](../03-policies/access-control/) — user-visible block policies
- [`../03-policies/dlp/inline-upload-block-regulated-data.md`](../03-policies/dlp/inline-upload-block-regulated-data.md) — content-inspection privacy posture
- [`../03-policies/genai/inline-prompt-dlp.md`](../03-policies/genai/inline-prompt-dlp.md) — clipboard-inspection DPIA
- [`../08-failure-modes/over-blocking-and-user-circumvention.md`](../08-failure-modes/over-blocking-and-user-circumvention.md) — the failure mode this discipline prevents
- [`../README.md`](../README.md) — repo-level navigation

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
