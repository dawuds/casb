# Zscaler Internet Access (ZIA) — CASB / Data Protection

> Research draft. Vendor `help.zscaler.com` HTML pages render via JavaScript and returned empty bodies to WebFetch on 2026-06-10. Material below is drawn from Zscaler help-portal URLs surfaced via search snippets, Zscaler innovation/blog pages, the Zscaler Data Protection Reference Architecture PDF, and the Zscaler/Microsoft Copilot deployment guide. Claims that rely on snippet text rather than a directly-read full page are flagged **[snippet]**. Unverified inferences are flagged **[unverified]**.

## Lineage
- **Current product name:** Zscaler Internet Access (ZIA), with the CASB feature set marketed as **Zscaler CASB** and split into **Inline CASB** (in the ZIA proxy data path) and **Out-of-Band CASB / SaaS Security API** (API connectors to sanctioned SaaS). The configuration-time console name is **"SaaS Security"** for the API side and **"Cloud App Control"** for the inline side. Adjacent feature: **3rd-Party App Governance** (formerly the AppTotal product, an OAuth-grant-and-app-supply-chain governance plane).
- **Renames / acquisitions:**
  - Zscaler founded 2007; ZIA is the original product, ZPA added later, ZDX added for digital experience, the platform marketed as the Zero Trust Exchange / SSE.
  - **ShiftRight** acquired 2022 → folded into SaaS posture remediation workflow.
  - **Canonic Security** acquired 2023 → became **3rd-Party App Governance** / "AppTotal", the SaaS-to-SaaS OAuth governance product. Now reachable from inside the ZIA console under SaaS Security. **[snippet]**
  - **Avalor** acquired 2024 → data fabric / "Data Fabric for Security", not CASB-core but feeds risk telemetry.
- **CASB housing:** Inside an SSE platform. CASB is **not** a separable SKU end-state — it sits inside ZIA's traffic flow (inline) and ZIA's API connectors (out-of-band). Practitioners license it as part of ZIA, not as a stand-alone CASB.
- **Licensing tiers (where CASB capabilities live):**
  - **ZIA Essentials** — no meaningful CASB. URL filtering, basic firewall.
  - **ZIA Business** — Cloud App Control + Shadow IT report ship here. Inline DLP basics. **[unverified for current 2026 SKU sheet — confirm against the current Zscaler price book; tier names have changed historically]**
  - **ZIA Advanced / Transformation** — Out-of-Band CASB (SaaS Security API), advanced DLP (EDM, IDM, OCR), Cloud Browser Isolation, SSPM, 3rd-Party App Governance. Public reseller pages describe a "Transformation Edition" bundle that explicitly lists "OOB CASB Standard". **[snippet]**
  - **Data Protection add-on** — packaged separately on some price lists, includes inline DLP, SaaS API DLP, email DLP, unmanaged-device controls. **[snippet]**

## Deployment modes supported

- **Forward-proxy (inline)** — the primary mode. Traffic steering options:
  - **Zscaler Client Connector** (endpoint agent) — the default for managed endpoints; tunnels port-80/443 (and configurable other ports) to the nearest ZIA service edge.
  - **PAC file** — browser-only, no agent; fragile on BYOD.
  - **GRE / IPsec tunnel** from a branch firewall or SD-WAN device.
  - **IdP-routed / SAML proxy** ("Source IP Anchoring", Browser Access for ZPA) — used for unmanaged devices to force SaaS traffic through Zscaler without an agent.
  - **SSL/TLS inspection** is required for any meaningful inline CASB or DLP action — without decryption, ZIA sees only the hostname / SNI. Inspection uses a Zscaler intermediate CA (or a customer intermediate) trusted on the endpoint.
- **Reverse-proxy** — Zscaler does **not** market a classic Bitglass/Netskope-style reverse-proxy mode (no SAML rewrite of the IdP sign-in URL to inject the CASB into the auth flow for unmanaged devices). The equivalent on the Zscaler platform is **agentless Cloud Browser Isolation** triggered by an IdP integration: the IdP redirects unmanaged-device sessions through Zscaler CBI, which renders the SaaS app and ships only pixels to the device. **[snippet — confirm by reading the CBI policy guide end-to-end]**
- **API connectors (Out-of-Band CASB / SaaS Security API).** Documented native API connectors (from vendor partner pages and the SaaS Security API help-portal index):
  - Microsoft 365 (OneDrive, SharePoint Online, Exchange Online, Teams) **[snippet]**
  - Google Workspace (Drive, Gmail) **[snippet]**
  - Box **[snippet]**
  - Dropbox **[snippet]**
  - Salesforce **[snippet]**
  - ServiceNow **[snippet]**
  - Slack **[snippet]**
  - Atlassian (Jira / Confluence) — SSPM scope **[snippet]**
  - GitHub — Cloud App Control granular activity controls confirmed; API DLP scope **[unverified]**
  - Webex — listed in vendor data sheets **[unverified for current API parity]**
