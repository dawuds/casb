# GenAI discovery and auto-unsanction

> Status: v0.0 — MDA column lens-reviewed from playbook v1 (Policy 8 — Day 90); other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [GenAI-app discovery + Risk-based session policy + endpoint enforcement integration](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Log-based discovery + endpoint enforcement (MDE Network Protection in block mode) OR SWG GenAI-category block.
> MDA playbook reference: [Policy 8](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Auto-unsanction GenAI above risk threshold).

## Purpose

Continuously discover GenAI app usage; auto-unsanction apps in the Generative AI category exceeding a risk threshold; propagate the unsanction to endpoint enforcement to block at the device. Reduces data leakage to consumer-tier GenAI. Counters MITRE ATT&CK `T1567 Exfiltration Over Web Service` (Detect on Discovery; Prevent only via MDE NP block); ATLAS `AML.T0024 Exfiltration via AI Inference API` partial. [MITRE ATLAS: https://atlas.mitre.org/techniques/AML.T0024]

## Microsoft architectural anchor

This policy operationalises the Application Zero Trust pillar Initial Deployment Objectives published at [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications]:

- **Objective II** — Discover Shadow IT (Cloud Discovery feed into MDA Cloud App Catalog).
- **Objective III** — Protect sensitive information automatically (Tag-as-Unsanctioned and propagation to endpoint enforcement).
- **Objective IV** — Deploy adaptive access (risk-score-driven enforcement).
- **Objective V** — Detect and respond to cyber threats in apps (App Governance behavioural-anomaly detection as supplementary signal).

For agentic-AI coverage Microsoft publishes Defender AI Agent Protection (Nov 2025 release-notes cohort) at [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/whats-new] `[VERIFY against current Defender release notes]`.

## What organisations use this for

For 2024-2026 deployments this is **the** GenAI governance control most firms reach for first — because Shadow IT discovery already produces a GenAI inventory, and tagging-to-unsanction reuses an existing pipe. The hard part is not the toggle. The hard part is the **sanction taxonomy** that sits behind it: which AI vendors are approved-for-sensitive-data, which are approved-for-non-sensitive-only, which are blocked, which are tolerated-pending-review. Most firms ship this policy in three phases: (1) discover and report, (2) block consumer-tier consumer AI after a named incident or board ask, (3) build out an enterprise-AI allowlist with a documented intake process. Skipping straight to phase 3 fails because the operator has not yet seen what their users are actually doing.

The dominant 2026 reality: an organisation deploys this policy, blocks consumer ChatGPT and friends, and the users move to their phones. The control reduces the **corporate-endpoint** leak surface; it does almost nothing on BYOD. That trade-off must be named on day one, not discovered six months in by a board member who watched their CFO paste a memo into Claude on a personal iPhone.

### Use case 1 — Tier-1 ASEAN universal bank, post-Mint-Sandstorm consumer-AI clamp

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, MAS-licensed subsidiary in SG, board has named "Responsible AI" as a 2026 standing risk
- **Trigger:** internal investigation surfaced ~40 employees pasting client correspondence and credit-memo drafts into consumer ChatGPT and Claude over a six-month window — none malicious, all "trying to save time". Board demanded a hard control before next quarterly risk forum
- **Scope:** all employees + contractors on managed endpoints (~28k devices); MDA Shadow IT discovery via MDE feed; auto-unsanction on the Generative AI category at risk score ≤ 5 with custom App tags distinguishing "AI coding assistant" / "AI image generator" sub-uses; allowlist = ChatGPT Enterprise (tenant SSO'd), M365 Copilot, internal AI gateway
- **Outcome:** ~120 consumer-AI domains blocked at MDE Network Protection within 14 days; ~3k weekly blocked-attempt events trending down to ~500 by week 8 (users learn the allowlist); mobile-BYOD remains an open gap acknowledged on the risk register and routed to a separate Intune MAM workstream. Claimed reduction in sensitive-data-to-consumer-AI events is operator-reported and not independently verified [flag — internal metric only]

### Use case 2 — Merchant acquirer, pre-PCI-DSS-v4.0 AI-overlay finding

- **Org type:** payments processor across MY / SG / HK / TH, in-scope for PCI DSS v4.0, ~6k employees, cardholder data flows through QA / Engineering teams who had begun using GitHub Copilot for free against repos that contained test-PAN fixtures
- **Trigger:** QSA pre-assessment for the PCI DSS v4.0 cycle flagged "AI Coding Assistant exposure to cardholder-data repositories" as a finding under Req. 6 (secure development) and Req. 12.3 (acceptable use). PCI Council 2024 guidance on AI/ML in CHD environments cited [VERIFY against current PCI SSC AI guidance]
- **Scope:** Engineering + QA + Product BUs (~800 users); MDA discovery + MDE NP block on Generative AI category with a custom App tag `ai-coding-assistant` applied to GitHub Copilot Free / Codeium / Cursor / Tabnine / Replit Ghostwriter / CodeWhisperer entries; allowlist = GitHub Copilot Enterprise on CDE-isolated tenant only; secondary control = pre-commit hook scrubbing PAN-format from any AI-bound payload
- **Outcome:** finding closed at follow-up; ~15 AI-coding tools blocked (CodeWhisperer, Tabnine free, Cursor free-tier, Codeium, Replit Ghostwriter, etc.); QSA accepted Copilot-Enterprise-on-CDE-tenant as compensating control for Req. 6.4; ongoing dispute with Engineering on developer experience — partially mitigated by sanctioning Copilot Enterprise within 30 days

### Use case 3 — Digital-native challenger bank, transition from block-all to sanction taxonomy

- **Org type:** neobank, ~600 employees, M365 E5, regulated by BNM as a digital bank, pre-IPO posture, heavy AI tooling already in the engineering org pre-rollout
- **Trigger:** initial "block everything AI" reaction from the board after a competitor's data-leak headline; six months in, Engineering and Product had built shadow workarounds (personal devices, personal accounts, mobile hotspots); CTO escalated that the blanket block was costing more in productivity than it was buying in risk reduction
- **Scope:** transition project — pull together a vendor-risk-assessment template specifically for AI vendors (training-data policy, customer-isolation, model-residency, sub-processor list, AML.T0010 supply-chain posture); intake workflow targets 3-week SLA from request to allowlist decision; MDA allowlist becomes the enforcement-layer expression of the taxonomy
- **Outcome:** 12 vendors approved over 9 months across four risk tiers (sensitive-data-OK / internal-data-OK / public-data-OK / blocked); blocked count drops from ~60 to ~25; user satisfaction recovers; pre-IPO due diligence accepts the AI-vendor risk-management process as evidence of operational maturity

### Use case 4 — Tier-1 ASEAN bank post-Storm-2372 OAuth cleanup, GenAI-app-via-OAuth overlap

> **Practitioner inference:** the linkage between Storm-2372 device-code phishing and downstream GenAI-OAuth grant abuse is field observation in 2024-2025 incident triage, not a Microsoft-documented attribution. Microsoft Security Blog Storm-2372 reporting focuses on the device-code-phishing technique itself; GenAI-OAuth as a downstream pivot is practitioner-attested. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming] for the threat-actor-naming taxonomy.

- **Org type:** universal bank, ~25k employees, M365 E5, recovering from Storm-2372 consent-phishing wave in 2024 that surfaced ~80 over-permissioned OAuth grants to AI tools (Notion AI, ChatGPT-via-MSA-OAuth, AI plugin marketplace apps)
- **Trigger:** post-Storm-2372 OAuth review identified that OAuth-mediated AI access bypassed the network-egress discovery entirely — apps that connected to M365 via Graph permissions never appeared in the SWG / MDE feed. AI governance had to converge with OAuth governance
- **Scope:** joint workstream — MDA OAuth apps page (Policy 2a) + App Governance behavioural-anomaly detection (dynamic detection model, June 2025 — Policy 2b) + GenAI auto-unsanction (this policy). Approved-AI vendors must pass BOTH the network-discovery allowlist AND the OAuth-grant verified-publisher restriction in Entra. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity] (Objective IV — Restrict user consent to applications) + [Microsoft Learn: https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants]
- **Outcome:** ~50 AI-via-OAuth apps banned; verified-publisher restriction enforced in Entra for AI-category-tagged apps; documented interlock between OAuth-governance team and AI-governance team (had been separate functions); residual gap on Power Platform AI Builder service-principal grants under-surfaced (per Policy 2b limits)

## Implementation pattern

Typical 10-week rollout sequence for a tier-2 BFSI tenant with MDA Shadow IT discovery already in place:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Confirm Day-0 preconditions: MDE coverage % on managed endpoints, MDE NP in **block** mode (not audit), Shadow IT discovery already producing a 30-day GenAI inventory, governance forum to own the allowlist | Preconditions signed-off; gap list for un-onboarded endpoints |
| W2 | Pull 30-day GenAI baseline from Cloud Discovery: app list with custom App tags applied to distinguish coding-assistant / image / video / voice-avatar use within the single Microsoft-documented "Generative AI" category; user count per app; traffic-volume rank | Baseline inventory; named long-tail vs head-of-curve apps |
| W3 | Sanction taxonomy design: 4-tier taxonomy (Sensitive-OK / Internal-OK / Public-OK / Blocked); vendor-risk-assessment template specifically for AI (training-data, customer-isolation, model-residency, AML.T0010 supply-chain, sub-processor list, DPA); intake SLA decision (typical 2-3 weeks) | Taxonomy + intake template signed-off by Risk + Legal + IT |
| W4 | Allowlist seed: which AI vendors are pre-approved (typically the enterprise SaaS the firm already has — ChatGPT Enterprise, M365 Copilot, GitHub Copilot Enterprise, internal AI gateway); capture app IDs | Seeded allowlist; rollout-comms drafted |
| W5 | Policy in **alert-only** mode: App discovery policy with category filter = Generative AI + custom App tag filter for the coding-assistant / image / video sub-uses targeted in phase 1, risk score ≤ 5, users/day > 10, action = Alert (NOT Tag as Unsanctioned yet) | Alert volume measured; FP candidates identified |
| W6 | FP review of alert-only week: legitimate trials, contracted-vendor-not-in-allowlist, mis-categorised apps, approved vendor scoring ≤ 5 due to "SOC 2 = No" in MS catalogue | Allowlist exceptions added; mis-categorisation feedback to MS via in-product mechanism |
| W7 | Flip to **Tag as Unsanctioned + Alert**: MDE NP propagation begins on managed endpoints; user-facing block page configured to redirect to sanctioned alternatives | First-block-events flowing; help-desk runbook tested |
| W8 | Rollout comms wave: user-comms naming the sanctioned alternatives + the intake path for new AI requests; targeted comms to engineering / product / finance teams whose AI use was discovered in W2 | Help-desk ticket volume monitored; user-comms acknowledged |
| W9 | Broaden custom App tag scope from coding-assistant-only to additional sub-uses (image, video, voice-avatar) as appropriate per the firm's posture; tighten risk-score threshold or users-per-day threshold based on W7-W8 data | Full sub-use coverage live within Generative AI category |
| W10 | Quarterly review cadence stood up: allowlist review (vendor-risk re-attestation), block-list review (re-evaluation of recently-blocked vendors that gained user demand), risk-score-drift review on existing allowlist members | Steady-state ops handoff; quarterly metric defined |
| W11+ | Steady-state — quarterly review + ad-hoc intake-ticket response + AML.T0010 supply-chain incident triage for any allowlisted vendor | Allowlist-as-a-control evidence pack maintained |

The W3 taxonomy work is the load-bearing piece. Without it, the allowlist becomes a sprawl of one-off exceptions and the operator loses control of the policy's intent.

## Action

- Primary: **tag as Unsanctioned + alert**; MDE Network Protection propagates the block to managed Windows / macOS endpoints
- Phase-1 alternative: **alert-only** for at least 2 weeks to baseline FP rate and seed the allowlist before propagating to endpoint enforcement
- Allowlist: approved AI vendors (ChatGPT Enterprise, Copilot, GitHub Copilot Enterprise, internal AI services) exempted by app ID

## Scope

- **Users:** all (excluding documented exception groups — e.g. AI-research team with sanctioned-for-research scope)
- **Apps:** Microsoft's published Cloud App Catalog category is the single "Generative AI" classification, with adjacent categories "AI – MCP Server" and "AI – Model Provider". The five-way practitioner split (LLM / AI Coding Assistant / AI Image / AI Video / AI Voice-Avatar) is **not** in the published Microsoft taxonomy as of June 2026. Use **custom App tags** to distinguish sub-uses within the single Generative AI category. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score] → Supported cloud app catalog categories.
- **Device posture:** managed endpoints with MDE Network Protection in block mode; BYOD bypasses (separate Intune MAM workstream)
- **Network position:** corporate network + roaming via MDE agent; off-corp via MDE NP if endpoint reports in
- **Traffic:** all (block by domain/IP via Network Protection — partial against CDN-fronted egress)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Microsoft Defender Portal → Cloud Apps → Policies → Policy management → Shadow IT → Create App discovery policy | Category filter = **Generative AI** (single Microsoft-documented category — cite `risk-score`); custom App tags for sub-uses; Risk score ≤ 5 on the documented 0-10 scale (lower = higher risk; cite [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score]); Users per day > 10; Action = Tag as Unsanctioned + Alert; Allowlist approved AI vendors by app ID; **MDE Network Protection in block mode (not audit)** | Endpoint enforcement requires MDE deployed; Windows/macOS supported builds per current MDE NP supported-OS page; Endpoints reporting to same tenant | "Generate block script for SWG" produces flat IP/domain list that does not match modern GenAI vendor egress (CDN-fronted, JA3-fingerprinted, dynamic domains) — partial fail. Microsoft documents 90+ risk factors but does not publish the factor weights — the score is opaque on the weighting side, not the factor-count side. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score]. Auditor cannot accept opaque score as sole basis for automated enforcement; document parallel internal risk-acceptance / appeal process. Approved-AI allowlist supply-chain compromise (ATLAS AML.T0010) requires quarterly review + contingency. **Tag-as-Unsanctioned in audit-mode MDE NP is the single most common Day 90 misconfiguration — produces zero endpoint enforcement and the operator does not get an alert about the audit-vs-block state mismatch** |
| Netskope | `[unverified]` — Netskope CCI + native SSE GenAI-category block | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access GenAI security | | | |
| Skyhigh | `[unverified]` — Skyhigh GenAI controls | | | |
| Zscaler | `[unverified]` — Zscaler ZIA GenAI category | | | |

