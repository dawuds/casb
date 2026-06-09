# Product lens review — Skyhigh Security (Skyhigh Cloud / CASB)

Reviewer stance: hands-on CASB administrator (McAfee MVISION Cloud → Trellix → Skyhigh era), multiple BFSI deployments, has personally hit the policy-tree fragmentation, reverse-proxy SAML-trip footguns, and Cloud Connector log-ingest rough edges in production.

---

## Findings to act on (highest impact first)

1. **Cloud Registry / CloudTrust numbers are vendor marketing — treat as advertising, not capability.** Section "Sanctioned-app inventory" (line 24) and the GenAI claim (line 35) repeat "50,000+ services, 1,900+ AI services, ~110 risk attributes" verbatim from the Skyhigh home page. There is no published methodology, no inter-rater scoring, no audit. Practically: CloudTrust *category* assignment is often stale, the "1,900 AI services" number is a count of crawled domains (many duplicates: `chat.openai.com`, `chatgpt.com`, `openai.com`, `platform.openai.com` are separate registry entries), and the "110 risk attributes" includes attributes you cannot change and many you cannot see the source data for. The draft already partially flags this in line 213; promote the warning into the capability claim itself, not the footnotes.

2. **Reverse-proxy is described as a current first-class capability — in 2026 it is on life support.** Line 16 lists Salesforce, ServiceNow, Slack, Workday, SuccessFactors, Box, G Suite as documented reverse-proxy targets. In live BFSI deployments since ~2022, reverse-proxy fails on: (a) Microsoft Teams desktop client (non-SAML socket auth), (b) Outlook Mobile / native mail clients using Modern Auth / OAuth2 directly, (c) Salesforce Mobile (cert-pinned), (d) any Slack desktop client after the 2023 auth refactor, (e) Workday Mobile. What still works cleanly via reverse proxy is *browser-only* access on the listed apps. The draft does not say this. A practitioner reading line 16 will assume reverse proxy covers those apps end-to-end and will commit it to an architecture diagram. Add a hard caveat: "reverse proxy works for browser sessions only; native and mobile clients bypass."

3. **SAML reverse-proxy "SAML-trip" footgun missing.** When the IdP SAML endpoint is rewritten to land at Skyhigh first, any IdP outage, Skyhigh POP latency spike, or certificate rotation at Skyhigh breaks SSO *for the entire SaaS app*, not just the inspection path. Several BFSI customers have had Saturday-night M365 outages traced to the Skyhigh reverse-proxy SAML certificate not being renewed in their IdP. This is the highest-blast-radius operational risk in the product and the draft does not mention it.

4. **"Same policy surface covers Shadow-IT and Sanctioned-IT" is overstated (line 19).** The draft hedges with "not fully unified" but understates it. There are at minimum **four** disjoint policy planes a practitioner authors in: (i) SWG Web Policy (inline / Shadow-IT), (ii) Sanctioned DLP Policies (API content scan), (iii) Cloud Access Policies (proxy access decisions), (iv) Connected Apps Policies (OAuth). A single DLP classifier authored in one place does **not** automatically apply across all four — you re-bind the classifier per policy object. Line 217 partially notes this but it belongs in the capability narrative, not as a reviewer note.

5. **"Inline DLP enforced via reverse-proxy paths" is technically true but operationally weak (line 25).** Reverse-proxy inline DLP only fires on content that traverses the proxy — for SharePoint/OneDrive that is web upload; the OneDrive sync client uses a Graph API endpoint that does **not** flow through reverse proxy. So "inline DLP on OneDrive" via reverse proxy misses the most common file-egress path on a managed laptop. Sync client uploads need either the Skyhigh Client (forward proxy) or API-mode post-upload scan. Draft needs to say so.

