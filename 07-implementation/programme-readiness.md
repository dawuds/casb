# CASB programme readiness

> What an organisation must know about itself before a CASB / SSE platform deployment will succeed. Discovery and policy decisions only land on a base that has these answers. Vendor-agnostic.

## The four readiness axes

| Axis | Question | Why it bites if unanswered |
|---|---|---|
| **SaaS estate** | What sanctioned + tolerated + unknown SaaS does the firm use, and which contain regulated data classes? | CASB onboarding effort scales with the SaaS estate. Without an inventory, the rollout is reactive, the policy library is sized wrong, and the licence cost is wrong |
| **Identity posture** | Is there a single IdP source of truth? Where is the federation seam (Entra primary, Okta legacy, ADFS, Ping)? Is MFA enforced for all access? | CASB inline policies depend on the IdP. Identity drift is the dominant cause of policy gaps. Multi-IdP estates pay 3-5× the onboarding cost per app |
| **Endpoint posture** | Managed vs unmanaged distribution. Mobile BYOD population. MDM/MAM coverage. SSL-inspection compatibility (cert pinning, banking apps) | Every CASB deployment trade-off (forward-proxy vs API, block vs read-only on BYOD) collapses to the endpoint posture answer |
| **Regulatory perimeter** | Which obligations actually bite (BNM RMiT, MAS TRM, HKMA, PDPA MY 2024, GDPR, DORA, sectoral)? What is the audit cadence? What records survive an examination? | Drives the must-have policies, the audit evidence requirements, and the residency/cross-border decisions. Treated as an afterthought it forces re-architecture late |

## Inventory checklist (before vendor selection)

| Check | Owner | Outcome required |
|---|---|---|
| Top-50 SaaS by user count (from IdP sign-in logs) | Identity team | Named list with data-classification tag per app |
| Top-50 SaaS by traffic volume (from SWG / firewall logs) | Network team | Named list including the long-tail Shadow IT not behind the IdP |
| Sanctioned vs tolerated vs unsanctioned classification | Risk + Business | Documented, with sponsoring business owner per sanctioned app |
| Data classification scheme | Information Protection team | Named classes (PII, PCI, KYC, etc.) with examples and labelling guidance — not just "Confidential / Internal / Public" |
| Sensitive-data exposure assessment | Risk + Information Protection | Where the regulated data classes actually live across the SaaS estate (Microsoft Purview / Google Vault / equivalent already deployed?) |
| IdP inventory | Identity team | Primary IdP, secondary IdP, federation chain, app coverage per IdP |
| MFA enforcement state | Identity team | What % of users / apps are MFA-protected. Gap analysis against CASB session-policy assumptions |
| Endpoint management state | Endpoint team | % managed (MDM enrolled), % unmanaged, % BYOD-mobile, MDM platform (Intune, Workspace ONE, Jamf, Kandji) |
| SSL-inspection feasibility | Network + Endpoint | Cert pinning audit on key apps (banking, vendor SDK clients, mobile native). Personal-traffic inspection legal/HR position |
| Egress topology | Network team | Where users exit to the internet (corp network, VPN, BYOD direct, SD-WAN). Determines where the CASB proxy can sit |
| SOC capability | Security operations | SIEM in use, SOAR available, alert handling capacity (alerts/analyst/day), current FP rate on existing controls |
| Regulatory inventory | Risk + Legal | Named obligations with article/clause references. Cross-border data-flow constraints. Audit cadence per regulator |
| Existing DLP estate | Information Protection | What is already enforcing (endpoint DLP, email DLP, service-side SaaS DLP). Where is the duplication risk |
| Change-management capacity | IT operations | How fast can you push agent/PAC/policy changes. Communications channels for end-user impact |

If five or more rows above are blank or "unknown", **CASB selection is premature** — close the inventory gap first. Tooling buyers who skip this end up paying for capabilities they cannot deploy.

## Sponsorship and ownership

A CASB programme needs four named owners signed off before kickoff:

| Role | Owns |
|---|---|
| **Executive sponsor** | The business case; the budget; the cross-functional unlock when Legal / HR / business push back |
| **Programme lead** | The roadmap; the phased rollout; vendor management; the politics of policy decisions |
| **Technical lead** | Platform architecture; integration plumbing; the on-call rotation for the CASB itself |
| **Compliance lead** | Control mapping; audit evidence; the regulator conversation; sign-off on policies that touch privacy |

Programmes with three of four end up with mis-scoped policies (no compliance lead), shelfware (no exec sponsor), or controls that work technically but fail audit (no compliance lead). Programmes with two of four fail.

## Decisions that must be locked before policy design

| Decision | What it determines |
|---|---|
| **In-scope user populations** | Employees only? Contractors? B2B partners? Service accounts? Each population has different policy semantics |
| **In-scope SaaS scope** | Sanctioned only? Sanctioned + tolerated? All-discovered? Sets the licensing budget and the implementation surface |
| **BYOD position** | Allow with controls / block / read-only / device-trust-required. The single most political decision in a CASB rollout |
| **Personal-use position** | Are users allowed to access personal SaaS (personal Gmail, personal cloud drives) on corporate networks / devices? Determines the SSL-inspection privacy posture |
| **Action-class default** | Audit / coach / block. Most programmes start in audit, escalate. Block-first programmes generate a workaround culture that defeats the controls |
| **Exception process** | Who can grant an exception? On what evidence? For how long? Audit trail of exceptions |
| **Privacy posture** | What end-user notice is given? What is logged about user activity? Where does the log live? How is access governed? Critical under PDPA MY 2024 / GDPR / HKPCPD |

Don't run vendor selection without these locked. Vendor demos look identical if the requirements are vague; they look different when the requirements name a policy decision.

## Common readiness failure modes

| Failure | Symptom | Root cause |
|---|---|---|
| **"We'll figure out the policies in implementation"** | Six months in, the policy library is half-defined and the rollout is stalled on internal disagreement | Treated CASB as a tooling decision, not a policy decision |
| **"The IT team will run it"** | Alerts accumulate; nobody triages; the platform becomes shelfware | No SOC integration plan; no operational capacity sized |
| **"We'll start with everything in block mode"** | High user friction within two weeks, business escalates to executive, programme is rolled back | No graduated action-class plan; no exception process; no business engagement |
| **"BYOD is allowed but unmanaged"** | Policies cover ~30% of actual exfil risk; auditor flags the gap | BYOD position wasn't decided; controls inherited a "do what you can" scope |
| **"The vendor will handle the regulator question"** | First audit cycle exposes that no control mapping exists; vendor docs cited as evidence | No compliance lead; no control mapping; vendor self-attestation treated as audit evidence |

## Outputs of the readiness phase

When this phase is genuinely complete, you have:

1. A **scoped requirements document** that names: user populations, SaaS scope, BYOD/personal-use position, action-class defaults, exception process, privacy posture.
2. A **populated SaaS estate inventory** with risk-classification, data-classification, and ownership per app.
3. A **named team** — sponsor, programme, technical, compliance — with calendar commitment.
4. A **regulatory inventory** with named clauses per obligation, audit cadence, evidence requirements.
5. A **gap analysis** between current controls (endpoint DLP, email DLP, IdP, SIEM) and the CASB target — where the duplication is, where the seams are, what gets retired.
6. A **business case** the executive sponsor signs that names the risk reduction (not the feature count).

Without these six outputs, vendor selection is theatre.
