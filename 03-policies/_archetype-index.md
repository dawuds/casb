# Cross-vendor policy archetypes

Source: 5 vendor practitioner drafts in `../raw/`, accessed 2026-06-10. Lens-review unreliability flags are surfaced inline with `[unverified]`.

## How to read this

- "Vendor support" rows quote the *vendor's* claim from its draft. They are not equivalent feature-for-feature; the deployment-mode column and per-vendor caveat narrow the equivalence.
- `[unverified]` = the lens reviewers flagged the source draft as snippet-grade, internally contradictory, or operationally wrong on this specific point.
- "Where this maps in `03-policies/`" is a forward pointer for the canonical promotion step, not a current path.

## Archetype index

| # | Archetype |
|---|-----------|
| 1 | Shadow-IT discovery + risk-tagging (sanction / unsanction / tolerate) |
| 2 | Block / coach access by app-risk score and category |
| 3 | Tenant restriction (corporate-instance-only for M365 / Workspace / etc.) |
| 4 | Inline DLP on upload to sanctioned and unsanctioned SaaS (pre-upload block) |
| 5 | API-mode DLP scan + remediate (post-upload, data at rest) |
| 6 | External-share / anonymous-link governance on sanctioned storage |
| 7 | Malware scan in SaaS storage (inline upload + at-rest API) |
| 8 | Block download / read-only session to sanctioned SaaS from unmanaged devices (BYOD) |
| 9 | OAuth third-party-app discovery and revoke (M365 / Workspace / Salesforce) |
| 10 | UEBA / anomalous-activity detection (mass download, off-hours, bulk ops) |
| 11 | Compromised-account detection (impossible-travel, brute-force) |
| 12 | GenAI app discovery + inline prompt / upload control |
| 13 | Remote Browser Isolation overlay for risky / unsanctioned sessions |
| 14 | SaaS Security Posture Management (SSPM) — tenant misconfiguration drift |

---

## 1. Shadow-IT discovery + risk-tagging

**Purpose:** Discover all SaaS apps users touch from network telemetry; score each against a vendor catalogue; let practitioners tag apps Sanctioned / Tolerated / Unsanctioned to drive downstream policy.

| Vendor | Support + caveat |
|---|---|
| Netskope | Cloud Confidence Index, ~80k apps, ~100 attrs, CCL Poor→Excellent; tags drive policy (per netskope draft § Capability set / CCI). |
| Microsoft Defender for Cloud Apps | 31,000+ catalogue, 90+ risk indicators; tags Sanctioned / Unsanctioned / Monitored; **discovered-subdomains feature deprecates 2025-12-31** (per microsoft-defender draft § Shadow-IT discovery + § Limitations). |
| Palo Alto Prisma Access | SaaS Security Inline reads Strata Logging Service only — no third-party syslog ingest; Sanctioned/Tolerated/Unsanctioned tags (per palo-alto draft § Shadow-IT, § Limitations). |
| Skyhigh | Cloud Registry + CloudTrust rating 1–9 (1–3 enterprise-ready, 7–9 too risky); ~110 risk attributes (per skyhigh draft § Sanctioned-app inventory). `[unverified]` — lens reviewers flagged "1,900+ AI services" and "50,000+ services" as vendor-asserted with no published methodology. |
| Zscaler | Shadow IT Report against catalogue "20,000+ apps", Application Risk Index 1–5; **discovery only sees users on Zscaler — off-client traffic is invisible** (per zscaler draft § Shadow-IT, § Limitations). `[unverified]` — entire vendor draft sourced from JS-gated snippets per zscaler-architect lens. |

- **Deployment-mode requirement:** Log-based discovery (own proxy/firewall for Zscaler/PAN; third-party log ingest for Netskope/Skyhigh/MDA).
- **Common false-positive risk:** Over-counting (a user appearing in multiple log feeds is triplicated — MDA lens flagged this); business-needed niche apps scored low for lack of public audit evidence get auto-blocked.
- **Maps to `03-policies/`:** `discovery/shadow-it-catalogue-and-tagging.md`.

---

## 2. Block / coach access by app-risk score and category

**Purpose:** Once apps are scored, block (or interstitially coach) users away from the high-risk / unsanctioned tail; allow the corporate-approved alternative.

