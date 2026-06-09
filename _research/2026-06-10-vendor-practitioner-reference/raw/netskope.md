# Netskope SSE (CASB Inline + CASB API)

## Lineage
- Current product names: **Netskope One SSE** platform; CASB capability surfaces as **Netskope One CASB Inline** (forward-proxy / reverse-proxy real-time controls) and **Netskope One CASB API** (out-of-band API connectors, marketed as "Next Generation API Data Protection" / "Classic API Data Protection"). Adjacent modules in the same console: Next-Gen SWG, ZTNA Next (NPA), SSPM, Cloud Firewall, RBI, AI Gateway, AI Guardrails.
- No major rename. Netskope (founded 2012) has run a single tenant/console since launch. Notable additions over the last 24 months: **AI Gateway** and **AI Guardrails** (genAI inline controls), **Next Generation API Data Protection** platform (replacing portions of "Classic" API), and expanded SSPM 3rd-party-app discovery. Acquisitions that touched the CASB stack: Kindite (2020, encryption/tokenisation), Infiot (2022, SD-WAN, peripheral to CASB), Dasera (2024, DSPM — adjacent but listed here because it now feeds risk context into CASB API alerts).
- CASB housing: **inside the SSE platform**, single tenant/console (Netskope One). Same policy engine, same DLP engine, same identity stack across Inline and API. There is no stand-alone CASB SKU separate from the SSE tenant.
- Licensing tiers (publicly described): CASB Inline ships in the **Netskope One SSE** / **Netskope One Platform** bundles; CASB API is a **separately licensed** add-on (per-app or platform), as is **AI Guardrails** and **AI Gateway**. Exact SKU names vary by reseller/region — call out as **Unverified** in the lens review; vendor pricing page does not enumerate SKUs publicly.

## Deployment modes supported
- **Forward-proxy (inline).** Steering options documented under `docs.netskope.com/en/netskope-help/traffic-steering/`:
  - **Netskope Client** (endpoint agent, Win/Mac/Linux/iOS/Android) — default for managed endpoints.
  - **IPSec** and **GRE** tunnels from edge / SD-WAN / branch routers to Netskope NewEdge PoPs.
  - **Explicit proxy** and **PAC files** (for browser-based steering and reverse-tunnelled appliances).
  - **Proxy chaining** (upstream from a legacy SWG to Netskope).
  - **IdP-based steering** via SAML Forward Proxy — auth flow redirects user to Netskope as the inline interception point even without an agent.
  - **SSL inspection** is required for content-level CASB controls (DLP, threat scan, activity controls deeper than the URL). Certificate-pinned mobile apps fail under MITM and must be exception-bypassed.
- **Reverse-proxy.** Two flavours documented:
  - **SAML Reverse Proxy** (a.k.a. "SAML Proxy") — IdP-integrated; the IdP (Okta, Entra ID, Ping, CyberArk Idaptive, etc.) issues the SAML assertion via Netskope, which re-signs the assertion and inserts itself in the data path post-auth. Targets specific sanctioned SAML SaaS apps (Salesforce, Workday, ServiceNow, Box, M365 OWA, Google Workspace, etc. — exact list per `docs.netskope.com/en/saml-reverse-proxy/` is feature-gated, **mark as Unverified for the precise app list**).
  - **Reverse Proxy as a Service (RaaS) with Microsoft Entra ID / Universal Reverse Proxy** — agentless steering for **BYOD / unmanaged** devices into Netskope without requiring the Netskope Client; conditional-access policy in Entra forces session through Netskope.
- **API connectors (CASB API / Next Generation API Data Protection).** Sanctioned apps with API support (per `docs.netskope.com/en/apps-supported-in-classic-and-next-generation-api-data-protection/`, accessed 2026-06-10):
  - **Both Classic + NextGen:** Box, Cisco Webex Teams, Dropbox, Egnyte, GitHub, Google Drive, Microsoft OneDrive (Commercial), Microsoft 365 Outlook (Commercial), Microsoft SharePoint (Commercial), Microsoft Teams (Commercial), Salesforce, ServiceNow, Slack Enterprise.
  - **NextGen only:** Atlassian Confluence, Atlassian Jira, ChatGPT Enterprise, Gmail, Google Calendar, Microsoft Copilot, M365 OneDrive GCC/GCC High, M365 Outlook GCC, M365 SharePoint GCC/GCC High, M365 Teams GCC High, M365 Yammer, Okta, ShareFile, Workday, Zendesk, Zoom.
  - **Classic only:** Slack Team, Workplace from Meta (deprecated direction — NextGen is where new connectors land).
