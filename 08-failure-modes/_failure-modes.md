# Cross-vendor CASB failure modes — synthesis

> Source: aggregated "Limitations / what this product does NOT cover" sections from five vendor drafts at `../raw/` (Microsoft Defender for Cloud Apps, Netskope, Palo Alto Prisma Access, Skyhigh Security, Zscaler Internet Access). Lens-review caveats at `../lens-reviews/` consulted; `[unverified]` prefix denotes a claim the lens reviewers flagged as snippet-grade, marketing-grade, or operationally uncorroborated.
>
> Convention: vendors abbreviated MDA (Microsoft Defender for Cloud Apps), NSK (Netskope), PAN (Palo Alto Prisma Access / SaaS Security), SKY (Skyhigh), ZS (Zscaler ZIA). All vendor drafts dated 2026-06-10.

---

## 1. BYOD / unmanaged-endpoint blind spot

**Definition.** Inline controls (DLP, activity restriction, tenant pinning) only fire when traffic is steered through the CASB. BYOD without an enrolment vehicle — agent, MDM-pushed VPN profile, PAC, or IdP-routed reverse proxy — bypasses the inline plane entirely.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | Strict Entra-bound model — Conditional Access App Control reverse-proxy is the *only* BYOD path and works for browser sessions only (per MDA draft Limitations bullet 3; reinforced by architect lens finding 6) |
| NSK | Yes | Has "Reverse Proxy as a Service with Entra ID" and "Universal Reverse Proxy" for agentless BYOD steering (per netskope draft § Deployment modes, "Reverse-proxy") — best-managed of the five for browser-based BYOD into federated SaaS |
| PAN | Yes (worst) | Only Clientless VPN — "limited to web apps; used for unmanaged/BYOD access to a subset of sanctioned SaaS" (per palo-alto draft § Deployment modes); no IdP-routed reverse-proxy equivalent (Limitations bullet 11) |
| SKY | Yes | Reverse-proxy via SAML rewrite is the historical differentiator; agentless for federated web apps only (per skyhigh draft § Deployment modes); fails for direct OAuth, service-local credentials, static API keys (Limitations bullets 1, 4) |
| ZS | Yes | No SAML-rewrite reverse-proxy at all — only "agentless Cloud Browser Isolation triggered via IdP" as substitute, "not a 1:1 substitute" (per zscaler draft Limitations bullet 3) |

**Root cause.** Inline CASBs need a control point in the data path. Without OS-level network capture (agent), forced network egress (PAC/tunnel), or IdP-mediated session rewriting, BYOD traffic exits the corporate envelope before any policy can apply.

**Compensates with (non-CASB).** MDM enrolment, MAM / Intune App Protection Policies, Conditional Access on the IdP, Zero-Trust Network Access for sanctioned-app delivery, or an explicit BYOD-prohibition policy backed by acceptable-use.

**Promote to 08-failure-modes/?** Yes — universal across all five vendors, distinct technical root cause, distinct compensating control class.

---

## 2. Mobile-native-app bypass (modern-auth + cert-pin)

**Definition.** Mobile SaaS apps (Outlook Mobile, Teams Mobile, OneDrive sync, Drive iOS, etc.) authenticate via modern-auth / MSAL / WAM rather than SAML browser flow, and frequently certificate-pin. Both properties defeat IdP-redirect reverse-proxy and TLS-intercepting forward-proxy respectively.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | Browser sessions and IdP-authenticating subset only; "Outlook Mobile, Teams Desktop, OneDrive sync client and most mobile SDK-based apps bypass the proxy" (per MDA draft Limitations bullet 3) |
| NSK | Yes | "Mobile native apps with certificate pinning ... must be exception-bypassed" — accept the visibility loss (per netskope draft Limitations) |
| PAN | Yes | "Certificate pinning rejects the gateway-resigned cert; either traffic bypasses the proxy or the app fails to connect" (per palo-alto draft Limitations bullet 6) |
| SKY | Yes | "Mobile native apps frequently use non-SAML auth flows" → reverse-proxy doesn't help (per skyhigh draft Limitations bullet 3); "Pinned cert ... app either refuses connection ... or bypasses the proxy" |
| ZS | Yes | "Pre-upload blocking for mobile SaaS apps that use certificate pinning" → not supported (per zscaler draft Limitations bullet 2) |

**Root cause.** Two compounding properties: (a) modern-auth flows establish session tokens without round-tripping through the IdP's web flow that the reverse-proxy hijacks; (b) cert-pinning rejects any non-pinned CA chain, including a CASB's MITM intermediate.

