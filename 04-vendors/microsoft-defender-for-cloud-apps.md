# Microsoft Defender for Cloud Apps — practitioner's playbook (v1)

> v1 — 2026-06-10. Supersedes v0 by incorporating five specialist reviews (QA, cyber/MITRE, MDA-deep-expert, system integration, documentation). v0 retained at `microsoft-defender-for-cloud-apps-practitioner-playbook.md`; changelog at `playbook-v0-to-v1-changelog.md`.
>
> **Status:** still in `_research/`. The full BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS clause mapping is deferred to a separate compliance deliverable (`06-compliance/microsoft-defender-for-cloud-apps-control-mapping.md`). Third-party attestation citations to be pulled from the Microsoft Service Trust Portal. Promotion to `04-vendors/` after those land.
>
> **Verification posture:** this playbook is written against MDA / Defender XDR portal as documented at Microsoft Learn accessed 2026-06-10. Re-validate after any of: Defender portal redesign; Purview DCS schema change; Entra Conditional Access schema change; MDE Network Protection update; `CloudAppEvents` schema change. Subscribe to the Microsoft Defender XDR what's-new feed for schema-stability tracking.

---

## Read this if

You are a **security architect or platform engineer at a regulated financial-services firm** whose tenant is already licensed for Microsoft 365 E5 (or has the standalone MDA SKU) and you need to know **what to switch on, in what order, what risk each switch reduces, and what MDA will not do for you.**

## Skip this if

You are doing a CASB vendor evaluation, scoring an RFP, or modelling licence cost. For vendor comparison, see [`../02-capabilities/capability-matrix.md`](../02-capabilities/capability-matrix.md). For market mechanics, see `cyber-business/`.

## How to read this playbook

1. Run the **Day 0 precondition checklist** — if anything fails, stop. The body assumes those preconditions hold.
2. The **Gaps and compensating controls** table near the top is the single canonical "what MDA cannot do for you" list. Read it before any policy.
3. **Day 1 / 30 / 90** is *roll-out cadence*, not maturity classification. Day 30 policies are not "worse" than Day 1 — they are deferred because they require CAAC (Conditional Access App Control) and per-app regression testing.
4. Every policy is structured identically: Risk reduced → What it does → ATT&CK technique countered → Critical prerequisite → Console path → Key configuration → Trap to avoid → Defeated by → Auditable evidence → SIEM artefact.

---

## One-page summary card

| Item | Detail |
|---|---|
| **Licence prerequisite** | M365 E5 OR standalone MDA SKU. **Office 365 E5 alone = OCAS subset (~20% of capabilities; sunsetting — verify EoL date)** |
| **Entra ID prerequisite** | Entra ID P1 required for any CAAC-dependent policy (all Day 30 + several Day 90) |
| **MDE prerequisite** | Required for off-corp Shadow IT discovery AND for Policy 8 endpoint enforcement (Network Protection in block mode) |
| **The bilateral CAAC trap** | Every CAAC policy needs BOTH an Entra Conditional Access policy AND an MDA session policy. Two consoles, two policy objects. Either alone produces a silent no-op |
| **Day 1 policies (4)** | Shadow IT discovery / OAuth post-consent cleanup / Mass-download alert / Impossible-travel alert |
| **Day 30 policies (6)** | Sensitive-label download block (CAAC) / Auto-label PCI / Quarantine stale external shares / Mass-delete anomaly / B2B exfiltration alert / App Governance Predictive Risk |
| **Day 90 policies (4)** | Auto-unsanction GenAI / Block clipboard-paste to ChatGPT / Terminated-user activity in connected SaaS / IRM signal-boost integration |

---

## Day 0 — precondition checklist (do these first; if any fail, do not proceed)

| Check | Verify how | Why it matters |
|---|---|---|
| Licensing | M365 Admin Center → Licensing. M365 E5 OR standalone Defender for Cloud Apps. If only O365 E5: ~80% of this playbook is not deployable; you have OCAS subset only and the SKU is sunsetting (verify end-of-support date at the current `editions-cloud-app-security-o365` doc) | The playbook silently does not work on the wrong SKU |
| Entra ID P1 | Entra portal → Tenant overview → Licences. Required for CAAC | Without it, no inline DLP, no download/upload blocks, no session controls |
| Tenant primary-data region | M365 Admin Center → Org settings → Org profile → Data location. Verify against the current **Microsoft Trust Center / Microsoft Cloud regions** page (accessed `<date>`) | Japan East was added in 2025 H2; India is on the public roadmap for H2 2026. For Malaysian / Singapore / Hong Kong tenants this materially changes the cross-border-transfer calculus under PDPA / GDPR. Do not propose this product to a residency-sensitive FI without verifying the current tenant region |
| Defender for Endpoint coverage | Defender portal → Settings → Endpoints → Onboarding. Confirm fleet coverage % | Required for off-corporate-network Shadow IT discovery. **Required-and-in-block-mode** for Policy 8 to actually block GenAI on endpoints |
| MDE Network Protection mode | Endpoint security baseline policy assigned to managed devices. **Confirm block mode, not audit mode** | "Tag as Unsanctioned" produces zero endpoint enforcement if NP is in audit mode — the single most common Day 90 misconfiguration |
| Microsoft Purview labels published | Purview portal → Information Protection → Labels and policies → Publish status. Labels exist AND are published to the affected user scope | Auto-label policies silently fail if the label is not published to the user, or if DCS is not onboarded to the SharePoint sites |
| Conditional Access policy backup | Export current CA policies via Microsoft Graph or the CA portal export feature | Adding "Use Conditional Access App Control" on top of 30–80 existing CA policies frequently triggers unintended evaluation order; you need a rollback |
| Break-glass exclusions | Documented set of accounts excluded from EVERY CAAC-touching CA policy. **The "Microsoft Defender for Cloud Apps – Session Controls" Enterprise Application must NEVER be in a blanket Block-Access policy** | If misconfigured, every protected user is locked out simultaneously. There must be a documented break-glass for IT / SOC / executive |
| Sentinel + Defender XDR connector | Sentinel workspace exists; the **Microsoft Defender XDR data connector** (canonical for `CloudAppEvents`, `AlertInfo`, `AlertEvidence`) is enabled — NOT the Office 365 connector and NOT Diagnostic Settings on Entra alone | Default activity-log retention is 30 days standard tier; 90 hot + 90 warm = 180 days with the Defender XDR unified data lake (GA 2025); Sentinel forwarding for 1–7 year BFSI retention. Pick the right path |
| `CloudAppEvents` schema version pinned in SIEM queries | Document the schema fields your queries depend on; subscribe to the Defender XDR what's-new blog | Schema renamed `ActivityType → ActionType` (2023) and changed `RawEventData` JSON-string → object (late 2024). Pin and re-test quarterly |

---

## What MDA *is* (in concrete terms)

MDA gives you **four distinct capability sets**, only the fourth requires CAAC:

1. **API governance for sanctioned SaaS.** API connectors to M365 (first-class), plus a defined set: Google Workspace, Salesforce, Box, Dropbox, ServiceNow, Workday, Webex, GitHub Enterprise, Okta, AWS / GCP / Azure resource activity, Atlassian. Connector depth varies; M365 has the largest governance-action set.
2. **Shadow IT discovery from network-egress logs.** Defender for Endpoint feed (roaming devices), log-collector container fed Syslog from your SWG / firewall, or direct integrations with Zscaler / iboss / Corrata / Menlo. Microsoft asserts ~31,000 apps in the catalogue (vendor-self-reported; verify the current figure on the Cloud Discovery overview page, accessed `<date>`).
3. **OAuth grant surveillance and revocation.** Delegated-permission OAuth grants made by users to third-party apps in M365 / Workspace / Salesforce surface in the OAuth apps view. Discovery, ban, per-user revoke, and built-in anomaly detection on misleading-publisher / malicious-consent patterns. **Application-permission (client-credentials / service-principal) grants are mostly NOT in this view** — see Policy 2 and Gaps table.
4. **In-session DLP via CAAC.** Browser sessions to sanctioned apps are redirected through Microsoft's proxy at a `*.mcas.ms` URL (DigiCert G2 chain in 2026). TLS is terminated there; per-app session controls (block download, block upload, block paste, apply sensitivity label) are applied to the rewritten session. **This is the Entra-bound piece** — strip Entra ID P1 and this entire capability collapses.

It is **not** a CASB peer to Netskope / Zscaler / Skyhigh in the architectural sense. No PAC. No agent. No SSL-inspection appliance. The "proxy" is the Entra-mediated CAAC redirect — IdP-mediated reverse-proxy — and it works only for browser sessions and the small set of native apps that authenticate through the IdP.

### DSPM-for-AI prompt-capture depth (clarified)

