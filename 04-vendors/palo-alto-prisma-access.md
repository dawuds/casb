# Palo Alto Networks — Prisma Access (SaaS Security / Next-Generation CASB)

## Lineage
- **Current product names:**
  - **Prisma Access** — the SSE delivery platform (FWaaS, SWG, ZTNA, inline CASB).
  - **SaaS Security** — the umbrella name covering three sub-modules:
    - **SaaS Security Inline** (formerly the inline CASB function inside Prisma Access; "Discovered Apps" / shadow-IT view in Strata Cloud Manager).
    - **SaaS Security API** (formerly **Aperture**, briefly **Prisma SaaS**; "Data Security" in the Cloud Management Console).
    - **SaaS Security Posture Management (SSPM)**.
  - **AI Access Security** — separately-licensed GenAI-app discovery and policy module (Jan 2024 GA; integrates with SaaS Security Inline).
  - **Next-Generation CASB / CASB-X** — the marketing umbrella + cross-platform license SKU that bundles the above.
- **Renames / acquisitions:**
  - **CirroSecure** acquired April 2015 → rebranded **Aperture** (API-CASB).
  - **Aperture** renamed **Prisma SaaS** (2019, with the broader Prisma rebrand of GlobalProtect Cloud Service → Prisma Access).
  - **Prisma SaaS** rebranded **SaaS Security** (~2021), repositioned as part of "Next-Generation CASB".
  - AI Access Security launched 2024; SSPM third-party-plugin discovery expanded 2024–2025.
- **CASB housing:** Inside the **SSE platform (Prisma Access)**. The API-CASB and SSPM run on a separate "Cloud Management Console" / Strata Cloud Manager tenant but share identity, logging (Strata Logging Service), and Enterprise DLP profiles with Prisma Access.
- **Licensing tiers where CASB capabilities live:**
  - **CASB-X** (Next-Generation CASB for Prisma Access *and* NGFW, cross-platform) — bundles SaaS Security Inline + SaaS Security API + SSPM + Enterprise DLP + Behavior Threats.
  - **CASB-PA** — Prisma-Access-only add-on (Inline DLP + Data Security + SSPM).
  - **Prisma Access Enterprise Edition** — includes SaaS Security Inline by default.
  - **Data Security All Apps** — user-based standalone API CASB license (requires separate add-on for public-cloud storage: S3, Azure Storage, GCS).
  - **AI Access Security** — separate license.
  - **Enterprise DLP** — separate activation via Common Services, bundled in CASB-X / CASB-PA.

## Deployment modes supported
- **Forward-proxy (inline):**
  - Steering options:
    - **GlobalProtect agent** (Win / macOS / iOS / Android / Linux / ChromeOS) — primary path for managed endpoints.
    - **IPSec / GRE tunnels** from branch / SD-WAN edge to Prisma Access service connections.
    - **PAC / explicit-proxy** — Prisma Access Explicit Proxy is supported but is a different licensed deployment topology.
    - **Clientless VPN** — limited to web apps; used for unmanaged/BYOD access to a subset of sanctioned SaaS.
  - SSL/TLS decryption is **required** for SaaS app identification at the action level (upload/download/share). Without decryption you get App-ID at the connection level only, not in-session activity.
- **Reverse-proxy:** Not the primary delivery model. Palo Alto's API approach plus Clientless VPN is positioned as the equivalent for unmanaged access; there is no broadly-marketed IdP-routed reverse-proxy CASB mode comparable to Microsoft Defender for Cloud Apps Conditional Access App Control.
- **API connectors (Data Security supported sanctioned apps):** Airtable, Amazon S3, Bitbucket, Box, ChatGPT (Enterprise), Cisco Webex Teams, Confluence (Cloud + Data Center), Dropbox, GitHub, Gmail, Google Chat, Google Cloud Storage, Google Drive (My Drive + Shared Drives), Jira (Cloud + Data Center), Microsoft Azure Storage, Microsoft Exchange, Microsoft 365 (OneDrive + SharePoint), Microsoft Teams, Salesforce, ServiceNow, ShareFile, Slack (Enterprise + Pro/Business), Workday, Zendesk, Zoom. (~28 apps as of mid-2026; see Sources.)
- **Log-based discovery:** SaaS Security Inline ingests session logs from **Strata Logging Service** (formerly Cortex Data Lake), populated by Prisma Access and on-prem NGFWs. No support for third-party-firewall log ingestion the way Netskope CCI or Zscaler ShiftLogs accept generic syslog from non-PAN firewalls — this is a PAN-NGFW / Prisma Access ecosystem lock-in.
- **Hybrid considerations:**
  - Inline and API are **separately licensed and separately configured**. Policies in the Cloud Management Console (Data Security) do not auto-mirror to Prisma Access SaaS policy rules and vice versa — this is the seam the priority note calls out. Practitioners must build and maintain two policy sets that conceptually cover the same data.
  - Enterprise DLP profiles **are** shared between the two — one DLP profile can be referenced by an inline SaaS policy and an API asset policy. This is the main consolidation point.
  - SSPM is its own console area again, with its own policy primitives (posture checks, third-party plugin governance).