- **Log-based discovery** — Shadow IT report consumes ZIA's own proxy/firewall logs. Third-party log ingestion (Palo Alto, Cisco, etc.) is **not** Zscaler's discovery model; the discovery is downstream of users-on-Zscaler traffic. If users bypass the Zscaler client, those sessions are not in the Shadow IT report.
- **Hybrid considerations:**
  - Inline policies require traffic steering + SSL inspection. Devices that cannot install the Client Connector (some IoT, some contractor BYOD, some kiosks) are blind spots.
  - API mode is post-event: the file lands in the tenant, then the API scan finds it. Latency for first-scan can be minutes to hours depending on the SaaS app's webhook / change-feed.
  - 3rd-Party App Governance reads OAuth grants from the connected IdP/SaaS tenant — it only sees grants for tenants you've connected.

## Capability set

> URL access date for all entries below: **2026-06-10**. Where the help-portal URL returned empty rendered HTML, the snippet was sourced from search results; this is flagged inline.

- **Shadow-IT discovery (network-log-based and API-based)** — **Supported.** Network-log based. ZIA mines its own proxy logs and tags transactions against a vendor-maintained catalogue ("over 20,000 applications" per vendor copy; treat the number as marketing — what matters is that the catalogue is large enough for typical enterprise discovery). Each application is assigned an Application Risk Index 1–5 (1 = low, 5 = high). Practitioners can mark apps **sanctioned / unsanctioned**, which then drives Cloud App Control rules. AppTotal SSO integration extends the report with third-party-app exposure data. Source: `https://help.zscaler.com/zia/about-shadow-it-report` **[snippet]**, `https://help.zscaler.com/zia/about-cloud-application-risk-profile` **[snippet]**. Vendor-controlled. Access 2026-06-10.
- **Sanctioned-app inventory + risk-scoring** — **Supported.** Application risk attributes include compliance certifications, data-handling, security posture, hosting jurisdiction — visible per-app in the Shadow IT report. Source: `https://help.zscaler.com/zia/about-cloud-applications` **[snippet]**. Vendor-controlled. Access 2026-06-10.
- **Inline DLP for sanctioned SaaS (proxy mode — pre-upload prevention)** — **Supported.** ZIA's inline DLP engine inspects the HTTP body during upload to any web/SaaS destination it can decrypt, matches against DLP dictionaries / EDM / IDM / OCR, and blocks before the payload reaches the SaaS tenant. Effective only when (a) SSL inspection is enabled, (b) the SaaS app does not use certificate pinning that breaks decryption, (c) the user is steered through ZIA. Source: `https://help.zscaler.com/downloads/zia/reference-architecture/data-protection-secure-internet-and-saas-access/Data-Protection-with-Secure-Internet-and-SaaS-Access-ZIA-Reference-Architecture.pdf`. Vendor-controlled. Access 2026-06-10.
- **API-mode DLP for sanctioned SaaS (post-upload scan + remediate)** — **Supported.** The SaaS Security API DLP policy crawls connected tenants for data at rest, matches DLP rules, and applies remediation. Remediation actions: notify owner, notify admin, quarantine (Box/OneDrive/SharePoint to user root with tombstone template; Salesforce with Restore), remove external collaborators (via domain profiles), change sharing scope. Source: `https://help.zscaler.com/zia/configuring-saas-security-api-dlp-policy` **[snippet]**, `https://help.zscaler.com/zia/about-saas-security-api-dlp` **[snippet]**, `https://www.zscaler.com/innovations/enhanced-quarantine-capabilities-saas-security-api-dlp-policy` **[snippet]**. Vendor-controlled. Access 2026-06-10.
- **OAuth-app discovery and governance (third-party app grants to M365 / Workspace)** — **Supported with caveats.** Under "3rd-Party App Governance" (former Canonic/AppTotal). Connects to M365, Google Workspace, Salesforce, Slack, GitHub tenants and enumerates OAuth grants, scopes, publisher reputation, behaviour. Caveat: only enumerates grants for connected tenants — direct-to-app OAuth on unmanaged identities is invisible. Source: `https://help.zscaler.com/zia/what-3rd-party-app-governance` **[snippet]**, `https://help.zscaler.com/zia/3rd-party-app-governance-policies` **[snippet]**. Vendor-controlled. Access 2026-06-10.
- **Anomalous-activity / UEBA detection** — **Supported with caveats.** ZIA exposes Security & UEBA Alerts: failed-login bursts, brute force, location anomalies, bulk uploads, off-hours activity, encrypted uploads. The detection draws from ZIA's own traffic telemetry, not from per-SaaS audit log streams (cf. Defender for Cloud Apps' Activity Log). For SaaS-tenant-side detections beyond what the proxy sees, Zscaler positions integrations with third-party UEBA (e.g. Gurucul). Source: `https://help.zscaler.com/zia/about-security-ueba-alerts` **[snippet]**, `https://help.zscaler.com/zscaler-technology-partners/zscaler-and-gurucul-deployment-guide` **[snippet]**. Vendor-controlled. Access 2026-06-10.
- **Compromised-account detection (impossible travel, brute-force, anomalous-OAuth)** — **Supported with caveats.** Impossible-travel and brute-force are surfaced by the UEBA alerts above. Anomalous-OAuth is surfaced by 3rd-Party App Governance. The signal is only as good as the connected sources — an attacker authenticating directly to the SaaS tenant from outside ZIA's traffic path will not produce a ZIA-side impossible-travel event. **[unverified — confirm whether SaaS Security API ingests M365 sign-in logs directly]**.
- **Data-residency / geo-policy enforcement** — **Supported with caveats.** ZIA service edges exist across regions; tenants can constrain which Zscaler regions handle decryption / DLP scanning. SaaS-side residency (where the SaaS itself stores the file) is the SaaS vendor's concern, not Zscaler's. Source: vendor reference architecture PDF (above). **[unverified for the specific list of regions where SaaS API scanning runs vs inline]**.
- **Share-link governance (external sharing, anonymous links, expiry)** — **Supported.** SaaS Security API policies can detect publicly-shared files, anonymous-link files, and externally-shared files in OneDrive / SharePoint / Google Drive / Box, and remediate by removing the link, removing collaborators by domain, or quarantining. Source: `https://www.zscaler.com/innovations/oob-casb-granularity-and-control-enhanced-control-domain-profiles-and-collaborator` **[snippet]**. Vendor-controlled. Access 2026-06-10. Expiry-policy enforcement is the SaaS vendor's primitive; Zscaler can trigger remediation but not set tenant-wide expiry defaults. **[unverified]**
- **Malware scanning in SaaS storage** — **Supported.** SaaS Security API Malware Detection policy scans files at rest in connected tenants, with quarantine / notify remediation. Source: `https://help.zscaler.com/zia/configuring-saas-security-api-malware-detection-policy` **[snippet]**, `https://help.zscaler.com/zia/about-saas-security-api-malware-detection` **[snippet]**. Vendor-controlled. Access 2026-06-10.
- **Encryption / tokenisation at gateway** — **Not supported** as a documented native feature in ZIA CASB. (Classic CASB-gateway tokenisation — encrypt-before-storage with format-preserving tokens — is not a Zscaler positioning.) **[unverified — confirm by checking the "Cloud DLP" feature index; no positive evidence found in search]**.
- **Risk-based session policy (step-up auth, read-only, watermark, block)** — **Supported with caveats.**
  - **Block / Caution / Allow / Isolate** are the four Cloud App Control actions. "Caution" displays an interstitial warning; "Isolate" hands the session to Cloud Browser Isolation, which can enforce read-only (no download, no copy, no paste, no print) and watermarking. Source: `https://help.zscaler.com/zia/about-cloud-app-control` **[snippet]**, `https://help.zscaler.com/zia/adding-rules-cloud-app-control-policy` **[snippet]**.
  - **Step-up auth / MFA** is not a Zscaler primitive — Zscaler steers, the IdP authenticates. Step-up is configured on the IdP (Okta, Entra) and is triggered by Zscaler signals only via integration, not natively. **[unverified for any current "force re-auth via IdP from Zscaler policy" feature]**.