- **Log-based discovery.** Netskope ingests logs from common firewalls/SWGs (Palo Alto, Cisco ASA/Firepower, Fortinet, Check Point, Zscaler, Forcepoint, generic syslog/CEF) and feeds shadow-IT discovery into the **Cloud Confidence Index (CCI)** — a database scoring ~80,000+ apps (~100 attributes per app, 0–100 score, CCL levels Poor/Low/Medium/High/Excellent).
- **Hybrid considerations.** Inline + API is the documented "belt and braces" deployment: API mode catches data already at rest in the sanctioned tenant; Inline catches data in motion to sanctioned tenants AND to unsanctioned tenants/apps. The shared DLP engine means a single classifier is authored once and referenced from both policy stacks — though the **policy objects themselves are configured separately** (Real-time Protection Policy vs. API Data Protection Policy), which is a common source of drift.

## Capability set

- **Shadow-IT discovery (network-log-based AND API-based)** — **Supported.** Two ingestion paths: (a) **Inline** via the Netskope Client / forward-proxy traffic (real-time), and (b) **Log-based** via uploaded firewall/SWG logs (batch). Both pivot on the **Cloud Confidence Index (CCI)** for risk scoring. CCI scores ~80k+ apps using ~100 attributes adapted from CSA CCM. `https://docs.netskope.com/en/cloud-confidence-index/` (vendor-controlled, accessed 2026-06-10).
- **Sanctioned-app inventory + risk-scoring** — **Supported.** CCI risk scores drive policy ("only allow Excellent/High CCL, coach on Medium, block Poor/Low"). Per-app instance distinction is a first-class concept — Netskope can tell `corporate-tenant` Box from `personal` Box. `https://docs.netskope.com/en/understand-the-risk-of-cloud-services-utilization-by-leveraging-cci/` (vendor-controlled, accessed 2026-06-10).
- **Inline DLP for sanctioned SaaS (proxy mode — pre-upload prevention)** — **Supported.** DLP profiles (rules referencing data identifiers) attach to Real-time Protection / Cloud Apps policies; action set includes block, alert, user-justification ("coach"), and quarantine. Same DLP engine across web/cloud/email/endpoint. `https://docs.netskope.com/en/data-loss-prevention/` (vendor-controlled, accessed 2026-06-10).
- **API-mode DLP for sanctioned SaaS (post-upload scan + remediate)** — **Supported with caveats.** Configured in API Data Protection Policies; remediation actions include **quarantine, restrict share, change owner, delete (where the connector exposes it), and notify**. Crucially, this is **after-the-fact** — the vendor's own page says "visibility and control are after-the-fact versus proactive and real-time." Action support varies per connector (e.g., Salesforce delete is soft-delete only). `https://docs.netskope.com/en/casb-api-protection/` (vendor-controlled, accessed 2026-06-10).
- **OAuth-app discovery and governance (third-party app grants to M365 / Workspace)** — **Supported with caveats.** Surfaced under SSPM → 3rd Party Apps. Risk scoring 0–100 across five tiers (Critical/High/Medium/Low/Unknown). Visible: app name, connected SaaS, instance, scope/permission strings, granting user, grant timestamp. Coverage explicitly described for **Azure AD / Entra ID, Salesforce, Google Workspace**. Actions documented are visibility + verification status updates; full **revoke from the Netskope console** is connector-dependent (Entra/Workspace yes; verify per-app). `https://docs.netskope.com/en/view-3rd-party-apps/` (vendor-controlled, accessed 2026-06-10).
- **Anomalous-activity / UEBA detection** — **Supported.** Behavior Analytics produces a **User Confidence Index (UCI)** score and named detection scenarios; covers insider threat, compromised account, compromised device, data exfiltration, lateral movement. Tiered as Standard vs. Advanced (separate SKU). `https://docs.netskope.com/en/behavior-analytics/` (vendor-controlled, accessed 2026-06-10).
- **Compromised-account detection (impossible travel, brute-force, anomalous-OAuth)** — **Supported.** Listed as a UEBA detection family. Specific signal names (impossible travel velocity threshold, brute-force window) not explicitly enumerated on the public page — **mark for lens-reviewer verification**.
- **Data-residency / geo-policy enforcement** — **Supported with caveats.** Inline policies can condition on user-location and destination-region. Tenant-data residency is set at provisioning (NewEdge PoP region). DLP profile data residency for incident metadata (the bits Netskope stores about a hit) is configurable; verify with vendor for in-region scanning — **flag for reviewer.**
- **Share-link governance (external sharing, anonymous links, expiry)** — **Supported.** API Data Protection policies act on share-permission state (Public, Anyone-with-link, External-domain, Internal-only) for the storage connectors (OneDrive, SharePoint, Drive, Box, Dropbox). Actions: restrict-sharing, change-owner, alert.
- **Malware scanning in SaaS storage** — **Supported.** Threat Protection Policy (TSS profile) overlays on both Real-time and API policies. ML and signature engines; sandboxing via Netskope Cloud Sandbox. `https://docs.netskope.com/en/threat-protection/` (vendor-controlled, accessed 2026-06-10).
- **Encryption / tokenisation at gateway** — **Supported with caveats.** Inherits the **Kindite** (acquired 2020) cryptographic stack — searchable encryption / tokenisation for specific fields, primarily Salesforce-flavoured deployments. Heavy implementation lift; not the common path; ask why before proposing it.
- **Risk-based session policy (step-up auth, read-only, watermark, block)** — **Supported with caveats.** Inline policy actions include block, alert, user-justify, isolate (via Netskope RBI), restrict-activity (granular: prevent download / prevent upload / prevent post / prevent share / read-only). **Watermarking** is achieved via RBI overlay, not directly in the proxy. **Step-up auth** is via IdP integration (Entra/Okta CAE/conditional-access) rather than native to Netskope — flag.
- **GenAI-app discovery + governance (ChatGPT, Claude, Copilot, etc.)** — **Supported.** Two-stack approach: (1) **Gen AI App Governance** under CASB Inline catalogs **1,800+ GenAI apps** with activity-/instance-based policies; (2) **AI Guardrails** does inline content moderation, prompt-injection / jailbreak detection, classified against MITRE ATLAS + OWASP LLM Top 10; (3) **AI Gateway** sits app-to-LLM (OpenAI, Anthropic Claude, Google Gemini) for auth, rate-limit, content filter. Plus API-side: **ChatGPT Enterprise** and **Microsoft Copilot** are API connectors (NextGen). `https://docs.netskope.com/en/ai-security/` (vendor-controlled, accessed 2026-06-10).

