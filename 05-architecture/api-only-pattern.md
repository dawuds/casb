# API-only deployment pattern

> CASB API connectors to sanctioned SaaS; near-real-time monitoring; retroactive remediation; what cannot be enforced inline. Vendor-agnostic.

## When API-only is the right choice

| Driver | Why API-only fits |
|---|---|
| Quick visibility win with low operational friction | No agent, no PAC, no SSL inspection, no per-app regression test |
| Sanctioned SaaS list is narrow and has good API connectors | M365 first-class; Google Workspace, Salesforce, Box, Dropbox, ServiceNow have decent depth |
| Acceptable trade-off: post-event detection + retroactive remediation | Some risk classes don't need real-time prevention (sharing drift, OAuth grant cleanup, terminated-user activity) |
| Heavy BYOD where inline coverage is impractical | API mode doesn't depend on endpoint coverage |
| Forward-proxy / SSL-inspection out of scope (privacy / cost / political) | API mode avoids the TLS-inspection trade-off entirely |

## When API-only is NOT enough

| Driver | Why |
|---|---|
| Real-time DLP enforcement required (regulator demands prevention, not post-event remediation) | API is post-event; inline is required for prevention |
| Inline activity blocking required (block paste, block download, block print) | Only proxy mode has these |
| Apps without API connectors are in scope | API mode is silent on apps it cannot connect to |
| GenAI inline DLP needed | Most GenAI vendors don't expose CASB-grade APIs; inline is the only path |

## Reference topology

```
SaaS application (M365, Workspace, Salesforce, Box, etc.)
   |
   | (1) CASB authorises via OAuth/API token (one-time setup)
   v
CASB API connector
   |
   | (2) Event subscription: file events, sharing events, auth events, admin actions
   v
CASB event-processing engine
   |
   | (3) File policy matches, anomaly policies, OAuth governance
   v
Governance actions:
   - Apply sensitivity label
   - Move to admin quarantine
   - Remove external collaborators
   - Trash file
   - Suspend user (Entra-integrated)
   - Revoke OAuth grant
   |
   v
SIEM / SOC for alert routing
```

### Connector depth varies

| Connector | M365 | Google Workspace | Salesforce | Box | Dropbox | ServiceNow | Workday |
|---|---|---|---|---|---|---|---|
| Read activity | Full | Full | Full | Full | Full | Partial | Limited |
| Scan files | Full | Yes (Drive) | Attachments | Yes | Yes | No | No |
| Apply sensitivity label | Yes (Purview-aware) | (Vendor-specific) | Limited | (Some vendors) | (Some) | No | No |
| Quarantine file | Yes (multi-action) | Yes (some actions) | Limited | Yes | Yes | No | No |
| Remove external collaborators | Yes | Yes | Yes | Yes | Yes | N/A | N/A |
| Suspend user | Via Entra | Limited | (Custom) | (Custom) | (Custom) | (Custom) | (Custom) |
| Revoke OAuth grant | Yes | Yes | Yes | N/A | N/A | N/A | N/A |
| Configuration assessment (SSPM-light) | Some | Some | Some | Limited | Limited | Limited | Limited |

M365 is the deepest connector across all major CASB vendors. Other connectors have varying depth — verify per-vendor before assuming a governance action is available.

## What API-only can do well

| Capability | Notes |
|---|---|
| **OAuth governance** | Discovery + ban; per-user revoke; anomaly detection on misleading-publisher / malicious-consent |
| **Sharing drift cleanup** | External-share scanning, quarantine, owner notification |
| **Data-at-rest sensitivity labelling** | Auto-label on file content; encryption via the label propagates |
| **Anomalous-activity detection** | Mass-download, impossible travel, terminated-user activity in connected apps |
| **Compromised-account signal** | Activity from unusual ISP, anomalous OAuth grant, suspicious mailbox manipulation |
| **Configuration assessment (light SSPM)** | A subset of connected apps' admin-settings can be policy-checked |

## What API-only cannot do

| Gap | Why |
|---|---|
| Real-time inline DLP | Events arrive post-event |
| Block download to BYOD | No control point in the user's session |
| Block clipboard paste to ChatGPT | Browser session is not in the API control plane |
| Apply sensitivity label at download | API mode applies labels at scan time, not at user action time |
| Cover non-API SaaS | If the SaaS doesn't expose an API connector, API mode is silent |
| Inline malware block | Detection only; some vendors can quarantine post-detect |

## Connector latency

API-mode events arrive after the user action. Typical latencies:

| SaaS | Typical event latency |
|---|---|
| M365 SharePoint / OneDrive | Near-real-time per Microsoft (no SLA; plan for <15 min normal; longer during throttling) |
| M365 Power BI / Dynamics 365 | 24-72 hours |
| Google Drive | Minutes |
| Google Workspace audit log | Minutes |
| Salesforce | Minutes to hours, connector-dependent |
| Box / Dropbox | Minutes |
| ServiceNow / Workday | Hours to a day; less mature connectors |

