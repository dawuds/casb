# Vendor lock-in

> CASB policy languages don't port between vendors. Classifier definitions don't port. Custom integrations don't port. The "we'll switch CASB next year" plan rarely survives contact with the migration estimate.

## The failure

A firm picks Vendor A for CASB. Builds 30-50 policies over 18 months. Tunes the classifier set. Integrates with their SIEM. Builds SOC runbooks against the vendor's alert schema. Trains the team.

Three years later, Vendor B has shipped a meaningfully better capability (better GenAI inspection, better cost-per-user, better SSE integration). The firm wants to switch. The actual cost:

- Policy migration: every policy must be redefined in Vendor B's policy language. Some semantics don't translate (vendor-specific actions, vendor-specific scope filters).
- Classifier migration: regex / SIT / DLP signatures rebuilt against Vendor B's classifier engine. FP rates re-baselined.
- Integration migration: SIEM connector swapped; KQL queries rewritten; SOAR playbooks updated for the new alert schema.
- Runbook migration: SOC training. Console paths in every runbook. Escalation contacts updated.
- Audit-evidence-trail discontinuity: the migration is itself an audit event; pre- and post-migration evidence is in different formats.
- Parallel-run period: both vendors run for 3-6 months to validate coverage.
- Total cost: typically 6-12 months of platform-team + SOC + risk effort plus the cost of running two platforms in parallel.

## What enables it

| Mechanism | Why it locks in |
|---|---|
| **No standard for CASB policy expression** | Each vendor has their own policy DSL or UI. No portable JSON schema, no equivalent of Open Policy Agent for CASB |
| **Vendor-specific schema for alerts and audit events** | Each vendor's SIEM schema is different; downstream integrations are vendor-bound |
| **Per-app onboarding work** | CAAC / inline app onboarding is per-app, per-vendor; switching means re-doing this for every app |
| **Custom integrations** | Power Automate flows, ServiceNow connectors, SOAR playbooks — built against the vendor's API surface |
| **DLP classifier libraries** | Vendor-specific SIT definitions, regex syntax, ML model behaviour |
| **Console-knowledge inertia** | The team knows Vendor A's console; they don't know Vendor B's |
| **Audit-trail continuity** | Pre-migration evidence in Vendor A's format; post-migration in Vendor B's. Auditor / regulator continuity question |
| **EA / contract economics** | Year 3-5 of a 5-year EA — vendor knows you can't realistically leave |

## What you cannot port

| Asset | Portability |
|---|---|
| Policy logic (block-download-of-Confidential-from-unmanaged) | Re-implementable, but every clause requires re-mapping to the new vendor's primitives |
| Risk-score thresholds | Different vendors use different scoring models; thresholds reset |
| FP-rate tuning | Re-baselined |
| Allowlists / exception lists | Re-keyed against new vendor's app IDs |
| Sensitivity labels (Purview) | Portable across CASBs that integrate with Purview; not portable across non-Microsoft DLP engines |
| OAuth-app review state | Each vendor's catalogue is different; per-app sanction decisions re-applied |
| Custom SIT / classifier definitions | Often not portable |
| SIEM dashboards + alert correlation rules | Re-built against new schema |
| SOAR playbooks | Re-built against new alert + governance-action API |
| Internal runbooks | Re-written |
| User-facing block messages | Re-styled per vendor's customisation surface |

What you CAN port:
- Standards control mappings (these are vendor-agnostic by design)
- The list of regulated data classes
- The business case
- The team's general CASB literacy

## Why the failure exists

CASB grew from many independent pure-play products (Adallom, Bitglass, Netskope, Skyhigh, etc.). No standards body wrote a portability spec early. SSE platforms inherited each vendor's policy language. Vendors have no commercial incentive to make migration easier; the inverse is true.

The closest analogue is firewall policies — every NGFW vendor has their own rule language; migrating from one to another is non-trivial. Networking has at least the OSI layers as semantic anchors; CASB has no equivalent semantic standard.