## Capability set

**Shadow-IT discovery (network-log-based and API-based) — Supported.** SaaS Security Inline consumes Strata Logging Service data from Prisma Access / NGFWs and produces a Discovered Apps inventory with risk attributes; risk score is computed from 55+ of 80+ tracked attributes per app. No native browser-extension-based or endpoint-agent-only discovery (i.e. no equivalent to the Zscaler Client Connector log-only mode for non-tunnelled traffic). Source: https://docs.paloaltonetworks.com/saas-security/saas-security-admin/saas-security-inline/get-started-with-saas-security-inline/whats-saas-security-inline (vendor doc, accessed 2026-06-10).

**Sanctioned-app inventory + risk-scoring — Supported.** Discovered apps can be tagged **Sanctioned / Tolerated / Unsanctioned**. Risk score is vendor-computed and exposes the underlying attributes (compliance certifications, data-at-rest encryption, breach history, etc.). Source: https://docs.paloaltonetworks.com/saas-security/saas-security-inline/manage-saas-security-inline-policy/guidelines-for-saas-policy (vendor doc, accessed 2026-06-10).

**Inline DLP for sanctioned SaaS (proxy mode — pre-upload prevention) — Supported.** Delivered through Prisma Access decryption + Enterprise DLP data profiles applied to SaaS app traffic; can block uploads pre-write. Requires decryption to be enabled for the destination and managed endpoint with GlobalProtect (or a steered branch path). Source: https://docs.paloaltonetworks.com/saas-security and Enterprise DLP integration docs (vendor doc, accessed 2026-06-10).

**API-mode DLP for sanctioned SaaS (post-upload scan + remediate) — Supported.** Data Security scans assets at rest, applies data patterns / profiles, and offers automated remediations: change sharing, quarantine, tombstone, alert. File-size ceiling **25 MB across all apps** is a hard limit. Forward-scan only for several apps (Gmail, Google Chat, Zoom, Zendesk, ChatGPT, Airtable). Source: https://docs.paloaltonetworks.com/saas-security/saas-security-admin/saas-security-api/get-started-with-saas-security-api/support-on-saas-security-api/supported-saas-applications (vendor doc, accessed 2026-06-10).

**OAuth-app discovery and governance (third-party app grants to M365 / Workspace) — Supported with caveats.** Delivered via **SSPM third-party plugins** view, which maps third-party apps connected to a marketplace tenant (Google Workspace, M365, Salesforce, Slack), shows scopes granted, surfaces GenAI-specific attributes, and can revoke high-risk apps. Caveat: only sees apps connected through the SSPM-onboarded tenant — consumer-account OAuth (personal Google sign-in to a third-party SaaS) is invisible. Source: https://docs.paloaltonetworks.com/saas-security/sspm/assess-posture-security/view-third-party-plugins (vendor doc, accessed 2026-06-10).

**Anomalous-activity / UEBA detection — Supported.** Delivered by **Behavior Threats** (included in CASB-X / CASB-PA / Data Security). Dynamic policies use up to 90 days of historical data for ML baselines; categories: activity spikes within an hour, bulk operations across hours, off-hours activity, unusual-location activity, uncommon sensitive-data access. Source: https://docs.paloaltonetworks.com/saas-security/behavior-threats/policies-to-detect-threats (vendor doc, accessed 2026-06-10).

**Compromised-account detection — Supported.** Static Behavior Threats policies cover impossible-travel, failed-login bursts, unsafe-VPN / risky-IP access, malware-tied activity. Detection only — automated session-kill / forced reauth back through the IdP is not a documented native action; SaaS Security raises an incident and integrations (XSOAR, IdP) handle response. Source: same as above (vendor doc, accessed 2026-06-10).