**Compensates with (non-CASB).** Intune App Protection Policies / equivalent MAM, vendor-native mobile DLP (Microsoft App Protection for the M365 mobile apps), conditional access requiring approved-client-app, or container/wrapper approaches (Hypori, AppDome) for sensitive workloads.

**Promote to 08-failure-modes/?** Yes — universal, distinct technical root cause from generic BYOD, distinct compensating control class (MAM, not MDM).

---

## 3. SSL/TLS inspection prerequisite (and inevitable bypass list)

**Definition.** Inline DLP, activity controls, prompt inspection, and tenant restriction all require TLS termination by the CASB. Cert-pinned apps, ECH-enabled endpoints, regulator-sensitive destinations, and trust-store-resistant clients all end up on a growing bypass list that *silently disables* CASB for those destinations.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | N/A (architecturally avoided) | No SSL-bump; CAAC terminates TLS at proxy and rewrites URL to `*.mcas.ms` (per MDA draft § Deployment modes); user's trust is in the rewritten domain (per architect lens finding 5) — but the model only works for IdP-fronted browser sessions |
| NSK | Yes | "SSL inspection is required for content-level CASB controls. Certificate-pinned mobile apps fail under MITM and must be exception-bypassed" (per netskope draft § Deployment modes); see also Limitations bullets on cert-pinning and encrypted-archive |
| PAN | Yes | "SSL/TLS decryption is required for SaaS app identification at the action level. Without decryption you get App-ID at the connection level only" (per palo-alto draft § Deployment modes); reinforced by Limitations bullet 12 |
| SKY | Yes | "Forward-proxy DLP on TLS 1.3 + ECH endpoints" structurally degrades inspection (per skyhigh draft Limitations bullet 5); "SSL/TLS inspection is required for most inspection use cases" (per § Deployment modes) |
| ZS | Yes (most exposed) | "SSL/TLS inspection is required for any meaningful inline CASB or DLP action" (per zscaler draft § Deployment modes); [unverified] architect lens finding 2 quantifies "standard 15-20% bypass list" — every bypassed destination is dark to CASB |

**Root cause.** TLS is end-to-end by design. Inserting an inspector requires re-issuing a certificate the client trusts, which conflicts with cert-pinning, FIDO/WebAuthn attestation paths, ECH, and any client that bundles its own trust store.

**Compensates with (non-CASB).** Endpoint DLP (operates pre-encryption), SaaS-native DLP (operates post-decryption inside the tenant — Purview, Workspace DLP, Salesforce Shield), data classification at source, change-management discipline for bypass-list governance, or accepting visibility loss with monitoring-only on bypassed destinations.

**Promote to 08-failure-modes/?** Yes — universal, structural (TLS architecture), independent compensating-control class.

---

## 4. API-mode is post-event, not prevention

**Definition.** API connectors scan tenant data after the file has been written. The vendor itself markets this as "after-the-fact, not real-time" — webhook + change-feed architecture imposes minutes-to-hours latency. A user can still upload, share, and have the file pulled before remediation fires.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | Power BI / Dynamics 365 log delay 24-72 hours specifically called out (per MDA draft Limitations bullet 13); file policies are post-facto by definition |
| NSK | Yes | "API connectors are not real-time — vendor itself describes the timing as 'after-the-fact'. Time-to-detect ranges from minutes to hours" (per netskope draft Limitations bullet 10); also "Inline policies do not catch data already at rest" (bullet 9) — the converse |
| PAN | Yes | Gmail / Google Chat / Zoom / Zendesk / Airtable / ChatGPT are "forward-scan only" — no historical backfill (per palo-alto draft Limitations bullet 4) |
| SKY | Yes | API connector scans "post-upload"; no real-time block via API path (per skyhigh draft Limitations bullet 1, bullet 12) |
| ZS | Yes | "API mode catches the file post-upload, not pre-upload" (per zscaler draft Limitations bullet 1); SaaS-side audit-log retention can be gated by the SaaS vendor's own licence (bullet 10) |

**Root cause.** SaaS providers do not expose synchronous pre-write hooks to third parties; the only API contract is webhooks-after-commit and polling. Vendor connectors cannot insert into the write path.

**Compensates with (non-CASB).** Inline proxy mode for the same flow (acknowledging the BYOD/cert-pin caveats above), endpoint DLP at source, SaaS-native pre-write policies (Purview Auto-labelling, Workspace context-aware access), or data-classification-led prevention before the file ever reaches the SaaS.