- **GenAI-app discovery + governance (ChatGPT, Claude, Copilot)** — **Supported.** "AI & ML Applications" is a Cloud App Control category. Granular per-activity controls documented for ChatGPT (Chat / Upload / Download / Delete / Share / Invite) and Microsoft Copilot (Chat / Delete / Rename / Share / Upload). 2025 additions: prompt classification + inspection, DLP applied to prompt bodies, WebSocket inspection (Copilot on the web uses WebSockets — without this, prompt content is opaque to the proxy). Source: `https://help.zscaler.com/zia/adding-ai-ml-applications-rule-cloud-app-control` **[snippet]**, `https://www.zscaler.com/blogs/product-insights/zia-innovation-launch-part-2-websockets-workflows-full-stack-security-genai` **[snippet]**, `https://www.zscaler.com/blogs/product-insights/securing-gen-ai-and-microsoft-copilot-how-zscaler-data-protection-keeps` **[snippet]**. Vendor-controlled. Access 2026-06-10.

## Configurable policies

Eleven policies a practitioner can actually configure. Console paths are given as the vendor labels them; menu paths shift between ZIA Admin Portal redesigns, so navigate by feature name.

### 1. Shadow-IT App Sanctioning + Risk Tagging
- **What it does:** Marks an application as Sanctioned / Unsanctioned / Any (default) and pins an Application Risk Index that other policies can filter on.
- **Where it lives:** ZIA Admin Portal → Analytics → Shadow IT Report → app row → "Sanction status" / "Mark as sanctioned/unsanctioned".
- **Key knobs:** Per-app status (Sanctioned / Unsanctioned / Any), per-app notes, per-app risk override.
- **Deployment-mode requirement:** None — this is a tagging primitive, but it only sees apps that appeared in ZIA traffic.
- **False-positive risk:** Low — it is a label, not an enforcement; downstream policies can still misfire.
- **Source:** `https://help.zscaler.com/zia/about-shadow-it-report` **[snippet]**, access 2026-06-10.