**Data-residency / geo-policy enforcement — Supported with caveats.** Prisma Access supports tenant placement in specific regions; inline policy can include source-location conditions. API CASB scanning of assets is region-bound by the SaaS app's own data residency, not Palo Alto's. There is no in-product data-residency policy primitive on a per-asset basis comparable to (e.g.) Microsoft Purview's location labels. Source: Prisma Access Administration / SaaS Security docs (vendor doc, accessed 2026-06-10).

**Share-link governance (external sharing, anonymous links, expiry) — Supported.** Data Security asset policies match on **exposure** (Public / External / Company / Private) and trigger remediations to change-sharing, remove collaborator, or quarantine. Per-app fidelity varies — works well on Google Drive / OneDrive / SharePoint / Box / Dropbox; weaker on Salesforce / ServiceNow where the sharing model is record-permission-driven. Source: https://docs.paloaltonetworks.com/saas-security/saas-security-admin/saas-security-api/manage-saas-security-api-policy/policy-types (vendor doc, accessed 2026-06-10).

**Malware scanning in SaaS storage — Supported.** WildFire-backed scanning of files at rest in sanctioned apps via Data Security; quarantine-and-tombstone available. Subject to the same 25 MB ceiling. Source: SaaS Security docs (vendor doc, accessed 2026-06-10).

**Encryption / tokenisation at gateway — Not supported.** Palo Alto does not market field-level tokenisation or proxy-injected encryption for sanctioned SaaS. Encryption is at the transport layer (TLS) and via Enterprise DLP profile actions (block / coach) rather than transformation. Source: absence from product docs (accessed 2026-06-10).