| AI tool | Capture depth |
|---|---|
| ChatGPT Enterprise (SSO'd to your tenant) | Deep — prompt body + response body + classification verdict |
| Copilot Chat / Copilot Studio | Deep — same as above |
| Gemini for Workspace | Shallow — metadata only (no body capture) |
| Anthropic Claude (claude.ai web) | Shallow — metadata only |
| Perplexity Enterprise | Shallow — metadata only |
| Consumer Copilot, consumer ChatGPT, personal-Gmail Gemini | Not captured (no SSO into your tenant) |

Prerequisites: Microsoft Purview Insider Risk Management (or equivalent) licensing, Purview-onboarded devices (Defender for Endpoint), federated SSO from your tenant to the AI provider, devices reporting to the same tenant. The BFSI buying argument for DSPM-for-AI hinges on which AI tools land in the "deep" column; do not assume Gemini / Claude capture parity with ChatGPT Enterprise.

---

## Gaps and compensating controls (read before any policy)

This is the canonical "what MDA cannot do for you" table. Every Day 1 / 30 / 90 policy assumes you have read this and have a compensating-control plan for the gaps relevant to your environment.

| Gap | Why MDA can't | Compensating control | ATT&CK technique(s) implicated | Cross-ref |
|---|---|---|---|---|
| **BYOD on iOS / Android** | CAAC only protects browser sessions and IdP-authenticating native apps. Mobile native (Outlook Mobile, Teams Mobile, OneDrive sync, third-party mail clients) bypass | **Intune App Protection Policies (MAM)** with copy-paste / save-to-personal-cloud restrictions inside the managed app; Entra CA requiring approved client app | T1530, T1567, T1213 | Policy 5 |
| **Apps not onboarded to CAAC** | Only the apps you explicitly onboard get session policies; everything else is API-mode (post-upload) or invisible | Onboard more apps to CAAC OR accept post-upload-only detection | T1530, T1567.002 | Policy 5 |
| **API-mode is post-upload, not prevention** | Connector latencies: M365 near-real-time per Microsoft (no published SLA — independent measurement varies, plan for <15 min in normal load, longer during throttling); Power BI / Dynamics 365 24-72 hours per `protect-office-365` (re-verify against current docs) | Inline policy via CAAC for prevention; accept API-mode for detection + remediation | T1530, T1567 | Policies 2 / 6 / 7 / 10 / 11 (mass-delete) |
| **OAuth Application permissions (client-credentials)** | Service-principal consents are mostly not in the OAuth apps view. Service-principal abuse is the dominant cloud-persistence pattern (estimated ~70% of OAuth-grant abuse) — Policy 2 covers ~30% | **Defender for Cloud Apps App Governance add-on** for service-principal hygiene; **Microsoft Entra Workload Identities + Conditional Access for Workload Identities**; **Microsoft Entra ID Governance** for periodic application access reviews. For Power Platform: **Power Platform Admin Center DLP** | T1098.001, T1136.003, T1528 | Policy 2b (Day 30) |
| **Watermarking on downloads** | "Apply sensitivity label on download" applies the label's encryption + permissions; visible markings (headers/footers/watermarks) are NOT applied per Microsoft Learn | Use Purview label visible-marking publish settings; Edge for Business native watermark (verify GA status H2 2026) | T1213 (insider screenshot exfil) | Policy 5 |
| **Real-time block of malware in Box / Dropbox / Google Workspace** | Detects only; third-party-app remediation. Only OneDrive / SharePoint scanned by Microsoft's own service before they land | Use the third-party app's own quarantine; CAAC inline malware-inspection if the app is CAAC-onboarded | T1204, T1566.001 | (Day 30 candidate; not promoted to a numbered policy in v1) |
| **No forward-proxy traffic interception** | MDA has no PAC, no agent, no SSL bump. Defender for Endpoint feeds discovery only — not enforcement | Microsoft Global Secure Access (Entra Internet Access) / a third-party SWG | T1567 | (architectural — affects every off-corp scenario) |
| **Certificate-pinned mobile / desktop apps** | CAAC's TLS interception fails on cert-pinned apps; they refuse the connection or silently bypass | **Intune App Protection Policies (MAM)** at the app layer (not at the network layer); restrict to managed apps via Entra CA | T1027 family (encrypted bypass), T1102 | (architectural) |
| **No policy-as-code interface** | Policies are clickops only; no Terraform provider; no documented REST API for creating session/access/file policies | Document the policy-as-code workaround: Git workflow with portal JSON exports as source of truth; `diff` review; an `AuditLogs` query that detects out-of-band portal edits | T1098.003 (defender-evading policy edit) | (operational) |
| **File-policy ceiling** | 200 per tenant (raised from 50 in February 2026 — verify against current `data-protection-policies` page accessed `<date>`); broad consolidated policies dilute audit traceability | Policy-consolidation strategy documented as a control-design decision; one consolidated policy per classification family rather than per SIT | N/A | Policy 6 |
| **Power BI / Dynamics 365 multi-day connector latency** | 24-72 hours per Microsoft Learn; verify against current docs | Inline Purview DLP at the Power BI workspace policy level (separate product surface) for any data that must be evaluated faster than the connector latency | T1530, T1213 | Policy 6 |
| **Cached `*.mcas.ms` redirects survive policy removal** | Deprovisioning a CAAC onboarding cleanly requires four steps across two consoles + session-cookie purge; no Microsoft cache-TTL SLA | Document the `mcas.ms` deprovisioning runbook; communicate user-side session-clear; expect 2–24 hours of "phantom" redirects post-removal | T1190 (residual proxy artefacts) | Appendix — CAAC deprovisioning runbook |
| **Multi-tenant / B2B guest** | CAAC behaviour on B2B guest sessions accessing the host tenant's SharePoint is non-trivial — device claims may come from the home tenant; Cross-Tenant Access Settings determine what is enforced | Cross-Tenant Access Settings (Entra) configured per partner; explicit B2B-guest policy scope in CA + MDA; B2B exfil policy (Day 30) | T1199 | Policy 6b (B2B exfil — Day 30) |
| **CAE-induced session re-establishment** | When Continuous Access Evaluation revokes a session mid-flight, the re-established session may bypass MDA session-policy re-evaluation. Documented as expected by Microsoft; practitioners treat as a control gap | Pair CAAC with Entra Token Protection (verify GA status); enforce sign-in-risk policies at the CA layer to force re-evaluation | T1550.001 | Policy 5 |
| **Default 30-day activity log retention (standard tier)** | Storage tradeoff inside Defender XDR | Sentinel forward + Defender XDR unified data lake (90 hot + 90 warm); budget accordingly | N/A (audit-trail control) | Day 0 checklist |
| **Email body content DLP in Exchange** | MDA policies attachments only; message body is Purview DLP territory | Microsoft Purview DLP (Exchange policy) | T1114 | (architectural) |
| **No DLP on Teams chat body** | Beyond Purview DLP for Teams | Purview DLP for Teams | T1213 | (architectural) |
| **Structured-data DLP inside SaaS** (Salesforce object / row-level) | MDA does not inspect SaaS database rows | Salesforce Shield; SSPM tools (vendor list shifts with M&A — verify currency) | T1213 | (architectural) |
| **OCAS subset SKU** (O365 E5 alone) | No third-party connectors, no CAAC beyond Office, no app governance, no GenAI category | Migrate to M365 E5 OR plan migration before SKU EoL | N/A (licensing) | Day 0 checklist |

---

## Concentration risk (top-of-stack)

A regulated FI running M365 + Defender XDR + Entra + Purview + MDA is single-vendor for IdP, EDR, CASB-equivalent, DLP, and (with Global Secure Access) SSE. In technique terms:

- **CAAC proxy compromise** = T1557 Adversary-in-the-Middle at internet-scale across every CAAC-onboarded app simultaneously.
- **Single Entra identity-plane compromise** = T1098.001 / T1556 / T1528 across the entire Microsoft stack at once.
- **Microsoft service-side incident** (the supply-chain class — Storm-0558 2023, Midnight Blizzard 2024) leaves no in-tenant detection.

This is a textbook concentration-risk pattern under BNM RMiT (third-party / outsourcing sections — verify against current edition), MAS Outsourcing, and EU DORA concentration-risk recitals. Promote MDA as a primary control with this risk on the register; compensating controls live outside the Microsoft stack (independent SIEM with raw log forwarding; non-Microsoft IdP for break-glass; non-Microsoft secrets management).

---

## Day 1 — turn-on essentials (no CAAC required)

**Prerequisites:** Day 0 checklist complete. SOC alerting destination decided. Exception process documented for the policies that will start firing alerts.

| Policy | Requires CAAC | Requires Entra P1 | Requires MDE | Requires Purview labels | Action class | Blast radius |
|---|---|---|---|---|---|---|
| 1. Shadow IT discovery | No | No | Recommended | No | Discovery + tag | Low (no enforcement) |
| 2. OAuth post-consent cleanup | No | No | No | No | Detect + manual ban | Medium (ban is tenant-wide) |
| 3. Mass-download anomaly (alert) | No | No | No | No | Alert | Low |
| 4. Impossible-travel alert | No | No | No | No | Alert | Low |

### Policy 1 — Shadow IT discovery + GenAI category triage

| | |
|---|---|
| **Visibility precondition for** | Risk decisions about unsanctioned SaaS and unsanctioned GenAI. Discovery does not reduce risk on its own; it enables enforcement decisions (Policy 8) and meets ISO 27001 / 27017 / NIST CSF ID.AM-04 expectations on SaaS inventory |
| **What it does** | Ingests network-egress logs (MDE feed, log-collector container fed by SWG/firewall Syslog, or direct Zscaler / iboss / Corrata / Menlo integration) and matches activity against Microsoft's cloud-app catalogue. Surfaces each discovered app with risk score, user count, and traffic volume |
| **ATT&CK technique(s)** | Visibility precondition for T1567 Exfiltration Over Web Service, T1102 Web Service C2 candidates; not a direct counter |
| **Critical prerequisite** | At least one data source connected (start with MDE for roaming visibility) |
| **Console path** | `security.microsoft.com` → Cloud apps → Cloud discovery. Configure data sources at System → Settings → Cloud apps → Cloud discovery → Automatic log upload. (Re-verify the 2026 unified-portal path on the current Cloud Discovery overview page, accessed `<date>`.) |
| **Key configuration** | Connect at least one data source. Sanction known-approved apps (tag = Sanctioned). For unknown apps, leave untagged for analyst review. Subscribe to weekly discovery report into a security-analyst queue |
| **WARNING — trap to avoid** | **The "discovered subdomains" capability was deprecated 2025-12-31 and is already not reliable as of today.** Any control evidence you built on subdomain-level Shadow IT differentiation (e.g. distinguishing your own SharePoint Online tenant from a rogue tenant) needs re-baselining before any compliance attestation cycle |
| **Defeated by** | Encrypted DNS-over-HTTPS bypassing the SWG, traffic from BYOD-mobile not visible to MDE / log collector, apps with rapidly-rotating domains (CDN-fronted GenAI vendors) |
| **Auditable evidence** | Table `CloudAppEvents` (events of class `DiscoveryReport` / app-tag changes); retention 30 days standard tier, 90+90 days with unified XDR data lake, Sentinel forwarding for 1-7 year BFSI retention. Weekly discovery report is a CSV with no tamper-evidence claim — treat as analyst input, not audit artefact |
| **SIEM artefact + connector** | Sentinel via the Defender XDR data connector |

### Policy 2 — OAuth post-consent cleanup + behavioural detection (split: 2a / 2b)

**Reframing from v0:** OAuth post-consent governance via MDA is a **cleanup control**, not a primary defence. The primary control for T1528 Steal Application Access Token is **Entra admin-consent workflow + verified-publisher restriction** (configured in Entra, not MDA). MDA Policy 2 inspects what already happened.

#### Policy 2a — Classic OAuth apps page anomalies

| | |
|---|---|
| **Risk reduced** | Cleanup of consented OAuth apps that match anomaly signatures (misleading publisher name, malicious-consent patterns), and per-user revocation of confirmed-bad grants |
| **What it does** | Catalogues delegated-permission OAuth grants to M365 / Workspace / Salesforce. Built-in anomaly detection fires on Misleading OAuth app name / Misleading publisher name / Malicious OAuth app consent. Manual workflow: Approve / Ban (tenant-wide) / Revoke (per-user) / Report-to-Microsoft |
| **ATT&CK technique(s)** | T1528 (Detect + Respond, post-consent); T1550.001 Application Access Token (Detect on subsequent token use); T1567 / T1071.001 (Detect, post-event exfil signal) |
| **Critical prerequisite** | None beyond an M365 / Workspace / Salesforce connector being live |
| **Console path** | Defender portal → Cloud apps → OAuth apps. Built-in anomaly policies under Cloud apps → Policies → Policy management → Threat detections. (Re-verify the 2026 portal path; "App governance" may be a separate left-nav item if the add-on is enabled.) |
| **Key configuration** | Day 1: review every High-permission, low-community-use, unverified-publisher app. **Manually ban known-malicious.** Do NOT enable auto-revoke on the built-in anomaly policy until you've measured FP rate for at least 4 weeks. Restrict user consent to verified-publisher apps at the Entra layer in tandem |
| **WARNING — trap to avoid** | **Banning is tenant-wide and immediate, propagating on next token validation; CAE-aware apps revoke faster.** No staged rollout. Before banning, query `CloudAppEvents` for user-count and recent usage. Practitioners ban a flagged app at 11pm and wake up to dozens of broken integrations |
| **Defeated by** | T1098.001 service-principal credentials added to a legitimate-looking app (Application-permission grants — see Policy 2b). Personal-account OAuth (user signs into a third-party app with personal Gmail / personal MSA — Entra never sees it) |
| **Auditable evidence** | `CloudAppEvents` records the grant / revoke; Entra `AuditLogs` records the consent action by admin/user. Field carrying before/after permission state: review the `RawEventData` schema (changed from JSON string to object late 2024 — pin version) |
| **SIEM artefact + connector** | Sentinel via the Defender XDR data connector + Entra ID Sign-in / Audit logs |

#### Policy 2b — App Governance Predictive Risk + OAuth threshold policies (Day 30 add-on)

| | |
|---|---|
| **Risk reduced** | Behavioural post-consent abuse — a legitimately-consented app starts exfiltrating after compromise of the developer, or a low-volume app suddenly behaves anomalously |
| **What it does** | App Governance Predictive Risk (shipped 2025 H2) machine-learned anomaly scoring of in-tenant OAuth-app behaviour. OAuth threshold policies allow configurable thresholds (count of grants per app per window, scope-expansion alerts, geographic anomalies) |
| **ATT&CK technique(s)** | T1098.001 Additional Cloud Credentials (Detect), T1550.001 (Detect), T1136.003 Cloud Account creation via service-principal (Detect) |
| **Critical prerequisite** | App Governance add-on licensed and enabled |
| **Console path** | Defender portal → Cloud apps → App governance → Predictive risk policies + OAuth threshold policies |
| **Key configuration** | Enable Predictive Risk in **notify-only** mode for 4 weeks; tune; enable auto-revoke only on highest-confidence signal classes |
| **WARNING — trap to avoid** | Predictive Risk false-positives on legitimate burst usage (DR rehearsals, content-migration runs). Build a per-app suppression workflow before auto-revoke |
| **Defeated by** | First-time abuse of a never-before-anomalous app — the model has nothing to compare against |
| **Auditable evidence** | App Governance event records in `CloudAppEvents`; cross-correlate with Entra `AuditLogs` for grant-event lineage |
| **SIEM artefact + connector** | Sentinel via Defender XDR data connector |

**Power Platform sub-class.** Power Platform service-principal consents are under-surfaced in App Governance. Heavy Power Platform tenants must additionally configure Power Platform Admin Center DLP (separate console).

### Policy 3 — Mass-download anomaly with alert (NOT auto-suspend)

| | |
|---|---|
| **Risk reduced** | Insider exfiltration via bulk download before resignation; compromised-account exfiltration after credential theft |
| **What it does** | Activity policy on download-class events from connected apps. Threshold-based; not behavioural baselining (it is rule-based with a 7-day learning window, not UEBA in the strict sense) |
| **ATT&CK technique(s)** | T1530 Data from Cloud Storage Object (Detect, post-event); T1213.002 SharePoint (Detect, post-event); T1567.002 (Detect, post-event) |
| **Critical prerequisite** | M365 connector live; SOC alert destination configured; service-account exclusion list maintained |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Create policy → Activity policy |
| **Key configuration** | Activity type = Download file. Apps = SharePoint, OneDrive, plus any other connected app where bulk download is plausible. **Threshold = starting heuristic only — derive your tenant's value as P95 of legitimate users over a 30-day baseline, tighten on a documented cadence.** For pilot, start at 5,000 in 30 minutes (high to avoid noise) and narrow. Action = Alert + email to SOC. **Do NOT enable auto-suspend on first turn-on.** Exclude service accounts by UPN regex; exclude documented migration accounts |
| **WARNING — trap to avoid** | **"Suspend user" governance action disables the Entra user object — it is NOT a soft suspension.** Underlying call is `POST graph.microsoft.com/v1.0/users/{id} accountEnabled=false` running under the Defender service principal. Downstream: SCIM cascade into Entra-provisioned apps (40-min default latency); Exchange mailflow breakage if the user is a shared-mailbox delegate; Power Automate flows under that account stop; SOC cannot un-suspend cleanly without resetting credentials. **Silently fails on hybrid-AD users with `onPremisesSyncEnabled=true` and no writeback** — attacker keeps the session. Validate `IdentityInfo` table that `AccountEnabled` actually flipped |
| **Defeated by** | Slow-and-low exfil — 100 files/day × 30 days defeats any sub-day threshold (see Appendix B for the slow-and-low Sentinel KQL). Native-sync staging via OneDrive sync client (T1074.002) produces sync events, not download events |
| **Auditable evidence** | `CloudAppEvents` carries the download events with `AccountObjectId`, `IPAddress`, `ActionType=FileDownloaded`, `Application`; the activity-policy match adds `PolicyId`, `PolicyName`. The "Suspend user" action: Entra `AuditLogs` records the actor as the Defender service principal; correlate to the originating policy match via `CorrelationId`. **For an auditor: state explicitly that the audit trail is two records minimum — `CloudAppEvents` (policy match) + `AuditLogs` (governance action). Sentinel forwarding preserves both; default retention 30 days standard, 90+90 with data lake** |
| **SIEM artefact + connector** | Sentinel via Defender XDR data connector |

### Policy 4 — Impossible travel + activity from infrequent country (alert-only)

| | |
|---|---|
| **Risk reduced** | Naive-credential-stuffing — attacker logs in from a different geography than the legitimate user, both within a window MDA's heuristic considers infeasible |
| **What it does** | Built-in anomaly policy with a tunable sensitivity slider. Country-level granularity; ML-based suppression of VPN endpoints and tenant-common locations (Microsoft's claim; independent FP rate not published) |
| **ATT&CK technique(s)** | T1078.004 Cloud Accounts (Detect, low-fidelity, **naive-credential-stuffing only**) |
| **Critical prerequisite** | Entra-as-IdP for the governance signal; documented "Frequent travellers" exclusion group with a named owner and review cadence |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Anomaly detection policy → Impossible travel (built-in; edit, don't create) |
| **Key configuration** | Sensitivity = Low or Medium initially. Scope = all users; exclude the documented "Frequent travellers" group. **Action = Alert only.** Pair with Entra ID Protection sign-in risk for higher-fidelity correlated signal |
| **WARNING — trap to avoid** | **Country-level only — no alerts within the same country or between bordering countries.** KL ↔ SG ↔ JB produces no signal. For an MY FI with cross-border insider scenarios this is a material gap. The compensating control is Entra ID Protection user-risk + sign-in-risk policies, not MDA |
| **Defeated by** | **T1550.001 Application Access Token replay from residential-proxy infrastructure placing attacker IP near the user.** Modern intrusions (T1566.002 → T1528 → T1550.001 token replay) produce zero impossible-travel signal. **MDA's impossible-travel is a naive-credential-stuffing-only detector** — the dominant 2024-2026 attack pattern bypasses it. Compensating controls: Entra Token Protection (verify GA status); Continuous Access Evaluation; sign-in-risk policies that enforce step-up |
| **Auditable evidence** | `AlertInfo` + `AlertEvidence` (Defender XDR) carry the alert with sign-in source, ISP, country. If the VPN-detection enrichment source is unavailable, the policy degrades silently — there is no documented operator notification for enrichment-source outage. Subscribe to Microsoft service-health for indirect signal |
| **SIEM artefact + connector** | Sentinel via Defender XDR data connector |

---

## Day 30 — hardening (CAAC required for policies 5, 6, 6b, 7)

**Prerequisites:** Day 1 stable. SOC alert tuning done. Per-app CAAC onboarding plan documented. **Per-app onboarding effort estimate:** standard M365 / Salesforce / Box / Workday on Entra = 1–3 days incl. regression test; Okta-fronted SaaS requiring re-federation = 2–4 weeks; cert-pinned native app = unsupported, drop from CAAC scope; B2B-guest CAAC = case-by-case.

**The bilateral CAAC trap (re-stated):** every CAAC session policy in MDA requires a matching Entra Conditional Access policy with "Use Conditional Access App Control" as the session control. **Two consoles. Two policy objects. Three when you count the Entra app registration / Enterprise Application for non-M365 apps.** Without the Entra side: the MDA policy never fires. Without the MDA side: the Entra side redirects the user through the proxy but nothing in-session blocks. Both silent failures.

### Pre-CAAC compatibility test (per app, before onboarding)

For each app you intend to onboard:
- SAML federation works against your IdP (Entra or Okta-federated-to-Entra)
- In-session features still work after URL rewrite — especially **`window.parent.postMessage` cross-origin failure** (dominant 2026 breakage class; affects Salesforce Lightning, ServiceNow Polaris, Workday Extend, Box file-preview, Atlassian Compass, Confluence whiteboard)
- The `*.mcas.ms` DigiCert G2 cert chain is accepted on managed endpoints (strict cert-pinning device profiles may reject — fails closed at TLS but looks fail-open at app layer)
- Anti-CSRF tokens survive rewrite
- OIDC iframe re-auth survives rewrite
- Third-party-cookie flows still work
- SaaS-to-SaaS embedded OIDC initiated mid-session still works
- Anti-bot / WAF on the SaaS vendor side does not block Microsoft proxy IP ranges

Practitioner cadence: test plan template, regression-test environment, sign-off on every app before policy 5 / 6 / 7 / 6b is scoped to it.

### Policy 5 — Block download of Confidential-labelled files to unmanaged browsers (M365)

| | |
|---|---|
| **Risk reduced** | Sensitive-document exfiltration via personal device — employee signs into SharePoint from a personal laptop / home browser and downloads Confidential files |
| **What it does** | When the user signs in to M365 from a non-Entra-hybrid-joined and non-Intune-compliant device, allow in-session view/edit but block download of any file labelled Confidential or above |
| **ATT&CK technique(s)** | T1530 via untrusted endpoint (Prevent for the download leg only). **Out of scope:** T1550.001 token-replay exfil; T1213 in-session screenshot; managed-endpoint exfil |
| **Critical prerequisite** | **Step 0: confirm app state = `Enabled` (not `Discovered`) on Settings → Cloud apps → Conditional Access App Control → Connected apps → Conditional Access App Control apps. Until that row exists per-user-first-touch, the session policy literally cannot match.** Also: Entra ID P1; matching Entra CA policy with "Use Conditional Access App Control"; Purview labels published and DCS onboarded |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create policy → Session policy. PLUS the Entra Conditional Access policy in `entra.microsoft.com` |
| **Key configuration** | Session control type = Control file download (with inspection). App = Office 365 bundle (NOT individual apps — picking individual apps causes inconsistent CAAC interception across Office workloads). Client app = Browser only (omit Mobile and desktop to avoid breaking Outlook / OneDrive sync clients). Device filter = Device tag ≠ MicrosoftEntraHybridJoined AND ≠ IntuneCompliant. File filter = Sensitivity label ∈ {Confidential, Highly Confidential}. Action = Block + custom user message linking to your device-enrolment page. `Always apply if data cannot be scanned` = **document the decision** (false = fail open on unscannable = data leakage via user-encrypted archive; true = block all unscannable = false-positive on every legitimate Zip / OneNote / proprietary file format). State the trade-off in your control documentation |
| **WARNING — trap to avoid** | **Bilateral failure:** the inverse trap of an Entra CA "Use Conditional Access App Control" without a matching MDA session policy = redirect to a no-op proxy that adds `*.mcas.ms` rewrite but does nothing visible. Always verify both halves with a test user before scoping broadly. **CAE-induced re-establishment:** when CAE revokes mid-session, the re-established session may bypass MDA policy without re-evaluation — pair with Entra Token Protection |
| **Defeated by** | T1550.001 token-replay from managed device (token issued legitimately, replayed elsewhere — attacker bypasses the device-tag filter entirely). T1027 user-encrypted archive — attacker 7zips with password, uploads/downloads anyway, label cannot read content. T1213 in-session screenshot of the file (the proxy cannot prevent the user taking a photo). Cert-pinned device profile rejecting the `*.mcas.ms` chain (fails closed at TLS — silent) |
| **BYOD read-only fallback** | Pair with a parallel session policy on the same scope: Action = Audit (not Block), Session control = limit downloads, Device filter = same. Spec the read-only policy as a peer to this one; do not leave "BYOD will use read-only" as a TODO |
| **Auditable evidence** | Three correlated records: Entra `SigninLogs` (auth event with `ConditionalAccessStatus = Success`); Entra `AuditLogs` (CA policy result); Defender XDR `CloudAppEvents` (session-policy match with `PolicyId`, `Verdict = Block`). **All three required for an auditor.** Sentinel KQL: `CloudAppEvents | join SigninLogs on $left.AccountObjectId == $right.UserId | join AuditLogs on $left.CorrelationId == $right.CorrelationId` within ±30s of `TimeGenerated`, intersected on `IPAddress` |
| **SIEM artefact + connector** | Sentinel via Defender XDR data connector (`CloudAppEvents`) + Entra ID Sign-in / Audit logs (separate connector) |

### Policy 6 — Auto-label PCI-containing files in OneDrive / SharePoint with *Confidential — Finance*

**Reframing from v0:** Policy 6 is **not an exfil control**. It is a **labelling precondition** for downstream encryption AND for Policy 5's `Sensitivity label ∈ {Confidential}` filter to fire. Without Policy 6 (or an equivalent labelling control), Policy 5's filter matches nothing.

| | |
|---|---|
| **Risk reduced** | Cardholder-data sprawl in OneDrive / SharePoint — credit-card numbers in spreadsheets, dragged into OneDrive, emailed to self. Policy 6 finds and labels them; the label carries encryption + permissions per Purview configuration |
| **What it does** | API-mode file policy. Scans existing and new files in OneDrive / SharePoint via Microsoft Graph; runs Purview Data Classification Service (DCS) for content inspection; applies the sensitivity label automatically on match |
| **ATT&CK technique(s)** | Precondition for label-encryption controls that frustrate T1565.001 (Stored Data Manipulation) and T1486 (Data Encrypted for Impact); not a direct counter |
| **Critical prerequisite** | The *Confidential — Finance* label must already exist in Purview, with encryption + permissions configured, label-publishing scope including affected users, DCS onboarded to the SharePoint sites. If any of these silently fail, the policy fires but no label is applied. Validate end-to-end with a known-positive test file before broad rollout |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Information protection → Create policy → File policy. Label management lives in Microsoft Purview → Information Protection → Labels and policies |
| **Key configuration** | App filter = OneDrive for Business + SharePoint Online. Parent folder scope = narrow to your finance / cardholder-data folders on day one; do NOT run tenant-wide initially (DCS quota + Graph throttling). File filter: Inspection method = Data Classification Service; SIT = Credit Card Number; **minimum violations = 5 as a recommended starting heuristic** (1 produces too many false positives on test data / mock cards / cancelled cards). Governance action = Apply Microsoft Purview sensitivity label → *Confidential — Finance*. Alert + email per match. **Run for 1 week with action = Alert-only; flip to label-apply after FP review** |
| **WARNING — trap to avoid** | **The 100-character matching-content preview snippet is itself cardholder data.** It is stored in Defender's incident metadata. Under PCI DSS 3.4 / 3.5, the storage and access control around that snippet is itself a regulated control. Document the snippet retention and access list. Applying a Purview encryption label via MDA does NOT propagate to existing co-author sessions — users with the file open retain unencrypted access until they reopen |
| **Defeated by** | Image-format card numbers (DCS does not OCR), password-protected files (cannot inspect content; tagged as `Password encrypted` and skipped), files larger than DCS content-inspection limit, Azure-RMS-pre-encrypted files |
| **Graph API rate budget** | File policies pull file content via Graph. At tenant-wide scope they saturate (~600 req/min/tenant SharePoint Online bucket); MDA backs off to partial coverage without operator notification. Scope by parent folder on rollout; monitor Graph throttling headers via Sentinel rather than relying on MDA UI signal |
| **File-policy ceiling** | 200 per tenant (verify against current `data-protection-policies` page). Consolidate per classification family (not per SIT); document consolidation as a control-design risk acceptance |
| **Auditable evidence** | Label-application event in M365 unified audit log (`MIPLabel`); MDA file-policy match event in `CloudAppEvents` with `PolicyId`. Auditor will ask: prove the label was applied at T0 by which policy version. Pin policy version in your runbook |
| **SIEM artefact + connector** | Sentinel via Defender XDR + M365 unified audit log (separate connector) |

### Policy 6b — B2B / partner-tenant exfiltration alert (Day 30 addition)

| | |
|---|---|
| **Risk reduced** | Trusted-relationship exfil — external guest user downloads / shares N+ files from your tenant in a window. The Storm-0539 / Storm-2372-style consent-phishing-into-B2B attack chains rely on this surface |
| **What it does** | Activity policy on external-domain guest user file operations. Alert when an external user crosses a download / share threshold |
| **ATT&CK technique(s)** | T1199 Trusted Relationship (Detect); T1213.002 SharePoint (Detect) |
| **Critical prerequisite** | M365 connector live; documented list of expected B2B partner tenants; Cross-Tenant Access Settings reviewed in Entra |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Create policy → Activity policy |
| **Key configuration** | Activity type = Download file OR Share file. User type = External user. Threshold = baseline-derived; start permissive (1,000 in 24h) and tighten. Action = Alert + email to identity admin and SOC |
| **WARNING — trap to avoid** | "External user" classification depends on the user's home tenant claim; if a user has been mis-classified as guest vs member at federation time the policy filter misses them |
| **Defeated by** | A compromised partner-tenant account behaving within threshold per session — slow-and-low B2B exfil. Pair with Cross-Tenant Access Settings restricting partner scope |
| **Auditable evidence** | `CloudAppEvents` with `AccountType=Guest` or `IsExternalUser=true` (verify current field) + `PolicyId` |
| **SIEM artefact + connector** | Sentinel via Defender XDR |

### Policy 7 — Quarantine externally-shared OneDrive files with no recent activity (two-stage)

| | |
|---|---|
| **Risk reduced** | Sharing drift — old external share links accumulate over years, each a small leak |
| **What it does** | API-mode file policy. Identifies OneDrive files shared externally with last-modified >180 days. Two-stage: notify owner with 14-day grace; quarantine on no-response |
| **ATT&CK technique(s)** | T1213 Data from Information Repositories (cleanup, long-tail) |
| **Critical prerequisite** | OneDrive connector live; documented owner-notification workflow; folder allowlist for known-shared content |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Information protection → Create policy → File policy (two separate policies, 14 days apart) |
| **Key configuration** | App filter = OneDrive for Business. Access level = External OR Public-with-a-link OR Public. Last modified = >180 days. **Stage 1 (notify-only):** governance action = Notify owner; severity = Medium. **Stage 2 (quarantine, separate policy 14 days later):** governance action = Put file in admin quarantine; severity = High. Exclude folder allowlist (e.g. /Shared Templates) |
| **WARNING — trap to avoid** | **Single-step quarantine breaks links for internal users and external recipients without warning; owner-notification emails often land in junk.** Always run the two-stage flow. **A quarantined file under Microsoft Purview eDiscovery legal hold may be spoliation** — Purview hold wins, but your MDA operator may not know about the hold at policy-design time. Consult Legal before policy design; cite the Purview eDiscovery preservation interaction docs |
| **Defeated by** | Files whose `last-modified` is touched by automated processes (BCM rehearsals, schema-update jobs); shares to known-trusted external domains that ought to persist |
| **Auditable evidence** | `CloudAppEvents` records the governance action; M365 unified audit log records the file move |
| **Graph API rate budget** | Same as Policy 6 — file enumeration via Graph; scope by parent folder |
| **SIEM artefact + connector** | Sentinel via Defender XDR + M365 unified audit log |

### Policy 11 — Mass-delete / mass-overwrite anomaly on M365 (Day 30 addition)

| | |
|---|---|
| **Risk reduced** | Ransomware-in-SaaS — Storm-0501 / BlackCat-on-SaaS variants delete or overwrite files at scale inside the tenant after compromise |
| **What it does** | Activity policy on `File deleted` and `File modified` events at high velocity per user |
| **ATT&CK technique(s)** | T1485 Data Destruction (Detect); T1486 Data Encrypted for Impact (Detect) — the SaaS variant |
| **Critical prerequisite** | M365 connector live; SOC alert destination |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Create policy → Activity policy |
| **Key configuration** | Activity type = File deleted OR File modified. Apps = SharePoint, OneDrive. Threshold = baseline-derived; start at 500 deletions in 30 minutes per user. Action = Alert + manual governance review (NOT auto-suspend on first turn-on) |
| **WARNING — trap to avoid** | Legitimate bulk-delete events: SharePoint site archival, content migrations, mailbox cleanup. Exclude documented migration / archival accounts by UPN |
| **Defeated by** | Slow-and-low destruction (100 files/hour over days); attacker pre-compromises an excluded account |
| **Auditable evidence** | `CloudAppEvents` with `ActionType=FileDeleted` / `FileModified`; SharePoint version history retains pre-deletion content for the retention period |
| **SIEM artefact + connector** | Sentinel via Defender XDR |

---

## Day 90 — advanced (higher blast radius; require operational maturity)

**Prerequisites:** Day 1 + Day 30 stable; SOC integration done; exception process and change management documented.

**Platforms that supersede MDA controls in 2026:** Microsoft Edge for Business `gen-ai.app-control` (H2 2026 GA, verify); Microsoft Global Secure Access (GSA) native Generative-AI category enforcement; Microsoft Purview Adaptive Protection for GenAI (Q1 2026 GA). For Edge-on-Windows estates with GSA licensed, prefer these over the MDA → MDE Network Protection chain — fewer dependencies, no audit-mode footgun, more reliable enforcement.

### Policy 8 — Auto-unsanction GenAI apps above risk threshold

| | |
|---|---|
| **Risk reduced** | Data leakage to consumer-tier GenAI — employees paste source code, customer data, financial models into ChatGPT / Bard / Claude / Perplexity. Consumer-tier accounts may train on the content |
| **What it does** | App-discovery policy on the Cloud Discovery stream. Tags GenAI-category apps above a risk threshold as Unsanctioned, which propagates to MDE-managed endpoints and blocks via MDE Network Protection |
| **ATT&CK technique(s)** | T1567 Exfiltration Over Web Service to AI endpoint (Detect on Discovery; Prevent only via MDE NP block) |
| **Critical prerequisite** | **MDE Network Protection in block mode** (not audit mode) — without this, "Tag as Unsanctioned" does nothing at the endpoint. Windows 10/11 supported builds per the current MDE Network Protection supported-OS page (re-verify; the legacy "1709+" line is anachronistic and 1709 reached end of support October 2019). Endpoints reporting to the same tenant. Approved-AI allowlist documented with quarterly review cadence |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Shadow IT → Create policy → App discovery policy |
| **Key configuration** | Category filter — **use the 2025 H2 sub-categories** (LLM, AI Coding Assistant, AI Image Generator, AI Video Generator, AI Voice/Avatar) rather than the broad "Generative AI" lump. Risk score **≤ 5 (where lower = higher risk on Microsoft's 1-10 scale)**. Users per day > 10. Action = Tag as Unsanctioned + Alert. Allowlist approved AI vendors (enterprise ChatGPT, Copilot, internal AI) by app ID |
| **WARNING — trap to avoid** | **Microsoft's risk score is opaque** (uncited "90+ factors", weights not disclosed). An auditor cannot accept it as the sole basis for automated enforcement. Document a parallel internal risk-acceptance / appeal process for when an approved-by-business vendor scores ≤5 (e.g. "SOC 2 = No" in Microsoft's catalogue but enterprise contract exists). The **"Generate block script for SWG"** output is a flat IP/domain list that does not match modern GenAI vendor egress (CDN-fronted, JA3-fingerprinted, dynamic domains) — partial fail at the SWG |
| **Defeated by** | BYOD / hotspot (off-corp network = MDE NP not in path); personal MSA or personal Google sign-in to ChatGPT (Entra never sees session); CDN-fronted vendor domains (T1090.002, T1568.002, T1102 — SWG block script defeated by class); image / audio bypass of clipboard inspection (DCS does not OCR / transcribe). **Supply-chain compromise of an approved AI vendor (AML.T0010)** — allowlist becomes the attack surface |
| **P0 control-design requirement** | The approved-AI allowlist is a **third-party concentration-risk decision**. Required: documented quarterly review cadence; kill-switch posture documented; supply-chain-compromise contingency (AML.T0010); change-management for adds/removes |
| **Auditable evidence** | `CloudAppEvents` with `ActionType` for App tag changed / App marked as unsanctioned (verify current field name — MS has renamed); MDE Network Protection block events at the endpoint via Defender XDR `DeviceNetworkEvents` |
| **SIEM artefact + connector** | Sentinel via Defender XDR data connector |

### Policy 9 — Block clipboard paste of sensitive data into ChatGPT browser session

| | |
|---|---|
| **Risk reduced** | Targeted GenAI data leakage at the browser session layer — block paste of cardholder data, SSN-class PII, source code, internal project codenames into a ChatGPT browser session |
| **What it does** | CAAC session policy on the ChatGPT app. Inspects clipboard Paste (and optionally Cut / Copy / Sent message) for DLP signatures and blocks on match |
| **ATT&CK technique(s)** | T1567 Exfiltration Over Web Service narrowed to clipboard sub-channel (Prevent — partial). **Not T1052 (Physical Medium)** |
| **Critical prerequisite** | **ChatGPT must be onboarded to CAAC via SAML federation. Consumer ChatGPT (chat.openai.com with free or personal Plus account) does NOT support SAML.** This policy is deployable ONLY against ChatGPT Enterprise or ChatGPT Team plan with SSO configured (verify against OpenAI's current SSO-by-plan page, accessed `<date>` — OpenAI plan parity has shifted in the past) |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create policy → Session policy. Plus the matching Entra CA policy |
| **Key configuration** | Session control type = Block activities. App filter = Custom ChatGPT Enterprise (Manual onboarding). Activity type = Clipboard Paste (and Sent message, with content inspection). Inspection method = DCS; SIT = Credit Card, SSN, PII; custom SIT for internal project codenames. Action = Block with custom user-education message. **Start with `Audit` action, escalate to `Block` per SIT after FP tuning** |
| **WARNING — trap to avoid** | Clipboard inspection is **high-intrusion processing** under GDPR Art. 35 / equivalent — DPIA before pilot. The audit log may retain truncated content snippets — itself regulated content. At 5k seats, clipboard inspection inflates Sentinel ingest by ~100 GB/month (sysint estimate, verify against actual ingest after pilot) |
| **Defeated by** | Image / screenshot / audio of the same data (DCS does not OCR / transcribe at session layer); **typing instead of pasting (no paste event fires)**; desktop ChatGPT apps (Mac / Windows clients exist now); personal account; non-Edge browsers without onboarding |
| **Step-up auth** | The "Require step-up authentication" action is still in preview after years — **do not list as a production action**; treat as preview-only |
| **2026 alternative** | Microsoft Edge for Business `gen-ai.app-control` (H2 2026) provides browser-layer enforcement for Edge-on-Windows estates without the SAML-federation prerequisite |
| **Auditable evidence** | `CloudAppEvents` session-policy match with `ActivityType=Paste` (verify field) and `PolicyId` |
| **SIEM artefact + connector** | Sentinel via Defender XDR |

### Policy 10 — Terminated-user activity in connected SaaS

| | |
|---|---|
| **Risk reduced** | Offboarding gap — terminated user retains an active local account in a connected SaaS app under a different email format (T1078.004 persistence) |
| **What it does** | Built-in anomaly detection. Correlates Entra-deleted user to still-active local account in connected SaaS |
| **ATT&CK technique(s)** | T1078.004 Cloud Accounts (persistence); precondition for detecting T1136 (Create Account, where the user created a SaaS-local account before leaving); T1098.001 (Additional Cloud Credentials added pre-termination); T1556 (Modify Authentication Process — MFA-method additions pre-termination) |
| **Critical prerequisite** | Entra as IdP for the deletion signal; connected SaaS apps with API connectors live |
| **Console path** | Defender portal → Cloud apps → Policies → Policy management → Threat detections → Anomaly detection policy → Activity performed by terminated user (built-in; edit, don't create) |
| **Key configuration** | Scope = all users. Per-app governance action: Notify admin; manual revoke (no auto-suspend for non-Entra accounts — Entra suspend cannot disable a non-Entra local account). Severity = High. Alert to identity admin + SOC |
| **WARNING — trap to avoid** | **Correlation depends on exact-match email between Entra deleted-user and SaaS-app account.** If Entra UPN is `user.surname@corp.com` but Salesforce account is `surname.user@corp.com`, no match → no alert → 0% coverage. Practitioner discipline: at every offboarding, run a manual SaaS-account audit via Advanced Hunting (`CloudAppEvents | where AccountObjectId == "<deleted-user-id>" | distinct Application, AccountId`) for the user across all connected apps |
| **Three additional offboarding sub-cases NOT covered by this built-in:** | (a) **Service-principal secrets / PATs / API keys added before termination** — check Entra `AuditLogs` for `AppRoleAssignment` and `Add service principal credentials` events for the leaving user. (b) **MFA-method additions or backup-email changes (T1556) before termination** — Entra Audit `User registered alternative authentication device` event. (c) **Accounts created by the terminated user in connected SaaS (T1136)** — per-SaaS admin audit, not in MDA. Reference an offboarding runbook that covers all three |
| **Defeated by** | Non-email-matched accounts; accounts where the user has a legitimate guest-account-of-different-email reason |
| **Auditable evidence** | `AlertInfo` + `AlertEvidence` for the anomaly; correlate to Entra `AuditLogs` for the deletion event; per-app audit for the local-account activity |
| **SCIM cascade** | Same SCIM-cascade table as Policy 3 (shared offboarding mechanic — see Appendix A) |
| **SIEM artefact + connector** | Sentinel via Defender XDR + Entra ID Audit logs |

### Policy 12 — Wire MDA signals into Purview Insider Risk Management adaptive scoping (Day 90 addition)

| | |
|---|---|
| **Risk reduced** | False-positive fatigue + missed signal — standalone MDA alerts are noisy at scale. Routing MDA activity-policy hits into Purview Insider Risk Management as boosted indicators turns Policies 3, 7, 10, 11 into high-fidelity insider-risk signals correlated with HR / Purview content / endpoint signals |
| **What it does** | Configure Purview IRM Indicators to consume Defender for Cloud Apps signals as risk-score boosters. Adaptive Protection adjusts DLP policy strictness based on the boosted score |
| **ATT&CK technique(s)** | Cross-cutting — boosts detection fidelity on T1530, T1567, T1078.004, T1485 |
| **Critical prerequisite** | Microsoft Purview Insider Risk Management licensed; MDA activity policies running; Adaptive Protection enabled in Purview |
| **Console path** | `compliance.microsoft.com` → Insider Risk Management → Settings → Indicators → Defender for Cloud Apps signals |
| **Key configuration** | Enable Defender for Cloud Apps as a signal source. Map specific MDA policies to specific IRM indicator categories. Set Adaptive Protection thresholds — most BFSI deployments use the Major / Moderate / Minor scoring without modification at first |
| **WARNING — trap to avoid** | IRM signal aggregation is privacy-sensitive — workforce monitoring scope must be documented; DPIA applies in EU/UK/equivalent regimes |
| **Defeated by** | The compensating-control class for the underlying policy still applies (e.g. Policy 3 slow-and-low exfil still bypasses regardless of IRM boost) |
| **Auditable evidence** | Purview IRM activity logs; cross-correlate to source `CloudAppEvents` |
| **SIEM artefact + connector** | Sentinel via the Microsoft Purview connector + Defender XDR |

---

## Appendix A — SCIM cascade catalogue (top 10 BFSI SaaS)

When MDA's "Suspend user" governance action fires (Policy 3, Policy 10, ad-hoc), what happens downstream depends on the SaaS app's provisioning model:

| SaaS app | SCIM provisioning model | Default latency on disable | Footgun |
|---|---|---|---|
| Salesforce | Entra-provisioned via Enterprise Application | ~40 min | Permission Sets remain sticky on deactivation; if the account is re-activated, the sets reattach. Audit accordingly |
| Workday | Entra-provisioned (newer connector) OR manual | 1–24 hr depending on Workday Worker vs Account distinction | Worker termination ≠ Account disable — separate processes |
| ServiceNow | Entra-provisioned | ~40 min | Inactive vs Locked Out semantics — Locked Out is reversible by the user, Inactive is not |
| AWS | Entra-provisioned to IAM Identity Center OR direct-to-Organization SCIM | ~40 min for IdC; longer for Organization SCIM | IAM Identity Center vs Organization SCIM divergence; check which path your tenant uses |
| Box | Entra-provisioned | ~40 min | Disabled users retain ownership of shared files; reassign before disabling |
| Dropbox | Entra-provisioned | ~40 min | Same — file ownership reassignment required |
| GitHub Enterprise | Entra-provisioned via Enterprise Managed User | ~40 min | Non-EMU GitHub setups use email-based linking; not SCIM-cascaded |
| Okta | Entra-provisioned OR Okta as primary IdP | If Okta as primary: SUSPEND IN OKTA, not Entra | If Entra is downstream of Okta, the "Suspend user" in Entra is reversed at the next Okta SCIM push |
| Google Workspace | Entra-provisioned | ~40 min | Suspension preserves data; deletion does not — different operations |
| Microsoft 365 (sanity-check row) | Native — Entra is the IdP | Immediate (with CAE-aware apps faster) | Hybrid-AD users with `onPremisesSyncEnabled=true` and no writeback: Suspend silently fails. Validate via `IdentityInfo` table |

Apps NOT in this list, or apps with local-user creation outside SCIM, require manual revocation as part of the offboarding runbook.

---

## Appendix B — Slow-and-low detection layer (Sentinel KQL)

Fast-and-loud thresholds (Policies 3, 11) leave a wide-open channel for slow-and-low exfil and ransomware staging. These KQL queries supplement.

### B.1 — Per-user weekly download volume vs 90-day baseline

```kql
let baseline_window = 90d;
let detection_window = 7d;
let lookback_stats = CloudAppEvents
  | where TimeGenerated > ago(baseline_window) and TimeGenerated < ago(detection_window)
  | where ActionType == "FileDownloaded"
  | summarize Baseline = avg(todouble(1)) by AccountObjectId, bin(TimeGenerated, 7d)
  | summarize BaselineWeekly = avg(Baseline), StdDev = stdev(Baseline) by AccountObjectId;
CloudAppEvents
  | where TimeGenerated > ago(detection_window)
  | where ActionType == "FileDownloaded"
  | summarize WeeklyDownloads = count() by AccountObjectId
  | join kind=inner lookback_stats on AccountObjectId
  | where WeeklyDownloads > BaselineWeekly + 3 * StdDev
  | project AccountObjectId, WeeklyDownloads, BaselineWeekly, StdDev
```

### B.2 — OAuth grant additions in the last 30 days, high-scope only

```kql
CloudAppEvents
  | where TimeGenerated > ago(30d)
  | where ActionType in ("Consent to application", "OAuthGrantCreated")
  | where RawEventData has_any ("Mail.ReadWrite", "Files.ReadWrite.All", "Sites.FullControl.All", "User.ReadWrite.All")
  | project TimeGenerated, AccountObjectId, Application, RawEventData
```

### B.3 — External-share creation rate per user, weekly

```kql
CloudAppEvents
  | where TimeGenerated > ago(7d)
  | where ActionType in ("AnonymousLinkCreated", "ExternalShareInvitation", "SharingSet")
  | summarize ExternalShares = count() by AccountObjectId
  | where ExternalShares > 10
  | order by ExternalShares desc
```

### B.4 — MFA-method additions immediately before termination event

```kql
let terminated_users = AuditLogs
  | where TimeGenerated > ago(30d)
  | where OperationName == "Delete user"
  | project DeletedUser = tostring(TargetResources[0].userPrincipalName), DeletionTime = TimeGenerated;
AuditLogs
  | where OperationName has_any ("User registered alternative authentication device", "Add user", "Update authentication methods")
  | join kind=inner terminated_users on $left.InitiatedBy_userPrincipalName == $right.DeletedUser
  | where TimeGenerated < DeletionTime and TimeGenerated > DeletionTime - 7d
  | project DeletedUser, AddedMFAEvent = TimeGenerated, DeletionTime
```

### B.5 — DSPM-for-AI sensitive-data hits by AI app

```kql
CloudAppEvents
  | where TimeGenerated > ago(7d)
  | where ApplicationCategory == "Generative AI"
  | where ActionType has "SensitiveDataDetected"
  | summarize Hits = count(), Users = dcount(AccountObjectId) by Application
  | order by Hits desc
```

### B.6 — The three-way audit-correlation join (use for any CAAC policy-match attestation)

```kql
let session_window = 30s;
CloudAppEvents
  | where TimeGenerated > ago(1d)
  | where PolicyId == "<your-policy-id>"
  | project CloudAppTime = TimeGenerated, AccountObjectId, IPAddress, CorrelationId
  | join kind=inner (
      SigninLogs
      | where TimeGenerated > ago(1d)
      | project SigninTime = TimeGenerated, UserId, IPAddress, ConditionalAccessStatus, CorrelationId
    ) on $left.AccountObjectId == $right.UserId
  | where abs(datetime_diff('second', CloudAppTime, SigninTime)) < session_window
  | where IPAddress == IPAddress1
```

---

## Appendix C — CAAC deprovisioning runbook

Removing a CAAC onboarding is a four-step process across two consoles with no Microsoft-published cache-TTL SLA:

1. **Disable the MDA session policy** (Defender portal → Cloud apps → Policies → toggle Off).
2. **Disable the Entra Conditional Access policy** (Entra portal → Conditional Access → toggle to Report-only, then Off after observation period).
3. **Remove the app from the Conditional Access App Control apps page** (Defender portal → Settings → Cloud apps → Conditional Access App Control → Connected apps).
4. **Communicate user-side session-clear** — affected users must close all browser sessions / clear cookies for the affected SaaS. Expect 2–24 hours of "phantom" `*.mcas.ms` redirects post-removal during which user reports look like the policy is still active.

Document this runbook in change management; do not assume "policy off = policy gone."

---

## Appendix D — DR / fail-mode matrix

| Scenario | Component affected | Behaviour | Operator notified? |
|---|---|---|---|
| MDA service regional outage | All inline session policies | Per-policy `Always apply if data cannot be scanned` setting — `true` = fail closed (sessions blocked), `false` = fail open (sessions pass without inspection) | Microsoft service health, not in-product |
| MDA service regional outage | API-mode governance actions | Queue (post-recovery replay) for short outages; drop for long outages — Microsoft does not publish drop threshold | No documented notification |
| MDA service regional outage | Alert forwarding to Sentinel | Sentinel ingest pauses; logs back-fill on recovery (variable lag) | Sentinel ingest-rate anomaly alert (configure separately) |
| Entra ID outage | All CAAC policies | Sessions fail at the IdP layer before reaching MDA | Entra service health |
| MDE outage on managed endpoint | Policy 8 endpoint enforcement | Network Protection inactive — unsanctioned apps reachable | MDE health alert (configure separately) |
| Purview DCS quota saturation | Policies 6, 9 content inspection | `Always apply if data cannot be scanned` setting determines fail-open vs fail-closed | None — silent |
| Graph API throttling | Policies 6, 7 file enumeration | MDA backs off — partial coverage of the tenant scan | None — silent |
| CAE token revocation | All CAAC session policies | Session re-established may not re-evaluate MDA policy | Documented as expected behaviour |

---

## Appendix E — ATT&CK + ATLAS coverage matrix

For each MITRE technique relevant to this playbook: MDA verdict (Prevent / Detect / Respond / Out of scope) + compensating control where Out of scope.

| Technique | Name | MDA verdict | Compensating control (if Out of scope) |
|---|---|---|---|
| T1078.004 | Cloud Accounts | Detect (Policy 4, Policy 10) — naive credential-stuffing + offboarding gap only | Entra ID Protection user-risk; sign-in-risk policies; Token Protection |
| T1098.001 | Additional Cloud Credentials | Detect (Policy 2b — App Governance) | Entra Workload Identities; Entra ID Governance access reviews |
| T1098.003 | Additional Cloud Roles | Out of scope | Entra Privileged Identity Management; Entra Audit logs alerting |
| T1136.003 | Create Cloud Account | Detect partial (Policy 10) | Per-SaaS admin audit; offboarding runbook |
| T1199 | Trusted Relationship | Detect (Policy 6b) | Cross-Tenant Access Settings; B2B guest scoping |
| T1204 | User Execution | Out of scope | Defender for Office 365 attachment scanning; SafeLinks |
| T1213 | Data from Information Repositories | Detect (Policy 7) | Purview DLP; data classification |
| T1213.002 | SharePoint | Detect (Policies 3, 7, 11) | Purview DLP for SharePoint |
| T1485 | Data Destruction | Detect (Policy 11) | SharePoint version history; backup |
| T1486 | Data Encrypted for Impact (SaaS variant) | Detect (Policy 11) | Backup; immutable storage |
| T1528 | Steal Application Access Token | Detect + Respond (Policy 2a) | **Primary control: Entra admin-consent workflow + verified-publisher restriction** |
| T1530 | Data from Cloud Storage Object | Detect (Policy 3); Prevent partial (Policy 5 — download leg only) | Intune MAM; Purview DLP; Token Protection |
| T1550.001 | Application Access Token | Detect (Policy 2a) only on subsequent token use; **bypasses Policies 4, 5** | Entra Token Protection; Continuous Access Evaluation |
| T1556 | Modify Authentication Process | Detect (Policy 10 sub-case) | Entra Audit log alerting on MFA-method additions |
| T1557 | Adversary-in-the-Middle | Concentration risk — MDA contributes to risk at internet-scale via CAAC proxy | Independent monitoring outside Microsoft stack |
| T1567 / T1567.002 | Exfiltration Over Web Service / Cloud Storage | Detect (Policies 1, 3); Prevent partial (Policies 5, 8, 9 — narrow channels) | SWG; DLP-at-rest; Endpoint DLP |
| T1102 | Web Service | Detect (Policy 1 discovery) | SWG category filtering |
| T1090.002 | External Proxy | Out of scope (anonymising proxy bypass of geo filters) | SWG; network IDS |
| T1027 | Obfuscated Files | Out of scope at content inspection (user-encrypted archive) | Endpoint DLP at pre-encryption stage |
| T1074.002 | Local Data Staging — SaaS sync client | Out of scope of Policy 3 download-event threshold | Endpoint DLP on sync client; OneDrive Known Folder Move restrictions |
| T1199 (re-stated for B2B) | Trusted Relationship | Detect (Policy 6b) | Cross-Tenant Access Settings |
| **ATLAS AML.T0010** | ML Supply Chain Compromise | Out of scope | Approved-AI vendor risk-review process |
| **ATLAS AML.T0024** | Exfiltration via ML Inference API | Out of scope | DSPM for AI; Edge for Business gen-ai.app-control (H2 2026) |
| **ATLAS AML.T0040** | ML Model Inference API Access | Detect partial (Policies 8, 9 metadata) | RBAC at the AI vendor side |
| **ATLAS AML.T0051** | LLM Prompt Injection | Out of scope — note: prompt injection causing OAuth grant escalation is an undetected technique class today | Application-layer prompt validation; isolated agent execution |
| **ATLAS AML.T0052** | Phishing via LLM-generated content | Out of scope | Defender for Office 365 phishing detection |

---

## Appendix F — Threat-actor profile per policy (named TTP clusters)

Use this to map each policy to the named threat clusters it materially counters. None of these policies counters a trained-adversary fully; the table sizes the relevance.

| Policy | Counters (named clusters where public reporting matches) |
|---|---|
| Policy 2a / 2b (OAuth) | Storm-2372 (mass consent phishing 2024); Storm-1283 (OAuth abuse 2023-24); MIDNIGHT BLIZZARD service-principal abuse (2024); generic TA453-class consent-phishing |
| Policy 3 (mass-download) | Insider exfil patterns (no named cluster); compromised-account exfil after T1566 phishing |
| Policy 4 (impossible travel) | Naive credential-stuffing botnets — **NOT** modern token-replay actors (MIDNIGHT BLIZZARD, Storm-0558) |
| Policy 5 (download block) | Insider exfil via BYOD; compromised-account exfil from non-managed endpoint |
| Policy 8 (GenAI unsanction) | Insider negligence + opportunistic data leakage to consumer AI; **NOT** trained-adversary data exfil (multiple bypass channels) |
| Policy 9 (clipboard block) | Same — insider negligence + opportunistic |
| Policy 10 (terminated-user) | Disgruntled-leaver patterns; departed-employee credential reuse |
| Policy 11 (mass-delete) | Storm-0501 ransomware-in-SaaS variants; opportunistic insider sabotage |

---

## Appendix G — Glossary

| Term | Expansion |
|---|---|
| **CAAC** | Conditional Access App Control — MDA's reverse-proxy session-policy mechanism invoked via Entra Conditional Access |
| **CAE** | Continuous Access Evaluation — Entra's near-real-time token revocation feature |
| **CNAPP** | Cloud-Native Application Protection Platform (cloud infrastructure posture; Microsoft Defender for Cloud) |
| **CSF** | Cybersecurity Framework — NIST CSF 2.0 |
| **DCS** | Data Classification Service — Microsoft Purview's content-inspection engine |
| **DfE / DfI / DfO** | Defender for Endpoint / Identity / Office 365 |
| **DPIA** | Data Protection Impact Assessment (GDPR Art. 35 / equivalent) |
| **DSPM** | Data Security Posture Management — Purview overlay for AI prompt visibility |
| **EDM** | Exact Data Match — Purview structured-data classification method |
| **MAM** | Mobile Application Management — Intune App Protection Policies |
| **MDE** | Microsoft Defender for Endpoint |
| **MIP** | Microsoft Information Protection — Purview sensitivity labels |
| **OCAS** | Office 365 Cloud App Security — subset SKU bundled with O365 E5; not full MDA |
| **PDPA** | Personal Data Protection Act (Malaysia 2010 + 2024 amendments; SG has its own) |
| **RBI** | Remote Browser Isolation |
| **RMiT** | Risk Management in Technology (Bank Negara Malaysia regulatory document) |
| **SCIM** | System for Cross-domain Identity Management — provisioning protocol |
| **SIT** | Sensitive Information Type — Purview DLP classifier |
| **SSPM** | SaaS Security Posture Management |
| **TRM** | Technology Risk Management (MAS regulatory document) |

---

## Three-lens sign-off

| Lens | Reviewer / run | Verdict on v0 | Verdict on v1 | Outstanding |
|---|---|---|---|---|
| **Architect (original)** | workflow `wcr0nuuwc` (2026-06-10) | NO | ADDRESSED — re-review pending | Tenant primary-region requires accessed-date Trust Center pull on the day of promotion |
| **Product (original)** | workflow `wcr0nuuwc` (2026-06-10) | NO | ADDRESSED — re-review pending | Quantitative DCS-latency benchmarks (vendor doesn't publish; document as residual) |
| **Compliance (original)** | workflow `wcr0nuuwc` (2026-06-10) | NO | PARTIAL — outstanding | Full BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS clause mapping deferred to `06-compliance/microsoft-defender-for-cloud-apps-control-mapping.md`; third-party attestation citations from Service Trust Portal; sub-processor list URL; full DPIA scaffolding |
| **QA (v1 specialist)** | workflow `wxs8cfuni` (2026-06-10) | NO-WITH-CRITICAL-FIXES | ADDRESSED — re-review pending | Every numeric / dated / Microsoft-named claim still needs the `refs.md` companion file with URL + accessed date + quoted phrase per claim-id |
| **Cyber / MITRE (v1 specialist)** | workflow `wxs8cfuni` (2026-06-10) | NO-WITH-CRITICAL-FIXES | ADDRESSED — re-review pending | Purple-team / atomic-test cadence inline TBD; threat-actor mapping refined as named-cluster public reporting evolves |
| **MDA-deep-expert (v1 specialist)** | workflow `wxs8cfuni` (2026-06-10) | NO-WITH-CRITICAL-FIXES | ADDRESSED — re-review pending | Schema-versioning subscriber + portal-path re-verification on Defender XDR redesigns |
| **System integration (v1 specialist)** | workflow `wxs8cfuni` (2026-06-10) | NO-WITH-CRITICAL-FIXES | ADDRESSED — re-review pending | Tenant integration prerequisites + Graph throttling budget could grow into separate appendices; Okta addendum currently inline |
| **Documentation (v1 specialist)** | workflow `wxs8cfuni` (2026-06-10) | NO-WITH-CRITICAL-FIXES | ADDRESSED — re-review pending | A 2026-Defender-portal-path verification sweep is required before promotion |

**Status:** stays in `_research/` pending (a) the compliance mapping deliverable, (b) the `refs.md` companion file with full citation discipline, (c) a 2026-portal-path verification sweep. Promotion candidate to `04-vendors/microsoft-defender-for-cloud-apps.md` after those three.