## Configurable policies

The Netskope policy taxonomy (per `docs.netskope.com/en/netskope-help/admin-console/policies/`, accessed 2026-06-10) splits into **Real-time Protection** (inline; conditions on Cloud Apps / Web / Firewall / DNS / NPA / Mail Relay), **API Data Protection** (out-of-band), and **add-on overlays** (DLP, Threat Protection, Browser Isolation). The following are policies a practitioner actually configures.

### 1. Block upload of regulated data to non-corporate cloud storage
- **What it does**: Prevents end users from uploading files containing regulated content (PCI, PII, source code) to any cloud storage app instance that is not the corporate-sanctioned tenant.
- **Where it lives**: Policies → Real-time Protection → New Policy (type: Cloud App) + attach DLP Profile.
- **Key knobs**:
  - User filter: AD group `Employees-All`
  - App filter: Category `Cloud Storage` + Activity `Upload`
  - Instance filter: NOT-equals `corp-onedrive-tenant`, NOT-equals `corp-box-tenant`
  - Condition: DLP Profile `Regulated-Data-Strict` (rules: PCI-DSS predefined detector + PII-MY-NRIC custom regex + ML-trained "source code" classifier)
  - Action: **Block** + user-notification template `regulated-upload-blocked`
- **Deployment-mode requirement**: Proxy (inline). Requires SSL inspection enabled for the destination domain.
- **False-positive risk**: **Medium.** Named scenario: developers uploading legitimate test fixtures containing fake-but-regex-matching credit card numbers (e.g. `4111-1111-1111-1111`) — mitigate with sample-luhn-validation and per-group exception.
- **Source**: `https://docs.netskope.com/en/data-loss-prevention/` (vendor-controlled, accessed 2026-06-10).