### DSPM-for-AI and Purview AI coverage matrix

Per [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/ai-chatgpt-enterprise] and [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/ai-claude-enterprise]:

| Capability | ChatGPT Enterprise | Claude Enterprise (preview) | Google Gemini |
|---|---|---|---|
| DSPM-for-AI prompt capture / audit | Supported | Supported | Third-party-AI-sites coverage via browser extension + endpoint DLP only |
| Microsoft Purview Audit | Supported | Supported | n/a |
| Data Lifecycle Management (DLM) | Supported | NOT supported | n/a |
| Data classification | Supported | NOT supported | n/a |
| Insider Risk Management (IRM) | Supported | NOT supported | n/a |
| Purview eDiscovery | Supported | NOT supported | n/a |
| Purview DLP | **NOT** supported | **NOT** supported | n/a |
| Sensitivity labels | **NOT** supported | **NOT** supported | n/a |

The previous "deep vs shallow" framing in this file conflated DSPM presence with broader Purview coverage. The above matrix is the authoritative breakdown — Claude Enterprise (preview) has DSPM-for-AI + Audit only; ChatGPT Enterprise has DSPM + Audit + Data classification + IRM + eDiscovery + DLM (still no DLP and no sensitivity labels).

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant after FP-tuning is complete:

```yaml
policy:
  name: "GenAI auto-unsanction — Generative AI category, coding-assistant + general LLM"
  type: AppDiscoveryPolicy
  category_filter:
    include:
      - "Generative AI"                        # single Microsoft-documented category
    exclude: []
  app_tag_filter:                              # custom App tags (not MS taxonomy)
    include:
      - "ai-coding-assistant"                  # tenant-defined custom tag
      - "ai-general-llm"                       # tenant-defined custom tag
      # ai-image, ai-video, ai-voice-avatar added in phase 2 per posture
  risk_filter:
    risk_score_max: 5                          # documented 0-10 scale; lower = higher risk
  usage_filter:
    users_per_day_min: 10                      # avoid alerting on lone-trial vendors
    days_in_use_min: 3                         # rule out one-day spikes
  app_allowlist:                               # exempted by app ID; quarterly review
    - id: "chatgpt-enterprise"                 # tenant SSO'd
    - id: "microsoft-copilot"
    - id: "github-copilot-enterprise"          # CDE-isolated tenant for engineering
    - id: "internal-ai-gateway"
    - id: "anthropic-claude-enterprise"        # if contracted
  governance:
    action: TagAsUnsanctioned
    secondary_action: Alert
    swg_block_script_generation: false         # script is partial-fail; manage at MDE NP
  endpoint_propagation:
    target: MDE_Network_Protection
    required_mode: Block                       # NOT Audit
    propagation_window_minutes: 60             # practitioner observation; not Microsoft-published SLA
  alerts:
    severity: Medium
    email_recipients: [genai-governance@example.com, soc-l1@example.com]
    daily_alert_limit: 100
  user_facing_block_page:
    custom_message: "AI tool not approved. Use [internal-AI-portal] or request via [intake-form]."
    intake_link: "https://intranet.example.com/ai-intake"
  review_cadence:
    allowlist_review: Quarterly
    blocklist_review: Quarterly
    risk_score_drift_review: Quarterly
    supply_chain_incident_review: AdHoc        # AML.T0010 triggered
```