| Vendor | Support + caveat |
|---|---|
| Netskope | Real-time Protection policy; action set includes Block, User Alert (Coach with justification), Isolate (per netskope draft policy #2 "Coach users away from low-CCI shadow apps"). |
| Microsoft Defender for Cloud Apps | App discovery policy auto-tags Unsanctioned, propagates to Defender-for-Endpoint for enforcement; **enforcement requires MDE-managed endpoint** (per microsoft-defender draft policy #7). |
| Palo Alto Prisma Access | SaaS Security Inline policy recommendation pushed as Prisma Access security rule; Continue-with-response-page action for coach (per palo-alto draft policy #10). |
| Skyhigh | Web Policy + Cloud Registry Service Group, CloudTrust threshold; Block / Coach / Monitor (per skyhigh draft policy #8). |
| Zscaler | Cloud App Control rule; actions Allow / Block / Caution / Isolate (per zscaler draft policy #2). |

- **Deployment-mode requirement:** Inline (forward proxy) for live block/coach. MDA path is endpoint-agent-enforced.
- **Common false-positive risk:** Broad category blocks ("all File Sharing") catch a sanctioned-but-newly-categorised app; CloudTrust/CCI misclassification of niche regional fintech.
- **Maps to `03-policies/`:** `access-control/category-and-risk-block.md`.

---

## 3. Tenant restriction (corporate-instance-only)

**Purpose:** Allow the corporate tenant of M365 / Workspace / Salesforce / Dropbox; block (or read-only) personal or other-org tenants of the same SaaS, typically via the SaaS vendor's tenant-restriction HTTP header spec.

| Vendor | Support + caveat |
|---|---|
| Netskope | Real-time Protection with App Instance condition; tenant-ID extraction requires SSL inspection (per netskope draft policy #10). |
| Microsoft Defender for Cloud Apps | Not surfaced as a dedicated archetype in the draft; Entra Conditional Access + CAAC carries equivalent for M365 (inferred, not explicitly enumerated as a policy in the MDA draft). |
| Palo Alto Prisma Access | HTTP Header Insertion of `Restrict-Access-To-Tenants` + `Restrict-Access-Context` on `login.microsoftonline.com` (per palo-alto draft policy #4; cross-cited Microsoft Learn). |
| Skyhigh | Web Policy → Tenant Restrictions using Microsoft / Google tenant-restrictions spec (per skyhigh draft policy #12). |
| Zscaler | Cloud App Control → Tenant Profiles → Tenant Restriction rule (per zscaler draft policy #3). `[unverified]` — snippet-grade source. |

- **Deployment-mode requirement:** Inline (forward proxy) + SSL inspection.
- **Common false-positive risk:** M&A entity tenant not yet listed → org users locked out; cross-tenant-collaborator (consultant) breaks.
- **Maps to `03-policies/`:** `access-control/tenant-restriction.md`.

---

## 4. Inline DLP on upload (pre-upload block)

**Purpose:** Inspect HTTP/S upload body in transit; block uploads of regulated content (PCI / PII / source / secrets) to sanctioned-but-wrong-instance or unsanctioned destinations before the bytes reach the SaaS.

| Vendor | Support + caveat |
|---|---|
| Netskope | DLP profile attached to Real-time Protection policy; same engine across web/cloud/email/endpoint (per netskope draft policy #1). |
| Microsoft Defender for Cloud Apps | Only via CAAC Session Policy → *Control file upload (with inspection)*; **browser sessions only, requires Entra ID P1**, native apps with own auth bypass (per microsoft-defender draft policy #3, § Capability set inline DLP). |
| Palo Alto Prisma Access | Inline rule + Enterprise DLP profile; requires SSL decryption + GlobalProtect (per palo-alto draft policy #1). |
| Skyhigh | Web/Cloud Web Policy + DLP classifier; via Skyhigh Client or PAC (per skyhigh draft policy #2). `[unverified]` — skyhigh-architect lens flagged "Forward proxy OR reverse proxy" as wrong: reverse proxy only catches browser uploads, not sync clients / mobile / native. |
| Zscaler | Inline DLP rule with predefined dictionaries / EDM / IDM / OCR; **SSL inspection prerequisite**; sessions bypassing Client Connector are invisible (per zscaler draft policy #4). `[unverified]` — snippet-grade source. |

- **Deployment-mode requirement:** Forward proxy (Netskope/PAN/Skyhigh/Zscaler) or reverse proxy CAAC (MDA); SSL/TLS inspection mandatory; certificate-pinned apps bypass.
- **Common false-positive risk:** Regex on PCI/SSN fires on test fixtures, employee IDs, order numbers, fake-but-Luhn-valid sample card numbers.
- **Maps to `03-policies/`:** `dlp/inline-upload-block.md`.

---

## 5. API-mode DLP scan + remediate (data at rest)

**Purpose:** Crawl connected SaaS tenant for sensitive content already stored; quarantine, notify owner, change sharing, or apply label. Day-one safety net for pre-existing data.

| Vendor | Support + caveat |
|---|---|
| Netskope | API Data Protection Policy on Box/OneDrive/SharePoint/Drive/etc; vendor own copy calls it "after-the-fact, not proactive"; Salesforce delete is soft-delete only (per netskope draft policy #4, § API-mode DLP). |
| Microsoft Defender for Cloud Apps | File Policy; **50 file policies per tenant hard limit**; "only the first triggered policy is guaranteed to apply" (per microsoft-defender draft policy #2, § Limitations). |
| Palo Alto Prisma Access | Data Security asset policy; **25 MB file ceiling claimed universal** (per palo-alto draft § Capability set, policy #2). `[unverified]` — palo-alto-product lens flagged ceilings are 20–50 MB per-app per scan-type, not a uniform 25 MB. Forward-scan-only on Gmail/Chat/Zoom/Zendesk/Airtable/ChatGPT for at-rest. |
| Skyhigh | Sanctioned DLP Policy; quarantine, encrypt (Ionic/Seclore), notify, save-evidence ≤ 250 MB to S3/Azure (per skyhigh draft policy #1). |
| Zscaler | SaaS Security API DLP; remediation = notify / quarantine (Box/OneDrive/SharePoint with tombstone, Salesforce Restore) / remove external collabs / change to private (per zscaler draft policy #5). `[unverified]` — snippet-grade source. |

- **Deployment-mode requirement:** API connector to the sanctioned tenant.
- **Common false-positive risk:** Day-one scan against stale historical libraries (5+ years) mass-quarantines old marketing/HR/finance docs; OCR over scanned PDFs is noisy.
- **Maps to `03-policies/`:** `dlp/api-data-at-rest.md`.

---

## 6. External-share / anonymous-link governance

**Purpose:** Detect files in sanctioned storage shared externally / via anonymous link / to non-allow-listed domains; downgrade-share, remove collaborator, quarantine.

| Vendor | Support + caveat |
|---|---|
| Netskope | API DLP policy on share-state Public / Anyone-with-link / External-domain; per-app variation on what actions actually work (per netskope draft policy #4). |
| Microsoft Defender for Cloud Apps | File Policy with Access Level filter; "files shared with 'anyone' might incorrectly show up as private" known fidelity gap on OneDrive/SharePoint (per microsoft-defender draft policy #10, § Share-link governance). |
| Palo Alto Prisma Access | Data Security asset policy match on exposure Public / External / Company / Private; weaker on Salesforce/ServiceNow record-permission models (per palo-alto draft policy #2, § Share-link governance). |
| Skyhigh | Sanctioned DLP Policy collaboration controls; remove share / downgrade view-only / notify (per skyhigh draft policy #5). |
| Zscaler | SaaS Security API DLP with Domain Profile allow/deny; remove sharing / remove by domain / quarantine; **expiry-default is the SaaS's primitive, not Zscaler's** (per zscaler draft policy #7, § Share-link governance). `[unverified]` — snippet-grade. |

- **Deployment-mode requirement:** API.
- **Common false-positive risk:** JV / counsel / customer-onboarding domains not yet on the allow-list → legitimate external collab torn down; stale ACLs on historical files trigger mass alerts day-one.
- **Maps to `03-policies/`:** `dlp/external-share-remediation.md`.

---

## 7. Malware scan in SaaS storage

**Purpose:** Detect malware/grayware in files uploaded to or already in sanctioned storage; quarantine or alert.

| Vendor | Support + caveat |
|---|---|
| Netskope | Threat Protection (TSS) overlay on Real-time + API policy; ML + signature + Cloud Sandbox (per netskope draft policy #9). |
| Microsoft Defender for Cloud Apps | Built-in Malware Detection policy with File Sandboxing for Box/Dropbox/Workspace; **OneDrive/SharePoint scanned by Microsoft service, not MDA**; "doesn't automatically block" in third-party apps (per microsoft-defender draft policy #11, § Malware scanning). |
| Palo Alto Prisma Access | WildFire scan of files at rest via Data Security; quarantine + tombstone; subject to the 25 MB ceiling caveat (per palo-alto draft policy #8). |
| Skyhigh | Threat Protection paired with sanctioned-app instance; AV + sandbox; main risk is sandbox latency on large files (per skyhigh draft policy #11). |
| Zscaler | SaaS Security API Malware Detection policy; Cloud Sandbox tier if licensed (per zscaler draft policy #6). `[unverified]` — snippet-grade. |

- **Deployment-mode requirement:** API for at-rest; inline forward proxy for pre-upload block (where supported).
- **Common false-positive risk:** Low for signatures; grayware sandbox verdicts on legitimate IT utilities and macro-bearing finance templates.
- **Maps to `03-policies/`:** `threat/malware-saas-storage.md`.

---

## 8. Block download / read-only from unmanaged devices (BYOD)

**Purpose:** Allow web access to sanctioned SaaS from BYOD/unmanaged endpoints but block file download, copy/paste, print; force read-only in-session.

| Vendor | Support + caveat |
|---|---|
| Netskope | Reverse Proxy (SAML RP or RaaS with Entra) for unmanaged; Block Download / Print / Upload, allow View (per netskope draft policy #5). |
| Microsoft Defender for Cloud Apps | CAAC Session Policy → Control file download (with inspection); device tag filter `≠ Entra hybrid joined AND ≠ Intune compliant`; **requires Entra ID P1**; browser sessions only (per microsoft-defender draft policy #1, policy #5). `[unverified]` — mda-product lens flagged device-tag filter is incomplete (misses Compliant network + cert-based identity, can fail open or closed depending on `Always apply...if unscannable`). |
| Palo Alto Prisma Access | Inline rule + HIP (Host Information Profile) match ≠ Corporate-Managed; **HIP needs GlobalProtect — does not work for genuinely agentless BYOD** (per palo-alto draft policy #3, § Reverse-proxy "not the primary model"). |
| Skyhigh | Cloud Access Policy with device-certificate + MDM compliance signal; reverse proxy preferred, agentless (per skyhigh draft policy #3). `[unverified]` — skyhigh-architect lens flagged reverse-proxy pitch as "agentless for browsers on federated web apps only", not BYOD generally. |
| Zscaler | **No classic reverse-proxy mode**; equivalent is agentless Cloud Browser Isolation triggered via IdP redirect (per zscaler draft § Reverse-proxy, policy #9). `[unverified]` — snippet-grade and the lens reviewer noted CBI is a different UX, not 1:1 substitute. |

- **Deployment-mode requirement:** Reverse proxy (Netskope/MDA/Skyhigh) or RBI substitute (Zscaler); inline + endpoint posture (PAN). Identity hard-tied to IdP — Okta-legacy estates pay a per-app re-federation cost on MDA.
- **Common false-positive risk:** Device-certificate / HIP-failed states (cert expired, AV behind, macOS keychain reset post-upgrade) silently downgrade legitimate managed devices to "unmanaged"; entire BYOD population blocked from quarterly reports.
- **Maps to `03-policies/`:** `access-control/byod-read-only-session.md`.

---

## 9. OAuth third-party-app discovery + revoke

**Purpose:** Enumerate third-party OAuth grants against connected M365 / Workspace / Salesforce / Slack tenants; risk-score; alert or auto-revoke on high-risk scopes.

| Vendor | Support + caveat |
|---|---|
| Netskope | SSPM → 3rd Party Apps; risk tiers Critical–Unknown; revoke is connector-dependent (Entra/Workspace yes) (per netskope draft policy #3). |
| Microsoft Defender for Cloud Apps | OAuth apps / App governance page; built-in detections for Misleading name / Malicious consent; **only Delegated permissions surfaced — Application (client-credentials) grants invisible** (per microsoft-defender draft policy #6, § Limitations). |
| Palo Alto Prisma Access | SSPM third-party plugins view; GenAI plugin attribute; auto-revoke (per palo-alto draft policy #6). |
| Skyhigh | Analytics → Connected Apps; initial full scan + incremental (per skyhigh draft policy #4). |
| Zscaler | 3rd-Party App Governance (ex-AppTotal / Canonic); Block grant / Revoke / Notify (per zscaler draft policy #10). `[unverified]` — snippet-grade. |

- **All vendors share the same blind spot:** grants made via personal/consumer identity (Google personal, Microsoft consumer) on a managed device are invisible — the OAuth handshake never touches the connected tenant. Surfaced explicitly across all 5 drafts.
- **Deployment-mode requirement:** API (connector to tenant + IdP).
- **Common false-positive risk:** Productivity apps with broad scopes (Grammarly, Calendly, Loom) auto-revoked → sales/users break; needs verified-publisher allow-list first.
- **Maps to `03-policies/`:** `oauth/third-party-app-governance.md`.

---

## 10. UEBA / anomalous-activity detection (mass download, off-hours, bulk ops)

**Purpose:** Detect insider exfil / compromised behaviour via ML baseline + named scenarios (mass download, bulk delete/share, off-hours, infrequent country, terminated user still active).

| Vendor | Support + caveat |
|---|---|
| Netskope | Behavior Analytics → User Confidence Index (UCI); Standard vs Advanced (separate SKU) (per netskope draft policy #7, § UEBA). |
| Microsoft Defender for Cloud Apps | Anomaly detection policies + Activity policies; **June 2025 migration to "dynamic threat detection model" — 8 legacy policies disabled and replaced; practitioners must re-enable custom governance actions** (per microsoft-defender draft policy #4, § Anomalous-activity). Activity policies auto-disable above 200k/day or 100k/3h. |
| Palo Alto Prisma Access | Behavior Threats (Dynamic + Static); 90-day ML baselines; **automated session-kill is not native — incidents only, requires XSOAR/IdP** (per palo-alto draft policy #7, § Compromised-account, § Limitations). |
| Skyhigh | UEBA model over API + proxy telemetry; access anomalies (per skyhigh draft policy #6). |
| Zscaler | Security & UEBA Alerts; bulk uploads, off-hours, encrypted uploads; **detection over ZIA traffic only, not SaaS-side audit logs** (per zscaler draft § UEBA). `[unverified]` — snippet-grade; reviewer asked whether SaaS Security API ingests M365 sign-in logs directly. |

- **Deployment-mode requirement:** API (for SaaS-side telemetry) + inline (for proxy-observable behaviour).
- **Common false-positive risk:** Quarter/month-end finance bulk pulls; DR rehearsals; BI export jobs by service accounts; need explicit service-account exclusion under User filter.
- **Maps to `03-policies/`:** `ueba/anomalous-activity.md`.

---

## 11. Compromised-account detection (impossible-travel, brute-force)

**Purpose:** Detect sign-in patterns suggesting credential theft — two geo-distant sign-ins in implausible interval, failed-login bursts, activity-from-anonymous-IP.

| Vendor | Support + caveat |
|---|---|
| Netskope | UEBA scenario; vendor describes thematically — velocity threshold etc. not enumerated publicly (per netskope draft policy #7, § Compromised-account). `[unverified]` — netskope-product lens flagged step-up auth chain as misrepresented: Netskope degrades UCI which feeds Entra Identity Protection at next token refresh, **not live mid-session step-up**. |
| Microsoft Defender for Cloud Apps | Impossible-travel with Low/Medium/High sensitivity; country-level only, no alerts within same/bordering countries; **"Suspend user" governance action disables the Entra user object — destructive, SCIM cascade** (per microsoft-defender draft policy #9, lens-flagged). |
| Palo Alto Prisma Access | Static Behavior Threats — impossible-traveler, failed-login bursts, unsafe-VPN; IP exclusion list for corp VPN egress (per palo-alto draft policy #5, § Compromised-account). |
| Skyhigh | Anomalous Access Location; brute-force; configurable thresholds (per skyhigh draft policy #6, #7). |
| Zscaler | UEBA alerts surface impossible-travel and brute-force; **only signals what ZIA's traffic path sees — direct-to-SaaS attacker outside ZIA invisible** (per zscaler draft § Compromised-account). `[unverified]` — snippet-grade. |

- **Deployment-mode requirement:** API (for sanctioned-app auth telemetry) + IdP integration.
- **Common false-positive risk:** Corp VPN egress in distant region; mobile carrier CG-NAT; legitimate business travel; Outlook reauth loops post-password-change firing as brute-force.
- **Maps to `03-policies/`:** `ueba/compromised-account.md`.

---

## 12. GenAI app discovery + inline prompt / upload control

**Purpose:** Discover use of GenAI apps (ChatGPT, Claude, Gemini, Copilot, etc.); inline-inspect prompt body via DLP; block, coach, or apply tenant-restriction on consumer-vs-corporate workspace.

| Vendor | Support + caveat |
|---|---|
| Netskope | AI Guardrails (inline prompt inspection, prompt-injection / jailbreak detectors, MITRE ATLAS + OWASP LLM Top 10 mappings); AI Gateway (app-to-LLM auth/rate-limit); vendor claims **1,800+ GenAI apps catalogued** (per netskope draft policy #6, § GenAI). `[unverified]` — vendor-claim count not independently corroborated. |
| Microsoft Defender for Cloud Apps | Generative AI category in catalogue; auto-unsanction by risk threshold propagates to MDE-managed endpoints; Purview DSPM-for-AI surfaces prompts only on Purview-onboarded endpoints (per microsoft-defender draft policy #7, policy #8). `[unverified]` — mda-product lens flagged policy #8 ChatGPT block-cut/paste as undeployable without ChatGPT Enterprise/Teams SAML federation. |
| Palo Alto Prisma Access | AI Access Security (separate licence); GenAI dictionary; inline prompt DLP via Enterprise DLP; ChatGPT Enterprise the only GenAI API connector (per palo-alto draft policy #1, policy #9, § GenAI). `[unverified]` — palo-alto-product lens flagged that Claude/Gemini workspace-ID tenant restriction was asserted but coverage "should be verified per app". |
| Skyhigh | Cloud Registry tracks AI services (vendor claim 1,900+); ChatGPT Enterprise + M365 Copilot as native sanctioned apps; **Claude/Gemini/Mistral not native — inline / Cloud Registry only** (per skyhigh draft policy #9, policy #10). `[unverified]` — skyhigh-product lens flagged Copilot "block" action does not exist pre-execution; only monitor/alert/save-evidence + downstream Purview. |
| Zscaler | AI & ML Applications Cloud App Control category; per-activity control (Chat / Upload / Download / Share / Delete / Invite); WebSocket inspection required for Copilot-on-web (per zscaler draft policy #8, § GenAI). `[unverified]` — snippet-grade. |

- **Deployment-mode requirement:** Inline (forward proxy) for prompt inspection + SSL + (Zscaler) WebSocket inspection; API for ChatGPT Enterprise / Copilot at-rest where available.
- **Common false-positive risk:** Benign code paste containing JWT-shaped strings or 16-digit identifiers triggers Credit Card / Secret classifiers; constant coach prompts desensitise users.
- **Maps to `03-policies/`:** `genai/inline-prompt-control.md`.

---

## 13. Remote Browser Isolation overlay for risky / unsanctioned sessions

**Purpose:** Render a SaaS app session in a remote browser, transmit pixels only; disable clipboard / print / download; optionally watermark. Often the only path to "watermark on download" outside Microsoft Purview labels.

| Vendor | Support + caveat |
|---|---|
| Netskope | Action = Isolate via Netskope RBI; watermark via RBI overlay only — **no native proxy-session watermark**; UCI-driven trigger (per netskope draft policy #12, § Risk-based session policy). |
| Microsoft Defender for Cloud Apps | **No RBI**. Closest equivalent is Purview sensitivity label applied at download — but docs explicit: "visual markings, headers, footers, watermarks are not applied" (per microsoft-defender draft § Risk-based session policy, § Limitations). |
| Palo Alto Prisma Access | **No native watermark / DRM**; allow/block/coach only; reads Purview labels (per palo-alto draft § Risk-based session, § Limitations). |
| Skyhigh | Skyhigh RBI integration referenced in GenAI policy for read-only browser session; vendor copy mentions download/copy-paste/printing restrictions (per skyhigh draft policy #9, § Risk-based session). `[unverified]` — first-class watermark action only via partner DRM (Ionic/Seclore) per skyhigh-compliance lens. |
| Zscaler | Action = ISOLATE on Cloud App Control rule → Cloud Browser Isolation profile (clipboard in/out, print, upload/download, watermark with user identifier) (per zscaler draft policy #9). `[unverified]` — snippet-grade. |

- **Deployment-mode requirement:** Inline (forward proxy) trigger + separate RBI service. MDA + PAN have no native equivalent.
- **Common false-positive risk:** UX degradation on graphics-heavy apps; cascading isolation from a single noisy UEBA signal.
- **Maps to `03-policies/`:** `session-control/rbi-overlay.md`.

---

## 14. SaaS Security Posture Management (SSPM) — tenant misconfiguration drift

**Purpose:** Continuously assess connected SaaS tenant settings against a posture-rule library (default sharing, MFA enforcement, public-link defaults, guest-access, app allow-list); flag drift; auto-remediate where the API permits.

| Vendor | Support + caveat |
|---|---|
| Netskope | SSPM module; 3rd-party-app discovery sits inside it (per netskope draft § CASB housing, § Capability set OAuth governance). |
| Microsoft Defender for Cloud Apps | Not separately marketed as SSPM in the draft; Microsoft Secure Score + app governance posture features are adjacent (inferred from microsoft-defender draft § Lineage and § OAuth governance — **not a discrete SSPM policy archetype in this draft**). |
| Palo Alto Prisma Access | SaaS Security Posture Management as a third sub-module (separate console area, separate policy primitives) (per palo-alto draft § Lineage, § Hybrid). |
| Skyhigh | Skyhigh DSPM in platform component list — not enumerated as a distinct policy in the configurable-policies section (per skyhigh draft § Lineage). `[unverified]` — skyhigh draft does not give an SSPM policy example. |
| Zscaler | SSPM Policy under SaaS Security API; posture rule library, CIS / NIST 800-53 mapping claimed (per zscaler draft policy #11). `[unverified]` — zscaler draft itself flags compliance-mapping for current rule set as unverified. |

- **Deployment-mode requirement:** API.
- **Common false-positive risk:** Alert fatigue from low-severity findings; auto-remediation breaking tenant-level settings that an admin set deliberately.
- **Maps to `03-policies/`:** `sspm/tenant-posture-drift.md`.

---

## Notes for reviewers

- **Watermarking** does not survive cross-vendor synthesis as its own archetype — every vendor either bolts it onto an RBI overlay or refers it out to Purview labels. Folded into Archetype #13.
- **Gateway encryption / tokenisation** is absent or `Not supported` across all 5 drafts — no recurring archetype, hence omitted.
- **Step-up authentication** is consistently *not* a CASB primitive across all 5 — it is an IdP signal/response loop. Surfaced inside Archetype #11 (compromised-account) as a known mislabel hazard (per netskope-product lens flag).
- **Inline-vs-API policy plane duplication** is a recurring practitioner pitfall (Netskope: same engine but separate objects; Palo Alto: separate consoles + only Enterprise DLP profiles shared; Skyhigh: adjacent menus; Zscaler: separate "Cloud App Control" vs "SaaS Security API" panes). Not an archetype, but worth a horizontal "drift surface" note in any vendor selection or policy-authoring guide.
- **Tier-1 vs Tier-B/C connector asymmetry**: every vendor publishes a connector count headline (Netskope NextGen list, Skyhigh ~40, PAN ~28, Zscaler 8–10 named, MDA ~14 named) but actual capability per connector varies materially. Sizing coverage off headline count is wrong — skyhigh-architect lens called this out explicitly.