### 2. Coach users away from low-CCI shadow apps
- **What it does**: When a user navigates to a cloud app whose CCI score is below threshold, present a justification-prompt page; if they proceed, log the event with the justification.
- **Where it lives**: Policies → Real-time Protection → New Policy (type: Cloud App), action = User Alert (Coach).
- **Key knobs**:
  - User filter: All managed users
  - App filter: `Cloud Confidence Level` IN (`Poor`, `Low`)
  - Activity: Any
  - Action: **User Alert** with template `low-cci-coach`; require justification text; auto-log to SIEM.
- **Deployment-mode requirement**: Proxy.
- **False-positive risk**: **Low.** Coaches rather than blocks. Named scenario: legitimate niche vendor portals scoring Low for lack of public audit info — provision an explicit allowlist instance.
- **Source**: `https://docs.netskope.com/en/cloud-confidence-index/` (vendor-controlled, accessed 2026-06-10).

### 3. Restrict OAuth-app grants on Microsoft 365
- **What it does**: Detects newly granted third-party OAuth apps against the Entra tenant; risk-score them; alert (and optionally revoke) on Critical/High risk grants.
- **Where it lives**: SSPM → 3rd Party Apps → Policies.
- **Key knobs**:
  - SaaS scope: `Entra ID tenant: corp.onmicrosoft.com`
  - Condition: Risk Tier IN (`Critical`, `High`) OR scope contains `Mail.ReadWrite` OR `Files.ReadWrite.All` granted by user NOT in group `IT-Admins`
  - Action: **Alert** to SOC channel; **Revoke** (Entra-connector dependent)
- **Deployment-mode requirement**: API (SSPM connector to Entra; same for Google Workspace / Salesforce equivalents).
- **False-positive risk**: **Medium.** Named scenario: legitimate productivity apps (e.g. Grammarly, Calendly) requesting broad mailbox scopes trigger High risk; whitelist verified-publisher IDs.
- **Source**: `https://docs.netskope.com/en/view-3rd-party-apps/` (vendor-controlled, accessed 2026-06-10).

### 4. Quarantine externally-shared sensitive files in OneDrive / SharePoint / Drive
- **What it does**: Scans existing files in the sanctioned storage tenant; if a file matches a DLP profile AND its share-state is External-or-broader, quarantine it (move to admin quarantine folder; replace original with tombstone).
- **Where it lives**: Policies → API Data Protection → New Policy.
- **Key knobs**:
  - App: `OneDrive – corp tenant` (also a parallel policy per app)
  - Object filter: File; Owner ANY
  - Condition: Share-state IN (`Public`, `Anyone-with-link`, `External-domain`) AND DLP Profile `Regulated-Data-Strict` matches
  - Action: **Quarantine** to `\_quarantine\<incident-id>\` + email file owner + alert SOC
- **Deployment-mode requirement**: API. Inline cannot do post-hoc quarantine.
- **False-positive risk**: **High.** Named scenario: legal teams legitimately share regulated case files externally with counsel; require a per-OU exception and pre-approval workflow before broad rollout. The API scan can also hit historical files with old (stale) ACLs, generating mass false alerts on day one — pilot with a single OU first.
- **Source**: `https://docs.netskope.com/en/casb-api-protection/` (vendor-controlled, accessed 2026-06-10).

### 5. Block download of corporate data from sanctioned SaaS to unmanaged devices (BYOD)
- **What it does**: When a user authenticates to a sanctioned app (e.g. M365) from an unmanaged device, enforce read-only / no-download via reverse-proxy session.
- **Where it lives**: Policies → Real-time Protection → New Policy + Reverse Proxy steering (configured under Settings → Security Cloud Platform → SAML Reverse Proxy / RaaS).
- **Key knobs**:
  - User filter: AD group `Employees-All`
  - Device filter: Device Classification = `Unmanaged` (no Netskope Client)
  - App filter: `M365 SharePoint – corp tenant`, `M365 OneDrive – corp tenant`
  - Action: Restrict Activity = **Block Download**, **Block Print**, **Block Upload** (allow View)
- **Deployment-mode requirement**: Reverse Proxy (SAML RP or RaaS with Entra ID).
- **False-positive risk**: **Low** (well-scoped to unmanaged). Named scenario: contractors on managed-but-non-AAD-joined devices misclassified as Unmanaged — fix with device certificate + Entra compliant-device claim.
- **Source**: `https://docs.netskope.com/en/reverse-proxy-as-a-service-with-microsoft-entra-id-1/` (vendor-controlled, accessed 2026-06-10).

