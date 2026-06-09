# Over-blocking and user circumvention

> Aggressive enforcement creates workaround culture. Workarounds defeat the control more reliably than any technical bypass.

## The failure

The programme team, under audit-cycle pressure or risk-mitigation enthusiasm, ships block-mode policies without tuning, exception process, or business engagement. End-users hit blocks on legitimate work. Within weeks, the user population has built workarounds:

- Personal-device upload of work data (because the work device is blocked)
- Personal-email forwarding of files (because the SaaS share is blocked)
- USB / external-drive sharing (because the cloud share is blocked)
- Phone-photographing screens (because the file download is blocked)
- Shadow Discord / WhatsApp channels (because Teams is monitored)

Each workaround is outside any CASB control. The block-mode policy didn't prevent the data movement; it just changed the channel.

## What enables it

| Mechanism | Why it kicks in |
|---|---|
| **No coaching / no audit-mode phase before block** | User hits a block with zero prior warning; their first impression is "IT is broken" |
| **No exception process** | User can't get unblocked through legitimate means; circumvents |
| **No business engagement** | Department head wasn't consulted; reports user complaints to executive; programme rolled back or workarounds entrenched |
| **One-size-fits-all policy** | Sales / Finance / Engineering have different legitimate workflows; uniform block breaks one of them |
| **Block-rate metrics not tracked** | Programme can't tell coaching success from over-blocking |
| **High friction on the legitimate path** | Even when there's a sanctioned alternative, friction drives the user back to the easier-bad path |

## Symptoms

| Symptom | Severity |
|---|---|
| Spike in help-desk tickets in week 1 of a policy | Normal — handle via documented FAQ and rapid tuning |
| Repeated tickets from same user / team | Tuning needed — policy is hitting a real workflow |
| Department head escalation to CISO / sponsor | Communications failure — engage the BU |
| Drop in sanctioned-SaaS usage after a block ships | Users may have moved to a workaround; investigate |
| Anomaly in personal-email forwarding (from M365 alerts) | Direct evidence of circumvention |
| Discovery shows new Shadow IT after a block — same data class | Direct evidence of circumvention |
| Anonymous internal feedback complaining about "IT lockdown" | Cultural symptom — block-first programme without engagement |

## Which programmes exhibit it

Programmes that:
- Skip Phase 2 audit-mode (per [`../07-implementation/phased-rollout.md`](../07-implementation/phased-rollout.md))
- Skip the exception process (per [`../07-implementation/change-management.md`](../07-implementation/change-management.md))
- Don't measure block-rate per policy
- Don't track help-desk ticket volume per policy
- Treat the CASB as a tooling decision rather than a policy + change-management decision

This is a process failure more than a vendor failure. Every CASB vendor's platform can be deployed well or poorly.

## Why the failure exists

Security teams under audit-cycle pressure choose visible enforcement (blocks) over invisible enforcement (controls that don't generate user-visible friction). Visible enforcement is easier to demonstrate to an auditor or executive: "We have a policy. It is blocking. Done."

But the goal of the control is data protection, not block-rate metrics. The block is a *mechanism*, not the *goal*. When the block fires on legitimate work, the user circumvents, the data still moves, and the audit posture is worse — the firm now has both an over-blocking finding AND an unmonitored exfil channel.

## What compensates

| Discipline | What it does |
|---|---|
| **Audit-mode → coach → block sequencing** | User sees the policy operating in non-blocking form; learns the behaviour; coaching message educates; block flips only after measured FP rate is low |
| **Documented exception process with tiered authority** | User has a legitimate path; doesn't need to circumvent |
| **Per-BU pilot before broad rollout** | BU head's team experiences the policy first; BU head signs off |
| **Block-rate metric tracking per policy** | Programme sees trend; rising block rate signals tuning failure or unanticipated workflow |
| **End-user comms in risk-speak, not compliance-speak** | "This reduces the risk of [X]. If you need to [Y], use [Z]" beats "This is required by compliance" |
| **Discovery monitoring after each block ships** | New apps appearing in Shadow IT in the same data class as the just-blocked sanctioned SaaS = direct circumvention signal |
| **Anonymous channel for user feedback** | Workarounds surface earlier when users can flag friction without naming themselves |

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "We've reduced data loss by 80% via blocking" | The blocked channel may have moved to an unmonitored one |
| "User friction is the cost of doing business" | Sometimes — but high friction without measurement creates the workaround |
| "Policy exceptions are not allowed" | They will happen anyway, off the record; the policy is then unaudit-able |
| "The block rate is rising — that means the control is working" | Or it means the policy is hitting legitimate work and users are still trying to do their jobs |

## Operational rule of thumb

**A policy with block rate >5% of attempted actions in steady state is either:**
- **Catching a lot of bad behaviour** (verify by sampling TPs)
- **Catching a lot of good behaviour with no alternative path** (workarounds incoming)
- **Catching a lot of FPs and needs tuning** (analyst review)

Distinguish by sampling. A 5% block rate without sampling is just a number.

## Public-incident notes

The 2024 wave of corporate-data leaks via personal ChatGPT use illustrated this at scale. Firms that responded with "block ChatGPT completely" found that:
- Users moved to personal devices / personal accounts / personal networks for AI work
- The blocked traffic was visible (audit data); the moved traffic was invisible (no audit data)
- The firms that responded with sanction-and-coach (provide ChatGPT Enterprise, coach on safe use) had better outcomes than the firms that blocked

The shape is the same for any over-blocked control class.

## Promotion candidate

Strong public-wedge candidate — directly contrarian to vendor marketing. Currently a residual-risk reference for the practitioner audience.