## Which vendors exhibit it

All of them. This is a category-wide problem.

| Vendor | Migration friction |
|---|---|
| **MDA** | Locked in to Microsoft Entra / Defender / Purview ecosystem; migration off MDA means migrating away from Microsoft-integrated policies |
| **Netskope / Zscaler / Palo Alto** | Vendor-specific policy DSLs; SSE-platform integration; multi-component (SWG + CASB + ZTNA) migrations are even harder |
| **Skyhigh** | Less SSE-platform-coupled; somewhat lower friction than the SSE-leader vendors |

## What compensates

| Discipline | What it does |
|---|---|
| **Policy intent documented separately from policy implementation** | The intent "Block download of Confidential files to unmanaged devices" is portable; the implementation in MDA / Netskope / etc. is not. Document both layers |
| **Standards-mapped policy library** (per [`../06-compliance/control-mapping-framework.md`](../06-compliance/control-mapping-framework.md)) | The standards mapping survives the migration; the per-vendor implementation is rebuilt against the mapping |
| **SIEM-level dedup + canonicalisation** | If the SIEM normalises vendor-specific events into a canonical schema, downstream SOAR + dashboard logic is less coupled |
| **Multi-vendor hybrid posture** | If part of the CASB stack is already multi-vendor (e.g. MDA API + third-party inline), the team has already paid the lock-in tax — migration is incremental |
| **Documented exit-cost estimate** | Year-1 procurement should require an exit-cost estimate (per [`../07-implementation/cost-of-ownership.md`](../07-implementation/cost-of-ownership.md)). Forces the conversation early |
| **Regulator-required exit-strategy** | BNM RMiT third-party / MAS Outsourcing / EU DORA all require documented exit strategies for cloud / outsourced services (verify against current editions). Use the regulator requirement as the lever for vendor exit-cooperation contracts |
| **Annual lock-in review** | Each year, review the cost-to-switch estimate; update; surface in risk register |

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "Switching vendors is straightforward" | It isn't. 6-12 months of platform work |
| "We don't have lock-in — we can leave any time" | The contract may say so; the operational reality says otherwise |
| "The standards mapping makes us vendor-agnostic" | The mapping does. The implementation doesn't |
| "We can run two CASBs in parallel during migration" | Parallel-running is real but expensive — double licence cost + double SIEM ingest + dual triage |

## What an auditor will ask

| Question | What you need |
|---|---|
| Show me the exit strategy for this vendor | Documented; updated annually; reviewed at vendor renewal |
| What is the cost to migrate? | Cost estimate with assumptions; updated annually |
| What is the recovery time if you have to leave on regulator order? | Realistic estimate; typically 6-12 months at minimum |
| Does the vendor's contract include exit-cooperation terms? | Should — covers policy export, log export, transition support |

## Public-incident notes

When a CASB vendor exits the market or pivots its product (Oracle CASB discontinued; Symantec → Broadcom transition; McAfee → Trellix → Skyhigh re-org), customer migration was the dominant cost line. None of these transitions was painless for customer organisations.

## Procurement-time mitigation

| Contract clause | What to require |
|---|---|
| **Policy export** | Vendor commits to a portable export of policy configuration on customer request |
| **Log export in defensible format** | Audit logs exportable with integrity guarantees; format documented |
| **Transition support** | Vendor commits to N hours / N months of support for transition to another platform |
| **Notice period for product changes** | Material changes (deprecation, schema change) given N months notice |
| **No retroactive feature removal** | Features can't be removed within the contract term without notice + alternative |
| **Pricing-cap at renewal** | Cap year-over-year price increases past initial term |

Year-1 procurement is the only time you have negotiation leverage on these. Year 3-5 is too late.

## Promotion candidate

Strong public-wedge candidate. Currently a residual-risk reference.