**Promote to 08-failure-modes/?** Yes — universal, structural (SaaS API design), distinct compensating-control class.

---

## 5. OAuth-grant blind spot — consumer/personal identity

**Definition.** SSPM / Connected Apps / OAuth governance modules enumerate grants made via the corporate IdP-connected tenants only. A user signing in to ChatGPT/Notion/Loom etc. with a personal Google or personal Microsoft account on a corporate device, granting OAuth scope to that personal mailbox/drive — invisible to all five CASBs.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | "OAuth grants to apps not federated through the connected IdP" not surfaced; also "Application-only (client-credentials) OAuth grants" not surfaced (per MDA draft Limitations bullets 4, 5) |
| NSK | Yes | "OAuth grants made on personal / unmanaged tenants" invisible — SSPM only sees connected tenants (per netskope draft Limitations bullet 2) |
| PAN | Yes | "Direct-to-app OAuth using a personal Google / Microsoft account (consumer flow) on a managed device is invisible to SSPM" (per palo-alto draft Limitations bullet 5) |
| SKY | Yes | "OAuth grants given via a consumer Google or personal Microsoft account on the same device are invisible because the OAuth handshake never touches the connected tenant" (per skyhigh draft Limitations bullet 2) |
| ZS | Yes | "OAuth grants made directly to a SaaS app from an identity that is not in the connected IdP / tenant" — invisible (per zscaler draft Limitations bullet 4) |

**Root cause.** SSPM connectors read OAuth audit logs from the tenants they are subscribed to. A consumer-identity OAuth flow never touches those tenants — the grant lives in the user's personal Google/Microsoft consumer account, which the corporate CASB has no API into.

**Compensates with (non-CASB).** Identity-side controls: block consumer-account sign-in on managed devices (Entra Conditional Access "block personal Microsoft accounts", Workspace context-aware access), browser-managed-profile separation (Edge for Business / Chrome Enterprise profile enforcement), endpoint policy preventing second-browser-profile sign-in, DNS/SWG block on consumer sign-in endpoints (`login.live.com`, personal-account Google flows).

**Promote to 08-failure-modes/?** Yes — universal, structural (identity scope of API), the compensating control class (identity policy) is conceptually distinct from CASB.

---

## 6. SaaS-to-SaaS / third-party-to-third-party data flow

**Definition.** Runtime data movement between two SaaS tenants via vendor-native integrations (Slack ↔ Salesforce, M365 ↔ ServiceNow, Workday ↔ Box) or iPaaS platforms (Zapier, Workato, Make) traverses neither the user endpoint nor the CASB proxy. Authenticates via service principal, not user.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | [unverified by draft itself — surfaced by architect lens] "Slack -> Jira via Slack's Jira integration, Salesforce -> DocuSign, Workday -> ServiceNow — none of this is visible to MDA" (per MDA architect lens, "Limitations the draft missed") — draft does not name it |
| NSK | Yes (implicit) | Not explicit in Limitations, but the inline-blind + API-post-event nature applies; architect lens calls this out generally |
| PAN | Yes (implicit) | Not explicit in Limitations; architect lens flags: "Server-to-server SaaS integrations (Salesforce ↔ Workday, ServiceNow ↔ Slack, Workday ↔ Box) traverse no endpoint, no Prisma Access PoP" |
| SKY | Yes | "Cross-vendor SaaS-to-SaaS data flow inspection (Zapier-style integrations) — When SaaS A pushes data to SaaS B via a vendor integration platform that authenticates with its own service principal, the traffic doesn't traverse Skyhigh forward proxy and isn't visible via either tenant's API unless the integration touches a covered tenant" (per skyhigh draft Limitations bullet 8) |
| ZS | Yes | [unverified by draft itself — surfaced by architect lens] "third-party-to-third-party API traffic ... SaaS Security API DLP catches data at rest in a connected tenant but not the in-flight movement between two SaaS tenants" (per zscaler architect lens finding 11); draft itself silent |

**Root cause.** Forward-proxy CASBs see user-to-SaaS traffic; API connectors read tenant audit logs. SaaS-to-SaaS data movement uses a service-principal token between two SaaS backends — the user endpoint is uninvolved, and neither tenant's audit log necessarily records the *content* of the transfer, only the integration call.

