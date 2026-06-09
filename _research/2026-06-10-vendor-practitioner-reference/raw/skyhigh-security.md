# Skyhigh Security (Skyhigh CASB)

## Lineage
- **Current product name(s):** Skyhigh CASB (component of the Skyhigh Security Service Edge / SSE platform). Adjacent platform components: Skyhigh Secure Web Gateway (Cloud + On-Prem), Skyhigh Private Access (ZTNA), Skyhigh DLP, Skyhigh DSPM, Skyhigh AI, Skyhigh Cloud Connector, Skyhigh Client.
- **Renames / acquisitions:**
  - Skyhigh Networks founded 2012 (pure-play CASB).
  - Acquired by McAfee, January 2018.
  - Rebranded **MVISION Cloud** inside McAfee Enterprise.
  - McAfee Enterprise split from McAfee consumer (2021); enterprise side combined with FireEye to form **Trellix** (2022).
  - SSE/CASB carve-out from Trellix (March 2022) became **Skyhigh Security** — a separate, Symphony-Technology-Group-owned company. CASB is now part of Skyhigh's SSE platform, not Trellix.
- **CASB housing:** Stand-alone product *and* a module inside Skyhigh's SSE platform (SWG + ZTNA + CASB + DLP + RBI). Documentation/console treats CASB as its own product surface, even when bundled. This is the last credible *stand-alone* enterprise CASB lineage (Bitglass→Forcepoint, Cisco Cloudlock, Palo Alto SaaS Security, Microsoft Defender for Cloud Apps, Netskope, Lookout all sit inside larger SSE/identity platforms with varying degrees of CASB-as-a-feature collapse).
- **Licensing tier(s):** Skyhigh does not publish per-tier feature matrices openly. CASB is sold as a standalone SKU or bundled in the Skyhigh SSE Enterprise / SSE Complete packaging; specific feature gating (e.g. UEBA, DSPM, RBI, GenAI controls) varies by SKU and is set at quote time. **Unverified from public docs — confirm with Skyhigh prior to design.**

## Deployment modes supported
- **Forward-proxy:** Supported via Skyhigh Client (endpoint agent) and/or PAC-file routing through the Skyhigh SWG cloud POPs; agentless deployment via PAC/explicit-proxy is also documented. Forward proxy is the inline path used for non-sanctioned/Shadow-IT control and for inline DLP on managed endpoints. SSL/TLS inspection is required for most inspection use cases.
- **Reverse-proxy:** Supported and is one of Skyhigh's historical differentiators. Implemented by modifying the SAML endpoint in the customer's IdP so SAML assertions land at the Skyhigh reverse proxy first; the proxy evaluates context (SAML attributes, device certificate, location, device type), then issues a new SAML assertion to the destination SaaS. Documented for Office 365 (OneDrive/SharePoint/Teams/Exchange Online), Salesforce, ServiceNow, Slack, Workday, SuccessFactors, Box, G Suite — agentless, so it works for BYOD/unmanaged devices that authenticate through the corporate IdP.
- **API connectors ("Native Sanctioned Apps"):** Vendor home page says **~40 API integration apps**; documented native integrations include Microsoft 365 (incl. Copilot, Teams, SharePoint, OneDrive, Exchange Online), Google Workspace (Drive, Chat), Salesforce, Box, Dropbox, Slack, ServiceNow, GitHub, Okta, DocuSign, Zoom, Zendesk, Egnyte, Jive, Smartsheet, Clarizen, Confluence, Workplace (by Meta), ShareFile, Bitbucket, Domo, FileCloud, Introhive, SAP S/4HANA, ChatGPT Enterprise. (Vendor page lists ~14 additional unlabelled logos — **count is fluid, treat 37–40 as the order of magnitude.**)
- **Log-based discovery:** Supported via **Skyhigh Cloud Connector** — an on-premises log ingest component that parses web access logs from SWG/firewall/proxy devices, matches IPs/URLs against the Cloud Registry, and uploads enriched events. Standard ingestion via syslog (direct from device or via SIEM relay); also supports file-share, SCP, FTP pulls. Common log sources: Skyhigh SWG, Cisco/Zscaler/Palo Alto/Check Point/Fortinet proxies and firewalls (vendor lists are not exhaustive in the public docs).
- **Hybrid considerations:** Same policy surface is meant to cover Shadow-IT (log + inline-SWG) and Sanctioned-IT (proxy + API) — but the two policy planes are *not* fully unified; DLP rules and access policies live in adjacent menus and need to be authored against the right object (sanctioned app instance vs. cloud service category). Reverse-proxy depends on the SAML flow going through Skyhigh; if a user authenticates to a service outside the IdP (consumer account, direct OAuth), reverse proxy is bypassed. Forward proxy depends on Skyhigh Client being installed *or* PAC enforcement on the network — neither is guaranteed on BYOD.