### 6. Detect and block prompt-injection / data-exfil to GenAI apps
- **What it does**: Inline scan of prompts sent to consumer GenAI apps (ChatGPT, Claude, Gemini, Perplexity, Copilot consumer); block prompts containing source code, secrets, or PII; detect prompt-injection patterns against the app's response too.
- **Where it lives**: Policies → Real-time Protection → New Policy (type: Cloud App) with AI Guardrails profile attached.
- **Key knobs**:
  - User filter: AD group `Employees-All` EXCLUDE `AI-Pilot-Group`
  - App filter: Category `Generative AI` AND Instance NOT-equals `enterprise-chatgpt-tenant`
  - Activity: `Upload` / `Post`
  - Condition: AI Guardrails profile (prompt-injection detector + jailbreak detector + ATLAS-mapped classifiers) OR DLP Profile `Source-Code + Secrets`
  - Action: **Block** with coach message; log full prompt-hash to SIEM (NOT full prompt — for privacy)
- **Deployment-mode requirement**: Proxy + AI Guardrails licence.
- **False-positive risk**: **Medium-High.** Named scenario: a developer pastes a stack trace containing what looks like a JWT but is a synthetic example — AI Guardrails classifier flags as secret-leak. Tune with code-block whitelisting and pilot group.
- **Source**: `https://docs.netskope.com/en/ai-security/` (vendor-controlled, accessed 2026-06-10).

### 7. Alert on impossible-travel / anomalous login to M365
- **What it does**: UEBA detection of geographically-impossible login sequences (e.g. KL at 09:00, Lagos at 10:00) for sanctioned app sessions.
- **Where it lives**: Behavior Analytics → Policies → New Detection (or default scenarios) + Real-time Protection downstream policy.
- **Key knobs**:
  - Scenario: `Impossible Travel Login`
  - Velocity threshold: > 800 km/h (default; tunable)
  - Scope: Sanctioned apps M365, Google Workspace, Salesforce, Slack
  - Action chain: UCI score decrement → if UCI < threshold, trigger Real-time Protection policy that requires step-up auth (via IdP) on next session
- **Deployment-mode requirement**: API (for login telemetry from sanctioned tenants) + IdP integration for step-up.
- **False-positive risk**: **Medium.** Named scenario: VPN / mobile-hotspot users with carrier-NAT exits in another country generate impossible-travel from a real-world local login. Calibrate with corp-VPN egress allowlist and ASN-based exclusions.
- **Source**: `https://docs.netskope.com/en/behavior-analytics/` (vendor-controlled, accessed 2026-06-10).

### 8. Block uploads of files without a corporate sensitivity label
- **What it does**: For sanctioned destinations, block upload of files that do NOT carry a Microsoft Purview / AIP sensitivity label, or carry an inappropriate label given the destination.
- **Where it lives**: Policies → Real-time Protection → New Policy + DLP Profile referencing MIP/AIP label.
- **Key knobs**:
  - App filter: `M365 SharePoint – external-collab-site`
  - Activity: Upload
  - Condition: File metadata `MIP Label NOT IN (Public, Confidential-External-OK)`
  - Action: **Block** + coach to label first
- **Deployment-mode requirement**: Proxy (label is read from file metadata in transit). Note: label-protection relies on Microsoft's labelling client / Purview being deployed — Netskope does not author labels itself.
- **False-positive risk**: **Medium.** Named scenario: legacy unlabelled documents created before MIP rollout; provide a one-time amnesty window or grandfather rule.
- **Source**: `https://docs.netskope.com/en/data-loss-prevention/` (vendor-controlled, accessed 2026-06-10).

### 9. Malware scan & quarantine in sanctioned storage
- **What it does**: Continuous API scan of files at rest in sanctioned storage; quarantine on malware detection (ML + signature + optional sandbox).
- **Where it lives**: Policies → API Data Protection → New Policy + Threat Protection profile (TSS).
- **Key knobs**:
  - App: `Box – corp tenant` (replicate per app)
  - Object: All files; on creation AND on share-change
  - Condition: Threat Protection profile `Default-Malware` + Sandbox `Cloud-Sandbox-Hi-Risk-Filetypes`
  - Action: **Quarantine** + revoke shares + alert owner