6. **Cloud Connector is treated as a benign ingest component (line 18) — in practice it is fragile.** Skyhigh Cloud Connector is a Java service that runs on a customer Windows or Linux host. Failure modes I have seen repeatedly: (a) syslog UDP drops at high EPS without backpressure feedback to the UI — you get a silent gap in Shadow-IT data and no alert; (b) certificate-trust failures during outbound upload after JDK security updates; (c) parser drift when the upstream proxy (Zscaler/Palo Alto) changes log format and Skyhigh has not updated the parser yet; (d) no HA — it's single-instance per site. None of this is in the draft.

7. **"Save evidence … max 250 MB per file" (line 26) is correct but the harder limit is missing.** Total evidence storage is capped per tenant and the customer's own S3/Azure bucket is preferred — but more importantly, *evidence capture is best-effort and not transactional with the policy action*. A quarantined file may have no readable evidence if the SaaS revoked the link before Skyhigh could pull it. For BFSI evidentiary use this matters: you cannot promise an auditor that every quarantine event has a content snapshot.

8. **M365 Copilot integration is sold heavily; what it actually does is narrower than the draft implies (lines 35, 113–119).** The integration surfaces Copilot *prompt and response telemetry* to Skyhigh for DLP classification and incident reporting. It does **not** block Copilot at the prompt boundary in real time — Copilot has no pre-prompt webhook that a third-party CASB can hook into. So "downstream Purview-label enforcement if labels exist" (line 116) is doing all the load-bearing work. If the customer has no Purview labelling rollout, the practical action set collapses to *monitor / alert / save evidence*. The draft should say this plainly.

9. **"Apply Protection via Ionic / Seclore DRM" is a partner integration with separate licensing (lines 26, 33, 44).** The draft notes this in the limitations footnote (line 180, 215) but in the capability table it reads as a built-in DLP action. Ionic was acquired by Twilio (2021) and the Skyhigh-Ionic integration story is murky; the active partner path is Seclore. Either way, this is **not** a Skyhigh-native action — it requires a Seclore (or Ionic-derived) tenant, separate contract, separate connectors. Promote the caveat into the capability cell.

10. **"~40 native sanctioned apps" — name the asymmetry, not just the number (line 17).** Not all 40 "native" connectors are equal. Tier-A (deep DLP at rest, Connected Apps OAuth visibility, anomaly telemetry, quarantine actions): M365, Google Workspace, Box, Salesforce, ServiceNow, Slack. Tier-B (some DLP, limited actions, no OAuth Connected Apps surface): Dropbox, GitHub, Egnyte, Zoom. Tier-C (essentially metadata + audit only): everything else on the list. The "40" headline number hides this. A practitioner sizing coverage off "40 apps" will be wrong.

---

## Capability claims to corroborate (do not promote without independent source)

- "Cloud Registry: 50,000+ services" — vendor home page, no methodology. Independent: Gartner has cited similar numbers historically; if a third-party number exists, cite it. Otherwise mark as "vendor-asserted, unverified."
- "1,900+ AI services" — vendor home page only. Almost certainly inflated by sub-domain counting.
- "110 risk attributes" — vendor claim. Skyhigh publishes the attribute *categories* on the CloudTrust page but not all 110 attributes nor their weights. Treat as opaque.
- "Pre-upload block + real-time user notification" inline DLP (line 25) — works for browser uploads to documented sanctioned apps. *Does it work for non-sanctioned long-tail SaaS via Skyhigh SWG inline-DLP path?* The draft conflates the two. Source needed for the long-tail case.
- "UEBA over user activity from API + proxy telemetry" (line 28) — model is opaque. Skyhigh has never published the feature set, training cadence, or false-positive benchmarks. Treat all "UEBA detected" claims as best-effort, tunable-by-customer.
- "Integrated anti-malware solution and inline emulation-based sandboxing" (line 32) — vendor product-page copy. Engine identity (was historically McAfee GAM + ATD; today unspecified post-Trellix divestiture) is not disclosed. Practitioners should ask Skyhigh: which engine, which version, what's the sample-submission flow, what's the sandbox dwell, who sees the verdict.
- "Impossible travel" detection (line 29) — works on documented sanctioned-app login telemetry only. For apps without login-telemetry surface (most Tier-B/C), there is no impossible-travel detection. Source the per-app coverage list before promoting.
- "Connected Apps initial full scan + incremental scans" (line 27) — incremental cadence is not documented publicly. In practice it has been 6–24 hours. New OAuth grants are not visible in real time. Source needed or mark as "near-real-time, lagged."
- "Tenant Restrictions" (line 129–135) — Microsoft Tenant Restrictions v2 requires specific header insertion *and* a corresponding Azure tenant config on the customer side. Skyhigh inserting headers is necessary but not sufficient. Source the customer-side prerequisites.