## Capability set

- **Shadow-IT discovery (network-log-based AND/OR API-based):** **Supported.** Network-log-based via Skyhigh Cloud Connector ingesting SWG/firewall/proxy logs and matching against the Cloud Registry (50,000+ services, 1,900+ AI services per vendor claim). Inline-discovery via Skyhigh SWG is the second path. There is no general API-based discovery of *other vendors'* SaaS without the SaaS having an API connector. Vendor: https://success.skyhighsecurity.com/Skyhigh_CASB/02_Skyhigh_CASB_Architecture/Shadow_IT/Shadow_IT_Deployment_Architectural_Components (accessed 2026-06-10).
- **Sanctioned-app inventory + risk-scoring:** **Supported.** Cloud Registry + **CloudTrust Ratings** (1–9 scale; 1–3 = enterprise-ready, 4–6 = consumer/department-grade, 7–9 = too risky). Each service evaluated against ~110 risk attributes. Sits at *Governance → Cloud Registry*. Vendor: https://success.skyhighsecurity.com/Skyhigh_CASB/05_Skyhigh_CASB_Governance/Cloud_Registry/CloudTrust_Ratings (accessed 2026-06-10).
- **Inline DLP for sanctioned SaaS (proxy mode — pre-upload prevention):** **Supported.** Inline DLP enforced via Skyhigh Client (forward proxy) and reverse-proxy paths; pre-upload block + real-time user notification documented. Vendor: https://www.skyhighsecurity.com/products/cloud-access-security-broker.html (accessed 2026-06-10).
- **API-mode DLP for sanctioned SaaS (post-upload scan + remediate):** **Supported.** Sanctioned DLP policies scan content in connected SaaS (M365, Google Workspace, Box, Slack, Salesforce, etc.), with actions including block/quarantine, encrypt (Ionic/Seclore DRM), notify, end-user remediate, and save-evidence (encrypted copy to S3/Azure, max 250 MB per file). Vendor: https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Sanctioned_DLP_Policies_and_Rules/Sanctioned_DLP_Policies/Save_CASB_Evidence (accessed 2026-06-10).
- **OAuth-app discovery and governance (third-party app grants to M365 / Workspace):** **Supported with caveats.** Feature is called **Connected Apps**, sits at *Skyhigh CASB → Analytics → Connected Apps*; scans Google Workspace and Microsoft 365 (initial full scan, then incremental) for OAuth third-party grants and produces a **Connected Apps Policies** object that can include/exclude users/groups and trigger responses. Caveat: scope limited to the M365/Workspace tenants connected via API — consumer Google sign-ins or grants to apps from outside the connected IdP/tenant are invisible. Vendor: https://success.skyhighsecurity.com/Skyhigh_CASB/07_Skyhigh_CASB_Analytics/Connected_Apps/About_Connected_Apps (accessed 2026-06-10).
- **Anomalous-activity / UEBA detection:** **Supported.** Machine-learning UEBA over user activity from API + proxy telemetry; documented anomaly types include access anomalies (block-list IP/geo/org, login failures), anomalous access location, brute-force, suspicious-IP. Vendor: https://success.skyhighsecurity.com/Skyhigh_CASB/Skyhigh_CASB_Incidents/Anomalies/Anomaly_Types/About_Access_Anomalies (accessed 2026-06-10).
- **Compromised-account detection (impossible travel, brute-force, anomalous-OAuth):** **Supported.** Impossible travel = "login attempts from geographically distant locations within a short time period"; documented separately. Anomalous OAuth grant detection rides on Connected Apps + UEBA. Vendor: see Access Anomalies + Connected Apps URLs above.
- **Data-residency / geo-policy enforcement:** **Supported with caveats.** Geo can be a condition in Cloud Access Policies (allow/block based on user location) and in DLP policies (recipient-domain restrictions). True *data-residency* (ensuring tenant data is stored in a specific region) is a function of the SaaS, not the CASB — Skyhigh enforces *access geography*, not storage geography. Vendor: https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Access_Control_Policies/About_Cloud_Access_Policies (accessed 2026-06-10).
- **Share-link governance (external sharing, anonymous links, expiry):** **Supported.** Collaboration controls on M365/OneDrive/SharePoint/Google Drive/Box: detect, restrict, or revoke external shares; anonymous-link detection; quarantine. Configured under sanctioned-app DLP / collaboration rules. Vendor: https://www.skyhighsecurity.com/products/cloud-access-security-broker.html (accessed 2026-06-10).
- **Malware scanning in SaaS storage:** **Supported.** "Integrated anti-malware solution and inline emulation-based sandboxing" referenced on the vendor product page; covers SaaS file uploads and content at rest via API. Vendor: https://www.skyhighsecurity.com/products/cloud-access-security-broker.html (accessed 2026-06-10).
- **Encryption / tokenisation at gateway:** **Supported with caveats.** Inline encryption / DRM via partner integrations (Ionic, Seclore) — applied as a DLP action (Apply Protection) rather than transparent gateway-level field encryption. Native field-level tokenisation (a 2014-era CASB feature) is not foregrounded in current docs; **unverified whether it still ships as a first-class action.** Vendor: https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Sanctioned_DLP_Policies_and_Rules/Sanctioned_DLP_Policies/Save_CASB_Evidence (accessed 2026-06-10) and https://www.seclore.com/integrations/skyhigh/ (accessed 2026-06-10, independent).
- **Risk-based session policy (step-up auth, read-only, watermark, block):** **Supported.** Cloud Access Policies (Policy → Access Control → Access Policies) support: block access, require step-up authentication, require device registration, allow/deny service uploads, restrict file downloads. Read-only/watermark are also referenced in vendor product copy (download/copy-paste/printing restrictions, tenant restrictions). Vendor: https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Access_Control_Policies/About_Cloud_Access_Policies (accessed 2026-06-10).
- **GenAI-app discovery + governance:** **Supported.** Cloud Registry tracks AI services (vendor claim: 1,900+); inline controls via SWG + Skyhigh Client; API integration with **ChatGPT Enterprise** and with **Microsoft 365 Copilot** (custom Azure OAuth app + Copilot instance in Skyhigh CASB, API-based, surfaces Copilot interactions to DLP). DLP can be applied at "service, user, and activity" granularity for AI apps. *Feature additions 2024–2026:* Skyhigh AI bundle (announced 2023 for ChatGPT/Bard/Bing Chat/Jasper/YouChat); M365 Copilot integration documented; ChatGPT Enterprise as a native sanctioned app. Vendor: https://www.skyhighsecurity.com/solutions/skyhigh-ai.html, https://success.skyhighsecurity.com/Skyhigh_CASB/06_Skyhigh_CASB_Sanctioned_Applications/01_Skyhigh_CASB_Native_Sanctioned_Apps/01_Skyhigh_CASB_for_Microsoft_365_Copilot/02_Integrate_Skyhigh_CASB_with_Copilot (both accessed 2026-06-10).

