# Hybrid deployment pattern

> Proxy + API combined; common chokepoints; failure modes when one half degrades. Vendor-agnostic. The typical end-state for a mature CASB deployment.

## Why hybrid is the typical end-state

| Driver | Why hybrid wins |
|---|---|
| API mode is needed for OAuth governance, sharing cleanup, labelling, anomaly detection | API depth varies by app; some apps only have API mode |
| Inline mode is needed for real-time DLP, download blocks, clipboard inspection | Cannot be done in API mode |
| Discovery feeds the sanction taxonomy that both modes depend on | Log-based discovery underlies both |
| Different apps need different depth | Tier-1 sanctioned apps justify inline; tier-2 stays API-only; tier-3 stays discovery-only |
| Regulator expectations span pre-event (prevent) and post-event (detect + respond) | Both modes contribute audit evidence |

A pure-API deployment misses real-time prevention. A pure-inline deployment misses OAuth governance, sharing cleanup, and the long tail of sanctioned apps. Hybrid is the architecture that matches the policy library.

## Reference topology

```
User device (managed / BYOD / mobile)
   |
   | Network steering: agent / PAC / IdP-mediated / direct (for off-net)
   v
SSE PoP
   |
   v
   SWG layer
   |
   v
   CASB inline (forward-proxy or reverse-proxy)
   |
   |   <-- Real-time: block download, block paste, block upload, apply label
   v
SaaS application (sanctioned, inline-onboarded)

In parallel:
   CASB API connectors -> Sanctioned SaaS API
   |   <-- Near-real-time: scan files, OAuth governance, anomaly, sharing cleanup
   v
SaaS application (sanctioned, API-connected)

Discovery feed:
   SWG logs / EDR network telemetry / firewall logs
   |
   v
   CASB Shadow IT discovery
   |   <-- Catalogue, risk-scoring, sanction taxonomy
```

## App tiering

Most regulated-FI hybrid deployments tier the sanctioned-SaaS list:

| Tier | Examples | Mode |
|---|---|---|
| **Tier 1 — Critical, high-data-risk, high-volume** | M365, Salesforce (if KYC-heavy), the HR system | Inline + API (full hybrid per app) |
| **Tier 2 — Important but lower-risk or lower-volume** | Box, Dropbox, ServiceNow, Workday, Atlassian | API only |
| **Tier 3 — Tolerated** | Long-tail SaaS — Slack channels, departmental tools | Discovery + risk-scored; no enforcement directly |
| **Tier 4 — Unsanctioned / Blocked** | Apps tagged Unsanctioned | Discovery + blocking via SWG (not CASB) |

Tier 1 has the highest per-app onboarding cost and the highest control depth. Tier 2 has lower cost and acceptable depth via API mode. Tier 3-4 use discovery as the visibility layer; enforcement (if any) happens at the SWG.

## Decision: which app belongs in which tier

| Question | Answer guides tiering |
|---|---|
| Does this app hold regulated data (PCI / PII / KYC / health)? | If yes — at least Tier 2; consider Tier 1 if volume is high |
| Is this app in scope of an audit / regulator obligation? | If yes — at least Tier 2 (need audit evidence) |
| Is real-time DLP required for this app (regulator demands prevent, not detect)? | If yes — Tier 1 (inline) |
| Does this app have an API connector available? | If no — Tier 3 / 4 at most |
| Does this app survive reverse-proxy onboarding (`window.parent.postMessage` compatibility, cert pinning)? | If no — Tier 2 (API only) |
| How many active users? | Volume drives the per-user economics of inline mode |
| Is this app behind the primary IdP? | If no (Okta-fronted in an Entra-primary tenant) — onboarding cost goes from days to weeks |

## Composition mechanics — what fires first

When the same event matches both inline and API policies, what happens?

| Action class | Inline (proxy) | API (post-event) | Conflict resolution |
|---|---|---|---|
| Block download | Real-time block at proxy | Cannot block — event already happened | Inline wins; API is silent |
| Apply sensitivity label | Apply on download (rewrite) | Apply on file-policy match | Both apply (potentially different labels); document precedence |
| Audit / alert | Generated at proxy | Generated post-event | Two records, often correlated; SIEM dedup may be needed |
| Quarantine file | Cannot — proxy blocks download but file is already in SaaS | Post-event quarantine via API | API wins for this action |
| Revoke OAuth grant | Cannot — proxy doesn't see grants | API revoke | API only |

The "two events for one action" pattern is common — make sure the SIEM correlation rules handle dedup (or accept the volume for evidence completeness).

## Failure modes specific to hybrid