The `risk_score_max: 5` is the MDA-published threshold heuristic on the documented 0-10 scale; the operationally-important guard is the `app_allowlist` + the **parallel internal risk-acceptance workflow** — the auditor will probe whether the firm relies on Microsoft's opaque score or on a documented internal decision. Document both.

## Variants

### Industry-specific

- **BFSI:** the dominant scope. Allowlist tightly curated; consumer-tier consumer AI universally blocked. The hard fight is between the engineering org wanting Copilot / Cursor / Replit Ghostwriter and the risk team wanting only the enterprise-tier with SSO + DPA. AML.T0010 supply-chain risk on the allowlist is named on the risk register. BNM RMiT third-party / outsourcing clauses apply to allowlisted vendors [VERIFY against current edition — illustrative only, not regulatory advice].
- **Healthcare:** PHI exposure is the dominant fear. AI vendors must pass HIPAA-BAA scrutiny before allowlisting; even "internal-data-OK" tier is suspect because internal data includes PHI. Image-generation use cases often universally blocked because of PHI-in-radiology concern.
- **Tech:** highest user-side resistance to the control. Sanction taxonomy is typically broader (more allowlisted vendors); intake SLA is tighter (1-week typical vs 2-3 for BFSI); IP / source-code concerns dominate over PII concerns. Custom SIT for internal codebase paths and project codenames is common (pairs with Policy 9 clipboard block).
- **Retail:** customer-data + payment data concerns; loyalty-programme PII the dominant data class; less classifier sophistication needed; high-volume blocked-attempt events (retail users trial more consumer tools than BFSI users).
- **Public sector:** sovereignty / data-residency concerns dominate. Allowlist constrained to vendors with in-jurisdiction infrastructure (or vendors with documented data-residency commitments). Classification-marking schemes (Official / Sensitive / Secret) determine which sanction-taxonomy tier a vendor sits in.