## Configurable policies

> Console paths below use the documented top-level area names (Policy / Cloud apps / Governance / Incidents / Analytics). Skyhigh's UI has reorganised across the McAfee → Trellix → Skyhigh transitions, so a working analyst should expect drift between docs and current console; the area names below match the 2026 documentation set.

### 1. Sanctioned-app DLP policy (post-upload, API mode)
- **What it does:** Scans content at rest in a connected SaaS (e.g. OneDrive) and acts when a classification fires.
- **Where it lives:** Policy → DLP → Sanctioned DLP Policies (per-service or cross-service).
- **Key knobs:** Service / instance scope; user/group scope; content classifier (built-in identifier, custom keyword/regex/proximity/ML classifier, EDM/IDM fingerprint, OCR); file properties (type, size, encrypted/not); collaboration scope (internal / external / anonymous / domain-list); action (notify user, notify admin, save evidence, quarantine, delete share, restrict collaboration, apply DRM via Ionic/Seclore, block).
- **Deployment-mode requirement:** API (sanctioned-app native integration must be live).
- **False-positive risk:** Medium-High — OCR + regex classifiers on shared file libraries; named scenario: a marketing OneDrive with PDF brochures that contain customer testimonials triggers "PII (name+email)" and quarantines the file, breaking the marketing share.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Sanctioned_DLP_Policies_and_Rules/Sanctioned_DLP_Policies/Save_CASB_Evidence (accessed 2026-06-10).