### 2. Cloud App Control — Block High-Risk Unsanctioned Apps
- **What it does:** Blocks access to any cloud app in a chosen category with risk index ≥ N for chosen user groups.
- **Where it lives:** ZIA Admin Portal → Policy → Cloud App Control → add rule (per category: File Sharing, AI & ML, Webmail, Social, Collaboration, etc.).
- **Key knobs:**
  - Filter: users / groups / departments / locations / device posture
  - App scope: by category, by specific app, or "all apps where Sanction status = Unsanctioned AND Risk Index ≥ 4"
  - Action: **ALLOW / BLOCK / CAUTION / ISOLATE**
  - Optional: enforce time-of-day window
- **Deployment-mode requirement:** Inline (forward-proxy + SSL inspection).
- **False-positive risk:** Medium — broad category blocks (e.g. "all File Sharing") routinely catch a sanctioned-but-uncategorised app the first time it appears.
- **Source:** `https://help.zscaler.com/zia/about-cloud-app-control` **[snippet]**, `https://help.zscaler.com/zia/adding-rules-cloud-app-control-policy` **[snippet]**, access 2026-06-10.

### 3. Tenant Restriction — Personal vs Corporate Instance
- **What it does:** Lets corporate M365 / Google Workspace / Salesforce / Dropbox tenants through, blocks (or downgrades to read-only) any other instance of the same SaaS app.
- **Where it lives:** ZIA Admin Portal → Policy → Cloud App Control → Tenant Profiles → create profile → reference the profile from a Cloud App Control rule.
- **Key knobs:**
  - Allowed tenant IDs (Entra tenant GUID for M365, Workspace domain for Google, etc.)
  - Action for non-listed tenants: Block | Allow read-only (no upload) | Caution
  - Scope: which user groups it applies to
- **Deployment-mode requirement:** Inline + SSL inspection.
- **False-positive risk:** Medium — newly-onboarded corporate tenants (e.g. acquired entity) get blocked until added. Personal-account sign-in to a Microsoft service the corporate identity also uses can block legitimate flows.
- **Source:** `https://help.zscaler.com/zia/policies/cloud-apps/tenant-restriction` **[snippet]**, `https://help.zscaler.com/zia/about-tenant-profiles` **[snippet]**, access 2026-06-10.

### 4. Inline DLP for SaaS Upload
- **What it does:** Inspects the body of HTTP/S uploads to any SaaS destination and blocks transmission if it matches a DLP rule.
- **Where it lives:** ZIA Admin Portal → Policy → Data Loss Prevention → DLP Rule.
- **Key knobs:**
  - User / group / location / device filter
  - Destination scope: URL category (e.g. "AI & ML Applications", "Webmail", "Personal Storage") or specific apps
  - Content match: predefined dictionaries (PCI, PII, PHI), custom regex, EDM (Exact Data Match), IDM (Indexed Document Match), OCR for images
  - Action: Block | Allow + Log | Confirm (user-justification) | Quarantine to ICAP
  - Notification template