### Maturity-based

- **Immature:** "block everything AI" via category filter, no allowlist process, no intake SLA; users move to BYOD / personal devices; control becomes shadow-IT-by-other-means. FP rate at policy layer is "low" because nothing is sanctioned and exceptions are denied, but the *real* FP rate (people doing legitimate AI work outside the corporate stack) is enormous and invisible. Common at 6 months post-deployment for under-resourced programmes.
- **Mature:** documented sanction taxonomy with 3-4 risk tiers; allowlist with named vendor owners; quarterly review cadence; intake SLA typically 2-3 weeks; user-comms naming the sanctioned alternatives; help-desk runbook for blocked-attempt support; FP rate <10% at policy layer; named integration with the OAuth-governance workstream (per Use case 4).
- **Advanced:** AI-vendor risk-assessment template integrated with the wider vendor-risk programme (not a separate workflow); Microsoft Purview Adaptive Protection ties AI-usage signal to user-risk score, which now wires into DLP, DLM, and Conditional Access — cite [Microsoft Learn: https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection]; **Browser DLP via Microsoft Edge for Business** overlays browser-layer enforcement on Edge-on-Windows estates — cite [Microsoft Learn: https://learn.microsoft.com/en-us/purview/dlp-browser-dlp-learn] and [Microsoft Learn: https://learn.microsoft.com/en-us/purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz] `[VERIFY GA timeline against current Microsoft 365 roadmap]`; GSA-native GenAI-category block for off-corp roaming; DSPM-for-AI prompt-capture deployed on the allowlisted enterprise-AI vendors per the capability matrix above. Quarterly **AML.T0010 supply-chain rehearsal** — what does the firm do if an allowlisted AI vendor announces a compromise.

## Control mappings

- **CIS Microsoft 365 Foundations Benchmark v5.0.0** (30 April 2025): **CIS 1.3.4** (L1) Restrict non-admin users from creating tenants; **CIS 5.1.2.2** (L2) Ensure third-party integrated applications are not allowed; **CIS 2.4.3** (L2) umbrella — MDA-dependent detections.
- **Microsoft Secure Score** improvement actions in the **Apps** group: "Sanction or unsanction cloud apps", "Block unsanctioned cloud apps". Cite [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score] and [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]. Secure Score now uses the four-group structure (Identity / Device / Apps / Data).
- **Microsoft Zero Trust — Application pillar:** Objectives II / III / IV / V per [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/applications]. Identity pillar Objective IV (consent restriction) covers Use case 4's OAuth interlock per [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity].
- BNM RMiT clause(s): [BNM RMiT third-party risk + DLP + AI governance overlay](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition — illustrative only, not regulatory advice]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 42001 (AI management system) — relevant where the firm has adopted; mapping to allowlist-governance + risk-assessment-process clauses `[VERIFY against ISO/IEC 42001:2023]`
- NIST CSF 2.0 subcategory(ies): `GV.SC` (cybersecurity supply chain), `ID.AM-04` (services from suppliers), `PR.DS-05` (data leak protection), `DE.CM-09` `[VERIFY]`
- NIST AI RMF — `GOVERN-1` (AI governance structure), `MAP-1` (context), `MEASURE-2` (AI risk), `MANAGE-3` (allocate resources to manage AI risk) `[VERIFY against NIST AI RMF 1.0]`
- MITRE ATLAS coverage: `AML.T0010` (ML Supply Chain Compromise) — partial via allowlist governance; `AML.T0024` (Exfiltration via AI Inference API) — partial via category block. [MITRE ATLAS: https://atlas.mitre.org/]
- EU AI Act: limited applicability to discovery-and-unsanction itself; relevant where allowlisted AI vendors fall into high-risk categories under the Act `[VERIFY against EU AI Act Annex III, current consolidated text — illustrative only, not legal advice]`

## False-positive risk

- Approved-by-business AI vendor scoring ≤ 5 due to "SOC 2 = No" in Microsoft's catalogue but enterprise contract exists — exempt by app ID
- New legitimate AI tools in catalogue Unknown class — may be tagged Unsanctioned during legitimate trial
- Image / video / voice AI generators handled within the same "Generative AI" Microsoft category as LLMs — different risk class; distinguish via custom App tags
- Vendor mis-categorisation in Microsoft's catalogue — e.g. a productivity tool with a small AI feature surfaced as "Generative AI" category; resolve via in-product feedback to MS
- Long-tail catalogue lag — a vendor acquired by an approved parent; the catalogue still shows pre-acquisition data
- Vendor risk score drift — risk score recalculated by Microsoft; an allowlisted vendor suddenly scores ≤ 5 and the auto-unsanction logic re-fires

## Real-world FP experience

> **Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.**

Typical FP-rate trajectory:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1-W2 | 30-50% | Alert-only baseline; legitimate trials of new tools; contracted vendors not yet allowlisted; mis-categorised "this is a productivity tool with AI features, not an AI tool" |
| W4 | 15-25% | After initial allowlist seeding + after mis-categorisation feedback to MS; contracted-vendor exemptions added |
| W8 | 8-15% | After two cycles of intake-and-allowlist; user-comms wave reduces lone-trial behaviour; help-desk runbook handles edge cases |
| W12 | 5-10% | Mature taxonomy; quarterly review captures drift; intake SLA running |
| Steady-state | 3-7% | Quarterly review cycle catches new vendors before they grow; AML.T0010 incident-driven re-evaluations occasional |

Named FP scenarios encountered repeatedly:

| Scenario | Mitigation |
|---|---|
| Productivity app with embedded AI feature (Notion AI, Otter.ai, Grammarly) mis-categorised as "Generative AI" | Allowlist by app ID; submit catalogue-correction to Microsoft |
| Approved enterprise vendor scoring ≤ 5 because "SOC 2 = No" or "Hosted in country X = No" — vendor's enterprise contract addresses the underlying control | Allowlist by app ID with documented internal risk-acceptance; do not adjust risk-score threshold |
| New AI tool gaining sudden user-base during a hackathon or innovation sprint — legitimate but unsanctioned | Time-bounded exception via intake fast-track; review at sprint-end |
| AI plugin / extension running inside an already-allowlisted parent app (ChatGPT plugin store, Copilot extensions) | Sub-app surfacing limited; treat parent-app sanction as covering child plugins; document the residual gap |
| Marketing-team use of AI image generators (DALL-E web, Midjourney, Adobe Firefly) — legitimate non-sensitive use case | Custom App tag-based sub-tier (Public-OK tier); typically allowlist Adobe Firefly enterprise; block Midjourney free / DALL-E web by individual workflow |
| Engineering team trial of new AI coding assistant after one-week proof-of-concept | Time-bounded exception via intake; convert to full allowlist or deny at end of POC |
| Approved vendor's domain rotation breaks block — domain block-script defeated by CDN-fronted egress | Endpoint-level URL-pattern in MDE NP; or rely on the *application-tag-based* enforcement rather than the script |
| Allowlisted vendor's risk score drifts to ≤ 5 (Microsoft catalogue update) — auto-unsanction re-fires | Allowlist-by-app-ID is sticky; verify in policy match logs; submit catalogue-correction if score change unwarranted |

## Operational cost

- **Exception-handling load:** medium — 5-15 intake-ticket requests per week typical at a tier-2 BFSI tenant (practitioner observation); quarterly allowlist review consumes ~2 days of cross-functional time (Risk + Legal + IT + business owner)
- **Triage load:** low — block at endpoint is silent (no user-side ticket unless they raise via intake); SOC alert load is low because endpoint blocks generate aggregated rather than per-event alerts; ~5-20 alerts per week per 1k users at maturity (practitioner observation)
- **End-user friction:** high if blocked AI was in legitimate use — coaching message + sanctioned alternative essential; user-comms wave and intake-link on the block page are the operational difference between a workable control and a circumvented one

Typical staffing (practitioner observation): 0.2 FTE platform admin (policy tuning + allowlist maintenance); 0.1 FTE governance-forum coordinator (intake + quarterly review); 0.1 FTE vendor-risk analyst for AI-specific assessments (often shared with the wider vendor-risk programme). Engineering / Product business owners commit episodic time during intake-ticket reviews.

## Privacy / data-protection considerations

- AI vendor discovery includes user identity tied to AI app usage — workforce-monitoring data; document under PDPA / GDPR Art. 88 / equivalent
- Approved-AI vendor's data-handling posture matters: training-data policy, customer-isolation, jurisdictional location, sub-processor list — all must be captured in the AI-vendor risk assessment
- DSPM-for-AI prompt capture (separate policy, see MDA Policy 9 — Inline Prompt DLP) is **high-intrusion processing** under GDPR Art. 35 — DPIA before pilot; truncated content snippets in audit log are themselves regulated content
- Workforce notice in employee handbook + AUP must cover AI-usage monitoring scope and the sanction-taxonomy posture
- Cross-border: AI vendors host in many jurisdictions; the firm's data-residency posture (per PDPA / GDPR / DORA) constrains which vendors can sit in the Sensitive-OK / Internal-OK tiers

## Integration with broader programmes

- **AI governance forum:** the sanction taxonomy + the allowlist are the operational expression of the firm's AI governance posture. Allowlist additions / removals are typically signed off at the forum; quarterly review is a forum agenda item.
- **Vendor-risk programme:** AI-vendor risk assessments feed the wider sub-processor / third-party-risk register. The AI-vendor assessment template extends the standard template with AI-specific items (training-data policy, model-residency, AML.T0010 posture).
- **Audit cycle:** allowlist + intake-process evidence + quarterly-review minutes feed BNM RMiT third-party / outsourcing attestation [VERIFY — illustrative only, not regulatory advice]; ISO 42001 AI-management-system controls if certified `[VERIFY]`; NIST CSF 2.0 GV.SC + ID.AM-04 evidence pack; **CIS 2.4.3 / 5.1.2.2 / 1.3.4 evidence pack**; **Microsoft Secure Score Apps-group improvement-action snapshot** at attestation date.
- **Board reporting:** quarterly metric — (a) allowlist size + change-rate, (b) blocked-attempt count + trend, (c) intake-ticket SLA performance, (d) AML.T0010 supply-chain incidents on allowlisted vendors. The trend matters more than the absolute number. **Third-party-concentration framing:** the AML.T0010 supply-chain emphasis derives from BNM RMiT third-party-concentration / DORA Art. 28-30 / MAS TRM third-party-risk supervisory framing — **not** Microsoft's own positioning. Microsoft frames the AI vendor integration as "unified XDR / end-to-end security" per MCRA April 2025. Both lenses are legitimate; the regulator-derived framing belongs in board reporting, the Microsoft-derived framing in architecture docs.
- **Incident response runbook:** AI-vendor supply-chain compromise (AML.T0010) is its own playbook — what does the firm do if ChatGPT Enterprise discloses a compromise; revoke OAuth grants, suspend tenant SSO, communicate to users, preserve audit logs. Separate from the BAU MDA-side operations.
- **OAuth governance interlock:** AI-via-OAuth apps bypass the network-egress discovery (per Use case 4); approved-AI vendors must pass both the network-discovery allowlist AND the OAuth-grant verified-publisher restriction in Entra. Document the interlock; do not let the two workstreams diverge. Cite [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity] (Objective IV) + [Microsoft Learn: https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants].
- **DSPM-for-AI integration:** prompt-body capture on allowlisted enterprise-AI vendors feeds Purview Insider Risk Management adaptive scoping (per MDA Inline Prompt DLP policy; "Policy 12" is internal playbook numbering, not Microsoft documentation); see the DSPM coverage matrix above for the per-vendor Purview-capability breakdown — cite [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/ai-chatgpt-enterprise] and [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/ai-claude-enterprise]. Adaptive Protection wires the IRM signal into DLP, DLM, and Conditional Access — cite [Microsoft Learn: https://learn.microsoft.com/en-us/purview/insider-risk-management-adaptive-protection].

## Anti-patterns specific to this policy

1. **"Block everything AI"** — guarantees user migration to BYOD / personal devices / mobile hotspots; the policy reduces the visible signal of AI usage while leaving the risk almost unchanged. Common at month 6 of an under-resourced rollout. The fix is the sanction taxonomy + intake SLA, not relaxing the block
2. **"Tag-as-Unsanctioned with MDE NP in audit mode"** — single most common Day 90 misconfiguration. Audit mode produces no endpoint enforcement; the policy looks live but blocks nothing. There is no automatic operator notification of the audit-vs-block mismatch. Verify MDE NP state quarterly
3. **"Rely on Microsoft's opaque risk score as sole basis for automated enforcement"** — auditor cannot accept it; Microsoft publishes the 90+ factor count but not the factor weights. Document a parallel internal risk-acceptance / appeal process that decides which vendors land in which sanction tier independent of the catalogue score
4. **"Assume the published category taxonomy splits Generative AI into sub-categories"** — Microsoft's published Cloud App Catalog (as of June 2026) lists a single "Generative AI" category plus the adjacent "AI – MCP Server" and "AI – Model Provider" categories per [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/risk-score]. The five-way split (LLM / AI Coding Assistant / AI Image / AI Video / AI Voice-Avatar) is **not** in the Microsoft taxonomy. Use **custom App tags** to distinguish sub-uses within the single category. Earlier drafts of this file treated the split as Microsoft-published — corrected at 2026-06-10 QA pass.
5. **"No intake SLA"** — legitimate-business-need AI requests pile up with no review cadence; users circumvent; the control's credibility collapses. A 2-3 week SLA is the minimum workable
6. **"Treat the SWG block script as the enforcement layer"** — the generated IP/domain list does not match modern GenAI vendor egress (CDN-fronted, JA3-fingerprinted, dynamic domains). MDE NP application-tag-based enforcement is the actual control; the SWG script is partial-fail at best
7. **"Forget the AML.T0010 supply-chain rehearsal"** — the allowlist *is* the attack surface; a compromise of an allowlisted vendor (training-data poisoning, inference-side data exfil, prompt-injection-mediated OAuth grant escalation) hits the firm at the speed of the API. There must be a documented incident playbook for "what if an allowlisted AI vendor announces a compromise"
8. **"Approve OAuth-mediated AI access without going through the same allowlist"** — Notion AI, ChatGPT plugins, AI-marketplace apps connect via Graph permissions and bypass the network-egress discovery entirely. The OAuth-governance workstream and the AI-governance workstream must converge (per Use case 4); allowlisted vendors must clear both gates
9. **"No user-facing block page with intake link"** — silent endpoint block frustrates the user; user opens a personal-device session and continues; the help-desk has no signal. The block page is the most operationally-important UX in the entire policy
10. **"No quarterly risk-score-drift review"** — Microsoft's catalogue updates; an allowlisted vendor's risk score drifts; the auto-unsanction logic re-fires and disrupts users mid-week. Quarterly review captures this before it bites

## Coverage gaps

- BYOD / hotspot — off-corp network, MDE NP not in path; see [`../../08-failure-modes/byod-bypass.md`](../../08-failure-modes/byod-bypass.md) `[VERIFY path]`
- Personal MSA or personal Google sign-in to ChatGPT — Entra never sees session
- CDN-fronted vendor domains (T1090.002, T1568.002) — SWG block script defeated by class
- Image / audio prompts — DCS does not OCR / transcribe at session layer
- Mobile native apps (ChatGPT iOS / Android, Claude mobile, Gemini mobile) — not in MDE NP scope on mobile platforms
- AI access via OAuth-mediated SaaS (Notion AI, AI plugins to Slack / Salesforce / etc.) — bypasses network-egress discovery; see Policy 2 interlock
- Supply-chain compromise of an approved AI vendor (AML.T0010) — allowlist becomes the attack surface
- Prompt injection causing OAuth grant escalation (ATLAS AML.T0051) — out of scope of this policy entirely; application-layer control only
- 2026 alternatives: **Browser DLP via Microsoft Edge for Business** ([Microsoft Learn: https://learn.microsoft.com/en-us/purview/dlp-browser-dlp-learn] + [Microsoft Learn: https://learn.microsoft.com/en-us/purview/dlp-create-policy-prevent-cloud-sharing-from-edge-biz]) `[VERIFY GA timeline against current Microsoft 365 roadmap]` + GSA-native GenAI category block supersede the MDA → MDE NP chain for Edge-on-Windows estates with GSA licensed; prefer these where deployable

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