| Failure | Symptom | Mitigation |
|---|---|---|
| Inline policy and API policy contradict | Two records, possibly two governance actions, conflicting outcomes | Document policy precedence; have one policy per intent, split inline/API by deployment mode not by intent |
| Discovery doesn't feed the sanction taxonomy | New SaaS apps in use, not tiered, no governance | Weekly discovery review with tiering decision |
| Inline mode degrades, API still runs | Apparent reduction in alert volume; gap in real-time prevention | Health-check monitor; alert on inline-mode unavailability |
| API mode degrades, inline still runs | Sharing cleanup queue grows; OAuth grant backlog | Health-check monitor; alert on connector staleness |
| App moved from Tier 2 to Tier 1 (added inline) but old API policies retained | Duplicate enforcement; policy noise | Tier-change runbook including policy reconciliation |
| Multi-vendor hybrid (e.g. MDA API + Zscaler inline) | Two vendors, two log streams, two policy languages | Document the integration explicitly; pick which vendor owns which decision; SIEM unification mandatory |

## Multi-vendor hybrid (the harder case)

Sometimes the inline mode is one vendor and the API mode is another — e.g. MDA API for M365 governance + Zscaler / Netskope / Palo Alto inline for forward-proxy. This is common in tenants that picked Microsoft for identity/M365 and a third-party SSE for the broader stack.

| Concern | Handling |
|---|---|
| Two vendor consoles | Document the per-policy ownership; train SOC on both |
| Two SIEM connectors | Unify in Sentinel / Splunk; correlate by user + IP + timestamp |
| Two policy languages | Each vendor's policy expression is different; build per-vendor runbooks |
| Two alert classes | Dedup or accept the volume |
| Two support contracts | Per-vendor escalation paths |
| Identity correlation | Both vendors must see the same user identifier; verify Entra UPN vs Okta UPN consistency |

The multi-vendor case is also the most realistic state for many regulated FIs — picking "all Microsoft" or "all Netskope" is rare in mid-to-large enterprises.

## Migration to hybrid

If you're starting from proxy-only or API-only, the additive path:

| Starting state | Adding the missing mode |
|---|---|
| **API-only** | Add inline starting with the Tier 1 apps. Pilot one app per onboarding sprint. Expect 1-3 days per Entra-federated app, 2-4 weeks per Okta-federated app |
| **Proxy-only** | Add API connectors to sanctioned apps. Faster than adding inline — mostly UI clicks. Build the OAuth governance and sharing-cleanup policies |
| **Discovery-only** | Add API connectors first (lower friction); add inline for the highest-risk apps after |

## Integration with the broader stack

Hybrid CASB composes with the rest of the SSE stack and adjacent disciplines:

| Component | Hybrid CASB interaction |
|---|---|
| **SWG** | Inline before CASB on the same flow; SWG handles URL category / generic web policy; CASB handles per-app SaaS policy |
| **ZTNA** | Parallel path; private-app access doesn't transit CASB |
| **IdP + Conditional Access** | Upstream of CASB inline; inline mode depends on identity |
| **Purview / Information Protection** | Labels applied by API-mode file policies; enforced by Purview encryption; inline mode also reads labels for action decisions |
| **EDR / Endpoint DLP** | Pre-encryption data classification at the device; CASB picks up what survives to SaaS |
| **SIEM / SOAR** | Canonical destination for both inline and API events; correlation rules dedup |

## Operating model

| Activity | Cadence | Owner |
|---|---|---|
| Inline policy tuning | Weekly | Platform team |
| API policy tuning | Weekly | Platform team |
| Tier reassessment of apps | Quarterly | Programme + Risk |
| Per-app regression test (inline) | Per SaaS vendor major release | Platform team |
| Connector health check (API) | Weekly | Platform team |
| Compatibility-matrix update | Quarterly | Platform team |
| SOC alert dedup rule review | Quarterly | SOC + Platform |

A mature hybrid deployment runs on 1-1.5 FTE of dedicated platform effort plus SOC capacity.

## Anti-patterns

| Anti-pattern | Consequence |
|---|---|
| **Putting everything inline** | Onboarding cost explodes; compatibility issues mount; team burns out |
| **Putting everything API-only** | Real-time prevention gaps; regulator question on prevent vs detect |
| **One-vendor lock-in without exit posture** | Policy languages don't port; year-3 vendor renegotiation is one-sided |
| **Skipping the tier-decision documentation** | Apps drift between tiers without explicit decisions; auditor finding |
| **No SIEM dedup** | Alert volume doubles; FP rate metrics distorted |