- **Deployment-mode requirement:** Inline + SSL inspection. Will not protect a session that bypasses the Client Connector.
- **False-positive risk:** High — wide PII dictionaries on chat-style apps fire constantly; tune by content threshold and by app scope.
- **Source:** ZIA Data Protection Reference Architecture PDF (vendor-controlled), access 2026-06-10.

### 5. SaaS Security API DLP — Data at Rest
- **What it does:** Crawls connected tenants for sensitive content already stored and remediates.
- **Where it lives:** ZIA Admin Portal → Policy → SaaS Security API → DLP.
- **Key knobs:**
  - Tenant scope (which connected SaaS tenant)
  - Content match: same DLP dictionaries / EDM / IDM as inline
  - Action: Notify owner | Notify admin | Quarantine (Box / OneDrive / SharePoint to user root, with optional tombstone template; Salesforce supports Restore) | Remove external collaborators by domain profile | Change file to private
  - Scan schedule: continuous (change-feed) or scheduled full scan
- **Deployment-mode requirement:** API (Out-of-Band).
- **False-positive risk:** Medium — quarantine on false positive disrupts the file owner; start with Notify until rule precision is known.
- **Source:** `https://help.zscaler.com/zia/configuring-saas-security-api-dlp-policy` **[snippet]**, `https://www.zscaler.com/innovations/enhanced-quarantine-capabilities-saas-security-api-dlp-policy` **[snippet]**, access 2026-06-10.

### 6. SaaS Security API Malware Detection
- **What it does:** Scans files at rest in connected SaaS tenants for malware (AV + sandbox), quarantines on hit.
- **Where it lives:** ZIA Admin Portal → Policy → SaaS Security API → Malware Detection.
- **Key knobs:**
  - Tenant scope
  - File-size / file-type filter
  - Action: Quarantine | Notify | Tombstone-replace
  - Sandbox tier (if Cloud Sandbox licensed)
- **Deployment-mode requirement:** API.
- **False-positive risk:** Low — AV match precision is high; sandbox verdict latency can delay file availability.
- **Source:** `https://help.zscaler.com/zia/configuring-saas-security-api-malware-detection-policy` **[snippet]**, access 2026-06-10.

### 7. External Sharing / Collaborator Governance (API)
- **What it does:** Finds files shared externally or via anonymous link in connected tenants and removes the share or specific collaborator domains.
- **Where it lives:** ZIA Admin Portal → Policy → SaaS Security API → DLP (file-sharing applications rule).
- **Key knobs:**
  - Tenant scope
  - Sharing scope filter: public-link | anyone-with-link | external | guest
  - Domain Profile: allow-list and deny-list of external collaborator domains
  - Action: Remove sharing | Remove collaborators by domain | Notify owner | Quarantine
- **Deployment-mode requirement:** API.
- **False-positive risk:** Medium — partner-collaboration domains that weren't on the allow-list get stripped; populate domain profiles before enforcing.
- **Source:** `https://www.zscaler.com/innovations/oob-casb-granularity-and-control-enhanced-control-domain-profiles-and-collaborator` **[snippet]**, `https://help.zscaler.com/zia/configuring-saas-security-api-dlp-policy-file-sharing-applications` **[snippet]**, access 2026-06-10.

### 8. AI & ML Application — Granular Activity Control
- **What it does:** Within the AI & ML category, permits the app but restricts which actions the user can perform.
- **Where it lives:** ZIA Admin Portal → Policy → Cloud App Control → AI & ML Applications rule.
- **Key knobs:**
  - App scope: specific AI app (ChatGPT, Copilot, Claude, Gemini, etc.) or all
  - Per-activity action (ChatGPT example): Chat allow / Upload block / Download block / Share block / Delete block / Invite block
  - Apply DLP profile to the prompt body (text classification)
  - Apply Browser Isolation on Allow
- **Deployment-mode requirement:** Inline + SSL inspection + (for Copilot-on-web) WebSocket inspection enabled.
- **False-positive risk:** Medium — prompt-DLP fires on benign code paste containing internal hostnames; tune dictionaries.
- **Source:** `https://help.zscaler.com/zia/adding-ai-ml-applications-rule-cloud-app-control` **[snippet]**, `https://www.zscaler.com/blogs/product-insights/zia-innovation-launch-part-2-websockets-workflows-full-stack-security-genai` **[snippet]**, access 2026-06-10.

### 9. Cloud Browser Isolation — Risky / Unsanctioned App Session
- **What it does:** Renders the SaaS app in a remote browser, transmits pixels only, disables clipboard/print/download.
- **Where it lives:** Selected as the action **ISOLATE** on a Cloud App Control rule, with a CBI profile referenced.
- **Key knobs:**
  - CBI profile: allow / block clipboard-in, clipboard-out, printing, file download, file upload, watermark on/off (with user identifier)
  - User-group scope
  - App scope: a category, a specific app, or "all unsanctioned apps with risk ≥ 4"