- **Deployment-mode requirement**: API.
- **False-positive risk**: **Low** for malware (sigs/ML are tuned); **Medium** for sandbox verdicts on macros — exempt finance team templates by hash-list.
- **Source**: `https://docs.netskope.com/en/threat-protection/` (vendor-controlled, accessed 2026-06-10).

### 10. Restrict access to non-corporate instances of sanctioned apps (tenant pinning)
- **What it does**: Allow `corp-tenant` of M365 / Google Workspace / Box; block-or-restrict personal / partner tenants accessed from corp devices.
- **Where it lives**: Policies → Real-time Protection → New Policy (type: Cloud App) with App Instance condition.
- **Key knobs**:
  - App filter: `Microsoft 365`, `Google Workspace`, `Box`
  - Instance: NOT-equals `corp-tenant-id` AND NOT-in `approved-partner-tenants`
  - Activity: Login, Upload, Post
  - Action: **Block Upload + Post**; **Allow Login** (read-only) for personal Gmail / OneDrive (cultural concession, often pushed back by HR)
- **Deployment-mode requirement**: Proxy. Tenant ID extraction requires SSL inspection.
- **False-positive risk**: **Low** (instance IDs are deterministic). Named scenario: M&A integration period where a second corporate tenant is in flight but not yet listed — keep an explicit pending-merger instance allowlist.
- **Source**: `https://docs.netskope.com/en/cloud-confidence-index/` (vendor-controlled, accessed 2026-06-10).

### 11. Block file uploads exceeding size threshold to unsanctioned apps
- **What it does**: Bulk-exfil heuristic — block any single upload > N MB or aggregate > N MB/hour to apps outside the sanctioned list.
- **Where it lives**: Policies → Real-time Protection → New Policy + Constraint Profile.
- **Key knobs**:
  - App filter: Category `Cloud Storage` EXCLUDE sanctioned instances
  - Constraint: File size > 100 MB OR `Volume per User per Hour` > 500 MB
  - Action: **Block** + alert SOC
- **Deployment-mode requirement**: Proxy.
- **False-positive risk**: **Medium.** Named scenario: legitimate large-file transfers (CAD, video, datasets) to project partners — provide a request-an-exception workflow.
- **Source**: `https://docs.netskope.com/en/data-loss-prevention/` (vendor-controlled, accessed 2026-06-10).

### 12. Require browser isolation for risky sanctioned-app sessions
- **What it does**: When a user with degraded UCI (UEBA risk) accesses a sanctioned app, render the session inside Netskope's Remote Browser Isolation rather than letting the local browser hold tokens / cache.
- **Where it lives**: Policies → Real-time Protection → New Policy with action = Isolate.
- **Key knobs**:
  - User filter: UCI score < 50 (degraded)
  - App filter: Sanctioned apps (M365, Salesforce, etc.)
  - Action: **Isolate** in RBI with read-only + watermark + clipboard-block
- **Deployment-mode requirement**: Proxy + RBI licence.
- **False-positive risk**: **Medium.** Named scenario: a UCI drop from a single noisy detector cascades affected users into RBI for legitimate work; tune UCI thresholds and require multi-signal confirmation before isolating.
- **Source**: `https://docs.netskope.com/en/behavior-analytics/` (vendor-controlled, accessed 2026-06-10).

### Worked example — policy object (conceptual)

