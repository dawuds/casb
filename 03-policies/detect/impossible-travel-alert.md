# Impossible-travel alert

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [Compromised-account detection (impossible travel)](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector. Entra-as-IdP for governance signal.
> MDA playbook reference: [Policy 4](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Impossible travel + activity from infrequent country). **"Policy 4" is internal repo numbering — not Microsoft Learn nomenclature.**

## Purpose

Detect sign-in patterns that are geographically infeasible within the observed time window — credit-stuffing botnet / compromised-credential pattern. Counters MITRE ATT&CK `T1078.004 Cloud Accounts` (Detect, low-fidelity). **Important:** the dominant 2024-2026 intrusion pattern (T1566.002 → T1528 → T1550.001 token-replay from residential-proxy infrastructure) produces zero impossible-travel signal. Treat this control as **naive credential-stuffing-only**.

> **June 2025 anomaly-policy wave (applies to all MDA detect/* policies).** Several built-in anomaly policies were transitioned to a dynamic-threat-detection model in June 2025 and renamed — *Activity from suspicious IP addresses*, *Suspicious inbox forwarding*, *Suspicious inbox manipulation rules*, *Activity from anonymous IP addresses*, *Ransomware activity*, *Suspicious file access activity by user*, *Unusual ISP for an OAuth App*, *Suspicious email deletion activity*. The Impossible Travel anomaly itself was **not** disabled in that wave, but its operator-facing context (sibling policies in the same Threat detections group) changed materially. Source: [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy] — Important notice.

## What organisations use this for

For most regulated FIs this is a **Day-1 turn-on** because it has no CAAC dependency, no agent, no per-app onboarding, and the built-in anomaly policy is already shipped — you edit it, you do not create it. That low cost is also the trap: the policy is cheap to enable and emotionally satisfying ("we have impossible-travel detection") which leads programmes to over-credit it on the control register. The honest practitioner framing in 2024-2026 is that this control catches naive credential-stuffing botnets, supplies a baseline insider-risk signal, and satisfies an audit checkbox — but it does **not** catch the modern token-replay intrusion pattern, and the control register must say so.

The real architectural decision attached to this policy is not "do we enable it" — it is **where does the actual sign-in-risk decision live**. Entra ID Protection (sign-in risk + user risk) is the high-fidelity signal; MDA impossible-travel is a corroborating input. Programmes that invert that pairing (treating MDA as primary, Entra ID Protection as optional) end up with a credential-defence posture that the dominant attack class walks through.

### Use case 1 — Tier-1 ASEAN universal bank, credential-stuffing botnet response

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, mixed on-prem AD + Entra hybrid
- **Trigger:** Q3 2025 — credential-stuffing botnet (commodity, password-spray-class) hit the tenant's Entra sign-in surface; ~200k sign-in attempts over 72 hours; SOC asked "what would have caught a successful one?". Impossible-travel was the answer that didn't require new licensing
- **Scope:** all employees tenant-wide; built-in anomaly policy at Sensitivity = Low; documented Frequent-travellers exclusion group (~400 RMs, ops, audit) maintained by HR Operations with quarterly review; alert routed to SOC L1 with mandatory correlation to Entra ID Protection sign-in risk before any user-impacting action
- **Outcome:** in the 12 months following turn-on, ~180 alerts; ~12 escalated to L2; 2 confirmed credential-stuffing wins (botnet hit a service-account-style password); 0 confirmed token-replay catches (consistent with the bypass class). Programme retained the policy as a baseline and explicitly documented "does not catch modern intrusion" on the risk register for BNM RMiT third-party-cyber section attestation `[VERIFY against current edition]`

### Use case 2 — Digital-native challenger bank, baseline insider-risk signal

- **Org type:** neobank, ~600 employees, fully remote workforce, M365 E5, Entra-native (no on-prem AD)
- **Trigger:** Series-C raise → enterprise customer base required a documented insider-risk programme; board ask for "what monitoring covers leaving employees signing in from new locations"; impossible-travel was the lowest-friction signal to stand up
- **Scope:** all employees; Sensitivity = Medium (remote-first workforce = lots of new-location legitimate activity, so a noisier slider was unworkable); paired with a sign-in-risk Conditional Access policy that step-ups on Medium-or-above
- **Outcome:** ~30 alerts/month for first quarter; FP rate ~85% driven by remote workers using new home WiFi / coffee shops / personal VPNs; after Frequent-travellers exclusion expanded to "fully-remote workforce" reality and Sensitivity dropped to Low, alert rate fell to ~5/month with ~30% TP. Caught zero confirmed pre-departure exfiltration (no signal — leaving employees signed in from their normal locations). Reframed in board pack as "baseline coverage; not the primary insider-risk control"

### Use case 3 — Merchant acquirer, post-Storm-2372-style OAuth cleanup pairing

- **Org type:** payments processor, multi-jurisdiction (MY / SG / HK / TH), ~4k employees, M365 E5 + Salesforce + Workday
- **Trigger:** 2025 H2 incident-response after a Storm-2372-class device-code phishing campaign hit a partner ecosystem (Microsoft Threat Intel canonical naming — Storm-* cluster naming retained per [Microsoft Learn: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming]; related Iranian-nexus consent-phishing activity is tracked by Microsoft as **Mint Sandstorm**, not the Proofpoint label TA453); CISO asked "if a partner-tenant identity is compromised and the attacker pivots into our tenant via legitimate token, do we see it?". The honest answer was no — token-replay from residential proxy produces no impossible-travel signal. The policy was kept (cheap) and a parallel Entra ID Protection / Token Protection workstream was funded
- **Scope:** all employees + B2B guests; Sensitivity = Low; Frequent-travellers group built from Workday business-travel records (refreshed monthly); alert correlated with Entra ID Protection user-risk score and Continuous Access Evaluation events in Sentinel
- **Outcome:** the decision to **pair** MDA impossible-travel + Entra ID Protection + Token Protection (verify GA status) was the actual win — the impossible-travel signal alone caught 0 of 2 token-replay incidents that year; the paired control caught both via Token Protection re-evaluation. Programme treats this policy as a low-fidelity input, not the decision

### Use case 4 — Tier-2 healthcare insurer, audit-driven turn-on with no operational value

- **Org type:** regional health insurer, ~2k employees, M365 E5, PDPA-MY in scope, HIPAA-adjacent processing for cross-border medical-tourism flows
- **Trigger:** ISO 27001 surveillance audit — control `A.5.16 Identity Management` and `A.8.5 Secure Authentication` `[VERIFY]` flagged gap on anomalous-sign-in monitoring; auditor accepted impossible-travel as evidence of compliance with the control
- **Scope:** all employees; Sensitivity = Low; minimal exception list (Frequent-travellers = 8 named executives); alert routing to IT helpdesk (not SOC — they have no SOC)
- **Outcome:** alert resolved the audit finding; operationally the helpdesk does not triage the alerts (they go to a shared inbox, reviewed monthly); FP rate undocumented; TP rate undocumented. **Flag as unverified outcome** — this is the dominant pattern for under-resourced FIs and the honest answer is that the control is on the register but not actually operated. Auditor satisfied; risk reduction marginal

## Implementation pattern

Typical 8-week rollout for a tenant new to MDA anomaly policies:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Confirm Entra-as-IdP posture; document Frequent-travellers exclusion group with named owner + quarterly review cadence; pull 30-day baseline of sign-in geographies from `SigninLogs` to characterise tenant's normal travel-and-VPN footprint | Exclusion group exists in Entra; baseline geographic distribution documented |
| W2 | Configure built-in Impossible Travel anomaly policy (edit, don't create); Sensitivity = Low or Medium; Action = Alert only; alert destination = SOC L1 queue (or whichever team owns first-look) | Policy live in alert mode |
| W3 | Run a known-good test — sign in from corporate network, then 5 minutes later from a documented-test VPN in a distant geography. Confirm alert fires and lands in the correct destination. Run a known-bad-pattern test where the user is in the Frequent-travellers exclusion to confirm suppression | End-to-end signal flow validated |
| W4 | Triage week-1 alerts; classify TP / FP; identify FP patterns (VPN-driven, cloud-IP, frequent-traveller-not-in-group, same-country false alerts) | First-tuning-pass classifier of FP causes |
| W5 | Refine Frequent-travellers group; add named owners; document review cadence; integrate with HR business-travel data source where one exists (Workday business-travel module, T&E system) | Frequent-travellers list traceable to a system of record |
| W6 | Pair with Entra ID Protection — configure sign-in-risk Conditional Access policy that step-ups (MFA challenge) on Medium-or-above sign-in risk; this is the actual control, MDA impossible-travel is the corroborating signal | Paired control deployed; both alerts visible in Sentinel |
| W7 | Document the SOC triage runbook: alert arrives → correlate to `SigninLogs` for the same `UserId` / `IPAddress` → correlate to Entra ID Protection risk score → correlate to MDA OAuth-grant additions in last 24h → decision tree for L1 → L2 escalation | Runbook signed off |
| W8 | Document the policy as production-ready; record on risk register with explicit "naive credential-stuffing-only; does not catch token replay" annotation; define quarterly review cadence for Frequent-travellers list + Sensitivity slider | Steady-state handoff |
| W9+ | Quarterly tuning + Frequent-travellers list refresh + monthly correlation review with Entra ID Protection trend | Quarterly metric on alert volume + TP rate |

The W6 pairing is the work that matters. A programme that stops at W4 has a control on paper that the dominant 2024-2026 attack class walks through unnoticed.

## Action

- Primary: **alert-only** (do NOT auto-suspend)
- Secondary: **paired with Entra ID Protection sign-in-risk Conditional Access policy** — the actual user-impacting decision lives in Entra, not MDA
- Never: auto-suspend on impossible-travel signal alone (FP rate too high; false-positive suspension during legitimate travel is a reputational hit with the affected user and their manager)

## Scope

- **Users:** all (exclude documented Frequent travellers group with named owner + review cadence)
- **Apps:** sign-in events to any connected app (Entra-mediated)
- **Device posture:** any
- **Network position:** any
- **Traffic:** sign-in events

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → Policies → Policy management → Threat detections → Anomaly detection policy → Impossible travel (built-in; edit, do not create) | Sensitivity slider = Low or Medium initially; Scope = all users; Exclude documented "Frequent travellers" group; Action = Alert only; alert destination = SOC L1 (Sentinel-forwarded) | API-mode; country/region-level granularity only; ML-based VPN / tenant-common-location suppression per Microsoft (independent FP rate not published); 7-day learning window per the built-in anomaly framework | **Country-level only — "there will be no alerts for two actions originating in the same country/region or in bordering countries/regions" per [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy].** KL ↔ SG ↔ JB produces no signal — material gap for MY FIs with cross-border insider scenarios. Defeated by T1550.001 token replay from residential-proxy infrastructure (modern intrusions produce zero signal). VPN-detection enrichment source may be silently unavailable — no documented operator notification for enrichment outage. **Sensitivity slider exposes no numeric threshold** — opaque tuning, all tuning is by trial-and-error (Microsoft does not publish per-slider thresholds; same `anomaly-detection-policy` Microsoft Learn page documents the slider but not the underlying values — practitioner observation that tuning is therefore empirical) |
| Netskope | `[unverified]` — Netskope UEBA with impossible travel | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security anomaly | | | |
| Skyhigh | `[unverified]` — Skyhigh UEBA | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security UEBA | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration for a tier-2 ASEAN BFSI tenant after tuning is complete:

```yaml
policy:
  name: "Impossible travel — tenant-wide, alert-only"
  type: AnomalyDetectionPolicy
  template: BuiltIn-ImpossibleTravel       # edit the built-in; do not create
  sensitivity: Low                          # Low / Medium / High; Low = fewest alerts
  user_filter:
    scope: AllUsers
    exclude_groups:
      - "Frequent Travellers"               # HR-Ops-owned, quarterly review
      - "Field Auditors"                    # legitimately in 5+ countries/month
      - "Executive Committee"               # legitimately in cross-border travel
      - "Cross-border RMs (Cards BU)"       # business-travel via Workday feed
  app_filter:
    scope: AllConnectedApps                 # Entra-mediated sign-in to any connected app
  governance:
    action: Alert
    auto_suspend: false                     # NEVER true on this policy
    notify_user: false                      # do not tip off the actor on real TP
  alerts:
    severity: Medium                        # corroboration required before HIGH
    email_recipients: [soc-l1@example.com]
    sentinel_forward: true                  # mandatory — correlate with SigninLogs
    daily_alert_limit: 100                  # cap to avoid noise saturation
  pairing:
    entra_id_protection:
      sign_in_risk_policy: "CA-021-SigninRisk-StepUp"
      threshold: Medium                     # MFA challenge on Medium-or-above
    token_protection:
      enabled: true                         # verify GA status in current Entra docs
    continuous_access_evaluation:
      enabled: true
  triage_runbook:
    step_1: "Correlate alert IPAddress + UserId to SigninLogs ±30 min"
    step_2: "Pull Entra ID Protection user-risk and sign-in-risk score"
    step_3: "Pull MDA OAuth-grant additions for user in last 24h"
    step_4: "Check Frequent-travellers group membership at alert time"
    step_5: "L1 decision tree: corroborated High risk → L2; otherwise close-as-FP with reason code"
```

The Sensitivity slider has no numeric semantics published by Microsoft — `Low` produces fewest alerts, `High` produces most. Tuning is empirical. For most tier-2 BFSI tenants, `Low` is the sustainable steady-state.

## Variants

### Industry-specific

- **BFSI:** cross-border-RM workforce is the dominant FP source; Frequent-travellers group tied to a Workday business-travel feed where one exists; auditor expectation is that the policy is paired with Entra ID Protection, not relied on standalone; BNM RMiT cybersecurity-operations attestation accepts pairing language `[VERIFY]`
- **Healthcare:** medical-tourism flows + cross-border telemedicine produce legitimate same-time-different-country sign-ins; clinician on-call rotations include legitimate VPN-from-home patterns; HIPAA Security Rule `164.312(b)` audit-controls expectation `[VERIFY]` — alert exists, triage documented
- **Tech:** distributed-engineering workforce + extensive VPN use = high FP baseline; many programmes set Sensitivity = Low and accept the policy as marginal; investment goes into Entra ID Protection + Token Protection instead
- **Retail:** seasonal workforce + cross-border buyer travel; lower sensitivity to compromised-account risk than BFSI (lower-value sessions) so the policy is often deprioritised; PCI DSS Req. 10.2 / 10.4 sign-in-anomaly expectation may apply where the workforce touches CHD `[VERIFY]`
- **Public sector / legal:** lower workforce mobility (less VPN, less travel) = cleaner FP baseline; auditor expectation under public-sector audit regimes is sharper; legal-matter-coordinator cross-border travel is the named exception case

### Maturity-based

- **Immature:** built-in policy enabled at default Sensitivity; no exclusion group, or one that's months out of date; alerts route to a shared inbox no one reads; FP rate undocumented; no pairing with Entra ID Protection. Common at 12 months post-deployment for under-resourced FIs (Use case 4 archetype). The control is on the register but not operated
- **Mature:** Sensitivity tuned to tenant-appropriate level; Frequent-travellers group sourced from a system of record (Workday business-travel, T&E system) and refreshed quarterly; alerts routed to SOC L1 with documented triage runbook; paired with Entra ID Protection sign-in risk + Conditional Access step-up; quarterly review measures alert volume + TP rate
- **Advanced:** MDA impossible-travel is one of multiple signals feeding a UEBA correlation layer (Sentinel or third-party); per-user behavioural baseline considers individual travel patterns; Token Protection + Continuous Access Evaluation deployed to compensate for the token-replay bypass class; programme documents the policy explicitly as "naive credential-stuffing-only" on the control register and routes the real sign-in-risk decision through Entra ID Protection; the Purview Insider Risk Management Adaptive Protection integration (referenced as internal "MDA Policy 12" in the linked playbook — internal numbering, not Microsoft documentation) boosts the impossible-travel signal when corroborated with HR pre-departure flag. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection]. Note that Adaptive Protection now wires into DLP, DLM, and Conditional Access — not just Conditional Access alone

## Control mappings

- **CIS Microsoft 365 Foundations Benchmark v5.0.0 (30 April 2025):**
  - CIS 2.4.3 (L2) — *Ensure Microsoft Defender for Cloud Apps is enabled and configured* (umbrella; Microsoft explicitly names "Impossible travel detection" as MDA-dependent in the CIS benchmark rationale)
  - CIS 5.2.2.6 (L1) — *Ensure the user risk policy is enabled* (Entra ID Protection user-risk Conditional Access — the paired control that owns the user-impacting decision)
  - CIS 5.2.2.7 (L1) — *Ensure the sign-in risk policy is enabled* (the high-fidelity signal; MDA impossible-travel corroborates)
  - CIS 5.2.2.8 (L2) — *Ensure 'Sign-in frequency' is enabled and browser sessions are not persistent* (compensating control for the token-replay bypass class)
- **Microsoft Secure Score (Identity group):** improvement actions in the Identity group cover the paired Entra ID Protection user-risk / sign-in-risk policies; MDA impossible-travel is a Defender-for-Cloud-Apps detection feeding the same posture. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score] and [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]. The current Secure Score taxonomy is four groups — Identity / Device / Apps / Data — not the legacy five-group structure
- **Microsoft Zero Trust — Identity pillar Objective V (Threats are mitigated by Identity Protection signals):** the architectural anchor for this policy. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity]. **Architectural attribution:** the user-impacting sign-in-risk decision is owned by Entra ID Protection + Conditional Access (Identity ZT pillar Objective V); MDA impossible-travel provides a corroborating activity signal
- BNM RMiT clause(s): [BNM RMiT cybersecurity operations](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]` *(illustrative — not regulatory advice)*
- MAS TRM equivalent — sign-in-anomaly monitoring expectation `[VERIFY]` *(illustrative)*
- ISO 27001:2022 `A.5.16 Identity management`, `A.8.5 Secure authentication`, `A.8.15 Logging` `[VERIFY against current edition]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation — workforce-location data)
- NIST CSF 2.0 subcategory(ies): `DE.AE-02` (event data analysed), `DE.CM-07` (unauthorised personnel monitored), `DE.CM-09` (computing hardware/software monitored), `ID.RA-03` (threats identified) `[VERIFY]`
- PCI DSS v4.0 Req. 10.2.1 / 10.4 (anomaly detection on access events) where workforce touches CHD `[VERIFY]` *(illustrative)*

> **Regulatory-advice caveat.** Material on regulatory clauses, supervisory expectations, and audit thresholds in this file is illustrative — not legal or regulatory advice. Validate every regulatory citation against the current edition of the source instrument before relying on it for attestation.

## False-positive risk

- VPN endpoints geographically distant from user — registers as wrong country (Microsoft's ML suppression catches most common VPN ranges but not residential / smaller-vendor VPNs)
- Cloud-provider IPs (no physical location, classified to provider's HQ country)
- Frequent business travellers — name an exception group with documented owner and review cadence
- New WiFi / mobile-hotspot egress geo-located to carrier headquarters, not the user's location
- Personal VPN ("for privacy") used outside the corporate VPN, which then geo-locates to the VPN exit
- Tor / residential-proxy use (rare, but emits an extreme false signal when it happens legitimately for research / pen-test)

## Real-world FP experience

Typical FP-rate trajectory in a tier-2 BFSI tenant new to anomaly policies.

> Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 70-90% | Default Sensitivity catches every VPN-driven sign-in + every legitimate business trip not yet in exclusion group + cloud-IP-relayed sign-ins |
| W4 | 40-60% | After first Frequent-travellers exclusion pass + Sensitivity dropped to Low; remaining FP is mostly VPN + new-WiFi-egress |
| W8 | 20-35% | After exclusion group sourced from Workday business-travel feed; quarterly refresh cadence; SOC triage runbook in place |
| W12 | 15-25% | After pair-deployment with Entra ID Protection sign-in risk — high-fidelity correlation reduces L1 → L2 escalation rate |
| Steady-state | 10-20% | This is the realistic floor for a country-level anomaly with no per-user behavioural baseline; programmes that expect <10% are tuning a control that does not have the resolution to deliver it |

Named FP scenarios encountered repeatedly:

| Scenario | Mitigation |
|---|---|
| Employee at home on personal VPN (NordVPN / Mullvad / etc.) which geo-locates to another country | User-education + AUP statement that personal VPN use is permitted only off-corporate-resources; or accept as recurring FP and document |
| Cross-border RM legitimately in SG one hour, KL the next — flight time too short for MDA's heuristic | Add RM to Frequent-travellers group; document business-travel data feed |
| Auditor / consultant working from a partner firm's office in another country | Time-bounded exclusion during engagement; or accept as FP with named context |
| Cloud-IP sign-in (e.g. user runs a script from an AWS / Azure VM) registering to provider HQ country | Document the user-script-from-cloud-IP pattern; exclude the user where regular pattern; otherwise accept FP |
| Mobile hotspot or in-flight WiFi geo-located to carrier HQ (Lufthansa WiFi → Germany regardless of where the plane is) | Pattern-recognition by SOC; document repeating users |
| New home WiFi after move — user appears to "teleport" to new city | Auto-resolves after 7-day learning window; accept FP during transition |
| Legitimate Frequent-traveller missed from exclusion group (new hire, role change) | Quarterly Frequent-travellers list refresh tied to HR system; named owner |
| Residential-proxy use for legitimate research / pen-test | Time-bounded exclusion or named pen-test account |
| Token-replay attack — looks normal to MDA, no impossible-travel signal | **Not a FP — a true negative gap.** Compensating control = Entra Token Protection + CAE |

## Operational cost

- **Exception-handling load:** low — Frequent-travellers group updated quarterly; ad-hoc additions during business-travel surges (e.g. annual partner conference, quarterly board travel)
- **Triage load:** medium — alerts need correlation with sign-in risk and OAuth-grant history; high FP rate during travel periods makes the queue bursty
- **End-user friction:** low if alert-only; high if (incorrectly) configured to auto-suspend — false-positive suspension during legitimate travel is a trust-destroying event

Typical staffing: 0.05 FTE platform admin (quarterly tuning + Frequent-travellers refresh); 0.1-0.2 FTE SOC triage time depending on tenant size and FP rate. Identity admin commits ~0.05 FTE for the Entra ID Protection pairing maintenance.

## Privacy / data-protection considerations

- Sign-in location data = workforce-location processing; PDPA / GDPR Art. 88 / equivalent treatment required
- IP-address-to-country geo-location accuracy is not a regulated-grade signal — workforce-monitoring decisions must not rest on geo-location alone
- Workforce notice in employee handbook + AUP must cover sign-in monitoring scope (location, ISP, time, device fingerprint where collected)
- DPIA scope: anomaly-detection on sign-in events + cross-correlation with HR data (where the programme integrates with IRM)
- Cross-border consideration: the anomaly-detection processing happens in Microsoft's service region — for MY / SG / HK tenants, Japan East primary-data region is relevant. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/microsoft-365/enterprise/o365-data-locations] on the day of attestation. `[VERIFY against current Microsoft Cloud regions page — Japan East "added 2025 H2" claim is practitioner recall]`
- Compensating-control framing: practitioners must explicitly state on the control register that this is naive-credential-stuffing-only, not modern-attack coverage. Misrepresenting the control's coverage to a supervisor / auditor is the worst case

## Integration with broader programmes

- **Identity-and-access programme:** the actual high-fidelity signal lives in Entra ID Protection (sign-in risk + user risk); MDA impossible-travel is a corroborating input. Programme governance for both should sit with the IAM team, not the CASB team in isolation
- **Insider Risk Management:** the impossible-travel signal can feed Purview IRM Indicators (referenced as internal "MDA Policy 12 — IRM signal-boost integration" in the linked playbook; internal numbering, not Microsoft documentation); standalone the signal is weak, correlated with HR pre-departure flag and OAuth-grant additions it gains fidelity
- **SOC runbook:** alert lands in SOC L1 queue; runbook correlates with `SigninLogs` (Entra), `IdentityInfo` (Defender XDR), `CloudAppEvents` (OAuth-grant additions in last 24h), Entra ID Protection user-risk score; decision tree for close-as-FP vs L2 escalation
- **Annual audit cycle:** policy existence + measured FP rate + named-incident-investigation evidence feed the annual control-effectiveness review under SOX / NIS2 / equivalent operational-risk regimes; supervisory expectation is that the programme has documented the limitation (token-replay bypass) and has compensating controls (Entra Token Protection / CAE / sign-in-risk policies)
- **Board reporting:** quarterly metric — alert volume + TP/FP rate + confirmed credential-stuffing or token-replay events; honest framing is that this is a baseline control, not a primary defence; trend matters more than absolute number
- **Vendor risk / third-party cyber:** for BFSI under BNM RMiT third-party-cyber expectations `[VERIFY]`, the bank's identity-defence posture (including this policy + its pairing) becomes evidence in the supervisor dialogue
- **DPIA refresh:** annual DPIA includes review of sign-in-monitoring scope, workforce notice currency, and cross-border processing footprint

## Anti-patterns specific to this policy

1. **"Enable it, declare done, move on"** — the policy is on, the dashboard shows alerts, no one triages them; control on the register, not operated. This is Use case 4 — auditor-driven turn-on with no operational value
2. **"Treat it as the primary credential-defence control"** — the dominant 2024-2026 attack pattern (T1566.002 → T1528 → T1550.001 token replay from residential proxy) produces **zero** impossible-travel signal. Treating this policy as primary credential defence is a credential-defence posture the modern attacker walks through. The primary control is Entra ID Protection + Token Protection; this policy is corroborating
3. **"Auto-suspend on alert"** — FP rate of 10-30% steady-state means auto-suspend will suspend 10-30 legitimate users for every 100 alerts; legitimate-traveller suspension during business travel is a high-impact UX failure and a trust hit with the affected user's leadership
4. **"Don't bother with the Frequent-travellers exclusion group"** — every business-traveller becomes an FP; SOC queue fills up; analysts learn to close-all-as-FP; the one real TP gets closed-as-FP by reflex
5. **"Frequent-travellers list maintained ad-hoc by SOC, not by HR Ops"** — list drifts; users who left the role stay on the list; new RMs miss the list; quarterly refresh is the discipline
6. **"Trust the country-level signal between bordering / commuting jurisdictions"** — KL ↔ SG ↔ JB / HK ↔ Shenzhen / Geneva ↔ Lyon / NY ↔ NJ — all produce no signal. For MY FIs with cross-border insider scenarios this is a material gap; do not claim cross-border-commuter monitoring coverage from this policy
7. **"Tune the Sensitivity slider expecting numeric semantics"** — Microsoft does not publish the threshold values; tuning is empirical; expect 4-6 weeks of trial-and-error to find tenant-appropriate setting; document the chosen setting and the rationale (not the numeric value — there isn't one)
8. **"Ignore the VPN-detection enrichment outage class"** — the enrichment source can become silently unavailable; the policy degrades to "alert on every VPN sign-in"; FP volume spikes 10× overnight with no documented operator notification. Subscribe to Microsoft service-health as indirect signal; alert when alert-volume spikes >3σ above baseline
9. **"Notify the user when alerted"** — on real TP, tips off the actor who accelerates the remaining session; on FP, generates user confusion ("why is IT asking if I'm in Singapore — I'm in Singapore for a client meeting")
10. **"Skip the Entra ID Protection pairing"** — the actual sign-in-risk decision lives in Entra; without the pairing, this policy is a paper control that doesn't drive a user-impacting decision when it should

## Coverage gaps

- T1550.001 token replay from residential-proxy infrastructure — bypass class; the dominant modern intrusion pattern produces zero signal here. Compensating control: Entra Token Protection (verify GA status), Continuous Access Evaluation forcing re-evaluation
- T1556 Modify Authentication Process — attacker registers their MFA method before exfil, signs in cleanly from a colocated IP, no impossible-travel signal at exfil time. Compensating control: Entra `User registered alternative authentication device` audit + MFA-method addition alerting
- T1528 Steal Application Access Token — same colocation pattern; the consented OAuth grant runs from wherever the attacker has access. The original consent event **is** audit-logged in Entra `AuditLogs` (Consent to application) — the gap this policy cannot close is the absence of fresh **interactive sign-in** events for subsequent OAuth-app-as-actor API calls, which therefore produce no impossible-travel signal even when the actor is geographically implausible. Compensating control: MDA OAuth post-consent cleanup (internal Policy 2 — see playbook reference) + App Governance behavioural-anomaly detection (dynamic detection model, June 2025) `[VERIFY exact Microsoft Learn branding for App Governance behavioural-anomaly feature]`
- Same-country attacks — fundamental fidelity gap; KL ↔ SG / HK ↔ Shenzhen / cross-state US commute all invisible
- Cloud-provider IP sign-ins (legitimate user scripting from AWS / Azure / GCP) — geo-located to provider HQ, can produce signal or be suppressed inconsistently
- Personal VPN by the user (privacy-driven, not malicious) — recurring FP source
- BYOD-mobile sign-ins via carrier WiFi geo-located to carrier HQ — recurring FP
- Default activity-log retention (standard tier) — historical correlation requires Sentinel forwarding or unified data lake. `[VERIFY: 30-day retention figure against current Microsoft Learn data-retention page for Defender for Cloud Apps / Defender XDR advanced hunting]` — **Practitioner inference** until a specific Microsoft Learn URL is cited (XDR advanced-hunting retention is documented separately from MDA activity-log retention)
- See also [`../../08-failure-modes/token-replay-bypass.md`](../../08-failure-modes/token-replay-bypass.md) `[link-target may not yet exist]`

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