**Compensates with (non-CASB).** SSPM third-party-plugin policies that govern *which* integrations are permitted to be installed (controls the supply chain, not the runtime flow), SaaS-native scoped integration tokens with least-privilege, third-party SaaS-to-SaaS-aware tools (AppOmni, Obsidian Security, Adaptive Shield), data classification + tagging that the destination SaaS can act on, or contractual / DPA constraints on integration vendors.

**Promote to 08-failure-modes/?** Yes — universal architectural blind spot; only one vendor draft surfaces it explicitly; compensating-control class is distinct.

---

## 7. User-applied encryption / password-protected archives / E2EE content

**Definition.** A user 7zips with a password, applies PGP/S-MIME, uses Cryptomator/Boxcryptor on a Drive volume, or operates inside an E2EE SaaS context (Signal-style chat, EKM-mode Slack DMs, end-to-end-encrypted Teams threads). DLP regex/SIT/EDM matches nothing because the inspected bytes are ciphertext.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes (partial) | "Files larger than the scanner's content-inspection limit, password-protected files, Azure-RMS-encrypted files — content cannot be inspected; the file is tagged ... but DLP regex/SIT cannot fire" (per MDA draft Limitations bullet 9); [unverified] architect lens flags user-applied encryption / PGP / 7zip as missing from policy decisions |
| NSK | Yes | "Encrypted-archive (password-protected zip/7z/rar) DLP inspection — Netskope cannot read the contents. Reason: no key" (per netskope draft Limitations bullet 8); architect lens adds Cryptomator, S/MIME, PGP, MIP-without-decrypt-cert |
| PAN | Yes (implicit) | Not explicit in Limitations; architect lens finding 8: "User-side encryption (1Password vaults, Cryptomator containers in Drive, password-protected zips, S/MIME / PGP mail) ... API CASB scans plaintext; user-encrypted blobs evade DLP entirely" — draft silent |
| SKY | Yes | "End-to-end-encrypted SaaS content (E2EE Zoom recordings, encrypted Slack DMs in EKM mode, Signal-style E2EE workspaces) — API connector pulls metadata only; content body is unreadable" (per skyhigh draft Limitations bullet 6) |
| ZS | Yes | "DLP coverage of end-to-end-encrypted SaaS chat — proxy sees ciphertext; API sees ciphertext for E2EE content" (per zscaler draft Limitations bullet 7); architect lens finding 12 adds user-applied encryption (PGP, password-zip) as standard DLP-bypass tradecraft |

**Root cause.** Encryption is the explicit absence of cleartext for an unauthorised observer; the CASB is — by design from the user's perspective — an unauthorised observer of the encrypted payload. No key, no inspection.

**Compensates with (non-CASB).** Endpoint DLP that operates pre-encryption (clipboard, screen, file-open hooks), policy + AUP prohibition of user-applied encryption on corporate data, archive-format block at egress (block any password-protected archive to non-sanctioned destinations), data classification at source before encryption can occur, HYOK/BYOK arrangements with the SaaS so the customer holds keys.

**Promote to 08-failure-modes/?** Yes — universal, root cause is fundamental cryptography, compensating-control class is endpoint + policy, not CASB.

---

## 8. SaaS coverage outside the vendor's connector list

**Definition.** API-mode DLP, malware scan, SSPM, OAuth governance, and tenant pinning only work for SaaS apps the vendor has built a connector for. The long tail of niche SaaS, regional fintech, vertical industry SaaS, and self-hosted "looks-like-a-SaaS" portals gets inline-only visibility — no post-upload remediation, no OAuth surfacing.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | ~25 connectors with M365 as the only "first-class" one (per MDA draft § Deployment modes "API connectors"); 50-file-policy hard ceiling per tenant (Limitations bullet 10) compounds the constraint |
| NSK | Yes | NextGen connector list ~30 apps; "Real-time blocking of risky actions inside non-API-connected SaaS" not available (per netskope draft Limitations bullet 1); "OneNote, SharePoint Lists, Microsoft Loop sites" explicitly excluded (bullet 3) |
| PAN | Yes | "~28 sanctioned apps with API support; SSPM covers ~90 apps for posture but most do not have a Data Security DLP/asset connector" (per palo-alto draft Limitations bullet 10) |
| SKY | Yes | "Sanctioned-app coverage outside the ~40 native connectors — most niche SaaS, regional fintechs, vertical industry SaaS — has no API-mode inspection — only the inline/Shadow-IT view" (per skyhigh draft Limitations bullet 9) |
| ZS | Yes | "Real-time API control of every SaaS in the catalogue" not available; "Bring-your-own-cloud (private SaaS or self-hosted) coverage at API level" not available (per zscaler draft Limitations bullets 9, 11) |

