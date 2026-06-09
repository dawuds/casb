# Microsoft Defender for Cloud Apps — practitioner's playbook

> Draft v0 — written 2026-06-10. Synthesises the vendor draft + three-lens reviews from the workflow run earlier today. Still in `_research/`; another three-lens pass should happen before promotion to `04-vendors/microsoft-defender-for-cloud-apps.md`.
>
> **Read this if:** your firm already has Microsoft Defender for Cloud Apps (MDA) — typically because you're licensed for **Microsoft 365 E5** or you've added the standalone MDA SKU — and you want to know what to turn on, in what order, and what risk each switch actually reduces.
>
> Tone: terse practitioner. No vendor marketing. Where MDA cannot do something, that's stated; where a configuration has a known footgun, it's named.

---

## What MDA is *for*

In one sentence: **MDA is the Microsoft Defender layer that watches and governs your sanctioned SaaS apps via API connectors, runs Shadow IT discovery on your egress traffic, surfaces and revokes risky third-party OAuth grants on Microsoft 365 / Google Workspace / Salesforce, and — when paired with Entra ID Conditional Access — intercepts browser sessions to those apps to enforce in-session DLP and download/upload rules.**

It is **not** a CASB peer to Netskope, Zscaler, or Skyhigh in the architectural sense — there is no Defender-for-Cloud-Apps endpoint agent, no PAC file, no SSL-inspection appliance. The "proxy" piece is **Conditional Access App Control (CAAC)**: an Entra-mediated reverse-proxy where browser sessions to sanctioned apps are redirected through Microsoft's proxy at a `*.mcas.ms` URL, TLS is terminated there, and per-app session controls (block download, block upload, block paste, apply sensitivity label, etc.) are applied to the rewritten session. **Strip Entra ID P1 out of the licensing, and MDA collapses to API connectors + Shadow IT discovery — about half of what the marketing implies.**

### What it covers well

