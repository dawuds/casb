# Change management

> CASB rollouts fail more often on people than on technology. End-user impact, exception process, communications, business-unit onboarding. Vendor-agnostic.

## What end-users will actually notice

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

## Communications plan template

### Before the policy ships

| Channel | Audience | Content |
|---|---|---|
| **All-staff email + intranet** | Everyone affected | Policy intent (the *why*, not the *how*). Effective date. What to do if blocked. Exception path |
| **Team-lead briefing** | Department heads | Specific use-cases their team will hit; pre-emptive exception requests |
| **Help-desk briefing** | Frontline IT support | Scripted FAQ; triage decision tree; escalation path |
| **Business-app-owner briefing** | Per-SaaS owners | Per-app onboarding plan; regression-test coordination |
| **Privacy / HR / Legal briefing** | Risk + people | Especially for policies that inspect personal content (clipboard, downloads); data-handling implications |

### When the policy ships

| Channel | Content |
|---|---|
| **Block / coaching message** | The error the user sees. **Always include**: (a) why this is blocked, (b) what they can do (re-label, sign in from a managed device, use an alternative), (c) how to request an exception, (d) a link to the policy intent |
| **Help-desk tracker** | Real-time view of related tickets so the team can spot policy-misconfiguration vs user-education vs genuine block |
| **Policy state log** | Audit-trail-grade record of *when* the policy flipped from audit to block |

### After the policy ships

| Cadence | Activity |
|---|---|
| **Week 1** | Daily review with the help desk + SOC. Tune the message text; tune the exclusion lists; identify the FP patterns |
| **Week 2-4** | Move to weekly review; close the immediate-aftermath exception spike |
| **Month 2+** | Move to monthly tuning; track the trend |

## Exception process

The exception process is **not optional**. Every block-mode policy needs one. Programmes that say "no exceptions" find that exceptions happen anyway — they just happen off-the-record.

### Exception structure

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

### Exception escalation

| Tier | Authority | Latency |
|---|---|---|
| **L1 — Help desk grant** | Help desk lead | Same-day, for low-risk exceptions (e.g. add a known-good app to the allowed-AI list) |
| **L2 — Risk lead grant** | Programme risk lead | ≤48h, for medium-risk exceptions |
| **L3 — CISO grant** | CISO | ≤5 days, for high-risk or precedent-setting exceptions |
| **L4 — Executive override** | Exec sponsor + CISO + Risk Committee | Per-case; minutes captured |

Programmes without tiered authorisation push every exception to a single bottleneck (typically the CISO), creating either a backlog or rubber-stamping.

## Business-unit onboarding

| Stage | Activity | Output |
|---|---|---|
| **Engage** | Business unit owner identifies their critical SaaS apps and data classes | Per-BU SaaS inventory with sponsor sign-off |
| **Pilot** | Selected user group in the BU runs with the policy in audit mode | FP-rate data per BU; identified workflows that break |
| **Tune** | Per-BU exclusion lists, per-BU exception requests, per-BU communications | Tuned policy scope for the BU |
| **Roll out** | Policy flips to block for the BU's users | BU lead signs off after week 1 stable |
| **Steady state** | Quarterly BU review of CASB-affected workflows | Continuous tuning, no surprises at audit |

BU onboarding done in parallel across many BUs at once produces firefighting. BU-by-BU sequencing produces a tunable rhythm.

## Workforce-privacy posture

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

## Communications tone

| Avoid | Prefer |
|---|---|
| "We are deploying a new security tool to monitor your activity" | "We are adding a control that prevents accidental data sharing — here's what changes for you" |
| "Your downloads will be blocked from personal devices" | "Confidential documents now require a managed device. If you don't have one, [enrolment link] or contact [help desk]" |
| "This is required by compliance" | "This reduces the risk of [named risk]. Here's the business case [link]" |
| "Contact your manager for exceptions" | "Use [exception form link]. Most requests are decided within 48 hours" |

Plain language, named risk, named action, named timeline. Compliance-speak triggers user circumvention culture; risk-speak triggers cooperation.

## Roll-back protocol

For every policy that ships in block mode, a roll-back protocol must exist before go-live:

| Trigger | Action | Authority |
|---|---|---|
| **>5% of affected user base hits the block in week 1** | Flip policy to audit mode pending tuning | Programme lead |
| **Named business-critical workflow broken** | Targeted exclusion + tuning | Risk lead |
| **Executive escalation** | Flip to audit; review at executive sponsor level | Exec sponsor |
| **Regulator question** | Programme freeze + Legal involvement | CISO + Legal |

A documented roll-back is not weakness; it's resilience. Programmes without roll-back protocols escalate every issue to platform-removal pressure.

## Anti-patterns

| Anti-pattern | Why it fails |
|---|---|
| **Surprise go-live** | Users hit blocks with no context; first call is to the CEO's office |
| **"No exceptions" doctrine** | Exceptions happen off-record; audit trail breaks; control evidence breaks |
| **One-size-fits-all policy** | Finance, Engineering, HR have different legitimate workflows; uniform policies break all three |
| **Help desk untrained** | First-line support tells users to "submit a ticket" — adversarial relationship between IT and users |
| **No metrics on user friction** | Block-rate trends are invisible; programme can't tell coaching from over-blocking |
| **Comms in compliance-speak** | Users tune out; pushback at first error |