Policy: **"Block upload of regulated data to non-corporate cloud storage"** (policy #1 above). Conceptual shape — NOT literal product YAML; Netskope's console is GUI-first and policy export is JSON via the v2 REST API.

```json
{
  "policyName": "block-regulated-upload-noncorp-storage",
  "policyGroup": "Real-time Protection",
  "policyType": "CloudApp",
  "enabled": true,
  "order": 120,
  "source": {
    "users": { "include": ["group:Employees-All"], "exclude": ["group:Legal-EDiscovery"] },
    "devices": { "include": ["any"] },
    "locations": { "include": ["any"] }
  },
  "destination": {
    "categories": ["Cloud Storage"],
    "appInstances": {
      "exclude": ["onedrive:corp-tenant-id", "box:corp-tenant-id", "dropbox:corp-tenant-id"]
    },
    "activities": ["Upload"]
  },
  "condition": {
    "dlpProfile": "Regulated-Data-Strict",
    "fileType": ["any"]
  },
  "action": {
    "type": "Block",
    "userNotification": {
      "template": "regulated-upload-blocked",
      "requireJustification": false
    },
    "logging": { "severity": "High", "siemForward": true }
  }
}
```

## Limitations / what this product does NOT cover

- **Real-time blocking of risky actions inside non-API-connected SaaS** — because API connectors are post-upload only; if the app has no API connector (i.e. not in the supported list), all post-hoc remediation is impossible. Inline proxy compensates only when the user traffic actually traverses Netskope, which BYOD / mobile-native / certificate-pinned flows often bypass.
- **OAuth grants made on personal / unmanaged tenants** — SSPM 3rd-party-app discovery sees the **Entra / Workspace / Salesforce tenants you connect**; direct-to-app OAuth via consumer Google / Microsoft consumer accounts is invisible because the grant never touched the corporate IdP that Netskope is subscribed to.
- **OneNote, SharePoint Lists, Microsoft Loop sites** — explicitly excluded from M365 API connector coverage per the vendor's "3rd Party App Limitations" page. Reason: incomplete or absent Graph API surface for these object types at scan time.
- **Owner-level access changes on M365** — Netskope cannot modify Owner-level ACLs via Graph API even when it can delete content. Reason: API permission scope ceiling — Microsoft does not grant external apps Owner-revoke without delegated admin consent.
- **DLP file-owner attribution gaps on M365 API DLP** — vendor docs note "Unknown user and empty file owner in DLP alerts" caused by incomplete Graph API responses. Net effect: incident response cannot always trace "who put this here," so workflows that assume a notifiable owner break.
- **Salesforce "delete" is soft-delete only** — API connector moves objects to recycle bin; cannot be used as a legal-hold or hard-quarantine destination. Reason: Salesforce API does not expose hard-delete to third-party connectors without write-grant escalation; data is recoverable from recycle bin within retention window.
- **Mobile native apps with certificate pinning** — SSL inspection breaks the pin; the app refuses the connection or steers around the Netskope tunnel via app-specific bypass. Affected: many banking apps, some Apple services, parts of Microsoft Authenticator. Mitigation: exception-bypass at the steering layer, accept the visibility loss.
- **Encrypted-archive (password-protected zip/7z/rar) DLP inspection** — Netskope cannot read the contents. Reason: no key. Mitigation: policy to block password-protected archives to unsanctioned destinations.
- **Inline policies do not catch data already at rest** — the canonical inline gap. Day-one rollout against a 5-year-old SharePoint tenant requires API scan; inline alone returns false confidence.
- **API connectors are not real-time** — vendor itself describes the timing as "after-the-fact." Time-to-detect ranges from minutes to hours depending on webhook reliability and tenant scan queue depth. Reason: the underlying SaaS provider webhook + polling architecture.
- **Step-up authentication is not native** — Netskope can trigger it via IdP (Entra Conditional Access / Okta Adaptive MFA) but does not itself enforce MFA prompts. Reason: Netskope is not the IdP; it signals the IdP. Dependency: your IdP must support conditional access driven by external risk signals.
- **Slack Team (non-Enterprise) is on Classic API only** — NextGen does not support it. Slack Team also lacks the audit-and-DLP API surface that Slack Enterprise Grid provides, so coverage is shallower regardless of platform. Reason: Slack tier feature gating.
- **Watermarking on direct (non-RBI) sessions** — there is no native watermark applied to a normal proxy session; you need to route the session through RBI to get watermark, which adds latency and licence cost. Reason: watermarking requires rendering control, which the forward proxy alone doesn't have.
- **Smartsheet email-share events** — vendor docs note these bypass the permission-control event stream. Reason: Smartsheet's `/shares` API doesn't synchronously surface email-triggered shares.
- **Zoom external-user chat deletion + third-party meeting chat attachment scan** — explicitly excluded per the limitations page. Reason: Zoom API gaps for these object types.

## Sources

All accessed 2026-06-10.

- **Vendor-controlled:**
  - `https://www.netskope.com/products/casb` — CASB product overview (page load gated, fetched partially).
  - `https://docs.netskope.com/` — knowledge portal index.
  - `https://docs.netskope.com/en/casb-api-protection/` — API Protection overview.
  - `https://docs.netskope.com/en/apps-supported-in-classic-and-next-generation-api-data-protection/` — connector support matrix.
  - `https://docs.netskope.com/en/casb-api-feature-matrix/` — feature matrix (page returned no content; not used).
  - `https://docs.netskope.com/en/netskope-help/traffic-steering/` — steering options.
  - `https://docs.netskope.com/en/data-loss-prevention/` — DLP overview.
  - `https://docs.netskope.com/en/netskope-help/admin-console/policies/` — policy taxonomy (Real-time / API / overlays).
  - `https://docs.netskope.com/en/threat-protection/` — threat protection / TSS profile.
  - `https://docs.netskope.com/en/behavior-analytics/` — UEBA / UCI / detection scenarios.
  - `https://docs.netskope.com/en/ai-security/` — AI Gateway, AI Guardrails, Gen AI App Governance.
  - `https://docs.netskope.com/en/cloud-confidence-index/` — CCI.
  - `https://docs.netskope.com/en/understand-the-risk-of-cloud-services-utilization-by-leveraging-cci/` — CCI-driven policy.
  - `https://docs.netskope.com/en/view-3rd-party-apps/` — SSPM 3rd-party OAuth app discovery.
  - `https://docs.netskope.com/3rd-party-app-limitations/` — connector limitations.
  - `https://docs.netskope.com/en/saml-reverse-proxy/` — SAML reverse proxy.
  - `https://docs.netskope.com/en/reverse-proxy-as-a-service-with-microsoft-entra-id-1/` — RaaS with Entra (BYOD).
  - `https://docs.netskope.com/en/universal-reverse-proxy-2/` — Universal Reverse Proxy.
  - `https://www.netskope.com/blog/who-do-you-trust-challenges-with-oauth-application-identity` — vendor blog on OAuth identity (vendor-controlled).
- **Independent:**
  - `https://dope.security/post/top-10-netskope-alternatives-2026-honest-comparison` — third-party comparison (competitor blog; corroborative for CASB positioning, NOT for vendor-claimed features).
  - `https://expertinsights.com/cloud-security/top-cloud-access-security-brokers-casbs` — analyst-style comparison; corroborative only.
  - Gartner Magic Quadrant for SSE / SASE (referenced widely, not fetched directly here — independent corroboration of platform positioning would come from there).

## Notes for the lens reviewers

Items I'd want corroborated before promoting to canonical `04-vendors/`:

- **SKU / licensing tier names** — Netskope does not publish a clean SKU table; my "Netskope One SSE / Netskope One Platform + CASB API add-on + AI Guardrails add-on" mapping is from reseller-facing language, not a primary doc. Flag for sales-engineering confirmation. Marked **Unverified** above.
- **Reverse-proxy app support list** — the SAML Reverse Proxy page is feature-gated and my list of supported sanctioned apps (Salesforce, Workday, ServiceNow, Box, OWA, Google) is inferred from common reverse-proxy targets, not enumerated by the vendor on the public page. Lens reviewer should pull the gated list.
- **UEBA signal-level detail** — "impossible travel velocity threshold," "brute-force window," "OAuth anomaly" are described thematically by Netskope, not specified with thresholds publicly. Verify in the deployment guide.
- **AI Gateway release timing** — vendor page lists OpenAI / Anthropic / Gemini support but no GA date; competitive marketing on this changes quarterly.
- **"1,800+ GenAI apps catalogued"** — vendor claim from `ai-security/` page. Independent verification would be valuable; competitors make similar claims at different orders of magnitude.
- **Watermarking dependency** — I claimed watermarking requires RBI, not native proxy. Worth confirming a practitioner could not enable a session-overlay watermark in a non-RBI proxied session.
- **Data residency for incident metadata** — vendor doesn't put a hard statement on whether DLP-hit metadata can be pinned to in-region storage (e.g. for MY/SG regulatory needs); flag for vendor BNM-RMiT / MAS-TRM conversation.
- **Pricing language ("per-user", "per-app for API connectors")** — common reseller framing but not on the public site. Don't quote a price.
- **The `casb-api-feature-matrix` page returned no usable content on fetch** — the most useful vendor table for "which connector supports which action" was JS-gated. The Apps-Supported table I did fetch is a reasonable substitute but feature-by-app granularity is **incomplete** in this draft. This is the single most important gap to close before publication.