### 2. Inline DLP policy (proxy mode, pre-upload prevention)
- **What it does:** Blocks or coaches a file upload to a sanctioned or shadow SaaS *before* it leaves the endpoint.
- **Where it lives:** Policy → DLP → Web Policy / Cloud Web Policy (paired with the Cloud Access Policy that steers traffic).
- **Key knobs:** User/group; destination service or service category from Cloud Registry; classifier (built-in/custom/fingerprint/ML); action (block + user notification, coach/justify, monitor only).
- **Deployment-mode requirement:** Forward proxy (Skyhigh Client or PAC) OR reverse proxy. SSL inspection must be enabled.
- **False-positive risk:** Medium — SSL inspection breakage and classifier overmatch on non-business uploads (e.g. user's personal LinkedIn profile photo triggering "internal-document" image classifier).
- **Source:** https://www.skyhighsecurity.com/products/cloud-access-security-broker.html (accessed 2026-06-10).

### 3. Cloud Access Policy — managed-device-required for sanctioned-app access
- **What it does:** Allows access to a sanctioned app only if the endpoint presents a Skyhigh-issued device certificate (= managed). Otherwise, block or step-up.
- **Where it lives:** Policy → Access Control → Access Policies.
- **Key knobs:** App or app-instance; user/group; certificate status (managed/unmanaged); MDM compliance signal; IP/Allow List; geolocation; action (block, step-up MFA, require device registration, allow upload-only / download-only).
- **Deployment-mode requirement:** Reverse proxy (preferred — agentless) or forward proxy with Skyhigh Client.
- **False-positive risk:** High — certificate rollout breakage; named scenario: macOS keychain reset after OS upgrade strips the device cert; user is silently downgraded to "unmanaged" and locked out of SharePoint.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Access_Control_Policies/About_Cloud_Access_Policies (accessed 2026-06-10).

### 4. Connected Apps policy (OAuth third-party app governance)
- **What it does:** Allows or blocks OAuth grants from third-party apps to M365 / Google Workspace based on app risk, scope, or user.
- **Where it lives:** Analytics → Connected Apps → Connected Apps Policies (per Microsoft 365 / Google service instance).
- **Key knobs:** Service/instance; user/group include/exclude; rule predicates over discovered OAuth apps (e.g. risk score, scopes requested, vendor); action (alert, revoke grant, notify user). Initial full scan + incremental scans.
- **Deployment-mode requirement:** API (M365 / Google Workspace connector).
- **False-positive risk:** Medium — auto-revoke of "Zoom Scheduler" or other approved-but-noisy apps; named scenario: Calendly OAuth flagged on broad scope, auto-revoked, sales team loses scheduling links.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_CASB/07_Skyhigh_CASB_Analytics/Connected_Apps/About_Connected_Apps (accessed 2026-06-10).

### 5. External-collaboration / share-link policy
- **What it does:** Detects and restricts external or anonymous shares on M365, Google Drive, Box, Slack, etc.
- **Where it lives:** Policy → DLP → Sanctioned DLP Policies (collaboration controls).
- **Key knobs:** File classification (sensitive/not); share scope (internal/external/anonymous/specific-domain); recipient domain allow/block list; expiry rules; action (remove share, downgrade to view-only, notify owner, quarantine).
- **Deployment-mode requirement:** API.
- **False-positive risk:** Low–Medium — joint-venture domains accidentally on the block list cause legitimate external collaboration to be torn down.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Sanctioned_DLP_Policies_and_Rules/Sanctioned_DLP_Policies/Save_CASB_Evidence (accessed 2026-06-10).

### 6. Anomalous-access-location policy (UEBA)
- **What it does:** Flags or blocks logins from impossible-travel / block-listed / unusual geo for a user.
- **Where it lives:** Incidents → Anomalies → Anomalous Access Location (configure filters); UEBA model on by default for connected sanctioned apps.
- **Key knobs:** Block-list IPs / countries / orgs; learning window; sensitivity; action (incident only, alert, step-up via paired Cloud Access Policy).
- **Deployment-mode requirement:** API (login telemetry from sanctioned app) and/or proxy.
- **False-positive risk:** Medium — VPN, mobile-carrier CG-NAT, and short-trip business travel produce noise; named scenario: user on conference Wi-Fi geolocated to a different country than their VPN exit, triggers daily "anomalous location" tickets.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_CASB/Skyhigh_CASB_Incidents/Anomalies/Anomaly_Types/About_Access_Anomalies (accessed 2026-06-10).

### 7. Compromised-account / brute-force policy
- **What it does:** Detects repeated failed logins, atypical login bursts, and successful logins after failure spikes.
- **Where it lives:** Incidents → Anomalies → Access Anomalies (block-list + UEBA).
- **Key knobs:** Failure threshold; time window; user/group scope; action (incident, notify, require step-up via Cloud Access Policy).
- **Deployment-mode requirement:** API (auth events) and/or reverse proxy (SAML logs).
- **False-positive risk:** Low–Medium — application-bot retries (Outlook reauth loops on a password change) generate brute-force-looking events.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_CASB/Skyhigh_CASB_Incidents/Anomalies/Anomaly_Types/About_Access_Anomalies (accessed 2026-06-10).

### 8. Shadow-IT control policy — block-by-CloudTrust-rating
- **What it does:** Blocks or coaches access to any cloud service in the Cloud Registry above a chosen CloudTrust risk rating.
- **Where it lives:** Policy → Web Policy (Skyhigh SWG) or Governance → Cloud Registry → Service Groups (via Web Policy reference).
- **Key knobs:** Service Group selector; CloudTrust rating threshold (1–9); category (e.g. file-sharing, GenAI); user/group; action (block, coach with justification, monitor).
- **Deployment-mode requirement:** Forward proxy (Skyhigh SWG) OR log-only (monitor; cannot block from logs alone).
- **False-positive risk:** Medium — Service Group misclassification of a business-needed niche tool above the threshold; named scenario: a regional bank-rails fintech rated 7 because no public SOC 2 evidence is in CloudTrust gets blocked, breaking the AP workflow.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_CASB/05_Skyhigh_CASB_Governance/Cloud_Registry/CloudTrust_Ratings (accessed 2026-06-10).

### 9. GenAI usage policy (inline)
- **What it does:** Blocks or coaches uploads of sensitive content into AI apps (ChatGPT, Bard/Gemini, Bing Chat/Copilot, Jasper, YouChat) via the proxy path.
- **Where it lives:** Policy → Web Policy → AI Service Group (or named AI Service category from the Cloud Registry); paired with DLP classifiers.
- **Key knobs:** Service or AI Service Group; user/group; DLP classifier (PII, source code, customer data); action (block, coach with justify, monitor); optional integration with Skyhigh RBI for read-only browser session.
- **Deployment-mode requirement:** Forward proxy / Skyhigh SWG (plus Skyhigh Client or PAC for off-network).
- **False-positive risk:** Medium-High — prompt content matched by regex classifiers (e.g. credit-card-like number in a developer's sample code) causes constant coach prompts that desensitise users.
- **Source:** https://www.skyhighsecurity.com/solutions/skyhigh-ai.html (accessed 2026-06-10).

### 10. M365 Copilot DLP policy (API-mode AI control)
- **What it does:** Applies DLP to content surfaced or generated by Microsoft 365 Copilot via the connected Copilot API integration.
- **Where it lives:** Sanctioned Applications → Microsoft 365 → Copilot → DLP integration (configured after the Copilot API instance is added).
- **Key knobs:** Tenant/instance; user/group; DLP classifier for indexed source content; action (monitor, alert, save evidence; downstream Purview-label enforcement if labels exist).
- **Deployment-mode requirement:** API (Azure custom OAuth app + Copilot instance in Skyhigh).
- **False-positive risk:** Medium — Copilot grounding over a wide SharePoint corpus surfaces archive content classifiers were never tuned against; named scenario: 2017 HR docs migrated into SharePoint, all PII classifiers fire on every Copilot prompt touching that library.
- **Source:** https://success.skyhighsecurity.com/Skyhigh_CASB/06_Skyhigh_CASB_Sanctioned_Applications/01_Skyhigh_CASB_Native_Sanctioned_Apps/01_Skyhigh_CASB_for_Microsoft_365_Copilot/02_Integrate_Skyhigh_CASB_with_Copilot (accessed 2026-06-10).

### 11. SaaS malware scan policy
- **What it does:** Scans files uploaded to or already resident in connected SaaS storage and quarantines on detection.
- **Where it lives:** Policy → Threat Protection (paired with sanctioned-app instance).
- **Key knobs:** Service/instance; file scope (new uploads only / all); engines (signature AV + sandboxing); action (quarantine, alert owner, alert admin).
- **Deployment-mode requirement:** API for at-rest scan; forward proxy for inline scan during upload.
- **False-positive risk:** Low — modern AV; main risk is sandbox latency on large files (delivered then quarantined).
- **Source:** https://www.skyhighsecurity.com/products/cloud-access-security-broker.html (accessed 2026-06-10).

### 12. Tenant restrictions / unsanctioned-instance policy
- **What it does:** Lets users reach the *sanctioned* M365 / Google tenant but blocks personal or other-org tenants of the same service.
- **Where it lives:** Policy → Web Policy → Tenant Restrictions (uses HTTP headers per Microsoft/Google tenant-restrictions specs).
- **Key knobs:** Service; allowed tenant ID list; user/group; action (block / read-only).
- **Deployment-mode requirement:** Forward proxy (SWG inserts the tenant-restrictions header).
- **False-positive risk:** Low — but high blast radius if the wrong tenant ID is on the allow-list (whole org locked out of M365).
- **Source:** https://www.skyhighsecurity.com/products/cloud-access-security-broker.html (accessed 2026-06-10).

### Worked example — Policy #3, managed-device-required Cloud Access Policy (conceptual JSON)

```jsonc
{
  "policy_type": "CloudAccessPolicy",
  "name": "M365-managed-device-required",
  "service": "Microsoft 365",
  "service_instances": ["corp-tenant-prod"],
  "scope": {
    "users_groups_include": ["All Employees"],
    "users_groups_exclude": ["Break-Glass Admins"]
  },
  "conditions": [
    { "type": "device_certificate", "operator": "IS",     "value": "valid" },
    { "type": "geolocation",        "operator": "IS_NOT", "value": ["KP","IR","SY"] },
    { "type": "saml_attribute",     "name": "department", "operator": "IS_NOT",
      "value": "Contractor" }
  ],
  "action_on_match": "allow",
  "action_on_no_match": {
    "primary": "block",
    "user_message": "Your device is not managed. Enroll via MDM, or use the web-only portal.",
    "fallback_chain": [
      { "if": "user_in_group", "value": "Executives",
        "then": "step_up_mfa_then_allow_read_only" }
    ]
  },
  "logging": { "save_evidence": false, "incident_severity": "medium" },
  "deployment_mode": "reverse_proxy_via_idp_saml",
  "ordering": "evaluated_top_down_first_match_wins"
}
```

Not literal product syntax. The intent is to show that one policy carries: scope, ordered conditions with IS / IS_NOT, a positive action and a negative action with optional fallback, and a deployment-mode binding. The "first match wins" rule is documented behaviour — order matters.

## Limitations / what this product does NOT cover

- **Real-time blocking of file uploads to non-sanctioned SaaS for off-network BYOD** — Skyhigh Client / PAC is the only inline path; BYOD without the client and not routed through PAC bypasses inline DLP entirely. API connector does not exist for non-sanctioned SaaS, so post-upload remediation is also unavailable.
- **OAuth grants to apps not registered against the connected IdP / tenant** — Connected Apps discovers third-party apps tied to the Microsoft 365 / Google Workspace tenants Skyhigh has API access to. OAuth grants given via a consumer Google or personal Microsoft account on the same device are invisible because the OAuth handshake never touches the connected tenant.
- **Mobile native apps with certificate pinning** — SSL inspection breaks the pinned cert; the app either refuses the connection (and the user falls back to web / personal account) or bypasses the proxy. Reverse-proxy mode does not help here because mobile native apps frequently use non-SAML auth flows.
- **Reverse-proxy on services that do not use SAML / OIDC through the corporate IdP** — Reverse proxy depends on hijacking the SAML assertion. Anything that authenticates via service-local credentials, OAuth-to-personal-account, or static API keys does not flow through the reverse proxy and is invisible to it.
- **Forward-proxy DLP on TLS 1.3 + ECH (Encrypted Client Hello) endpoints** — TLS 1.3 with ECH defeats SNI-based steering and complicates SSL inspection at the proxy; this is a structural limit of the inline model, not a Skyhigh-specific bug, but it does mean Skyhigh's inline DLP coverage degrades as endpoints adopt ECH.
- **End-to-end-encrypted SaaS content (e.g. E2EE Zoom recordings, encrypted Slack DMs in EKM mode, Signal-style E2EE workspaces)** — API connector pulls metadata only; content body is unreadable. DLP classifiers cannot fire on opaque ciphertext.
- **Field-level / native tokenisation as a gateway action** — Documented "Apply Protection" actions route through Ionic / Seclore DRM. A turnkey, gateway-resident format-preserving-encryption / tokenisation action (the original 2014-era CASB feature) is not foregrounded in current docs; unverified whether it survives as a first-class action without partner integration.
- **Cross-vendor SaaS-to-SaaS data flow inspection (Zapier-style integrations)** — When SaaS A pushes data to SaaS B via a vendor integration platform that authenticates with its own service principal, the traffic doesn't traverse Skyhigh forward proxy and isn't visible via either tenant's API unless the integration touches a covered tenant.
- **Sanctioned-app coverage outside the ~40 native connectors** — Anything not in the published Native Sanctioned Apps list (most niche SaaS, regional fintechs, vertical industry SaaS) has no API-mode inspection — only the inline/Shadow-IT view, which means no post-upload remediation and no Connected-Apps OAuth discovery.
- **Data-residency enforcement at the storage layer** — Cloud Access Policies enforce *where the user is*, not *where the SaaS stores the data*. If M365 stores tenant data in EU but the user travels to a third country, Skyhigh can block the user, but it cannot move the data and cannot enforce "no replication" inside the SaaS provider.
- **Endpoint DLP / unmanaged-USB egress** — Skyhigh CASB is a network/API control. It does not cover local USB writes, screenshot, clipboard-to-personal-app, or print egress. A separate endpoint-DLP tool is required.
- **GenAI control inside an LLM SaaS at the conversation-content layer for non-integrated AI apps** — For AI apps without a dedicated API integration (most third-party GenAI), Skyhigh inspects the *upload* but cannot retrieve or remediate conversation history after the fact (unlike, e.g., the ChatGPT Enterprise or Copilot integrations, which do have API hooks).

## Sources

- [Skyhigh CASB product page](https://www.skyhighsecurity.com/products/cloud-access-security-broker.html) — vendor-controlled (accessed 2026-06-10).
- [Skyhigh Security documentation portal home](https://success.skyhighsecurity.com/) — vendor-controlled (accessed 2026-06-10).
- [Skyhigh CASB documentation index](https://success.skyhighsecurity.com/Skyhigh_CASB) — vendor-controlled (accessed 2026-06-10).
- [Skyhigh CASB Architecture](https://success.skyhighsecurity.com/Skyhigh_CASB/02_Skyhigh_CASB_Architecture/Skyhigh_CASB/Skyhigh_CASB_Architecture) — vendor-controlled (accessed 2026-06-10).
- [Shadow IT Deployment Architectural Components](https://success.skyhighsecurity.com/Skyhigh_CASB/02_Skyhigh_CASB_Architecture/Shadow_IT/Shadow_IT_Deployment_Architectural_Components) — vendor-controlled (accessed 2026-06-10).
- [CloudTrust Ratings](https://success.skyhighsecurity.com/Skyhigh_CASB/05_Skyhigh_CASB_Governance/Cloud_Registry/CloudTrust_Ratings) — vendor-controlled (accessed 2026-06-10).
- [About Cloud Access Policies](https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Access_Control_Policies/About_Cloud_Access_Policies) — vendor-controlled (accessed 2026-06-10).
- [Save CASB Evidence (Sanctioned DLP)](https://success.skyhighsecurity.com/Skyhigh_Data_Loss_Prevention/Sanctioned_DLP_Policies_and_Rules/Sanctioned_DLP_Policies/Save_CASB_Evidence) — vendor-controlled (accessed 2026-06-10).
- [About Access Anomalies](https://success.skyhighsecurity.com/Skyhigh_CASB/Skyhigh_CASB_Incidents/Anomalies/Anomaly_Types/About_Access_Anomalies) — vendor-controlled (accessed 2026-06-10).
- [Configure Anomalous Access Location Filters](https://success.skyhighsecurity.com/Skyhigh_CASB/Skyhigh_CASB_Incidents/Anomalies/Anomalous_Access_Location_Anomalies/Configure_Anomalous_Access_Location_Filters) — vendor-controlled (accessed 2026-06-10).
- [About Connected Apps (OAuth governance)](https://success.skyhighsecurity.com/Skyhigh_CASB/07_Skyhigh_CASB_Analytics/Connected_Apps/About_Connected_Apps) — vendor-controlled (accessed 2026-06-10).
- [Connected Apps for Microsoft 365](https://success.skyhighsecurity.com/Skyhigh_CASB/07_Skyhigh_CASB_Analytics/Connected_Apps/Connected_Apps_for_Microsoft_365) — vendor-controlled (accessed 2026-06-10).
- [Integrate Skyhigh CASB with Microsoft 365 Copilot](https://success.skyhighsecurity.com/Skyhigh_CASB/06_Skyhigh_CASB_Sanctioned_Applications/01_Skyhigh_CASB_Native_Sanctioned_Apps/01_Skyhigh_CASB_for_Microsoft_365_Copilot/02_Integrate_Skyhigh_CASB_with_Copilot) — vendor-controlled (accessed 2026-06-10).
- [Reverse Proxy for Office 365 via Azure AD](https://success.skyhighsecurity.com/Skyhigh_CASB/06_Skyhigh_CASB_Sanctioned_Applications/01_Skyhigh_CASB_Native_Sanctioned_Apps/Skyhigh_CASB_for_Office_365/Reverse_Proxy_for_Office_365_via_Azure_AD) — vendor-controlled (accessed 2026-06-10).
- [Configure IdPs for SaaS Applications and SAML Proxy Redirection](https://success.skyhighsecurity.com/Skyhigh_CASB/06_Skyhigh_CASB_Sanctioned_Applications/01_Skyhigh_CASB_Native_Sanctioned_Apps/SAML_Proxy_Deployment_Guide/Configure_IdPs_for_SaaS_Applications_and_SAML_Proxy_Redirection) — vendor-controlled (accessed 2026-06-10).
- [Skyhigh AI solution page](https://www.skyhighsecurity.com/solutions/skyhigh-ai.html) — vendor-controlled (accessed 2026-06-10).
- [Skyhigh CASB Datasheet (PDF, 2023)](https://www.skyhighsecurity.com/wp-content/uploads/2023/01/ds-cloud-access-securty-broker.pdf) — vendor-controlled (accessed 2026-06-10).
- [Seclore for Skyhigh Security CASB](https://www.seclore.com/integrations/skyhigh/) — **independent (Seclore corroborates DRM integration)** (accessed 2026-06-10).
- [Skyhigh CASB (formerly McAfee MVISION Cloud) overview — Macnica](https://www.macnica.co.jp/en/business/security/manufacturers/skyhighsecurity/casb.html) — **independent (distributor)** (accessed 2026-06-10).

## Notes for the lens reviewers

- **Licensing tiers are unverified.** Skyhigh does not publish a feature/SKU matrix publicly. The architect lens should require a vendor-provided licensing matrix before building cost or capability bets. The compliance lens should not rely on "Skyhigh supports X" without verifying X is in the licensed tier the customer actually bought.
- **Console paths drifted across the McAfee → Trellix → Skyhigh transitions.** Documentation set still has some `success.myshn.net` legacy URLs alongside `success.skyhighsecurity.com`. Treat any specific menu-path claim ("Policy → Access Control → Access Policies") as accurate-at-time-of-writing, not contractual.
- **Native sanctioned-app count of "37" → "40" on the vendor page is a moving target.** The Cloud Registry coverage claim (50,000+ services, 1,900+ AI services, 110 risk attributes) is also vendor-asserted with no published methodology. Product lens should pin Skyhigh on which specific apps in the buyer's stack have *native* coverage (DLP at rest + Connected Apps), versus only inline/Shadow-IT coverage.
- **Reverse-proxy is Skyhigh's historical differentiator and is still documented as first-class.** Worth verifying with the architect lens: how do customers actually deploy in 2026? If most deployments are forward-proxy via Skyhigh Client + API only, then the "stand-alone CASB with multi-mode" pitch is more brochure than reality. Several documented reverse-proxy targets (Salesforce, ServiceNow, Workday) are exactly the apps where mobile native + cert-pinning erodes inline value.
- **Field-level encryption / tokenisation:** could not confirm a first-class native action without partner DRM (Ionic / Seclore). Compliance lens should treat any tokenisation-at-gateway story as integration-dependent.
- **GenAI integration recency:** ChatGPT Enterprise and M365 Copilot are documented as native sanctioned apps, which is genuinely current (2024–2026). Claude, Gemini consumer/enterprise, and Mistral are *not* called out as native — they appear to be covered only via inline / Cloud Registry, which is a meaningful capability gap if the customer has standardised on a non-OpenAI / non-Microsoft AI stack.
- **DLP unification with sanctioned-app DLP and SWG/Web DLP** — vendor copy says "unified DLP" but the documentation tree shows distinct policy objects under DLP for sanctioned vs. web. Architect lens should verify whether one classifier authored once fires consistently across inline, API, and endpoint DLP paths, or whether duplication is needed.
- **Shadow-IT log-source list is not enumerated in the public docs.** Documentation references Skyhigh Cloud Connector with syslog/SCP/FTP/file-share ingest but does not publish a definitive list of supported log formats. Compliance lens should ask for the supported-vendor list before assuming an existing SIEM/SWG can feed it.
- **No CVE / public-incident search performed in this draft.** Independent corroboration is limited to one DRM partner page (Seclore) and one distributor page (Macnica). Lens reviewers should run a CVE/CISA/NVD pass on "Skyhigh", "MVISION Cloud", "McAfee MVISION Cloud", and "Skyhigh Networks" before promotion.