- **Deployment-mode requirement:** Inline (Client Connector or IdP-redirect for unmanaged).
- **False-positive risk:** Low to medium — UX degradation on graphics-heavy apps and on apps that depend on native browser features (drag-drop, certain WebRTC flows).
- **Source:** `https://help.zscaler.com/isolation` **[snippet]**, access 2026-06-10.

### 10. 3rd-Party App Governance — OAuth Grant Policy
- **What it does:** Blocks, quarantines, or notifies on new OAuth-app grants to M365 / Workspace / Salesforce / etc. based on publisher, scopes requested, risk score, and previous-grant behaviour.
- **Where it lives:** ZIA Admin Portal → 3rd-Party App Governance Admin Portal → Policies.
- **Key knobs:**
  - Connected tenant scope
  - Risk score threshold
  - Scope-sensitivity match (e.g. "any app requesting Mail.ReadWrite.All")
  - Publisher reputation tier
  - User-group scope
  - Action: Block grant | Revoke grant | Notify admin | Notify user | Auto-approve
- **Deployment-mode requirement:** API (against the connected tenant + IdP).
- **False-positive risk:** Medium — broad "block any app requesting these scopes" misfires on legitimate business apps; pilot in Notify mode.
- **Source:** `https://help.zscaler.com/zia/what-3rd-party-app-governance` **[snippet]**, `https://help.zscaler.com/zia/3rd-party-app-governance-policies` **[snippet]**, access 2026-06-10.

### 11. SSPM — SaaS Misconfiguration Policy
- **What it does:** Continuously assesses connected SaaS tenants against a posture-rule library (default tenant settings, sharing defaults, MFA enforcement, public-link defaults, guest-access controls, app-allow-list).
- **Where it lives:** ZIA Admin Portal → SaaS Security API → SSPM Policies.
- **Key knobs:**
  - Tenant scope (M365, Workspace, Salesforce, Slack, Atlassian, ServiceNow…)
  - Posture rule library: built-in + custom
  - Severity threshold
  - Auto-remediation: on / off (where the SaaS API supports write)
  - Compliance mapping (CIS, NIST 800-53 mappings) **[unverified for current rule set]**
- **Deployment-mode requirement:** API.
- **False-positive risk:** Low — these are configuration drift findings, not user actions; main risk is alert fatigue from low-severity findings.
- **Source:** `https://help.zscaler.com/zia/viewing-and-managing-supported-sspm-policies` **[snippet]**, `https://www.zscaler.com/products-and-solutions/saas-security` **[snippet]**, access 2026-06-10.

### Worked example — concept-shape configuration object

This is **conceptual** — the ZIA admin portal exposes these via web forms, and the Terraform / Pulumi providers (`zia.CloudAppControlRule`) use slightly different attribute names. Treat the keys below as practitioner-facing intent, not literal API syntax.

```jsonc
// Policy #8 — AI & ML granular activity control, applied to ChatGPT
{
  "policy_id": "cac_genai_chatgpt_dlp",
  "policy_type": "cloud_app_control",
  "category": "AI_AND_ML_APPLICATIONS",
  "rule_order": 50,
  "state": "ENABLED",

  "subject": {
    "user_groups": ["all_employees"],
    "exclude_user_groups": ["it_security_research"],
    "locations": "ANY",
    "device_trust_level": ["managed"]
  },

  "app_scope": {
    "apps": ["CHATGPT"],
    "tenant_profile_ref": null
  },

  "action": "ALLOW",

  "granular_activities": {
    "chat": "ALLOW",
    "upload": "BLOCK",
    "download": "ALLOW",
    "share": "BLOCK",
    "delete": "ALLOW",
    "invite": "BLOCK"
  },

  "content_inspection": {
    "ssl_inspection_required": true,
    "websocket_inspection": true,
    "dlp_engines": ["dlp_pii_my", "dlp_source_code_internal"],
    "on_dlp_match": "BLOCK",
    "user_notification": "notify_template_genai_block"
  },

  "isolation": {
    "fallback_to_cbi": false,
    "cbi_profile_ref": null
  },

  "logging": {
    "log_action": true,
    "alert_severity": "INFO"
  }
}
```

## Limitations / what this product does NOT cover