This latency is the dominant operational constraint of API-only mode. For any policy where the regulator clock starts at "the event", API-only doesn't satisfy a detection-and-respond window much shorter than the connector latency.

## Graph / API throttling

API connectors share rate budget with the SaaS vendor's tenant-level API throttling. At BFSI scale:

| Behaviour | Why |
|---|---|
| File-policy tenant-wide scans saturate Graph API bucket (~600 req/min/tenant SharePoint Online) | Multi-tenant CASB backend competes with the tenant's other API consumers (admin scripts, Power Automate, third-party integrations) |
| MDA backs off to partial coverage without operator notification | Silent failure; SIEM doesn't see the under-coverage; auditor catches the gap |
| Scope by parent folder, not tenant-wide | Reduces the load |
| Stagger scan windows | If multiple policies run tenant-wide on the same connector, sequence them |

Monitor Graph throttling headers via Sentinel (or vendor-specific dashboards) rather than relying on CASB UI signals.

## Integration with the SSE stack

API-only CASB pairs well with:

| Component | Composition |
|---|---|
| **SWG (Shadow IT discovery feed)** | Log-based discovery via SWG egress logs feeds the API connector ecosystem with the "what apps are in use" answer |
| **IdP (Entra / Okta)** | User identity correlation; suspend-user governance routes through the IdP |
| **EDR (Defender / CrowdStrike / SentinelOne)** | Endpoint discovery feed; complementary signal for compromised-account detection |
| **SIEM** | Canonical destination for CASB-generated alerts |
| **Purview / Information Protection** | Sensitivity labels applied by API-mode file policies; encryption + permissions enforced by the label |

API-only does not require:
- An endpoint agent
- A PAC file
- SSL inspection
- Per-app reverse-proxy onboarding
- A specific SSE platform — most CASB vendors support API connectors against the same set of major SaaS

## When to add inline mode

API-only typically evolves to hybrid (API + inline) when:

| Trigger | Why |
|---|---|
| Auditor or regulator requires real-time DLP for specific data classes | API mode is detect-and-remediate, not prevent |
| Sharing drift becomes high-volume; quarantining at scale breaks user workflows | Inline prevention reduces the volume |
| BYOD coverage becomes a documented gap | Inline (with MAM pairing) is the partial answer |
| GenAI exfiltration becomes a board-level risk | Inline (proxy or SAML-federated reverse-proxy) is the only enforcement path |

The inline-mode addition is incremental — pick one or two apps to onboard first, build the regression-test muscle, then expand. The full "everything inline" deployment is rarely the right end state — see [`hybrid-pattern.md`](hybrid-pattern.md).

## Costs specific to API-only

| Cost line | Notes |
|---|---|
| **Licensing** | Often cheaper than the full inline-capable tier; vendor-specific |
| **SIEM ingest** | Lower per-user than verbose-inline deployments; still non-trivial for activity-heavy connectors |
| **Implementation effort** | Lower than proxy-only — connector enablement is mostly UI clicks |
| **Operational headcount** | Lower triage volume than inline mode (no real-time blocks → fewer user-facing tickets) |
| **Governance-action API-cost** | Quarantine, label-apply, share-revoke each consume API quota; at scale this adds up |
| **Connector breakage** | SaaS vendor API changes can break connector functionality; monitor for breakage signals |

## Operating model

| Activity | Cadence |
|---|---|
| OAuth grant review | Weekly (catch new high-risk grants) |
| Sharing drift quarantine queue | Daily |
| Sensitivity-labelling progress | Weekly metric |
| Anomaly-policy triage | Daily |
| Connector health check | Weekly |
| Per-connector throttling review | Monthly |

A team can typically run an API-only CASB deployment with ~0.5 FTE of dedicated effort plus SOC alert triage capacity. Add inline mode and the operational requirement roughly doubles.

## Failure modes specific to API-only

| Failure | Symptom | Mitigation |
|---|---|---|
| Connector token expiry | Silent stop of event ingest | Monitor connector health; alert on stale connector |
| SaaS-vendor API schema change | Specific governance actions stop working | Subscribe to vendor API change notifications; vendor-CASB connector update cadence matters |
| File-policy ceiling hit | Excess policies silently not enforced | Document the ceiling (e.g. 200 for MDA per 2026 reality); consolidate policies |
| Power BI / Dynamics 24-72h latency | Real-time use-cases unsatisfied | Pair with Purview DLP for those workloads |
| Externally-owned files invisible | Files placed in your tenant by external users (where the external user is the data owner) are unscannable | Pair with org-level external-sharing restrictions; OneDrive reassigns ownership and is the partial exception |