**Root cause.** Every API connector is bespoke engineering against a specific SaaS API surface; vendors prioritise high-volume connectors by revenue. The long tail does not justify build cost.

**Compensates with (non-CASB).** Inline visibility + URL-category-level block on long-tail apps, a sanctioned-app inventory discipline that excludes long-tail SaaS by default, SaaS-native DLP/audit if the vendor ships one, third-party SSPM that may carry the connector (AppOmni, Adaptive Shield, Obsidian), or for self-hosted "SaaS-like" portals, route them through ZTNA + on-prem DLP.

**Promote to 08-failure-modes/?** Yes — universal, structural (build economics), compensating control is governance/scope-management rather than tech.

---

## 9. Over-blocking / false-positive blast radius

**Definition.** Aggressive policies (tenant restriction, OAuth auto-revoke, DLP block, geo block, device-compliance gating) misfire on legitimate use — M&A integration period, contractor laptops, frequent travellers, approved-but-noisy productivity OAuth apps, regulator portals. Cost: incident-grade business disruption.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | Policy #1 "Block download to unmanaged" FP risk = Medium; Policy #3 "Block upload of unlabelled PII" FP risk = High (DCS SSN regex hits 9-digit IDs); Policy #5 "Block sign-in from non-compliant device" FP risk = High — breaks BYOD entirely; Policy #8 "Block paste to ChatGPT" FP risk = High (per MDA draft § Configurable policies) |
| NSK | Yes | Policy #1 "Block regulated upload" Medium (Luhn-valid test fixtures); Policy #4 "Quarantine externally-shared" High (legal counsel external sharing); Policy #6 GenAI prompt scan Medium-High (synthetic JWT in stack trace) (per netskope draft § Configurable policies) |
| PAN | Yes | Policy #2 "Quarantine externally-shared Drive PII" High (customer onboarding docs); Policy #3 "Block SharePoint download to unmanaged" Medium (HIP-failed laptops blocked from legit work) (per palo-alto draft § Configurable policies) |
| SKY | Yes | Policy #1 "Sanctioned-app DLP" Medium-High (marketing PDFs with testimonials quarantined); Policy #3 "Managed-device-required" High (macOS keychain reset strips cert → silent lockout); Policy #9 "GenAI usage" Medium-High (credit-card-like number in dev sample code) (per skyhigh draft § Configurable policies) |
| ZS | Yes | Policy #4 "Inline DLP for SaaS Upload" High (wide PII dictionaries on chat-style apps fire constantly); [unverified] architect lens finding 6 quantifies: BFSI cost of FP-blocked upload to regulator portal (BNM, SC) is incident-grade |