**Risk-based session policy (step-up auth, read-only, watermark, block) — Supported with caveats.** Inline SaaS policy rules can apply by user/group, device-posture (managed vs. personal via GlobalProtect HIP), activity (upload / download / share), and Enterprise DLP profile. Actions: allow / block / continue (coach) / decrypt-and-inspect. **Watermarking** of downloaded files is not a native CASB action (no equivalent to Netskope's reverse-proxy watermarking). **Step-up MFA** is an IdP-side action triggered by an alert, not an inline session-control verb. Source: SaaS policy recommendation docs (vendor doc, accessed 2026-06-10).

**GenAI-app discovery + governance — Supported (2024–2026 additions).** Delivered by **AI Access Security** (separate license, integrates with SaaS Security Inline). Maintains a GenAI-app dictionary (ChatGPT, Claude, Gemini, Copilot, plus marketplace plugins) refreshed by Palo Alto; supports sanctioned / tolerated / unsanctioned tagging for GenAI apps specifically; inline prompt-content inspection via Enterprise DLP; fine-grained controls on upload/download to GenAI apps. SSPM also discovers GenAI plugins connected to marketplace tenants. ChatGPT Enterprise is the only GenAI app with an **API** connector for at-rest scanning of conversations and shared files. Source: https://www.paloaltonetworks.com/sase/ai-access-security and https://docs.paloaltonetworks.com/ai-access-security (vendor doc, accessed 2026-06-10).

## Configurable policies

Below: 10 policies a practitioner can build in the Strata Cloud Manager / Cloud Management Console. Where the policy lives across two consoles (the inline / Prisma Access side and the API / Data Security side), this is called out — that is the licensing seam.

### 1. Block upload of files matching PCI / PII data patterns to unsanctioned GenAI apps (inline)
- **What it does:** Inline DLP rule that blocks uploads to apps tagged "Unsanctioned" in the GenAI category when the request body matches a sensitive-data profile.
- **Where it lives:** Strata Cloud Manager → Manage → Configuration → NGFW and Prisma Access → Security Services → SaaS Security Inline → Policy Recommendations (pushed as a Security Policy rule), referencing an Enterprise DLP profile.
- **Key knobs:**
  - User/group filter: AD/Entra ID group ("All Employees" minus "AI Pilot").
  - App filter: Category = GenAI; Tag = Unsanctioned.
  - Condition: Enterprise DLP profile = "PCI-DSS-PAN + PII-Strict" with match on upload direction.
  - Action: Block + custom block page.
- **Deployment-mode requirement:** Inline (proxy). Requires SSL decryption to the GenAI endpoint and GlobalProtect on the endpoint.
- **False-positive risk:** Medium — generic regex hits in unrelated documents (e.g. policy docs *describing* PCI rules). Mitigate with confidence levels and document-type predicates.
- **Source:** https://docs.paloaltonetworks.com/ai-access-security and Enterprise DLP integration docs (vendor doc, accessed 2026-06-10).

### 2. Quarantine externally-shared files in Google Drive matching "PII-Strict"
- **What it does:** API CASB asset policy that quarantines a file at rest when (a) exposure ≥ External and (b) content matches a sensitive-data pattern.
- **Where it lives:** Cloud Management Console → Data Security → Policy → Data Asset Policy.
- **Key knobs:** App = Google Drive; Asset Exposure = External or Public; Data Profile = PII-Strict; Action = Admin Quarantine (copy to admin folder, tombstone original); Notify = file owner + DLP team.
- **Deployment-mode requirement:** API only.
- **False-positive risk:** High — quarantine of legitimately externally-shared assets (e.g. customer onboarding docs). Pilot with Alert-only first, then convert.
- **Source:** https://docs.paloaltonetworks.com/saas-security/data-security/remediate-risks-saas-security-api/automatically-remediate-incidents/quarantine (vendor doc, accessed 2026-06-10).

### 3. Block download of corporate SharePoint files to unmanaged devices
- **What it does:** Inline rule denying download activity from sanctioned M365 SharePoint when the endpoint does not satisfy HIP (Host Information Profile) "Corporate-Managed".
- **Where it lives:** Strata Cloud Manager → Security Services → SaaS Security Inline → Policy Recommendations → Activity = Download, with HIP match in the security policy rule.
- **Key knobs:** App = Microsoft SharePoint Online (specific tenant via custom App-ID or URL category); Activity = Download; HIP match ≠ Corporate-Managed; Action = Block.
- **Deployment-mode requirement:** Inline + GlobalProtect (HIP requires the agent).
- **False-positive risk:** Medium — laptops in HIP-failed states (cert expired, AV update behind) get blocked from legitimate work; needs a coach/notify exception path.
- **Source:** GlobalProtect HIP docs + SaaS policy recommendation docs (vendor doc, accessed 2026-06-10).

### 4. Microsoft 365 tenant restriction (corporate tenant only)
- **What it does:** Forces all M365 sign-in traffic to a corporate Entra tenant; blocks consumer / other-org M365 logins.
- **Where it lives:** Prisma Access Security Policy → URL filtering / HTTP Header Insertion rule, inserting the Entra tenant-restriction headers (`Restrict-Access-To-Tenants`, `Restrict-Access-Context`).
- **Key knobs:** Destination = login.microsoftonline.com; HTTP header insert with allowed tenant ID(s) and context tenant ID.
- **Deployment-mode requirement:** Inline (proxy). Decryption required.
- **False-positive risk:** Low when scoped correctly; high if rolled to users with legitimate cross-tenant access (consultants, M&A users).
- **Source:** https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-palo-alto-coexistence (independent, Microsoft Learn, accessed 2026-06-10).

### 5. Detect impossible travel on Salesforce
- **What it does:** Static Behavior Threats policy raising an incident when a Salesforce user authenticates from two geo-distant locations within an implausibly short interval.
- **Where it lives:** Cloud Management Console → Behavior Threats → Policies → Static → Impossible Traveler.
- **Key knobs:** App scope = Salesforce; Distance/time threshold (vendor default ~kmh); IP exclusion list (corporate VPN egress IPs); Severity = High.
- **Deployment-mode requirement:** API (sourced from app audit logs).
- **False-positive risk:** Medium — corporate VPN egress in distant regions; mobile users on cellular networks with NAT to remote PoPs. Maintain IP exclusion list.
- **Source:** https://docs.paloaltonetworks.com/saas-security/behavior-threats/policies-to-detect-threats (vendor doc, accessed 2026-06-10).

### 6. Revoke high-risk OAuth third-party apps in Google Workspace
- **What it does:** SSPM third-party-plugin policy that auto-revokes OAuth grants when the connected app has Read/Write scope on Drive/Gmail and a high vendor-risk score.
- **Where it lives:** Cloud Management Console → SSPM → Third-Party Plugins → Policy.
- **Key knobs:** Marketplace = Google Workspace; Scope contains "drive.file" OR "gmail.modify"; Vendor risk = High; GenAI plugin = true (optional); Action = Revoke.
- **Deployment-mode requirement:** API (SSPM connector).
- **False-positive risk:** Medium — sanctioned productivity plugins with broad scopes (Grammarly, Loom) get revoked; needs an allow-list of approved vendors first.
- **Source:** https://docs.paloaltonetworks.com/saas-security/sspm/assess-posture-security/view-third-party-plugins (vendor doc, accessed 2026-06-10).

### 7. Alert on bulk download from Box outside business hours
- **What it does:** Dynamic Behavior Threats policy that compares per-user hourly download volume against a 90-day baseline plus an off-hours predicate.
- **Where it lives:** Cloud Management Console → Behavior Threats → Policies → Dynamic → Bulk Operations + Time-Based.
- **Key knobs:** App = Box; Activity = Download; Volume = ML-baseline + n-sigma; Time = outside 08:00–19:00 user-local; Action = Incident.
- **Deployment-mode requirement:** API.
- **False-positive risk:** Medium — quarter-end finance / month-end accounting bulk pulls register as anomalies; needs an exclusion list of approved cyclical activity.
- **Source:** same as #5 (vendor doc, accessed 2026-06-10).

### 8. Enforce malware scan + WildFire verdict on OneDrive uploads
- **What it does:** API asset policy that scans new files at rest in OneDrive against WildFire; high-risk verdicts are quarantined.
- **Where it lives:** Cloud Management Console → Data Security → Policy → Data Asset Policy → Malware.
- **Key knobs:** App = OneDrive; Asset type = file; Malware verdict ∈ {malicious, grayware}; Action = Quarantine + notify owner + alert SOC.
- **Deployment-mode requirement:** API; WildFire subscription active on the tenant.
- **False-positive risk:** Low (WildFire grayware can flag legitimate IT utilities). 25 MB file ceiling applies — large installers escape scan.
- **Source:** Data Security docs + WildFire integration (vendor doc, accessed 2026-06-10).

### 9. Sanctioned-tenant control for ChatGPT / Claude / Gemini
- **What it does:** Inline AI Access policy permitting GenAI use only via the corporate workspace/tenant; logs and blocks consumer-account use.
- **Where it lives:** Strata Cloud Manager → AI Access Security → Policy.
- **Key knobs:** GenAI app = ChatGPT, Claude, Gemini; Sanctioned tenant ID = corporate workspace; Action on consumer = Block + log; Action on sanctioned tenant = Allow with DLP profile applied.
- **Deployment-mode requirement:** Inline. Tenant-restriction enforcement varies by app — ChatGPT supports a Workspace ID header; Claude and Gemini support equivalent headers but coverage in the AI Access dictionary should be verified per app at deployment time.
- **False-positive risk:** Medium — vendor app header schemas change; the policy can silently allow consumer use if the GenAI app dictionary is stale.
- **Source:** https://www.paloaltonetworks.com/sase/ai-access-security and AI Access Security docs (vendor doc, accessed 2026-06-10).

### 10. Coach users away from "Tolerated" file-sharing apps
- **What it does:** Inline rule that allows the session but inserts a coach page on first activity per-day for apps tagged Tolerated in the file-sharing category, steering users to the sanctioned alternative (Box or OneDrive).
- **Where it lives:** Strata Cloud Manager → Security Services → SaaS Security Inline → Policy Recommendations → Action = Continue with response page.
- **Key knobs:** App category = File Sharing; Tag = Tolerated; Activity = Upload OR Share; Action = Continue with custom page; Frequency = once per user per day.
- **Deployment-mode requirement:** Inline; decryption required for activity-level matching.
- **False-positive risk:** Low (it is informational, not blocking). The risk is the opposite — users learn to dismiss the page and behaviour does not change.
- **Source:** SaaS policy recommendation docs (vendor doc, accessed 2026-06-10).

### Worked example — Policy #2 (Quarantine externally-shared Google Drive files matching PII-Strict)

Conceptual configuration object (not literal product syntax):

```yaml
policy:
  type: data_asset_policy
  name: "GDrive - Quarantine externally shared PII"
  app:
    connector: google_drive
    scope: my_drive + shared_drives
  match:
    asset_type: file
    exposure_in: [external, public, anyone_with_link]
    data_profile: PII-Strict
    file_size_max_mb: 25      # platform ceiling
    direction: at_rest
  exclude:
    owners_in_group: legal-counsel
    folders_matching: "Client Onboarding/*"  # explicit business need
  action:
    primary: admin_quarantine     # copy to admin folder; tombstone original
    notify:
      owner: true
      group: dlp-ops
      severity: high
    response_sla_hours: 24
  remediation_window:
    business_hours_only: false
    auto_remediate: true          # set false during pilot
```

## Limitations / what this product does NOT cover

- **Two-console policy duplication for inline vs. API.** Configuring "block external sharing of PII" requires building the rule twice — once as an inline DLP rule in Strata Cloud Manager (pre-upload) and once as a Data Asset policy in the Cloud Management Console (post-upload). Reason: SaaS Security Inline and SaaS Security API have separate policy engines; only Enterprise DLP profiles are shared. This is the licensing/console seam called out as a priority.
- **Shadow-IT discovery on non-Palo-Alto network paths.** SaaS Security Inline ingests logs only from Prisma Access and PAN NGFWs via Strata Logging Service. Reason: no generic syslog/CSV ingestion path comparable to Netskope CCI or Zscaler ShiftLogs — discovery is locked to the PAN data plane.
- **API CASB coverage on files > 25 MB.** Files at rest above 25 MB are not scanned by Data Security in any supported app. Reason: documented per-app feature limit; large installers / video / data dumps escape malware + DLP at rest.
- **API-only forward-scan apps.** Gmail, Google Chat, Zoom, Zendesk, Airtable, ChatGPT (Enterprise) support **forward scan only** — historical content already in the tenant is not retroactively scanned. Reason: the API connector design or the SaaS app's API itself does not expose backfill at scale.
- **OAuth grants outside the IdP-connected tenant.** SSPM only sees third-party plugins connected to the marketplaces it is onboarded to (Workspace, M365, Salesforce, Slack). Direct-to-app OAuth using a personal Google / Microsoft account (consumer flow) on a managed device is invisible to SSPM. Reason: API connector scope is the corporate tenant, not the user's identity universe.
- **Mobile native apps with certificate pinning.** Pinned SaaS mobile apps (e.g. some banking SaaS clients, Authenticator apps, certain consumer GenAI mobile apps) refuse Prisma Access TLS decryption. Reason: certificate pinning rejects the gateway-resigned cert; either traffic bypasses the proxy or the app fails to connect, leaving the practitioner to choose a bypass.
- **Watermarking / DRM-style controls.** No native watermark-on-download or persistent rights management. Reason: Palo Alto's session-control model is allow/block/coach plus DLP-driven inspection, not content transformation; integrations with Microsoft Purview labels are read-only (match-on-label).
- **Field-level tokenisation / gateway encryption.** No proxy-injected encryption or tokenisation for sanctioned SaaS records. Reason: Palo Alto deprecated/never marketed this CASB pillar; rely on the SaaS app's native encryption or a third-party tokenisation tool.
- **Real-time session-kill on compromised-account detection.** Behavior Threats raises incidents; killing the session or forcing reauth requires an IdP-side workflow (Entra ID / Okta) via SOAR (XSOAR). Reason: SaaS Security does not hold an inline session-control plane equivalent to a reverse-proxy that can drop the user mid-session.
- **API connector breadth in Q2 2026.** Roughly 28 sanctioned apps with API support; SSPM covers ~90 apps for posture but most do not have a Data Security DLP/asset connector. Reason: connector roadmap prioritises the high-volume apps; long-tail SaaS coverage is left to inline visibility only.
- **Reverse-proxy / IdP-routed mode for unmanaged devices.** No conditional-access reverse proxy equivalent to MDA's "Conditional Access App Control". Clientless VPN partially fills this gap but is limited to web apps and a subset of sanctioned apps. Reason: architectural choice — Palo Alto leans on GlobalProtect-on-endpoint plus API-side controls instead of an IdP-routed reverse proxy.
- **Activity-level visibility without TLS decryption.** Without decryption to a sanctioned SaaS, the practitioner sees connection-level App-ID but not upload/download/share verbs. Reason: TLS encrypts the activity payload; App-ID alone identifies the app, not the action.

## Sources
- https://docs.paloaltonetworks.com/saas-security — **vendor-controlled**, accessed 2026-06-10.
- https://docs.paloaltonetworks.com/prisma/prisma-access — **vendor-controlled**, accessed 2026-06-10.
- https://docs.paloaltonetworks.com/next-gen-casb — **vendor-controlled**, accessed 2026-06-10.
- https://docs.paloaltonetworks.com/saas-security/saas-security-admin/saas-security-api/get-started-with-saas-security-api/support-on-saas-security-api/supported-saas-applications — **vendor-controlled**, accessed 2026-06-10. (Supported app list.)
- https://docs.paloaltonetworks.com/saas-security/activation-and-onboarding/license-types — **vendor-controlled**, accessed 2026-06-10. (Licensing.)
- https://docs.paloaltonetworks.com/saas-security/saas-security-admin/saas-security-api/manage-saas-security-api-policy/policy-types — **vendor-controlled**, accessed 2026-06-10. (API policy types.)
- https://docs.paloaltonetworks.com/saas-security/behavior-threats/policies-to-detect-threats — **vendor-controlled**, accessed 2026-06-10. (Behavior Threats.)
- https://docs.paloaltonetworks.com/saas-security/sspm/assess-posture-security/view-third-party-plugins — **vendor-controlled**, accessed 2026-06-10. (SSPM OAuth governance.)
- https://docs.paloaltonetworks.com/saas-security/saas-security-admin/saas-security-sspm/get-started-with-sspm/supported-saas-applications-on-sspm — **vendor-controlled**, accessed 2026-06-10. (SSPM app list.)
- https://docs.paloaltonetworks.com/ai-access-security — **vendor-controlled**, accessed 2026-06-10. (AI Access Security.)
- https://www.paloaltonetworks.com/sase/ai-access-security — **vendor-controlled** marketing page, accessed 2026-06-10.
- https://docs.paloaltonetworks.com/saas-security/saas-security-admin/saas-security-inline/get-started-with-saas-security-inline/whats-saas-security-inline — **vendor-controlled**, accessed 2026-06-10.
- https://docs.paloaltonetworks.com/saas-security/saas-security-inline/manage-saas-security-inline-policy/guidelines-for-saas-policy — **vendor-controlled**, accessed 2026-06-10.
- https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-palo-alto-coexistence — **independent** (Microsoft Learn) — corroborates tenant-restriction header insertion via Prisma Access, accessed 2026-06-10.
- https://www.paloaltonetworks.com/blog/sase/saas-supply-chain-security/ — **vendor-controlled** blog (third-party plugin discovery), accessed 2026-06-10.
- https://docs.paloaltonetworks.com/common-services/subscription-and-tenant-management/casb-x-activation — **vendor-controlled**, accessed 2026-06-10. (CASB-X licensing.)

## Notes for the lens reviewers
- **The inline-vs-API console seam is the single biggest practitioner pitfall.** Architect lens should validate whether Enterprise DLP profile sharing meaningfully reduces the double-configuration tax, or whether in practice rules drift. I'm confident the two policy planes are distinct; less confident on how much shared tooling exists in the latest Strata Cloud Manager UI.
- **Tenant-restriction policy (#4 and #9) header schemas are vendor-specific and time-bound.** I asserted ChatGPT / Claude / Gemini all support workspace-ID header enforcement via Prisma Access AI Access Security. The ChatGPT case is well-documented; the Claude and Gemini ones I marked as "verify per app at deployment time" — Compliance lens should pin this down before relying on it for regulator-facing controls.
- **Behavior Threats response actions.** Vendor docs describe detection thoroughly but are vague on automated response beyond incident creation. I treated session-kill / forced reauth as **not** native and requiring XSOAR or IdP workflow. Product lens should confirm whether 2025–2026 releases added native session response — I did not find evidence in the public docs.
- **Forward-scan-only apps (Gmail / Chat / Zoom / Zendesk / Airtable / ChatGPT).** This was extracted from the supported-apps matrix. Worth a second read of the per-app docs by the product reviewer before this gets promoted — the matrix wording can hide whether "forward only" means at the connector level or at the data-pattern level.
- **25 MB file ceiling** — stated as universal across Data Security. I treated this as a hard product limit; if there is an enterprise-tier increase available that's gated behind support, I did not find evidence of it.
- **The "Next-Generation CASB" / "CASB-X" branding is marketing-heavy.** I avoided repeating it as a capability claim. Reviewers should be alert to lifts of vendor phrasing if any reappear during the canonical promotion.
- **No public independent CVE / breach record** for Prisma Access SaaS Security components surfaced during research. I did not exhaust this — Compliance lens should run a targeted check before publishing.
- **Reverse-proxy gap** — I asserted there is no MDA-equivalent IdP-routed reverse proxy. Architect lens should challenge whether Clientless VPN materially closes this for the use cases that matter (unmanaged BYOD access to sanctioned SaaS).
