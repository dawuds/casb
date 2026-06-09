# API mode is not prevention

> API-mode CASB detects, classifies, and can remediate post-event. It does not prevent. Vendor marketing routinely conflates the two.

## The failure

API-mode policies fire **after** the user action. A practitioner reads "DLP for SaaS — Supported" in a vendor capability matrix, assumes pre-upload prevention, ships the policy, and finds in incident review that the sensitive data left the tenant 5-30 minutes before the API-mode policy classified it.

## What enables it

| Mechanism | Why API isn't inline |
|---|---|
| **API connector is event-subscription, not session-interception** | CASB receives events after SaaS records them |
| **Event delivery latency varies** | M365 SharePoint near-real-time (minutes); Power BI / Dynamics 24-72 hours |
| **Some events arrive in batches** | Connector throughput limits; vendor API rate limits |
| **Connector backoff during throttling** | Silent under-coverage during peak load |
| **No control-plane chokepoint** | The user's action completes; the CASB sees the record after |

## The detection-vs-prevention distinction

| Action class | Prevention (inline) | Detection + remediation (API) |
|---|---|---|
| **File upload of PII to SaaS** | Block at proxy (file never lands) | Detect after upload; quarantine post-event; data was exposed for the gap |
| **Sharing externally** | Block at proxy (share never created) | Detect after share-creation; revoke share; recipient may have already accessed |
| **OAuth grant** | Cannot prevent at proxy (consent screen is browser-native) | Detect after consent; revoke; grant was active for the gap |
| **Mass download** | (Some inline modes can throttle; rare) | Detect after the volume threshold; suspend after the fact |
| **Data classification** | Apply at upload (proxy reads file → applies label → blocks if needed) | Apply post-upload (scan file → apply label → may be too late for downstream sharing) |
| **Malware in SaaS storage** | Inspect at upload (block infected file) | Scan post-upload (file was in tenant for the gap; may have spread via sync) |

Each row's "API" column has a non-zero exposure window. Regulator obligations that name "prevent" — and many do — are not satisfied by API-mode policies alone.

## Which vendors exhibit it

All CASB vendors with API-mode capability. The depth and latency vary; the structural limitation is the same.

| Vendor | API latency (SharePoint / OneDrive, typical) |
|---|---|
| **MDA** | Near-real-time per Microsoft (no SLA; plan for <15 min normal, longer during throttling) |
| **Netskope** | Vendor-specific; comparable order of magnitude |
| **Zscaler / Palo Alto / Skyhigh** | Vendor-specific; all in the minutes-to-hours range for M365 |

For Power BI, Dynamics 365, and some Salesforce connectors, all vendors are 24+ hours.

## Why the failure exists (technical)

The SaaS app commits the action first (user uploads, share is created, OAuth grant is approved). The SaaS app then records the event in its audit log. The CASB subscribes to the audit log. There is no architectural point at which the CASB can intercept the action before commitment without sitting inline in the user's session — which is the proxy / inline mode.

API mode is *inherently* post-event. No vendor can solve this without changing the deployment mode.

## What compensates

| Need | Compensating control |
|---|---|
| Prevent pre-upload | **Inline / proxy mode** for the sensitive apps (M365 via CAAC; other apps via forward-proxy) |
| Catch what API misses (e.g. Power BI 72h latency) | **Endpoint DLP** at the user's device (pre-encryption inspection) |
| Reduce the exposure window when prevention isn't possible | **Tight detection + auto-remediation** (quarantine quickly; revoke share quickly; suspend account quickly) |
| Audit-evidence the prevention claim | **State explicitly in control documentation that API mode is detect-and-remediate**, not prevent; map to controls that accept detect-and-remediate (vs ones that require prevent) |

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "Our CASB prevents data loss to SharePoint" | If API-mode-only, it detects loss and quarantines; doesn't prevent |
| "DLP is enforced on all sanctioned SaaS" | API-mode is enforcement on the audit log; inline-mode is enforcement on the user session |
| "OAuth abuse cannot happen — we have governance" | The grant happens; the governance kicks in after |
| "Our control satisfies BNM RMiT / MAS TRM prevent-style requirements" (without inline mode) | Verify per-clause; some are detect-and-remediate-acceptable, some demand prevent |

## Operational implications

| Operational implication | Action |
|---|---|
| Auditor expects "prevent" — you have "detect" | Add inline mode for that data class, or accept the residual with documented compensating control |
| Incident response: data left the tenant before policy fired | Time-from-event-to-detect-to-respond becomes the headline metric; the prevention gap is the inherent exposure |
| Sharing drift accumulates faster than API quarantine runs | Pair with inline share-creation block in proxy mode |
| Mass-delete (ransomware-in-SaaS): API-only detects after files are gone | Pair with backup / version history; CASB alone is detection, not prevention |

## What an auditor will probe

| Question | Honest answer for API-only |
|---|---|
| "Show me the policy that prevents PII upload to OneDrive" | "We have detection + auto-label; prevention is via Purview Endpoint DLP at the device" |
| "Show me the policy that prevents external sharing of Confidential files" | "We detect sharing post-event and revoke; for prevention we require [inline policy in proxy mode]" |
| "Show me the time-from-event-to-action" | The metric must exist and be measured |
| "Show me the residual exposure window" | Per-policy: the connector latency + your triage SLA = the exposure window |

If you can't answer the last two in metric form (not vendor-marketing form), the audit conversation will identify the gap.

## Promotion candidate

Public-wedge candidate; currently a residual-risk reference.