**Root cause.** DLP classifiers (regex, SIT, ML) are imprecise at the data-character-class level (SSNs look like account numbers, PANs look like long IDs). Compliance/tenant logic depends on tags/labels that lag reality (M&A tenant not yet allowlisted, cert renewal lag). The cost asymmetry of FP-block (user can't do their job) versus FN-allow (low-volume actual breach) heavily punishes aggressive defaults.

**Compensates with (non-CASB).** Phased rollout discipline (Audit → Coach → Block over weeks, not days), per-OU pilot scoping, exception-workflow governance (ticket SLA, approver chain), classifier tuning with EDM/IDM/proximity rather than raw regex, change-management board for the bypass list, and *not buying CASB at all* for the destinations where the FP cost dominates.

**Promote to 08-failure-modes/?** Yes — pattern of FP-driven rollback is well-documented across all five vendors; compensating-control class is operational/process, not tech.

---

## 10. Inline-vs-API policy duplication (the two-console seam)

**Definition.** Inline (forward-proxy / reverse-proxy) and API CASB are separate policy planes. Configuring "block external sharing of PII" typically requires two rules — one inline (pre-upload), one API (post-upload on data at rest). Rules drift; remediation actions differ; one classifier authored once still produces two configuration objects.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Partial | Session policies (CAAC) and File policies (API) are separate objects, but the same Purview DCS sensitive-info-types feed both; lens reviewers flag this as less severe than for PAN/NSK |
| NSK | Yes | "Real-time Protection Policy vs API Data Protection Policy ... are configured separately, which is a common source of drift" (per netskope draft § Deployment modes "Hybrid"); architect lens finding 4: "#1 ops failure mode for Netskope deployments" |
| PAN | Yes (worst) | "Two-console policy duplication for inline vs. API ... requires building the rule twice — once as an inline DLP rule in Strata Cloud Manager (pre-upload) and once as a Data Asset policy in the Cloud Management Console (post-upload). Only Enterprise DLP profiles are shared" (per palo-alto draft Limitations bullet 1) |
| SKY | Yes | "DLP rules and access policies live in adjacent menus and need to be authored against the right object (sanctioned app instance vs. cloud service category)" (per skyhigh draft § Deployment modes "Hybrid"); [unverified] architect lens flags unification as a contractual question, not a fact |
| ZS | Yes | Cloud App Control + SaaS Security API + 3rd-Party App Governance + SSPM + CBI are five separate policy areas; architect lens product finding: practitioners will misnavigate the console |

**Root cause.** Inline and API are different runtime engines with different action vocabularies (inline blocks; API quarantines/notifies/removes-share). Even when DLP classifiers are shared, the policy *object* that binds classifier → scope → action remains plane-specific.

**Compensates with (non-CASB).** Policy-as-code workflow (Terraform, vendor APIs, Git diff between planes), single source-of-truth for DLP classifiers, change-management discipline requiring both planes updated in the same change ticket, periodic drift audits, and explicitly limiting the number of policies per plane so the cognitive load stays manageable.

**Promote to 08-failure-modes/?** Yes — universal architectural property of the inline+API hybrid model; compensating-control class is governance/IaC, not vendor feature.

---

## 11. Gateway encryption / tokenisation has effectively died

**Definition.** The original 2014-era CASB pillar — format-preserving encryption or tokenisation of fields before they land in the SaaS, with the CASB holding keys — is largely absent or partner-dependent in the current product lines.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes | "Encryption / tokenisation at gateway — Not supported" (per MDA draft § Capability set); Purview labels apply post-classification, not at gateway |
| NSK | Partial | Inherits Kindite (2020 acquisition) "searchable encryption / tokenisation for specific fields, primarily Salesforce-flavoured ... Heavy implementation lift; not the common path; ask why before proposing it" (per netskope draft § Capability set) |
| PAN | Yes | "Field-level tokenisation / gateway encryption — No proxy-injected encryption or tokenisation for sanctioned SaaS records" (per palo-alto draft Limitations bullet 8) |
| SKY | Partial | "Inline encryption / DRM via partner integrations (Ionic, Seclore) — applied as a DLP action (Apply Protection) rather than transparent gateway-level field encryption. Native field-level tokenisation (a 2014-era CASB feature) is not foregrounded in current docs; [unverified] whether it still ships as a first-class action" (per skyhigh draft § Capability set, Limitations bullet 7) |
| ZS | Yes | "Encryption / tokenisation at gateway — Not supported as a documented native feature in ZIA CASB" (per zscaler draft § Capability set, Limitations bullet 6) |

**Root cause.** Gateway encryption breaks SaaS-native search, indexing, and integration features; customers stopped buying it once SaaS providers shipped BYOK/HYOK and per-tenant encryption-at-rest. The functional + UX cost no longer matches the residual risk reduction.

**Compensates with (non-CASB).** SaaS-native BYOK/HYOK (Microsoft Customer Key, AWS KMS, Salesforce Shield Platform Encryption), data classification + access-control reducing what data lands in the SaaS at all, third-party tokenisation point-products (CipherCloud-derivative vendors, regulated-data-specific tokenisation services), tokenise-before-write at the application tier.

**Promote to 08-failure-modes/?** Yes — distinct historical-feature-retirement narrative; useful for buyers who still expect this pillar from CASB.

---

## 12. Compromised-account: detection vs response — no native session-kill

**Definition.** CASBs detect impossible-travel, brute-force, anomalous-OAuth signals well. Response — actually terminating the user's session and forcing re-authentication — almost always requires an IdP-side action triggered by SOAR/integration, not a CASB-native primitive.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Best of the five | "Suspend user (via Microsoft Entra ID)" and "Confirm user compromised + Suspend user" are documented governance actions on Activity / Anomaly policies (per MDA draft § Configurable policies #4, #9) — but only because MDA + Entra ID are the same vendor stack |
| NSK | Yes | "Step-up authentication is not native — Netskope can trigger it via IdP (Entra Conditional Access / Okta Adaptive MFA) but does not itself enforce MFA prompts" (per netskope draft Limitations bullet 11) |
| PAN | Yes | "Real-time session-kill on compromised-account detection — Behavior Threats raises incidents; killing the session or forcing reauth requires an IdP-side workflow (Entra ID / Okta) via SOAR (XSOAR)" (per palo-alto draft Limitations bullet 9) |
| SKY | Yes | Cloud Access Policies support "require step-up authentication" but this is a paired Cloud Access Policy enforcement, not native session kill (per skyhigh draft § Capability set) |
| ZS | Yes | "Native step-up MFA from a Zscaler policy decision — authentication lives at the IdP and Zscaler steers but does not authenticate" (per zscaler draft Limitations bullet 5) |

**Root cause.** Session tokens are issued and bound by the IdP. The CASB is in the data path (inline) or audit path (API) but does not hold the session-issuing authority. Killing a session requires revoking at the issuer.

**Compensates with (non-CASB).** IdP continuous access evaluation (Entra ID CAE, Okta CAEP), SOAR runbook that calls IdP revoke API on CASB alert, MDA-style single-vendor stack where CASB and IdP are the same trust domain, Continuous Access Evaluation Protocol (CAEP) — emerging standard for risk-signal-to-IdP feedback.

**Promote to 08-failure-modes/?** Yes — universal except for the single-vendor MDA/Entra case; compensating-control class is identity-platform, not CASB.

---

## 13. Shadow-IT discovery scope is bounded by traffic visibility

**Definition.** Network-log-based shadow-IT discovery only sees what the CASB's data path or log feed sees. Off-VPN, off-agent, split-tunnel-excluded, or non-PAN-network traffic produces no signal. Vendor lock-in on log sources reduces this further for ecosystem-bound vendors.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Best | Defender for Endpoint integration extends discovery to roaming endpoints (per MDA draft § Capability set "Shadow-IT discovery"); also accepts log uploads from Zscaler/iboss/Corrata/Menlo and 25+ firewall/proxy types — broadest third-party log ingestion of the five |
| NSK | Yes | Ingests logs from PAN, Cisco, Fortinet, Check Point, Zscaler, Forcepoint, generic syslog/CEF (per netskope draft § Deployment modes "Log-based discovery") — broad |
| PAN | Yes (worst) | "Shadow-IT discovery on non-Palo-Alto network paths ... ingests logs only from Prisma Access and PAN NGFWs via Strata Logging Service. No generic syslog/CSV ingestion path comparable to Netskope CCI or Zscaler ShiftLogs — discovery is locked to the PAN data plane" (per palo-alto draft Limitations bullet 2) |
| SKY | Yes | [unverified] "Shadow-IT log-source list is not enumerated in the public docs" (per skyhigh architect lens); references syslog/SCP/FTP/file-share ingest via Skyhigh Cloud Connector but no definitive supported-vendor list |
| ZS | Yes | "Shadow-IT discovery for users not steered through Zscaler ... off-VPN, off-Client-Connector traffic does not appear" (per zscaler draft Limitations bullet 12); does not ingest third-party logs at all |

**Root cause.** Discovery is downstream of telemetry. If the CASB is not in the data path and does not ingest external logs, the activity is dark.

**Compensates with (non-CASB).** Endpoint-agent-based discovery (browser extension, EDR cloud-app dictionary), third-party SaaS discovery point-tools (Productiv, Zylo, Torii), expense/finance-data SaaS discovery (matching corporate-card transactions against a SaaS catalogue), IdP-based discovery (SCIM / SSO connections as a proxy for sanctioned-app inventory), DNS-based discovery (DNS-resolver telemetry).

**Promote to 08-failure-modes/?** Yes — structural property of all network-based discovery; the PAN-lock-in subcategory is a distinct vendor-procurement risk.

---

## 14. Vendor / platform lock-in inside the SSE bundle

**Definition.** The "CASB inside SSE" packaging means CASB is no longer a discrete decision layer — it inherits the SSE's identity bindings, agent footprint, region constraints, licensing tiers, and outage blast radius.

| Vendor | Exhibits | Notes |
|---|---|---|
| MDA | Yes (severe) | Tightly coupled to Entra ID + Defender XDR; Conditional Access App Control "is literally Conditional Access App Control, an Entra feature" (per MDA draft § Lineage); architect lens finding 1: "It is not a deployable CASB peer to Netskope/Zscaler — it is an Entra-bound API+IdP-redirect overlay" |
| NSK | Yes | "There is no stand-alone CASB SKU separate from the SSE tenant ... CASB Inline + Next-Gen SWG share the same forward proxy and the same DLP engine — there is no separate 'SWG' data path" (per netskope draft § Lineage, architect lens finding 9) |
| PAN | Yes | "Inline CASB is not a separable SKU — it sits inside ZIA's traffic flow" equivalent: SaaS Security Inline sits inside Prisma Access; SaaS Security API sits in Strata Cloud Manager; "Inline and API are separately licensed and separately configured" (per palo-alto draft § Lineage, § Deployment "Hybrid") |
| SKY | Least locked-in | "Last credible stand-alone enterprise CASB lineage" — sold as standalone SKU or inside SSE bundle (per skyhigh draft § Lineage); but customer base shrinking and consoles drifted across McAfee → Trellix → Skyhigh transitions |
| ZS | Yes | "CASB is not a separable SKU end-state — it sits inside ZIA's traffic flow (inline) and ZIA's API connectors (out-of-band). Practitioners license it as part of ZIA, not as a stand-alone CASB" (per zscaler draft § Lineage); architect lens finding: "one outage, all CASB enforcement gone" |

**Root cause.** Vendor commercial strategy: SSE bundles increase ACV and reduce competitive shopping. The CASB capability follows the bundle, not the buyer's architecture preference.

**Compensates with (non-CASB).** Procurement-side controls: explicit clauses on data portability (SCIM/SAML/SIEM connector ownership), DLP-classifier export format (regex + EDM/IDM templates portable between vendors), policy-as-code with vendor-neutral abstraction layer, multi-vendor pilots before lock-in. Strategically: accept the lock-in and demand single-vendor breadth (MDA+Entra+Purview, ZS full-stack) or fight it and pay best-of-breed integration tax (Skyhigh + separate IdP + separate SWG).

**Promote to 08-failure-modes/?** Yes — strategic / procurement failure mode distinct from any technical one; useful framing for buyers.

---

## Categories that did NOT make the cut

- **"Watermarking is missing/limited"** — true across MDA (none native; Purview labels lack visual markings — per MDA draft Limitations bullet 6), NSK (RBI-only, not native proxy — per netskope draft Limitations bullet 12), PAN (no native — per palo-alto draft Limitations bullet 7). Real but narrow; subsumed under #11 "gateway-control pillars eroding" if 08-failure-modes/ wants depth here.
- **File-size ceiling on API scan** — PAN documents a 25 MB hard limit (per palo-alto draft Limitations bullet 3); MDA and others have content-inspection limits but less explicit. Operational annoyance rather than failure-mode category.
- **Endpoint DLP gap** (USB, clipboard, screenshot, print) — all five drafts acknowledge CASB is a network+SaaS plane, not endpoint (per skyhigh draft Limitations bullet 10, zscaler draft Limitations bullet 8). True but obvious; belongs in scope-definition, not failure-modes.

---

## Promotion summary

| # | Category | Promote to 08-failure-modes/ | Notes |
|---|---|---|---|
| 1 | BYOD / unmanaged-endpoint blind spot | Yes | Universal |
| 2 | Mobile-native-app bypass | Yes | Distinct from #1 — compensates with MAM, not MDM |
| 3 | SSL/TLS inspection prerequisite + bypass list | Yes | Structural to inline model |
| 4 | API-mode is post-event, not prevention | Yes | Structural to SaaS API design |
| 5 | OAuth-grant blind spot (consumer identity) | Yes | Universal, identity-policy compensates |
| 6 | SaaS-to-SaaS data flow | Yes | Most under-documented; surface in synthesis |
| 7 | User-applied encryption / E2EE / password archives | Yes | Cryptographic root cause |
| 8 | SaaS coverage outside connector list | Yes | Build-economics root cause |
| 9 | Over-blocking / false-positive blast radius | Yes | Operational pattern; process compensates |
| 10 | Inline-vs-API policy duplication | Yes | Universal architectural property |
| 11 | Gateway encryption / tokenisation has died | Yes | Historical-pillar retirement narrative |
| 12 | Compromised-account detection vs response | Yes | IdP-platform compensates |
| 13 | Shadow-IT discovery bounded by traffic visibility | Yes | PAN lock-in sub-pattern is distinct |
| 14 | Vendor / SSE bundle lock-in | Yes | Procurement failure mode, not technical |

All 14 categories promote-eligible. If 08-failure-modes/ should stay tight at 8–10, recommended cuts (in order): #11 (narrow narrative), #13 (overlaps with #1 — both about visibility), #14 (more strategy than failure-mode), #6 (least vendor-corroborated).