---

## Policies to refine

- **#1 Sanctioned-app DLP policy** — add explicit caveat: "DLP scan is post-upload; window from upload to scan is typically minutes but can be hours for large libraries during backfill." Add: "Quarantine action moves the file to a Skyhigh-controlled location in the same tenant; rolling back a wrongful quarantine at scale is manual." OCR is documented but accuracy on scanned PDFs and images is poor — name it. ML classifier results are not deterministic across re-scans — name it.

- **#2 Inline DLP policy** — "Forward proxy OR reverse proxy" (line 53) is wrong as written. Reverse proxy inline DLP fires only on file content posted via the browser session through the reverse proxy. Sync clients, mobile apps, and most desktop native clients are out of scope. Re-write the deployment-mode line to be: "Forward proxy (Skyhigh Client or PAC) for full coverage; reverse proxy for browser-only uploads on supported apps."

- **#3 Managed-device-required Cloud Access Policy** — the named scenario (macOS keychain reset) is real but understates: device-cert lifecycle is the #1 source of policy false-positives I have seen on Skyhigh. Add: "Plan cert renewal automation before this policy goes enforce; without it expect ~3–8% of fleet to drop out per quarter." Also flag: "Break-glass exclusion group is mandatory; the documented IdP-bypass path requires a parallel SAML route."

- **#4 Connected Apps policy** — "Auto-revoke" action (line 70) needs caution: revocation triggers user re-prompt; for OAuth scopes that bridge a critical workflow (e.g., Calendly to G Cal), users will re-grant the same scopes within minutes if not blocked at the IdP. Skyhigh cannot block re-grant — only the IdP can. So policy without IdP-side enforcement is a treadmill.

- **#6 Anomalous-access-location policy** — add: "Geo-IP database lag for mobile-carrier and cloud-VPN egress can cause sustained false positives for weeks. Tune block-list / allow-list before enabling." Also: this anomaly type does not benefit from inline action without a paired Cloud Access Policy — the draft says this but the policy authoring is two-step and not obvious from the UI.

- **#8 Shadow-IT control policy** — "log-only (monitor; cannot block from logs alone)" (line 101) is correct. Add the real-world constraint: blocking via Skyhigh SWG requires the customer to route web traffic through Skyhigh SWG. Many customers run Skyhigh CASB *with a different SWG* (Zscaler / Netskope inline, Skyhigh CASB for API + Shadow-IT visibility only). In that topology, Skyhigh CASB cannot block; the customer must push the block list to their actual SWG manually. Name this.

- **#9 GenAI inline policy** — "optional integration with Skyhigh RBI" (line 108) — Skyhigh RBI is a separate SKU and not all SSE bundles include it. Flag licensing dependency.

- **#10 M365 Copilot DLP policy** — see Finding #8. The "block" action does not exist for Copilot prompts pre-execution; primary actions are monitor/alert/save-evidence and downstream Purview label enforcement. Re-write the action list accordingly.

- **#11 SaaS malware scan policy** — "main risk is sandbox latency on large files (delivered then quarantined)" (line 126) is right but understates. Real risk: by the time the file is quarantined, end users may have already downloaded it from a shared location. Quarantine is **not** retroactive on downloads.

- **#12 Tenant Restrictions policy** — see corroboration note above. Add: "Requires Microsoft tenant cooperation; not a Skyhigh-unilateral control."