- **Real-time blocking of file uploads to SaaS apps that the user reaches without going through the Zscaler client** — because inline mode requires traffic steering; an unmanaged personal laptop hitting OneDrive directly is invisible to Zscaler. API mode catches the file post-upload, not pre-upload.
- **Pre-upload blocking for mobile SaaS apps that use certificate pinning** — because the Zscaler MITM certificate cannot be validated against the pinned cert and either the app refuses the connection or the practitioner adds the app to an SSL-inspection bypass list, which means no DLP on it. Vendor-documented limitation per the help portal.
- **Reverse-proxy mode with SAML rewrite for unmanaged-device access to sanctioned SaaS** — because Zscaler does not implement that pattern; the equivalent is agentless Cloud Browser Isolation triggered via IdP, which is a different UX (pixels-only, no native client) and not a 1:1 substitute for a Netskope/MDCA-style reverse-proxy session.
- **OAuth grants made directly to a SaaS app from an identity that is not in the connected IdP / tenant** — because 3rd-Party App Governance reads grants from the connected tenants only; consumer Google sign-in flows or personal Microsoft accounts that grant access to SaaS apps are invisible.
- **Native step-up MFA from a Zscaler policy decision** — because authentication lives at the IdP and Zscaler steers but does not authenticate; step-up requires an IdP integration that Zscaler signals to, not a Zscaler primitive.
- **In-tenant encryption / tokenisation of data before it lands in the SaaS provider** — because Zscaler does not market a gateway tokenisation product for SaaS storage; data at rest in OneDrive / Salesforce / Box stays encrypted by the SaaS provider, not by Zscaler. **[unverified — no positive evidence found; treat as "not a documented feature"]**
- **DLP coverage of end-to-end-encrypted SaaS chat (e.g. Signal-style, or any E2EE Teams/Slack-equivalent thread)** — because the proxy sees ciphertext; the API connector sees what the SaaS tenant exposes, which is also ciphertext for E2EE content.
- **Endpoint-resident data protection (file on disk, USB exfil)** — because ZIA is a network and SaaS plane; endpoint DLP needs a separate Zscaler Data Protection for Endpoint or third-party EDR/DLP. The Zscaler Client Connector is a steering agent, not an endpoint DLP.
- **Real-time API control of every SaaS in the catalogue** — because the API DLP / malware / SSPM connectors cover a defined list (M365, Workspace, Salesforce, Box, Dropbox, Slack, ServiceNow, Atlassian, GitHub, etc.); any sanctioned SaaS outside that list is inline-only.
- **Visibility into SaaS audit events that the SaaS does not expose via API** — because Zscaler can only consume what the vendor's API surfaces; some SaaS tenants gate audit-log APIs behind their own premium licence (M365 audit retention being the canonical example), in which case the practitioner's enforcement / forensic depth is bounded by the SaaS licence, not Zscaler.
- **Bring-your-own-cloud (private SaaS or self-hosted) coverage at API level** — because the API connectors target the public SaaS tenants Zscaler has built integrations for; an internal portal that "looks like" a SaaS still has to go through ZIA inline.
- **Shadow-IT discovery for users not steered through Zscaler** — because the discovery feed is ZIA's own logs; off-VPN, off-Client-Connector traffic does not appear. This is the structural difference from a CASB that ingests firewall/proxy logs from many sources.

## Sources

All sources accessed 2026-06-10.

