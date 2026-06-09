# Microsoft Defender for Cloud Apps

> Practitioner reference draft. Vendor docs accessed 2026-06-10. Marketing language has been stripped; named features and exact portal paths are quoted from Microsoft Learn.

## Lineage

- **Current product name(s):**
  - **Microsoft Defender for Cloud Apps** (since 2021 rebrand) — the full CASB / SSPM product.
  - **Office 365 Cloud App Security (OCAS)** — a strictly subset SKU that only protects the Office 365 connector (no third-party SaaS connectors, no Conditional Access App Control beyond Office apps, no app governance, no cross-SaaS DLP). Source: [Differences between Defender for Cloud Apps and Office 365 Cloud App Security](https://learn.microsoft.com/en-us/defender-cloud-apps/editions-cloud-app-security-o365).
- **Renames / acquisitions:**
  - **Adallom** (Israeli CASB) acquired by Microsoft September 2015.
  - Rebranded **Microsoft Cloud App Security (MCAS)** ~2016.
  - Rebranded **Microsoft Defender for Cloud Apps** in 2021, folded under the **Microsoft Defender XDR** umbrella; management UI migrated from `portal.cloudappsecurity.com` to the **Microsoft Defender portal** at `security.microsoft.com` under the **Cloud apps** node. Portal references in 2026 docs uniformly say *"In the Microsoft Defender Portal, under Cloud apps..."* (e.g. [Control cloud apps with policies](https://learn.microsoft.com/en-us/defender-cloud-apps/control-cloud-apps-with-policies)).
  - **App governance** (originally a separate add-on for OAuth-app risk) is now integrated into Defender for Cloud Apps and surfaced on the **App governance** page in the Defender portal.
- **CASB housing:**
  - Stand-alone SaaS, but tightly coupled to the **Microsoft Defender XDR** suite (incident correlation with Defender for Endpoint, Defender for Identity, Defender for Office 365) and to **Microsoft Entra ID Conditional Access** (the proxy mode is literally Conditional Access App Control, an Entra feature).
  - Not inside an SSE platform — Microsoft's SSE story is **Global Secure Access** (Entra Internet Access / Entra Private Access). Defender for Cloud Apps is the SaaS-layer CASB that sits alongside it, with some discovery feature overlap (Shadow AI discovery now also surfaces in Global Secure Access).
- **Licensing tier(s) where CASB capabilities live:**
  - **Microsoft 365 E5** (or M365 E5 Security add-on, or EMS E5) — full Defender for Cloud Apps.
  - **Microsoft 365 E5 Compliance** — does not include it on its own.
  - **Office 365 E5** — includes only **Office 365 Cloud App Security** (the subset).
  - **Defender for Cloud Apps** stand-alone SKU (per-user/month) — available outside the M365 bundle.
  - **Microsoft Entra ID P1** is a separate prerequisite for using Conditional Access App Control (session/access policies). Source: [Create access policies](https://learn.microsoft.com/en-us/defender-cloud-apps/access-policy-aad), Prerequisites.

## Deployment modes supported

- **Forward-proxy:** Not implemented as a traditional forward proxy in the Zscaler/Netskope sense. Microsoft's equivalent is **Conditional Access App Control (CAAC)** — a *reverse-proxy-after-IdP-handoff* model. Traffic steering is done at the identity layer: Entra ID (or a configured non-Microsoft IdP) sees a sign-in to a sanctioned app, the Conditional Access policy applies the **Use Conditional Access App Control** session control, and the user's browser is redirected through Microsoft's CAAC proxy (`*.mcas.ms` / `*.mcas-gov.us` suffix on URLs; a lock icon for Edge users in the in-browser-protection preview). There is **no PAC file, no agent, no SSL-bump appliance** — traffic steering is purely IdP-driven, which means it works for **browser sessions** and the subset of mobile/desktop apps that authenticate via the IdP. Source: [Create session policies](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad).
- **Reverse-proxy:** CAAC is reverse-proxy. Sanctioned apps fall into two onboarding paths: **Automated Azure AD onboarding** (any Entra-integrated app) and **Manual onboarding** (non-Microsoft IdP apps — Okta, PingFederate, ADFS, etc., onboarded via [proxy-deployment-featured-idp](https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-deployment-featured-idp) or [proxy-deployment-any-app-idp](https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-deployment-any-app-idp)). The CAAC proxy supports per-app session controls including **block download**, **block upload**, **block cut/copy/paste/print**, **apply sensitivity label on download**, **require step-up auth** (preview), and **block specific in-app activities** (e.g. *Sent Teams message*).
- **API connectors:** Native API integrations for **Microsoft 365** (SharePoint, OneDrive, Exchange, Teams, Power BI, Dynamics 365), **Google Workspace** (Drive, Gmail), **Salesforce**, **Box**, **Dropbox**, **ServiceNow**, **Workday**, **Webex**, **GitHub Enterprise**, **Okta**, **AWS** (resource activity), **GCP** (resource activity), **Azure** (resource activity), **Atlassian** (Jira/Confluence — newer connector), and a smaller set of others. M365 is the only "first-class" connector in the strict sense (it gets the largest set of governance actions and is the only one inside Office 365 Cloud App Security). Source: [Protect your Microsoft 365 environment](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365).
- **Log-based discovery (Shadow IT):** Cloud Discovery analyses traffic logs against the catalogue of 31,000+ apps. Supported sources, per [Cloud app discovery overview](https://learn.microsoft.com/en-us/defender-cloud-apps/set-up-cloud-discovery):
  - **Native Microsoft Defender for Endpoint integration** — agent-based, off-corp-network discovery on Windows/macOS endpoints (the "Defender — managed endpoints" stream in the dashboard).
  - **Log collector (Docker container)** — receives Syslog/FTP from any of the supported appliances.
  - **Direct API integration with SWGs:** **Zscaler**, **iboss**, **Corrata**, **Menlo Security**.
  - **Manual upload** (snapshot reports) — useful for one-off risk assessments.
  - Supported firewalls/proxies parsed natively: Barracuda WAF, Blue Coat ProxySG, Check Point, Cisco ASA / ASA-FirePOWER / FWSM / Cloud Web Security / IronPort WSA / Meraki, Clavister NGFW, ContentKeeper, Digital Arts i-FILTER, Forcepoint (LEEF and Web Security Cloud), Fortinet Fortigate / FortiOS, Juniper SRX / SSG, McAfee SWG, Menlo Security, MS Forefront TMG, Open Systems SWG, Palo Alto Networks, SonicWall, Sophos SG/XG/Cyberoam, Squid, Stormshield, Wandera, WatchGuard, Websense, Zscaler. Not all sources carry username (e.g. Check Point, Cisco ASA syslog, Cisco FWSM, Juniper SRX, Cisco Meraki, SonicWall do not).
- **Hybrid considerations:**
  - For on-prem custom apps you want CAAC to protect, you must front them with Entra ID (Application Proxy or SAML federation) and onboard them via the "any app" flow.
  - File monitoring must be explicitly enabled at **Settings → Cloud Apps → Files → Enable file monitoring**, and at least one file policy must exist or file monitoring auto-disables after seven days. Source: [File policies](https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies) — best practice #6.
  - The **Microsoft Defender for Cloud Apps – Session Controls** enterprise application in Entra must not be blocked by a generic "Block Access" Conditional Access policy, or all CAAC-protected sessions break. This is a real configuration trap and is called out explicitly in the docs.

## Capability set

### Shadow-IT discovery (network-log-based AND API-based) — **Supported**

Network-log-based discovery is mature: 31,000+ app catalogue with 90+ risk indicators, four-times-daily refresh, executive-summary PDF report, IaaS/PaaS resource discovery (Azure / AWS / GCP storage accounts, infra), and a Defender-for-Endpoint integration that extends discovery to roaming endpoints. The dashboard at **Cloud apps → Cloud discovery** shows top users, top source IPs, app HQ map, risk-score distribution. Source: [View discovered apps on the Cloud discovery dashboard](https://learn.microsoft.com/en-us/defender-cloud-apps/discovered-apps), accessed 2026-06-10. **Caveat:** the **discovered subdomains** feature is being deprecated 2025-12-31 per the same doc — practitioners relying on subdomain-level differentiation (e.g. distinguishing SharePoint tenants) will lose that fidelity.

### Sanctioned-app inventory + risk-scoring — **Supported**

The cloud app catalogue carries a risk score (0–10) per app derived from 90+ factors across General, Security, Compliance, Legal categories. Apps can be tagged **Sanctioned**, **Unsanctioned**, **Monitored**, or **Custom-tagged**. **Caveat:** the risk score is Microsoft's opinion, not an industry standard; weights are not user-tunable. Source: [Cloud app catalog and risk scores](https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score).

### Inline DLP for sanctioned SaaS (proxy mode — pre-upload prevention) — **Supported with caveats**

Available **only via CAAC session policies** with **Control file upload (with inspection)** session control type. Inspection methods include Microsoft Purview's **Data Classification Service** (sensitive info types like Credit Card, SSN, custom SIT), regex, fingerprints, trainable classifiers. Action set: **Block**, **Audit**, **Protect** (apply sensitivity label / custom permissions / step-up auth). **Caveat:** because CAAC is IdP-mediated reverse-proxy, this only works for browser sessions (and a subset of native apps that route through Entra). Native apps with their own auth flows bypass it. Source: [Create session policies](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad).

### API-mode DLP for sanctioned SaaS (post-upload scan + remediate) — **Supported**

The **File policy** type provides post-facto scanning against 20+ metadata filters and Microsoft Purview Data Classification Service for content inspection. Limit: **50 file policies per tenant**. Governance actions per app (M365: trash, quarantine, make private, remove external collaborators, remove specific collaborator, inherit parent folder permissions, apply/remove Purview sensitivity label; Google Workspace, Box, Dropbox: similar but app-specific subsets). **Caveat:** "Only the governance action of the first triggered policy is guaranteed to be applied" (per the docs) — policy ordering and overlap matter. Source: [File policies](https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies), accessed 2026-06-10.

### OAuth-app discovery and governance — **Supported**

The **OAuth apps** page (or the **App governance** page if app governance is enabled) lists every third-party OAuth app that has been granted **Delegated permissions** to Microsoft 365, Google Workspace, or Salesforce. Each app carries: permissions level (High/Medium/Low), authorized-by count, community use (common/uncommon/rare), publisher (with publisher-verification flag for M365), app state (approved / banned / undetermined). Admins can **Approve**, **Ban** (with optional user notification email), **Report app to Microsoft**, or **Revoke** permissions. Built-in OAuth anomaly detection policies fire on: **Misleading OAuth app name**, **Misleading publisher name for an OAuth app**, **Malicious OAuth app consent**. **Caveat:** only *Delegated* permissions are surfaced — Application permissions (app-only / client-credentials) are not in the OAuth apps view; you need Defender for Cloud / app governance for those. Sources: [Manage OAuth apps](https://learn.microsoft.com/en-us/defender-cloud-apps/manage-app-permissions), [Protect Microsoft 365](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365).

### Anomalous-activity / UEBA detection — **Supported**

Out-of-the-box UEBA via the **Anomaly detection policy** category — automatically enabled with a 7-day learning window. As of **June 2025**, Microsoft migrated several legacy anomaly detection policies (Activity from suspicious IPs, Suspicious inbox manipulation rules, Suspicious email deletion, Activity from anonymous IPs, Suspicious inbox forwarding, Unusual ISP for OAuth, Suspicious file access activity, Ransomware activity) to a **"dynamic threat detection model"** — the legacy policies were disabled in the portal and replaced with renamed dynamic detections (e.g. *Successful logon from a suspicious IP address*, *Ransomware payment instruction file uploaded to {Application}*). Practitioners who configured custom governance actions on the legacy policies must re-enable them on the new versions. Source: [Create anomaly detection policies](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy).

### Compromised-account detection (impossible travel, brute-force, anomalous-OAuth) — **Supported**

- **Impossible travel** — ML-based, ignores VPNs and tenant-common locations; tunable sensitivity slider (Low/Medium/High). Country-level only, no alerts within the same country or bordering countries.
- **Multiple failed login attempts**.
- **Activity from infrequent country**.
- **Activity performed by terminated user** (requires Entra as IdP — detects post-termination activity on other connected apps like AWS/Salesforce where the user's secondary account wasn't deleted).
- **Suspicious OAuth app file download activities**.
- **OAuth application activity from an unknown ISP** (replaces *Unusual ISP for an OAuth App*).
- **Malicious OAuth app consent**.

Source: [Create anomaly detection policies](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy).

### Data-residency / geo-policy enforcement — **Supported with caveats**

Geo-fencing is implemented through the **Location** filter on access/session/activity/anomaly policies (country-level granularity). **Defender for Cloud Apps tenants are themselves hosted in EU, US, UK, or AU datacentres** chosen at provisioning — primary-data location is fixed at tenant creation and not movable. **Caveat:** the docs are explicit that location accuracy is country-level only, and "absence of a clearly defined location may identify risky activities" — so policy logic on Location often needs an `is set` companion filter to avoid catching null-location events unintentionally.

### Share-link governance (external sharing, anonymous links, expiry) — **Supported**

Through **File policy** with **Access level = Public / Public with a link / External / Shared**. Governance actions per-app for sharing: **Remove external collaborators on file/folder**, **Remove a specific collaborator**, **Make file/folder private**, **Inherit parent folder permissions**, **Trash file/folder**. **Caveat:** for OneDrive/SharePoint, files publicly shared with 'anyone' "might incorrectly show up as private" per the M365 connector doc — known fidelity gap. Source: [Protect Microsoft 365](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365).

### Malware scanning in SaaS storage — **Supported**

Built-in **Malware detection policy** uses Microsoft Threat Intelligence. **File Sandboxing** for Box, Dropbox, Google Workspace; OneDrive/SharePoint files are scanned by the service itself, not Defender for Cloud Apps. **Caveat:** Defender for Cloud Apps "doesn't automatically block the file" in Box/Dropbox/Google — remediation depends on the third-party app's own quarantine. Source: [Create anomaly detection policies — Malware detection](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy).

### Encryption / tokenisation at gateway — **Not supported**

Defender for Cloud Apps does not provide gateway-level encryption or tokenisation of fields written to SaaS. The product's encryption story is **Microsoft Purview sensitivity labels** (MIP-AIP) applied to files post-classification — encryption sits in the file, not at the proxy. Practitioners who want field-level tokenisation must look at separate products (CipherCloud-style vendors).

### Risk-based session policy (step-up auth, read-only, watermark, block) — **Supported with caveats**

CAAC session policies support: **Audit**, **Block**, **Protect (apply sensitivity label + permissions)**, **Require step-up authentication** (preview — redirects mid-session to Entra Conditional Access for re-evaluation, e.g. enforce MFA on the download action specifically). Block specific in-app activities: print, copy, paste, share, send Teams message, etc. **Watermarking is not native** — the closest equivalent is applying a Purview sensitivity label that itself carries visual markings, but the docs note "When a document is labeled by Defender for Cloud Apps, visual markings, such as headers, footers, or watermarks, are not applied." Source: [Create session policies](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad).

### GenAI-app discovery + governance — **Supported (2024-2026 additions)**

A **Generative AI** category was added to the cloud app catalogue covering 1,000+ AI apps (Bing Chat, Google Bard / Gemini, ChatGPT, Claude, Copilot, Perplexity, Anthropic API endpoints, HuggingFace, and many more). Practitioners can: filter discovery by Generative AI category, view per-app risk score, **sanction/unsanction** specific apps (unsanctioned apps are auto-blocked at Defender-for-Endpoint-managed devices), set policies that auto-unsanction by threshold (risk score, user count, transactions), and combine with **Microsoft Purview Data Security Posture Management (DSPM) for AI** to see prompts/responses with sensitive data submitted to ChatGPT Enterprise, Gemini, and other supported sites. **Caveat:** DSPM-for-AI prompt visibility requires Purview-onboarded devices (Defender for Endpoint), not just the cloud discovery feed — and is limited to the [Supported AI sites](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview-supported-sites) list. Sources: [How do I discover AI apps and the sensitive data these use in my organization?](https://learn.microsoft.com/en-us/security/security-for-ai/discover) (accessed 2026-06-10); category rollout was announced mid-2023 and the catalogue has been expanded continuously through 2024-2026.

## Configurable policies

Policies live in **Microsoft Defender Portal → Cloud apps → Policies → Policy management**, organised into tabs: **All policies**, **Threat detection**, **Information protection**, **Conditional Access**, **Shadow IT**. Templates live under **Policies → Policy templates**.

### 1. Block download of sensitive files to unmanaged devices (M365 + browser)

- **What it does:** When a user signs in to SharePoint/OneDrive/Exchange from an unmanaged browser, allow read/edit in-session but block download of any file matching a Purview sensitivity label (e.g. *Confidential* or *Highly Confidential*).
- **Where it lives:** **Cloud apps → Policies → Policy management → Conditional Access tab → Create policy → Session policy**. Plus a matching **Entra Conditional Access policy** with **Use Conditional Access App Control** under **Session** controls.
- **Key knobs:**
  - **Session control type:** *Control file download (with inspection)*.
  - **App filter:** Microsoft 365 (Automated Azure AD onboarding).
  - **Device filter:** *Device tag* ≠ *Microsoft Entra hybrid joined* AND *Device tag* ≠ *Intune compliant*.
  - **File filter:** *Sensitivity label* ∈ {Confidential, Highly Confidential}.
  - **Inspection method:** Data Classification Service (optional, as a belt-and-braces for unlabelled files containing SSN/PII).
  - **Action:** *Block* + customised end-user message ("Download blocked — file is Confidential; sign in from a managed device").
- **Deployment-mode requirement:** Proxy (CAAC). Plus Entra ID P1.
- **False-positive risk:** **Medium** — a finance user on a BYO laptop urgently needing a quarterly report finds themselves blocked. Mitigate by scoping the policy to specific user groups first and rolling out gradually.
- **Source:** [Create session policies](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad), accessed 2026-06-10.

### 2. Auto-label files in OneDrive/SharePoint containing PCI data with *Confidential — Finance*

- **What it does:** Continuously scan all files in OneDrive/SharePoint, detect Credit Card Number content via Purview Data Classification Service, and apply the *Confidential — Finance* sensitivity label (which itself carries encryption per Purview label config).
- **Where it lives:** **Cloud apps → Policies → Policy management → Information protection tab → Create policy → File policy**.
- **Key knobs:**
  - **App filter:** OneDrive for Business, SharePoint Online.
  - **File filter:** *Parent folder* equals selected finance folders (narrow filter to limit scan cost).
  - **Inspection method:** *Data Classification Service*; *Sensitive information type* = *Credit Card Number*; minimum violations = 1.
  - **Governance action (OneDrive/SharePoint):** *Apply Microsoft Purview Information Protection sensitivity label* → *Confidential — Finance*.
  - **Alert:** Create alert per match; email to DLP queue.
- **Deployment-mode requirement:** API (M365 connector).
- **False-positive risk:** **Medium** — DCS sometimes hits on test files, mock data, expired/cancelled cards. The 100-character-before/after context preview helps triage, but the label is applied automatically. Run for a week with Audit/alert before flipping on the label action.
- **Source:** [Automatically apply sensitivity labels from Microsoft Purview Information Protection](https://learn.microsoft.com/en-us/defender-cloud-apps/use-case-information-protection), accessed 2026-06-10.

### 3. Block upload of unlabelled files containing PII to any SaaS app

- **What it does:** When a user attempts to upload a file containing PII (SSN/Passport/etc.) to any CAAC-protected app, if the file does not already carry a Purview *Confidential* or higher label, block the upload and instruct the user how to label.
- **Where it lives:** **Cloud apps → Policies → Policy management → Conditional Access tab → Create policy → Session policy**.
- **Key knobs:**
  - **Session control type:** *Control file upload (with inspection)*.
  - **App filter:** all apps marked Enabled on the *Conditional Access App Control apps* page.
  - **File filter:** *Sensitivity label* ∉ {Confidential, Highly Confidential}.
  - **Inspection method:** Data Classification Service; SIT = U.S. SSN, EU passport, etc.
  - **Action:** *Block* with custom user message ("Apply the Confidential label before upload — instructions: [link]").
- **Deployment-mode requirement:** Proxy (CAAC).
- **False-positive risk:** **High** — DCS regex on SSN matches many 9-digit strings; uploads of legitimate spreadsheets with employee IDs or order numbers can match. Always include `Always apply the selected action even if the data cannot be scanned` unchecked, and pilot on a small group.
- **Source:** [Create session policies — Protect uploads of sensitive files](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad), accessed 2026-06-10.

### 4. Mass-download anomaly (suspect insider exfil or compromised account)

- **What it does:** Alert and suspend user when a single user downloads >X files within Y minutes from any connected SaaS app.
- **Where it lives:** **Cloud apps → Policies → Policy management → Threat detections tab → Create policy → Activity policy**.
- **Key knobs:**
  - **Activity filter:** *Activity type* = *Download file*; *App* = SharePoint, OneDrive, Box, Google Drive (or all).
  - **Match parameters:** *Repeated activity* — count > 1,000 in 30 minutes, in a single app, by same user.
  - **Governance action (M365):** *Suspend user (via Microsoft Entra ID)*; *Notify user on alert*.
  - **Alert:** Severity High; email to SOC; send to Power Automate for ticketing.
- **Deployment-mode requirement:** API.
- **False-positive risk:** **Medium** — legitimate bulk downloads (DR rehearsals, content-migration runs, BI export jobs by service accounts). Exclude service-account UPNs explicitly under *User filter*.
- **Source:** [Create activity policies](https://learn.microsoft.com/en-us/defender-cloud-apps/user-activity-policies), accessed 2026-06-10.

### 5. Block sign-in from non-corporate, non-compliant device

- **What it does:** Deny browser access to SharePoint/Exchange/Teams unless the device is Entra-hybrid-joined or Intune compliant.
- **Where it lives:** **Cloud apps → Policies → Policy management → Conditional Access tab → Create policy → Access policy** (plus an enabling Entra Conditional Access policy with *Use Conditional Access App Control*).
- **Key knobs:**
  - **App filter:** Microsoft 365.
  - **Client app filter:** Browser (omit *Mobile and desktop* to avoid breaking Outlook/OneDrive sync clients).
  - **Device filter:** *Device tag* ≠ *Microsoft Entra hybrid joined* AND *Device tag* ≠ *Intune compliant*.
  - **Action:** *Block*.
- **Deployment-mode requirement:** Proxy (CAAC).
- **False-positive risk:** **High** — breaks BYOD entirely. Combine with a parallel **session policy** for unmanaged devices that grants read-only access rather than full block, or pair with a client-certificate-based access policy for trusted-but-unmanaged-by-Intune devices. Source: [Create access policies — Create access policies for identity-managed devices](https://learn.microsoft.com/en-us/defender-cloud-apps/access-policy-aad), accessed 2026-06-10.

### 6. Detect and revoke risky OAuth grants on M365

- **What it does:** Built-in OAuth anomaly detection — fires on *Misleading OAuth app name*, *Misleading publisher name for an OAuth app*, *Malicious OAuth app consent*. Practitioner-configurable governance: auto-revoke or alert + manual ban.
- **Where it lives:** **Cloud apps → OAuth apps** (or **App governance** if enabled). Anomaly policies under **Cloud apps → Policies → Policy management → Threat detections** (these are built-in and cannot be created — only edited/scoped).
- **Key knobs (for editing the built-in policy):**
  - **Scope:** specific users/groups to include/exclude.
  - **Governance action:** *Revoke OAuth app permission*.
  - **Alert** + Power Automate webhook.
  - **Per-app manual action:** *Ban app* from **OAuth apps** page with optional user notification.
- **Deployment-mode requirement:** API.
- **False-positive risk:** **Low** — Microsoft's catalogue+publisher-verification filter is reasonably tight. *Misleading publisher* fires on lookalike-publisher names (an "Microsoft" vs "Microsoft Corporation" mismatch).
- **Source:** [Manage OAuth apps](https://learn.microsoft.com/en-us/defender-cloud-apps/manage-app-permissions), [Protect Microsoft 365](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365), accessed 2026-06-10.

### 7. Auto-unsanction Generative AI apps above risk threshold

- **What it does:** Continuously discover GenAI apps from the cloud discovery stream; when an app in the Generative AI category exceeds a risk score / user-count threshold, automatically tag it as Unsanctioned, which propagates to Defender for Endpoint and blocks it on managed devices.
- **Where it lives:** **Cloud apps → Policies → Policy management → Shadow IT tab → Create policy → App discovery policy**.
- **Key knobs:**
  - **Filter:** *Category* = Generative AI; *Risk score* ≤ 5 (i.e. high-risk); *Users per day* > 10.
  - **Action:** *Tag as Unsanctioned* (auto-blocks on MDE-managed endpoints if MDE integration is on); *Alert*; *Generate block script* for SWG.
- **Deployment-mode requirement:** Log-based discovery (Defender for Endpoint integration recommended for endpoint enforcement).
- **False-positive risk:** **Medium** — risk scores include compliance factors (e.g. *SOC 2 = No*) that may flag apps the business has approved a side-channel exception for. Allowlist by app ID for known-approved exceptions.
- **Source:** [How do I discover AI apps...](https://learn.microsoft.com/en-us/security/security-for-ai/discover); [Govern discovered apps](https://learn.microsoft.com/en-us/defender-cloud-apps/governance-discovery), accessed 2026-06-10.

### 8. Block cut/copy/paste of corporate data into ChatGPT (browser session)

- **What it does:** When a user navigates to chat.openai.com (onboarded as a non-Microsoft IdP custom app), allow the session but block clipboard *Paste* of more than N characters, block Sent message activity when content matches a DLP signature, and audit all in-session activity.
- **Where it lives:** **Cloud apps → Policies → Policy management → Conditional Access tab → Create policy → Session policy**.
- **Key knobs:**
  - **Session control type:** *Block activities*.
  - **App filter:** Custom OpenAI ChatGPT (Manual onboarding).
  - **Activity type:** Clipboard *Cut*, *Copy*, *Paste*; Sent message.
  - **Use content inspection:** Data Classification Service; SIT = Credit Card, SSN, PII; custom SIT for internal project codenames.
  - **Action:** *Block* + customised user-education message.
- **Deployment-mode requirement:** Proxy (CAAC). Plus prior IdP onboarding for ChatGPT (via SAML federation to Entra or via a non-Microsoft IdP).
- **False-positive risk:** **High** — content inspection on copy/paste actions is noisy; benign code snippets and 16-digit identifiers trigger Credit Card SIT. Start with *Audit* action, then escalate to *Block* per SIT once tuned.
- **Source:** [Create session policies — Block specific activities](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad), accessed 2026-06-10. **Practitioner-named** — Microsoft does not ship a built-in template for this; it's a worked composition.

### 9. Impossible-travel alert with auto-suspend on confirmed compromise

- **What it does:** Detect impossible-travel sign-in pattern (two activities geographically too distant in too short a time), alert the SOC, and on confirmed-true-positive auto-suspend the Entra user.
- **Where it lives:** **Cloud apps → Policies → Policy management → Threat detections tab → Anomaly detection policy → Impossible travel** (built-in; edit, do not create).
- **Key knobs:**
  - **Sensitivity slider:** Low / Medium / High (Low suppresses common-tenant locations and user-common locations; Medium suppresses only system and user; High suppresses only system).
  - **Scope:** include All users; exclude *Frequent travellers* group (an explicitly managed exclusion).
  - **Governance action (M365):** *Require user to sign in again* + *Confirm user compromised* + *Suspend user*.
  - **Alerts:** email + Power Automate for ticket creation.
- **Deployment-mode requirement:** API. Requires Entra ID as IdP for the governance actions.
- **False-positive risk:** **Medium** — VPNs, cloud-provider IPs (no physical location), and frequent business travel all known to trigger. Use Low/Medium sensitivity to start, manage exclusions for known business travellers.
- **Source:** [Create anomaly detection policies — Impossible travel](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy), accessed 2026-06-10.

### 10. Quarantine externally-shared files in OneDrive with no recent activity

- **What it does:** Identify files in OneDrive shared externally that have not been modified in 6+ months — quarantine to admin-owned location pending owner confirmation.
- **Where it lives:** **Cloud apps → Policies → Policy management → Information protection tab → Create policy → File policy**.
- **Key knobs:**
  - **App filter:** OneDrive for Business.
  - **Access level:** *External* OR *Public with a link* OR *Public*.
  - **Last modified:** more than 180 days ago.
  - **Governance action (OneDrive):** *Put file/folder in admin quarantine*.
  - **Alert + notify owner.**
- **Deployment-mode requirement:** API.
- **False-positive risk:** **Medium** — old contract templates, reference docs intentionally left shared. Mitigate with a folder allowlist (`Parent folder ≠ /Shared Templates`).
- **Source:** [File policies — example policies](https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies), accessed 2026-06-10.

### 11. Malware on upload/download to Box / Dropbox / Google Workspace

- **What it does:** Detonate files in sandbox on upload or download for third-party storage apps; alert on hit, do not auto-block (third-party apps don't expose a clean block hook).
- **Where it lives:** **Cloud apps → Policies → Policy management → Threat detections tab → Malware detection policy** (built-in; edit, do not create). For inline blocking, pair with a **Session policy** using *Control file upload/download (with inspection)* → *Malware detection* inspection.
- **Key knobs:**
  - **File Sandboxing:** enabled (per-app).
  - **Apps:** Box, Dropbox, Google Workspace.
  - **Governance action:** alert + *Trash file/folder* (where supported).
  - **Session policy companion:** *Control file download (with inspection)* → Inspection method *Malware detection* → Action *Block*.
- **Deployment-mode requirement:** API for the scanner. Proxy (CAAC) if you want inline block at upload/download.
- **False-positive risk:** **Low** — Microsoft TI feed is generally reliable; sandbox detonation adds latency but high signal.
- **Source:** [Create anomaly detection policies — Malware detection / File Sandboxing](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy), accessed 2026-06-10.

### 12. Terminated user still active in connected SaaS (post-offboarding gap)

- **What it does:** Detects when an Entra-deleted account still performs activity in another connected SaaS app (AWS, Salesforce) — a common offboarding gap when a user had a secondary account separate from the SSO identity.
- **Where it lives:** **Cloud apps → Policies → Policy management → Threat detections tab → Anomaly detection policy → Activity performed by terminated user** (built-in; edit only).
- **Key knobs:**
  - **Scope:** all users.
  - **Governance action (per app):** for AWS/GCP/Salesforce — Notify admin; manual revoke (no auto-suspend for non-Entra accounts).
  - **Severity:** High.
- **Deployment-mode requirement:** API. Requires Entra as IdP.
- **False-positive risk:** **Low** — Microsoft's signal is reliable on deleted-Entra-user-active-elsewhere. False positives mainly when the user has a legitimate guest-account in the second app.
- **Source:** [Anomaly detection policies — Activity performed by terminated user](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy), accessed 2026-06-10.

### Worked example — Policy #1 expressed as a property list

```yaml
# Block download of sensitive files to unmanaged devices (M365 browser)
# Lives in: Cloud apps → Policies → Policy management → Conditional Access tab
# Requires Entra CA policy with "Use Conditional Access App Control" session control
policy:
  name: "Block download of Confidential files to unmanaged browser (M365)"
  type: SessionPolicy
  severity: High
  category: DLP
  description: |
    Browser sessions to M365 from non-managed devices: allow access and
    in-session view/edit, block file download of any file labelled
    Confidential or Highly Confidential.
  session_control_type: ControlFileDownloadWithInspection
  activity_filters:
    app:
      onboarding: AutomatedAzureADOnboarding
      apps: [Office365]   # covers SharePoint, OneDrive, Exchange Online, Teams (host app)
    client_app: Browser
    device:
      device_tag_not_in: [MicrosoftEntraHybridJoined, IntuneCompliant]
    user:
      include_groups: [All employees]
      exclude_groups: [SOC break-glass]
  file_filters:
    sensitivity_label_in: ["Confidential", "Highly Confidential"]   # Purview MIP labels
  inspection:
    # Optional belt-and-braces for unlabelled-but-sensitive content
    method: DataClassificationService
    sensitive_information_types: [Credit Card Number, U.S. Social Security Number (SSN)]
    minimum_violations: 1
  action:
    type: Block
    notify_user_email: true
    custom_block_message: |
      Download blocked: this file is classified Confidential. Sign in from
      a managed device, or contact IT to enrol your device. [link]
    always_apply_if_unscannable: false
  alerts:
    per_match_alert: true
    severity: High
    email_recipients: [soc-dlp@example.com]
    power_automate_flow: "dlp-ticket-create"
    daily_alert_limit: 200
```

## Limitations / what this product does NOT cover

- **Real-time blocking of file uploads to non-onboarded SaaS apps** — because CAAC is reverse-proxy-after-IdP and only sees traffic for apps onboarded to Conditional Access App Control. Any app not behind Entra (or a configured third-party IdP) is invisible to session policies; only post-upload API-mode DLP (limited to the small set of API-connected apps) can act on it.
- **True forward-proxy with PAC/agent traffic steering** — there is no Defender-for-Cloud-Apps PAC file and no Defender-for-Cloud-Apps endpoint agent for traffic interception (Defender for Endpoint feeds discovery data but does *not* route traffic through Defender for Cloud Apps). Roaming-user inline DLP requires another vehicle (Entra Internet Access / Global Secure Access, or a third-party SWG).
- **CAAC session controls on mobile and native desktop apps with their own auth flows** — only browser sessions and the apps that authenticate through the IdP are protected. Outlook Mobile, Teams Desktop, OneDrive sync client and most mobile SDK-based apps bypass the proxy. The Access policy *Client app* filter notes "**unless the Client app filter is specifically set to Mobile and desktop, the resulting access policy will only apply to browser sessions**" — but even mobile/desktop coverage depends on the app's auth flow.
- **OAuth grants to apps not federated through the connected IdP** — only delegated grants made through Entra (or Google Workspace / Salesforce) appear in the OAuth apps page. A user signing into ChatGPT with a personal Gmail account and granting permissions to ChatGPT-plugins is invisible.
- **Application-only (client-credentials) OAuth grants** — only *Delegated* permissions are surfaced. Service-principal / app-only consents granted by admins are not in the OAuth apps view; you need app governance or Defender for Cloud / Defender XDR for those.
- **Watermarking on downloaded files** — the *Protect* action applies a Purview sensitivity label, but the docs are explicit: "When a document is labeled by Defender for Cloud Apps, visual markings, such as headers, footers, or watermarks, are not applied." You only get the encryption / permissions enforcement, not visible markings.
- **Sensitivity labels on file types other than Office and PDF in session-mode protection** — *Protect* on download only supports docx/docm/dotm/dotx/xlsm/xlsx/xltx/xlam/pptx/pptm/potx/potm/ppsx/ppsm and unified-label PDF. Other file types (text, image, archives) cannot be labelled at download.
- **Externally-owned files in Google Drive / Box / Dropbox** — "Defender for Cloud Apps doesn't have access to these files" per the docs. Files placed into your Drive/Box/Dropbox by external users (where the external user is still the file owner under that platform's data model) are unscannable. Only OneDrive reassigns ownership and so makes external-uploaded files scannable.
- **Files larger than the scanner's content-inspection limit, password-protected files, Azure-RMS-encrypted files** — content cannot be inspected; the file is tagged with the special **Defender for Cloud Apps** labels (`Password encrypted`, `Azure RMS encrypted`, `Corrupt file`) but DLP regex/SIT cannot fire on the content.
- **More than 50 file policies per tenant** — hard limit per [File policies — Limitations](https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies). Practitioners must consolidate policies across apps; "Consolidate several file policies for the same service" is explicit best practice.
- **Sub-domain-level Shadow IT differentiation after 2025-12-31** — the discovered subdomains feature is deprecated. Differentiating SharePoint Online tenant URLs or distinguishing app variants by subdomain will no longer work.
- **Activity policies that exceed 200,000 matches/day or 100,000/3 hours** — auto-disabled. Practitioners must keep filters tight; broad activity policies for reporting purposes are better implemented as saved queries.
- **Power BI / Dynamics 365 log delay** — 24-72 hours for events to appear in Defender for Cloud Apps after auditing is enabled; no real-time signal for these workloads.
- **Multi-geo M365 deployments** — supported "only for OneDrive" per the M365 connector doc. SharePoint multi-geo events aren't fully captured.
- **Inline blocking of malware in Box, Dropbox, Google Workspace** — the malware policy detects but doesn't auto-block; remediation depends on the third-party app's own capabilities. Only OneDrive/SharePoint files are scanned by Microsoft's own service before they land.

## Sources

- **vendor-controlled** — [Overview - Microsoft Defender for Cloud Apps](https://learn.microsoft.com/en-us/defender-cloud-apps/what-is-defender-for-cloud-apps) — accessed 2026-06-10.
- **vendor-controlled** — [Control cloud apps with policies](https://learn.microsoft.com/en-us/defender-cloud-apps/control-cloud-apps-with-policies) — accessed 2026-06-10.
- **vendor-controlled** — [Protect your Microsoft 365 environment](https://learn.microsoft.com/en-us/defender-cloud-apps/protect-office-365) — accessed 2026-06-10.
- **vendor-controlled** — [Automatically apply sensitivity labels from Microsoft Purview Information Protection](https://learn.microsoft.com/en-us/defender-cloud-apps/use-case-information-protection) — accessed 2026-06-10.
- **vendor-controlled** — [Manage OAuth apps](https://learn.microsoft.com/en-us/defender-cloud-apps/manage-app-permissions) — accessed 2026-06-10.
- **vendor-controlled** — [View discovered apps on the Cloud discovery dashboard](https://learn.microsoft.com/en-us/defender-cloud-apps/discovered-apps) — accessed 2026-06-10.
- **vendor-controlled** — [Create session policies](https://learn.microsoft.com/en-us/defender-cloud-apps/session-policy-aad) — accessed 2026-06-10.
- **vendor-controlled** — [Create access policies](https://learn.microsoft.com/en-us/defender-cloud-apps/access-policy-aad) — accessed 2026-06-10.
- **vendor-controlled** — [Create anomaly detection policies](https://learn.microsoft.com/en-us/defender-cloud-apps/anomaly-detection-policy) — accessed 2026-06-10.
- **vendor-controlled** — [File policies](https://learn.microsoft.com/en-us/defender-cloud-apps/data-protection-policies) — accessed 2026-06-10.
- **vendor-controlled** — [Create activity policies](https://learn.microsoft.com/en-us/defender-cloud-apps/user-activity-policies) — accessed 2026-06-10.
- **vendor-controlled** — [Cloud app discovery overview](https://learn.microsoft.com/en-us/defender-cloud-apps/set-up-cloud-discovery) — accessed 2026-06-10.
- **vendor-controlled** — [Differences between Defender for Cloud Apps and Office 365 Cloud App Security](https://learn.microsoft.com/en-us/defender-cloud-apps/editions-cloud-app-security-o365) — accessed 2026-06-10.
- **vendor-controlled** — [How do I discover AI apps and the sensitive data these use in my organization?](https://learn.microsoft.com/en-us/security/security-for-ai/discover) — accessed 2026-06-10.
- **vendor-controlled (cross-product)** — [Microsoft Purview data security and compliance protections for Microsoft 365 Copilot and other generative AI apps](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview) — accessed 2026-06-10 (light reference, used for DSPM for AI context).
- **independent corroboration:** none added in this draft — see *Notes for reviewers* below for flagged claims that would benefit from third-party corroboration.

## Notes for the lens reviewers

- **June 2025 anomaly-policy migration** — the docs are explicit that 8 legacy anomaly policies were disabled and migrated to a "dynamic threat detection model" with renamed detections. I have NOT verified operationally that the new dynamic detections fire at the same fidelity as the old ones. Practitioners with prior MCAS deployments should diff their custom governance actions; a reviewer with hands-on access should validate that the new policies behave as documented.
- **GenAI catalogue size claim** — Microsoft says "more than a thousand generative AI-related apps". Catalogue count drift between docs (31,000 total apps in [set-up-cloud-discovery](https://learn.microsoft.com/en-us/defender-cloud-apps/set-up-cloud-discovery), 34,000+ in [editions-cloud-app-security-o365](https://learn.microsoft.com/en-us/defender-cloud-apps/editions-cloud-app-security-o365)). I quoted both. Reviewer should pick the latest authoritative figure.
- **Conditional Access App Control reverse-proxy claim** — I described CAAC as reverse-proxy in practitioner terms because functionally it is one (TLS termination, URL rewriting via `*.mcas.ms`), but Microsoft does not use the word "reverse proxy" in current docs. Architect-lens reviewer should sanity-check the framing.
- **Watermarking gap** — I have stated unambiguously that Defender for Cloud Apps does not watermark downloads. The Purview-label encryption story can substitute for some watermark use-cases (perimeter control), but not for visual deterrent watermarks (insider screenshot). Compliance-lens reviewer should consider whether this matters for any regulatory frame they're scoping against.
- **Step-up auth on session is in preview** — flagged as such per the session-policy doc. Practitioners building this into a runbook should validate GA status as of their actual deployment date.
- **The `protect-office-365` doc references a January 2026 change** ("In January 2026, the default values were added to support complete security coverage..."). I treated this as authoritative because the doc itself dates updated 2026-06-01. If the lens reviewers find evidence to the contrary, the change date should be revisited.
- **The Defender for Cloud Apps tenant-region claim** (EU/US/UK/AU primary data locations fixed at provisioning) was paraphrased from general practitioner knowledge; I did not surface a specific doc URL for it in this pass. Reviewer with portal access should confirm the current set of geos.
- **The "GenAI category was added mid-2023"** date is from practitioner memory + the release-notes page surfaced in search. I did not fetch the actual release-notes entry to confirm the month — that page would be the canonical source for the lens reviewer to cite.
- **The 50-file-policy limit and the 200k/100k activity-policy auto-disable thresholds** are quoted from current docs and are real operational ceilings that practitioners regularly hit at scale. They are easy to miss in a pre-sales conversation. Worth surfacing in the architect-lens review.
- **Office 365 Cloud App Security (the subset SKU)** — the documented feature gap is significant (no cross-SaaS DLP, no app governance, no third-party connectors, no SSPM beyond Microsoft Secure Score). I have not separately documented OCAS as a distinct vendor; practitioners working in O365 E5 without M365 E5 should be steered to either upgrade or to a different CASB. Compliance-lens reviewer should consider whether this licensing nuance belongs in the canonical 04-vendors entry.