---

## Limitations the draft missed

- **No native Slack Enterprise Grid org-level visibility** — Slack EG has org-level admin APIs that most CASBs (Skyhigh included) have inconsistent support for. DLP on shared channels across workspaces in an Enterprise Grid is partial.
- **No support for Microsoft Loop, Microsoft Lists, or Microsoft Whiteboard** in API-mode DLP. These are M365 surfaces but they are not in the Skyhigh M365 connector scope. Practitioner gap.
- **No support for Salesforce Files (Chatter Files) DLP at parity with Salesforce object DLP.** Salesforce connector is record-centric, not file-centric.
- **No control over SaaS-to-SaaS Power Automate / Power Apps / Workato / Make / Zapier flows.** Draft hints at this (line 181) but does not name the platforms.
- **No native support for Atlassian Cloud (Jira, Confluence Cloud) at first-class DLP-action depth.** Confluence is on the connector list (line 17) but the depth is metadata + collaboration, not at-rest content DLP with quarantine.
- **No native API integration for Notion, Airtable, Coda, Linear, Figma, Miro, Asana, Monday, ClickUp, Trello** — modern collaboration stack is largely off-platform. Inline-only.
- **No DSPM-grade data classification at rest across non-connected stores.** Skyhigh DSPM is a separate product; the CASB does not see data in S3 / Azure Blob / GCS / Snowflake unless those are surfaced via DSPM. Draft conflates "CASB" and "Skyhigh platform" — they are not the same SKU.
- **No native support for inspecting WebSocket-based SaaS traffic at the inline DLP layer for content classification.** Generic limitation but worth naming because GenAI apps and modern collaboration tools use WebSocket heavily.
- **No detection of DLP classifier drift / regression over time.** When a built-in classifier is updated by Skyhigh, existing incidents may not be re-graded; new false-positive bursts after a classifier update are common and there is no notification.
- **No audit trail on policy author actions sufficient for SOX-style change control out of the box.** Admin activity logs exist but exporting them for an external GRC tool requires manual API work; no first-class SIEM/GRC connector for admin actions specifically.
- **No documented SLO / SLA for API-mode scan latency, Connected Apps incremental sync cadence, or anomaly detection latency.** Customer cannot tie any of these to a contractual KPI.
- **No support for client-side encryption keys (BYOK / HYOK) on the Skyhigh-stored evidence or telemetry data itself.** Customer-managed keys for the *content evidence* bucket are possible if the customer points to their own S3/Azure; for Skyhigh's own telemetry storage, no BYOK.
- **No published data-residency map for Skyhigh's own telemetry storage.** Where is the CASB's own incident, telemetry, evidence-metadata stored? EU customers have asked, answers have been vague. Material for any BFSI deployment.
- **CVE / vulnerability history not surveyed in draft.** The draft itself flags this (line 219); flagging that flag — without a CVE pass, a BFSI security architect cannot promote this vendor. McAfee MVISION Cloud / Skyhigh has had CVEs (e.g., CVE-2021-31840 in the Cloud Connector component, authenticated path traversal). At minimum a one-line "no critical unpatched CVEs at time of writing" must be in scope before promotion.
- **No mention of administrative SSO + RBAC depth.** Skyhigh's role model is coarse (System Admin, Policy Admin, Incident Manager, etc.); attribute-based access control on incidents (e.g., "EU SOC team sees only EU-tenant incidents") is limited. Important for global BFSI customers with regional SOC separation.

---

## Overall verdict

- **Promote as-is to 04-vendors/?** **NO**
- **One-line reason:** Draft is structurally sound but launders vendor marketing on Cloud Registry numbers, oversells reverse-proxy currency, understates the four-plane policy fragmentation, and omits the operational footguns (Cloud Connector fragility, SAML-trip blast radius, sync-client DLP gap, Tier-A/B/C connector asymmetry) that a BFSI practitioner needs before signing.