- `https://help.zscaler.com/zia/about-saas-security` — **vendor-controlled** (page render returned empty body; relied on search snippets for content)
- `https://help.zscaler.com/zia/about-cloud-app-control` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/about-shadow-it-report` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/about-cloud-application-risk-profile` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/understanding-cloud-app-categories` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/adding-rules-cloud-app-control-policy` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/adding-ai-ml-applications-rule-cloud-app-control` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/policies/cloud-apps/tenant-restriction` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/about-tenant-profiles` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/policies/saas-security-api` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/configuring-saas-security-api-dlp-policy` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/about-saas-security-api-dlp` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/configuring-saas-security-api-dlp-policy-file-sharing-applications` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/configuring-saas-security-api-malware-detection-policy` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/about-saas-security-api-malware-detection` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/what-3rd-party-app-governance` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/3rd-party-app-governance-policies` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/connecting-your-platforms-3rd-party-app-governance` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/viewing-and-managing-supported-sspm-policies` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/about-security-ueba-alerts` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/certificate-pinning-and-ssltls-inspection` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/zia/about-ssl-inspection` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/isolation` — **vendor-controlled** (snippet-only)
- `https://help.zscaler.com/downloads/zia/reference-architecture/data-protection-secure-internet-and-saas-access/Data-Protection-with-Secure-Internet-and-SaaS-Access-ZIA-Reference-Architecture.pdf` — **vendor-controlled** (PDF reference architecture)
- `https://help.zscaler.com/downloads/zscaler-technology-partners/l-p/zscaler-and-microsoft-copilot-deployment-guide/Zscaler-Microsoft-Copilot-Deployment-Guide-FINAL.pdf` — **vendor-controlled** (deployment guide)
- `https://www.zscaler.com/products-and-solutions/saas-security` — **vendor-controlled** (marketing page)
- `https://www.zscaler.com/innovations/enhanced-quarantine-capabilities-saas-security-api-dlp-policy` — **vendor-controlled** (innovation announcement)
- `https://www.zscaler.com/innovations/enhanced-dlp-quarantine-capability-saas-security-api-file-sharing-apps` — **vendor-controlled**
- `https://www.zscaler.com/innovations/oob-casb-granularity-and-control-enhanced-control-domain-profiles-and-collaborator` — **vendor-controlled**
- `https://www.zscaler.com/innovations/apptotal-sso-integration-enhanced-risk-clarity-shadow-it-report` — **vendor-controlled**
- `https://www.zscaler.com/blogs/product-insights/zia-innovation-launch-part-2-websockets-workflows-full-stack-security-genai` — **vendor-controlled**
- `https://www.zscaler.com/blogs/product-insights/securing-gen-ai-and-microsoft-copilot-how-zscaler-data-protection-keeps` — **vendor-controlled**
- `https://www.zscaler.com/blogs/product-insights/zscaler-data-protection-tour-controlling-personal-apps-tenant-restrictions` — **vendor-controlled**
- `https://www.zscaler.com/products-and-solutions/cloud-access-security-broker-casb` — **vendor-controlled** (marketing)
- `https://www.pulumi.com/registry/packages/zia/api-docs/cloudappcontrolrule/` — **independent** (third-party IaC provider docs, confirms ALLOW/BLOCK/CAUTION/ISOLATE action set)
- `https://www.securityscientist.net/blog/12-questions-and-answers-about-zscaler-casb/` — **independent** (third-party practitioner write-up)

## Notes for the lens reviewers

- **Architect lens** — the structural difference vs a stand-alone CASB is that Inline CASB is not a separable decision layer; it is policy on top of the existing ZIA proxy. That means (a) every inline CASB capability requires the same SSL-inspection prerequisite and the same Client-Connector / tunnel steering as the rest of ZIA, (b) inspection-bypass lists added for ZIA stability automatically blind the CASB, (c) there is no "CASB-only" deployment option without the rest of ZIA. Worth pressure-testing whether this is a net positive (one chokepoint, one policy plane) or a net negative (one outage, all CASB enforcement gone) for the target customer.
- **Product lens** — the "Cloud App Control" + "SaaS Security API" + "3rd-Party App Governance" + "SSPM" + "CBI" naming is internally inconsistent. Practitioners will misnavigate the console. The schema I used groups by function, not by vendor menu. Reviewer should pressure-test whether to keep that or restructure to mirror Zscaler's own menu so the doc is searchable by what the customer sees on screen.
- **Compliance lens** — I have flagged unverified claims around SSPM-to-compliance-framework mapping, regional residency for SaaS API scanning, and the existence of any gateway tokenisation feature. These are the three items most likely to appear in an RFP "yes/no" matrix and most likely to be answered inaccurately by a non-specialist. Verify against the current price book and a current Zscaler SE before any compliance-driven recommendation.
- **General uncertainty:**
  1. The vendor help-portal HTML pages render via JavaScript and returned empty bodies to WebFetch. Every help.zscaler.com claim above is **search-snippet-grade**, not full-page-grade. Before promotion to `04-vendors/`, a reviewer with browser access should walk the help portal end-to-end and re-verify each "Where it lives" path.
  2. Licensing tier mapping for 2026 SKUs is the most volatile element. Zscaler restructured ZIA editions historically (Professional/Business/Transformation → Essentials/Business/Transformation → current). Treat the tier names in the Lineage section as 2024-vintage and re-confirm.
  3. The exact list of SaaS apps with API-mode DLP / Malware / SSPM coverage shifts quarterly. The list given is what is named in vendor partner pages and search snippets; not authoritative.
  4. I have **not** corroborated any of the limitations against a public customer incident, CVE, or independent deployment guide. They are mechanical inferences from how inline-proxy CASB architectures work; an independent practitioner write-up (Reddit, community.zscaler.com, partner blog) should be consulted before they go into a customer-facing artefact.