- **Sanctioned SaaS sitting behind Entra ID** — Microsoft 365, plus a defined list of API-connected apps (Google Workspace, Salesforce, Box, Dropbox, ServiceNow, Workday, Webex, GitHub Enterprise, Okta, AWS / GCP / Azure resource activity, Atlassian). M365 is first-class; everything else is connector-by-connector with varying governance-action depth.
- **Shadow IT discovery** at the network egress layer — through Defender for Endpoint feed (for roaming endpoints), a log collector container fed Syslog from your SWG/firewall, or direct integrations with Zscaler / iboss / Corrata / Menlo. ~31,000-app catalogue (Microsoft's count, vendor-self-reported).
- **OAuth-app governance** for delegated permissions granted by users to third-party apps in M365 / Workspace / Salesforce — discovery, ban, per-user revoke, automatic anomaly detection on misleading-publisher / malicious-consent.
- **In-session DLP and DRM-style controls on browser sessions** — but only for apps onboarded to CAAC, and only on browsers (and the subset of native apps that authenticate via the IdP).
- **GenAI app discovery** — categorical visibility into 1,000+ generative-AI apps in the catalogue, plus the Microsoft Purview "DSPM for AI" overlay that can surface the actual prompts/responses to ChatGPT Enterprise / Gemini / supported AI sites (privacy implications — see Day 90 below).
- **UEBA / anomaly detection** — built-in policies for impossible travel, mass download, terminated-user activity in connected apps, suspicious OAuth grants.

### What it does NOT cover (state plainly to anyone buying it)

1. **BYOD on iOS / Android = effectively uncovered.** Mobile native apps (Outlook Mobile, Teams Mobile, OneDrive sync, third-party mail clients) authenticate through their own flows and bypass CAAC. The compensating control is **Intune App Protection Policies (MAM) + Entra Conditional Access**, *not MDA*. If your BFSI risk concentration is mobile-BYOD, MDA is not the control.
2. **Apps not onboarded to Conditional Access App Control = no inline DLP, no session-mode controls.** Only the apps you've onboarded to CAAC get session policies. Everything else is API-mode (post-upload) or invisible.
3. **API-mode is post-upload, not prevention.** Policy #2 (auto-label PCI), Policy #4 (mass-download), Policy #10 (quarantine externally-shared), Policy #11 (malware on Box/Dropbox/Google), Policy #12 (terminated-user activity) all detect *after* the fact — typical M365 connector latency is <15 minutes, Power BI / Dynamics 365 is 24–72 hours.
4. **OAuth Application permissions (client-credentials / app-only)** are NOT surfaced in the OAuth apps page. Only delegated grants are visible. Service-principal consents — the higher-risk class for exfil — need Defender XDR / app governance / Defender for Cloud to govern.
5. **Watermarking on downloaded files.** The "Protect on download" action applies a Microsoft Purview sensitivity label (which can carry encryption + permissions) but **does not apply visible headers, footers, or watermarks**. Source: Microsoft Learn, session-policy docs.
6. **Real-time blocking of malware in Box / Dropbox / Google Workspace.** Detects, doesn't auto-block — third-party-app remediation only. Only OneDrive / SharePoint files are scanned by Microsoft's own service before they land.
7. **Forward-proxy-style traffic interception.** No PAC, no agent, no SSL bump. MDA does not see traffic to non-onboarded SaaS in inline mode — Defender for Endpoint feeds *discovery* data, not enforcement.
8. **Certificate-pinned apps.** CAAC's TLS interception fails on cert-pinned mobile and desktop apps — they refuse the connection or silently bypass.
9. **A policy-as-code interface.** Policies are clickops-only. No Terraform provider, no documented REST API for creating session/access/file policies. Configuration changes are manual and audit-traceability through change management is harder.
10. **>50 file policies per tenant.** Hard ceiling. Regulated FIs with multi-classification × multi-app needs hit this within months. Plan policy consolidation strategy upfront.
11. **Office 365 Cloud App Security (OCAS) — the O365 E5 subset SKU.** If you're licensed O365 E5 but *not* M365 E5, you get OCAS: M365 connector only, no third-party SaaS connectors, no Conditional Access App Control beyond Office apps, no app governance, no GenAI category, no SSPM. ~80% of this playbook does not apply. Check the licence before building runbooks.
12. **Tenant primary-data region.** Defender for Cloud Apps tenants land in EU / US / UK / AU (paraphrased — confirm against the Microsoft Trust Center for your tenant). There is **no APAC primary region as of writing**. For Malaysian / Singapore / Hong Kong tenants this is a cross-border-data-transfer determination under PDPA (MY 2010 Act + 2024 amendments) / PDPA (SG) / HKPCPD — material to data-residency assessments. **Verify via the Trust Center before committing.**

### What it touches (you must accept these dependencies)

| Component | Why it matters |
|---|---|
| **Entra ID P1** | Mandatory for CAAC (session/access policies). Without it, no inline DLP, no download blocks, no Conditional Access App Control. |
| **Microsoft Defender XDR** | MDA alerts surface in the unified Defender portal at `security.microsoft.com → Cloud apps`. Without DfE / DfI / DfO, MDA's XDR incident correlation is shallow and alerts are noisier. |
| **Microsoft Purview (DLP + sensitivity labels)** | Auto-labelling, encryption on labels, DLP signatures via the Data Classification Service — all live in Purview. MDA's DLP enforcement is a third control point on top of Purview Endpoint DLP and Purview service-side DLP. Plan ownership: who configures the labels (usually Information Protection team in Purview), who configures the MDA policies that enforce them. |
| **Defender for Endpoint (MDE)** | Required for off-corporate-network Shadow IT discovery on roaming endpoints; required for actual *enforcement* of the "tag GenAI app as Unsanctioned → block" pattern (via MDE Network Protection — must be in block mode, not audit mode). |
| **Microsoft Sentinel / Log Analytics** | The default 30-day Activities log retention in MDA is insufficient for BFSI audit retention requirements (typically 1–7 years). Plan to ship logs to Sentinel or Log Analytics for the retention you need — additional cost. |
| **Conditional Access (Entra)** | Every CAAC session policy in MDA requires a matching Entra Conditional Access policy with "Use Conditional Access App Control" as the session control. **Two consoles. Two policy objects. Both required.** This is the single most common reason "I configured everything in MDA and nothing happened." |

### Concentration risk

A regulated FI running M365 + Defender XDR + Entra + Purview + MDA is single-vendor for IdP, EDR, CASB-equivalent, DLP, and (with Global Secure Access) SSE. This is a textbook concentration-risk pattern under BNM RMiT (third-party / outsourcing sections — verify against the current edition), MAS Outsourcing, and EU DORA's concentration-risk recitals. **Promote MDA as a primary control with this risk on the register.**

---

## Day 1 — turn-on essentials

Configure these four within the first two weeks of go-live. They reduce the highest-likelihood incident classes with the lowest risk of breaking business operations. None requires CAAC (Conditional Access App Control); all are API-mode or built-in detections.

### Policy 1 — Shadow IT discovery + GenAI category triage

| | |
|---|---|
| **Risk reduced** | Unsanctioned SaaS / unsanctioned GenAI in use without IT awareness. Without this you cannot enumerate the SaaS apps your employees actually use, cannot make risk-acceptance decisions about them, and cannot meet ISO 27001 / 27017 / NIST CSF ID.AM-04 expectations on SaaS inventory. |
| **What it does** | Ingests network-egress logs (via the MDE feed, the log-collector container fed from your SWG/firewall, or a direct Zscaler / iboss / Corrata / Menlo integration) and matches activity against the cloud-app catalogue. Surfaces each discovered app with risk score, user count, and traffic volume. |
| **Console path** | **Microsoft Defender Portal → Cloud apps → Cloud discovery**. Configure data sources at **Settings → Cloud apps → Cloud discovery → Automatic log upload**. |
| **Key configuration** | Connect at least one data source (start with MDE for roaming visibility). Sanction your known-approved apps (tag = Sanctioned). For unknown apps, leave untagged for review. **Subscribe to weekly discovery report** for the security analyst queue. |
| **Trap to avoid** | The "discovered subdomains" capability **deprecated 2025-12-31**. Any control evidence you built on subdomain-level Shadow IT differentiation (e.g. distinguishing your own SharePoint Online tenant from a rogue tenant) is no longer reliable. Re-baseline before any compliance attestation cycle. |
| **Auditable evidence** | `CloudAppEvents` table in Advanced Hunting; default retention 30 days unless forwarded to Sentinel. |

### Policy 2 — OAuth app discovery + ban known-malicious

| | |
|---|---|
| **Risk reduced** | **OAuth-grant exfiltration.** A user clicks Consent on a malicious third-party app and the app exfiltrates mail, files, or contacts via Graph. This bypasses every network-layer control because the traffic is app-to-Microsoft, not user-to-anywhere. The 2022 mass-OAuth-phishing campaigns made this a board-level risk. |
| **What it does** | Catalogues every OAuth app that has been granted delegated permission to M365 / Workspace / Salesforce. Built-in anomaly detection fires on *Misleading OAuth app name*, *Misleading publisher name*, *Malicious OAuth app consent*. You can Approve, Ban (tenant-wide kill), Revoke (per-user), or Report-to-Microsoft each app. |
| **Console path** | **Cloud apps → OAuth apps** (or **App governance** if you've enabled it as a separate add-on). Built-in anomaly policies are under **Cloud apps → Policies → Policy management → Threat detections**. |
| **Key configuration** | Day 1: review every High-risk-permission, low-community-use, unverified-publisher app. **Manually ban known-malicious. Do NOT enable auto-revoke on the built-in anomaly policy until you've measured FP rate for at least 4 weeks.** Restrict user consent to verified-publisher apps (Entra ID consent governance — outside MDA, but configured in tandem). |
| **Trap to avoid** | **Banning is tenant-wide and immediate.** Every user who consented loses access at the next token refresh. There's no staged rollout. Practitioners ban a flagged app at 11pm and wake up to 40 broken integrations. *Before banning, query Advanced Hunting for the user-count and the apps' actual usage.* |
| **Critical limitation** | **Application (client-credentials / app-only) OAuth grants are NOT in this view.** Service-principal consents — admin-granted, unattended, broad-scope — are the higher-risk class and need Defender XDR / app governance / Defender for Cloud. State this on the risk register as a residual gap. |
| **Auditable evidence** | `CloudAppEvents` records the grant / revoke; `AuditLogs` (Entra) records the consent action by admin/user. |

### Policy 3 — Mass-download anomaly with alert (NOT auto-suspend) on M365

| | |
|---|---|
| **Risk reduced** | **Insider exfiltration via bulk download** before resignation; or **compromised-account exfiltration** by an attacker with valid creds. The standard pattern: 1,000+ files downloaded from SharePoint / OneDrive in a short window by a single user. |
| **What it does** | Built-in activity-policy template, customised to your environment. Detects per-user volume + velocity of download events from connected apps. |
| **Console path** | **Cloud apps → Policies → Policy management → Threat detections tab → Create policy → Activity policy**. |
| **Key configuration** | *Activity type* = Download file; *Apps* = SharePoint, OneDrive, plus any other connected app where bulk download is plausible; *Match parameters* = Repeated activity, count > 5,000 in 30 minutes by same user, in a single app. **Action: Alert + email to SOC. Do NOT enable auto-suspend on the first turn-on.** Exclude service accounts and known migration accounts by UPN. |
| **Trap to avoid** | **The "Suspend user" governance action disables the Entra user object — it is NOT a soft suspension.** Downstream cascade: SCIM-deprovisioning waves into every connected app, broken Exchange mailflow if the user is a shared-mailbox delegate, broken Power Automate flows under that account, SOC cannot un-suspend cleanly without resetting credentials. **Practitioner discipline:** auto-suspend only after correlated signal (Entra ID Protection sign-in risk = High AND mass-download policy hit AND not on the frequent-traveller / known-migration exclusion list). |
| **Auditable evidence** | `CloudAppEvents` carries the download events; the policy match event carries policy ID and user ID. Map to **NIST CSF DE.CM-03 (personnel activity monitoring)** and **BNM RMiT** logging/monitoring expectations (verify against the current edition). |

### Policy 4 — Impossible travel + activity from infrequent country, alert-only

| | |
|---|---|
| **Risk reduced** | **Account compromise** — particularly password-spray / credential-stuffing scenarios where the attacker is in a different geography than the legitimate user. |
| **What it does** | Built-in UEBA anomaly. ML-based, ignores VPNs and tenant-common locations (Microsoft's claim — independent FP rate not published). Country-level granularity only. |
| **Console path** | **Cloud apps → Policies → Policy management → Threat detections → Anomaly detection policy → Impossible travel**. Built-in; edit, don't create. |
| **Key configuration** | Sensitivity = Low or Medium on day one. Scope = all users; exclude a managed "Frequent travellers" group (maintained separately, with documented owner and review cadence). **Action: Alert only. Do NOT auto-suspend.** Pair with Entra ID Protection sign-in risk for higher-fidelity signal. |
| **Trap to avoid** | **Country-level only — no alerts within the same country or between bordering countries.** For Malaysia / Singapore / Hong Kong — impossible travel between KL and SG produces *no alert*. For an MY FI with cross-border insider scenarios this is a material gap; the compensating control is Entra ID Protection user-risk + sign-in-risk policies, not MDA. |
| **Auditable evidence** | `AlertInfo` + `AlertEvidence` (Defender XDR) carries the alert; default retention 30 days unless forwarded to Sentinel. |

---

## Day 30 — hardening

These three are higher-impact but riskier to misconfigure. Run them in audit mode (or "alert only") for at least 2 weeks before moving to enforce. All three require Conditional Access App Control (CAAC) and therefore Entra ID P1.

### Policy 5 — Block download of sensitivity-labelled files to unmanaged browser sessions on M365

| | |
|---|---|
| **Risk reduced** | **Sensitive-document exfil via personal device.** An employee signs into SharePoint from a personal laptop / home browser, downloads Confidential files. Without this policy, the only friction is "should I have done that?" — there is no technical block. |
| **What it does** | When the user signs in to M365 from a non-Entra-hybrid-joined and non-Intune-compliant device (the device-tag definition of "unmanaged"), allow in-session view/edit but block download of any file labelled Confidential or above. |
| **Console path** | **Cloud apps → Policies → Policy management → Conditional Access tab → Create policy → Session policy**. PLUS a matching **Entra Conditional Access policy** with "Use Conditional Access App Control" as the session control on the same user / app scope. **Both required. Two consoles. Without the Entra side, the MDA side never fires.** |
| **Key configuration** | Session control type = *Control file download (with inspection)*. App = Office 365 (use the bundle, not individual apps — picking individual apps causes inconsistent CAAC interception). Client app = Browser only (omit "Mobile and desktop" to avoid breaking Outlook / OneDrive sync clients). Device filter = Device tag ≠ MicrosoftEntraHybridJoined AND ≠ IntuneCompliant. File filter = Sensitivity label ∈ {Confidential, Highly Confidential}. Action = Block + customised user message linking to your "how to enrol your device" page. **`Always apply if data cannot be scanned` = false** (default fail-open on unscannable; the trade-off is data leakage on unscannable files vs blocking everything — document the decision). |
| **Trap to avoid** | **The `*.mcas.ms` URL rewrite breaks things.** Once CAAC is on, browser bookmarks to canonical SaaS URLs may break; internal SIEM rules pattern-matching `sharepoint.com` need updating; some SaaS-to-SaaS OIDC flows initiated inside a rewritten session fail; some apps have hardcoded canonical URLs or anti-CSRF tokens that don't survive the rewrite. **Treat every CAAC onboarding as a per-app regression test event, not a config change.** |
| **BYOD note** | This is "block download from BYOD." Some BFSI users (senior execs on iPad) will hit it immediately. Pair with a parallel **read-only access** session policy that allows view-only on unmanaged devices, rather than full block, for that population. |
| **Auditable evidence** | The block action lives in `CloudAppEvents` (MDA's session-policy event) AND in **Entra ID Sign-in logs → Conditional Access** (the CA policy result that triggered the redirect). **An auditor needs both correlated** — your SIEM query must join them by session correlation ID. |

### Policy 6 — Auto-label PCI-containing files in OneDrive / SharePoint with *Confidential — Finance*

| | |
|---|---|
| **Risk reduced** | **Cardholder-data sprawl.** Even with PCI DSS scope discipline, credit-card numbers end up pasted into spreadsheets, dragged into OneDrive, emailed to themselves. This policy continuously finds them and applies the Confidential — Finance label (which itself carries encryption + access restrictions, via Purview label config). |
| **What it does** | API-mode (file policy). Scans existing and new files in OneDrive / SharePoint, runs Microsoft Purview Data Classification Service (DCS) on the content, applies the sensitivity label automatically when *Credit Card Number* matches. |
| **Console path** | **Cloud apps → Policies → Policy management → Information protection tab → Create policy → File policy**. Requires the Purview sensitivity label already configured under **Microsoft Purview → Information Protection → Labels**. |
| **Key configuration** | App filter = OneDrive for Business + SharePoint Online. **Parent folder scope** — narrow to your finance / cardholder data folders first; do NOT run tenant-wide on day one (DCS quota + matching-content storage). File filter = Inspection method = Data Classification Service, SIT = Credit Card Number, minimum violations = 5 (not 1 — 1 produces too many false positives on test data / mock cards). Governance action = Apply Microsoft Purview sensitivity label → *Confidential — Finance*. Alert + email per match. **Run for 1 week with action = Alert-only first; flip to label-apply after FP review.** |
| **Trap to avoid** | The **matching-content preview snippet** (the 100-character before/after window the policy stores for triage) is itself cardholder data. It lives in Defender's incident metadata. Under PCI DSS 3.4 / 3.5 the storage and access control around that snippet is itself a regulated control. State the snippet retention and access list in your control documentation. |
| **50-policy ceiling** | This is policy 1 of 50. A tenant with multi-classification × multi-app needs typically wants ~30+ file policies for the M365 stack alone. Consolidate per classification family (not per individual SIT) and document the consolidation as a control-design risk acceptance. |
| **Auditable evidence** | Label-application event in M365 unified audit log (look for `MIPLabel`) + MDA file-policy match event in `CloudAppEvents`. Auditor will ask: prove the label was applied at T0, by which policy version. |

### Policy 7 — Quarantine externally-shared OneDrive files with no recent activity

| | |
|---|---|
| **Risk reduced** | **Sharing drift.** A user shares a contract externally for one deal, the deal closes, the share link stays. Over years, hundreds of stale external shares accumulate. Each is a small leak. This policy catches them. |
| **What it does** | API-mode (file policy). Identifies OneDrive files shared externally that haven't been modified in 6+ months. **Two-stage flow recommended:** alert the owner with a 14-day grace period, then quarantine (move to admin-owned location) on no-response. |
| **Console path** | **Cloud apps → Policies → Policy management → Information protection → Create policy → File policy**. |
| **Key configuration** | App filter = OneDrive for Business. Access level = External OR Public-with-a-link OR Public. Last modified = >180 days ago. **Stage 1 policy:** governance action = Notify owner; severity = Medium. **Stage 2 policy** (separate file policy, 14 days later): governance action = Put file in admin quarantine; severity = High. Exclude folder allowlist for known-shared content (`Parent folder ≠ /Shared Templates`, etc.). |
| **Trap to avoid** | **Single-step quarantine breaks links for internal users who still reference the file**, breaks external recipients without warning, and the owner-notification email is generic and often lands in junk. Always run the notify-then-quarantine two-stage flow. **Also: a quarantined file under Microsoft Purview eDiscovery legal hold can be spoliation** — Purview hold wins, but your MDA operator may not know about the hold at policy-design time. Coordinate with Legal. |
| **Auditable evidence** | `CloudAppEvents` records the governance action; the M365 unified audit log records the file move. |

---

## Day 90 — advanced

These are higher-blast-radius and require operational maturity (SOC integration, exception process, change management). Don't enable until the Day 1 + Day 30 set is stable and tuned.

### Policy 8 — Auto-unsanction Generative-AI apps above risk threshold (with caveats)

| | |
|---|---|
| **Risk reduced** | **Data leakage to consumer-tier GenAI.** Employees paste source code, customer data, financial models into ChatGPT / Bard / Claude / Perplexity. Consumer-tier accounts train on the content. Once leaked, the data is in the model. |
| **What it does** | Continuously discovers GenAI apps from the cloud discovery stream. When an app in the Generative AI category exceeds your risk threshold (or user / transaction threshold), automatically tags it as Unsanctioned, which propagates to MDE-managed endpoints and blocks it via MDE Network Protection. |
| **Console path** | **Cloud apps → Policies → Policy management → Shadow IT tab → Create policy → App discovery policy**. |
| **Key configuration** | Category = Generative AI. Risk score ≤ 5 (high-risk). Users per day > 10 (avoid one-off curiosity hits). Action = Tag as Unsanctioned + Alert. Build a parallel allowlist of approved AI vendors (your enterprise ChatGPT Enterprise, Copilot, internal AI services) — exclude by app ID. |
| **Prerequisites you must verify before enabling block** | (a) Defender for Endpoint deployed and reporting to the same tenant. (b) **MDE Network Protection in block mode, not audit mode** — this is the single most-missed prerequisite; without it, "Tag as Unsanctioned" results in *nothing* happening at the endpoint. (c) Windows 10 1709+ or supported macOS builds. (d) Endpoints actually connected to MDE policy. |
| **Trap to avoid** | The **"Generate block script for SWG"** export produces a flat IP/domain list that does not match modern GenAI vendor egress patterns (CDN-fronted, dynamic domains, JA3-fingerprinted). The script will partially fail at the SWG. Don't sell this as a complete enforcement chain. **Pair with SWG-native AI-category enforcement (Zscaler / Netskope / Palo Alto have these) for actual coverage.** |
| **Risk-acceptance note** | Microsoft's risk score is opaque (90+ factors, weights not disclosed, not industry-standard). An auditor cannot accept it as the sole basis for an automated enforcement decision. Document a parallel internal risk-acceptance / appeal process for when an approved-by-business AI vendor scores ≤5 (e.g. due to "SOC 2 = No" in Microsoft's catalogue). |
| **Auditable evidence** | App-tag-change event in `CloudAppEvents`; MDE Network Protection block events at the endpoint. |

### Policy 9 — Block clipboard paste of sensitive data into ChatGPT browser session

| | |
|---|---|
| **Risk reduced** | **Targeted GenAI data leakage.** Even if you've allowed ChatGPT Enterprise for the firm, you may want to block clipboard paste of cardholder data, SSN-class PII, source code, or internal project codenames. This policy enforces at the browser session layer. |
| **What it does** | CAAC session policy on the ChatGPT app. Blocks the *Paste* clipboard activity (and optionally *Cut* / *Copy* / *Sent message*) when the content matches a DLP signature. |
| **Console path** | **Cloud apps → Policies → Policy management → Conditional Access tab → Create policy → Session policy**. Plus the matching Entra CA policy with "Use Conditional Access App Control." |
| **Critical prerequisite** | **ChatGPT must be onboarded to CAAC via SAML federation. Consumer ChatGPT (chat.openai.com with a free or personal Plus account) does NOT support SAML federation. This policy is deployable ONLY against ChatGPT Enterprise or ChatGPT Team plan with SSO configured.** Without the prerequisite, this policy is unbuildable. |
| **Key configuration** | Session control type = *Block activities*. App filter = Custom (ChatGPT Enterprise, Manual onboarding). Activity type = Clipboard Paste (and Sent message, with DLP inspection). Inspection method = Data Classification Service; SIT = Credit Card Number, SSN, PII; custom SIT for internal project codenames. Action = Block with custom user-education message. **Start with `Audit` action, escalate to `Block` per SIT after FP tuning.** |
| **Trap to avoid** | This policy only covers the **browser session on the CAAC-onboarded ChatGPT URL.** Opening ChatGPT in a different browser, in the desktop app (Mac / Windows ChatGPT clients now exist), on a personal device, or via a personal account = bypass. State the residual risk on the register. Also: clipboard inspection produces a **lot** of noise initially — code snippets, 16-digit identifiers, employee IDs all trigger Credit Card SIT. The privacy treatment is also non-trivial — clipboard content inspection is high-intrusion processing under GDPR Art. 35 / equivalent. DPIA before pilot. |
| **Auditable evidence** | Session-policy match event in `CloudAppEvents`; depending on configuration, the truncated content snippet may also be retained — which is itself regulated content. |

### Policy 10 — Terminated-user activity in connected SaaS (offboarding-gap detection)

| | |
|---|---|
| **Risk reduced** | **Offboarding gap exploitation.** A user is terminated and their Entra account deleted, but they had a secondary local account in a connected SaaS app (AWS, Salesforce, GitHub) under a different email format. The local account stays active and the ex-employee continues to access. |
| **What it does** | Built-in anomaly detection (*Activity performed by terminated user*). Detects when an Entra-deleted account corresponds to a still-active account in a connected SaaS app. Requires Entra as the IdP for the deletion signal. |
| **Console path** | **Cloud apps → Policies → Policy management → Threat detections → Anomaly detection policy → Activity performed by terminated user**. Built-in; edit, don't create. |
| **Key configuration** | Scope = all users. Governance action per app = Notify admin; manual revoke (no auto-suspend for non-Entra accounts — the suspend action only works for Entra accounts). Severity = High. Alert email to identity admin + SOC. |
| **Trap to avoid** | Correlation between Entra-deleted-user and SaaS-app-active-user depends on a **shared identifier — usually email**. If the user's Entra UPN is `user.surname@corp.com` but their Salesforce account is `surname.user@corp.com`, no match, no alert. **Practitioner discipline:** at offboarding, run a manual SaaS-account audit for the user across all connected apps via Advanced Hunting query against `CloudAppEvents`, not relying solely on this anomaly policy. |
| **Auditable evidence** | Anomaly alert in `AlertInfo` / `AlertEvidence`. Maps to **NIST CSF PR.AA-05 (access permissions)** and BNM RMiT / MAS TRM identity management expectations (verify clauses against current editions). |

---

## Residual risks MDA does NOT address (compensating controls required)

| Residual risk | Why MDA can't | Compensating control |
|---|---|---|
| **BYOD-mobile data exfil** | CAAC only protects browser sessions and IdP-authenticating native apps. Mobile native (Outlook Mobile, Teams Mobile, OneDrive sync, third-party email) bypass. | **Intune App Protection Policies (MAM)** for selective wipe + copy-paste controls; Entra Conditional Access requiring approved client app. |
| **Cert-pinned mobile / desktop apps** | CAAC's TLS termination fails on pinned apps. | Manage exposure at the identity layer (Entra CA) and the endpoint layer (Intune compliance) rather than in-session. |
| **OAuth application (client-credentials) grants** | Only delegated grants are surfaced. | **Microsoft Defender for Cloud (CNAPP)** / **App governance** add-on / Entra Workload Identities + Conditional Access for Workload Identities. |
| **Email body content DLP in Exchange** | MDA can policy on attachments, not message body. | **Microsoft Purview DLP (Exchange policy)** — different product surface. |
| **Field-level tokenisation at the gateway** | MDA does not encrypt SaaS field data at the proxy. | If you need this — separate tokenisation product (CipherCloud-style). Most BFSI use Purview encryption labels as a softer equivalent. |
| **Native browser isolation / RBI** | MDA has none. | Zscaler / Netskope / Palo Alto Prisma all have native RBI; consider their SSE alongside MDA, or **Microsoft Edge for Business** with isolation profiles. |
| **SaaS Security Posture Management (SSPM) depth** | MDA does configuration assessment for a handful of apps (Salesforce, ServiceNow, GitHub, Okta, Box, Dropbox, Zendesk). Depth is below SSPM specialists. | If your concern is SaaS misconfiguration drift, look at **Adaptive Shield / AppOmni / Obsidian / Wing** alongside MDA. |
| **Policy-as-code / Terraform-managed CASB config** | Clickops only. No documented REST API for creating session/access/file policies. | Manual change-management process with annual policy review. Export portal config snapshots on a schedule. |
| **>30 day Activities retention** | Default in-product retention is 30 days. | **Forward to Sentinel / Log Analytics** for 1-7 year retention. Budget for the storage cost. |
| **Real-time DLP on Teams chat body** | MDA does not inspect chat-message body content. | Microsoft Purview DLP for Teams. |
| **Structured-data DLP inside SaaS** (Salesforce object-level) | MDA does not inspect SaaS database rows/columns. | Salesforce Shield (Salesforce-native); SSPM tools where supported. |

---

## What this playbook does NOT cover (out of scope)

- **Detailed control mapping to BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS clauses.** The compliance lens reviewer flagged this as mandatory before promotion to canonical `04-vendors/` but the mapping is a separate deliverable; capture it in `06-compliance/microsoft-defender-for-cloud-apps-control-mapping.md` once the per-vendor file is promoted. Material on regulatory clauses is illustrative; not legal/regulatory advice — confirm against gazetted text and your assurance lead.
- **Third-party attestation citations** — SOC 2 Type II, ISO 27001:2022, ISO 27017, ISO 27018, CSA STAR, FedRAMP, C5, IRAP, MTCS, OSPAR. Pull from the **Microsoft Service Trust Portal** before relying on any of these in a regulator submission.
- **Sub-processor list and change-notification SLA** — required for ISO 27018 A.11. Pull from the Microsoft Trust Center.
- **DPIA / TIA scaffolding** for CAAC reverse-proxy of employee browser sessions and DSPM-for-AI prompt capture. Both are high-risk processing under GDPR Art. 35 / PDPA equivalents and require formal assessments before pilot.
- **Exit / portability planning** — policy export, log export in defensible format, configuration backup-restore.
- **Detailed comparison against Netskope / Zscaler / Palo Alto / Skyhigh.** That is the cross-vendor synthesis in `synthesis/capability-matrix.md` from the same workflow run.

---

## What to do next (practitioner checklist)

1. **Verify licensing.** M365 E5 or MDA standalone — *not* O365 E5 alone. If O365 E5, ~80% of this playbook is not deployable.
2. **Verify Entra ID P1 is on the tenant** for any CAAC-dependent policy (Days 30 and 90).
3. **Verify MDE is deployed and reporting** if you intend to use Policy 8 (GenAI unsanction).
4. **Pull the current tenant primary-data region** from the Microsoft Trust Center (this playbook flagged the EU/US/UK/AU claim as needing verification — verify it for your tenant before any data-residency decision).
5. **Day 1 set (Policies 1-4)** — turn on in audit/alert mode first; gather 2 weeks of baseline data; tune; flip to enforce.
6. **Day 30 set (Policies 5-7)** — only after Day 1 is stable. Each is a CAAC onboarding event; budget for per-app regression testing.
7. **Day 90 set (Policies 8-10)** — only after Day 30 is stable and you have SOC integration + exception process documented.
8. **Forward to Sentinel** for retention beyond 30 days. Build SIEM queries that **correlate `CloudAppEvents` with Entra Sign-in logs** by session correlation ID — neither alone is sufficient audit evidence.

---

## Three-lens sign-off

| Lens | Reviewer / run | Verdict | Outstanding |
|---|---|---|---|
| Architect | workflow `wcr0nuuwc` (2026-06-10) | INCORPORATED | Concentration-risk and OCAS SKU-trap are surfaced; `*.mcas.ms` URL-rewrite blast radius surfaced; BYOD-mobile gap surfaced in residual-risks. Outstanding: tenant primary-region claim needs Trust Center verification. |
| Product | workflow `wcr0nuuwc` (2026-06-10) | INCORPORATED | CAAC onboarding regression-test requirement surfaced; "Suspend user = disable Entra object" cascade surfaced; MDE Network Protection prerequisite for Policy 8 surfaced; "step-up auth still preview" downgraded. Outstanding: API throttling and 30-day Activities retention surfaced as residual; quantitative DCS-latency benchmarks not captured (vendor doesn't publish). |
| Compliance | workflow `wcr0nuuwc` (2026-06-10) | PARTIAL — outstanding work needed | Concentration-risk surfaced; DSPM-for-AI privacy treatment surfaced; OAuth application-grant gap surfaced; auditable-evidence pointers added per policy. **Outstanding for canonical promotion:** full control-to-policy mapping to BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS (separate deliverable); third-party attestation citations from Service Trust Portal; sub-processor list URL; DPIA scaffolding; regulator-clock decision tree. |

**Status:** stays in `_research/` pending the outstanding compliance work. Promotion candidate to `04-vendors/microsoft-defender-for-cloud-apps.md` after the compliance mapping deliverable lands.
