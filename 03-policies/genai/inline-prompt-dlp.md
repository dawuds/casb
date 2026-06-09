# Inline prompt DLP

> Status: v0.0 — MDA column lens-reviewed from playbook v1 (Policy 9 — Day 90); other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [Risk-based session policy + Inline DLP + GenAI app governance](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC). Entra ID P1 required. SAML federation prerequisite on the AI vendor side.
> MDA playbook reference: [Policy 9](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Block clipboard paste of sensitive data into ChatGPT).

## Purpose

Block clipboard paste (and optionally Cut / Copy / Sent message) of cardholder data, SSN-class PII, source code, internal project codenames into sanctioned GenAI browser sessions. Counters MITRE ATT&CK `T1567 Exfiltration Over Web Service` narrowed to clipboard sub-channel (Prevent — partial). MITRE ATLAS `AML.T0024 Exfiltration via ML Inference API` — partial. **Not T1052 (Physical Medium)** — the original draft's mapping was wrong.

## What organisations use this for

This policy is what BFSI security teams reach for when the board has signed off on a ChatGPT Enterprise rollout and the audit committee wants a defensible answer to "what stops a developer pasting customer data into the prompt box". It is **prevention-leaning but session-scoped** — it only sees what passes through a CAAC-onboarded browser session against a SAML-federated AI tenant. It is not, and cannot be, the complete answer to GenAI data-loss risk. Most deployments pair it with DSPM-for-AI prompt-side visibility (where prompts and responses are captured natively by the AI vendor and surfaced to Purview / equivalent) to cover what session inspection cannot see — namely, what the user types directly rather than pastes.

The hardest single decision is the SIT mix. Too narrow = the policy looks decorative and gets challenged at audit. Too broad = developers complain to their MD that ChatGPT is unusable, and the exception list explodes within a month.

### Use case 1 — Tier-1 ASEAN universal bank, ChatGPT Enterprise rollout with developer pressure

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, in-scope for PCI DSS v4.0 (issuing + acquiring); board approved a 5k-seat ChatGPT Enterprise pilot for developer productivity
- **Trigger:** board sign-off for ChatGPT Enterprise was conditional on "DLP at the prompt boundary" — Chief Risk Officer asked Group Security to demonstrate a control that stops customer data and source code leaving the prompt box. Internal audit added it to the FY26 control inventory before pilot started
- **Scope:** pilot ring 1 = 500 developers + 200 wealth-management RMs; ChatGPT Enterprise SAML-federated to Entra; CAAC reverse-proxy on Edge for Business managed devices only; SITs = Credit Card + Malaysian NRIC + IBAN + custom "internal-codename" SIT + source-code-fragment classifier (Purview)
- **Outcome:** pilot ran 12 weeks audit-only; ~3,800 audit events triaged; 18 confirmed true-positives (customer PII pasted into prompts, mostly RM workflow); FP rate at flip-to-block was 11% (down from 47% at W1); developer-side friction surfaced "typing the data instead of pasting" within 3 weeks of block-action — closed with a parallel DSPM-for-AI prompt-capture rollout in pilot ring 2

### Use case 2 — Pre-IPO fintech, source-code-leak prevention as a category-1 risk

- **Org type:** digital-native challenger bank, ~800 employees, M365 E5, pre-IPO due diligence; engineering team uses ChatGPT Team plan with SSO; IP value concentrated in a payments-rail codebase
- **Trigger:** pre-IPO due diligence flagged "source code exposure to consumer LLMs" as a category-1 risk; CTO had personally observed engineers pasting whole files into prompts; board asked for a control before the next funding round
- **Scope:** engineering BU (~280 users); ChatGPT Team plan SAML-federated to Entra; custom SIT for internal codebase paths (e.g. `payments-rail/core/` family) + API-key regexes (the standard AWS / GCP / GitHub PAT family) + custom regex for the firm's own internal-secret-store format; ALL engineering devices managed
- **Outcome:** in W1-W4 audit mode the API-key SIT alone fired ~120 times — every single one was a legitimate secret being leaked (mostly from `.env` paste-overs); the policy became the engineering-side justification for proper secrets management (Vault rollout was already on the roadmap; the audit-mode evidence accelerated it by two quarters); steady-state FP rate ~6%, mostly from public-library identifier strings that resemble internal paths

### Use case 3 — Healthcare insurer, PHI-into-LLM prevention under HIPAA + state-level AI regs

- **Org type:** US-domiciled healthcare insurer with US + EU operations, ~12k employees, M365 E5, HIPAA covered entity + EU GDPR + California / Colorado / Texas AI-related rules in 2026; Copilot Chat + ChatGPT Enterprise both sanctioned
- **Trigger:** the firm's HIPAA officer flagged a near-miss — a claims-processor had pasted a screenshot-OCR'd claim summary into ChatGPT for "help drafting a denial letter"; that screenshot included member name + MRN + diagnosis codes
- **Scope:** all claims + medical-review staff (~4k users); ChatGPT Enterprise + Copilot Chat both under CAAC; SITs = US SSN + MRN-format + ICD-10 code pattern + claim-ID format; DPIA refreshed before pilot; coordinated with HIPAA Privacy Officer + DPO under one workforce-monitoring notice update
- **Outcome:** ~85% of audit-mode events were true PHI-in-prompt; cultural finding — staff treated ChatGPT as a "drafting tool" with no awareness that the prompt was processed by a third party. The control's main value at month 6 was **behaviour change** (PHI-in-prompt audit-event count fell ~70% from W1 to W24), not block-rate. Block was kept narrow (SSN + claim-ID only) — broader SITs remained audit-only to preserve the behavioural-feedback loop

### Use case 4 — Consulting / professional services firm, internal-project-codename leak as M&A risk

- **Org type:** management-consulting firm, ~15k consultants globally, M365 E5, frequent advisory work on M&A deals; project codenames are themselves market-moving information
- **Trigger:** an external incident at a peer firm (codename surfaced in a press leak; not the same firm but the same risk shape) prompted the General Counsel to commission a "codename hygiene" control; ChatGPT and Copilot Chat were the most-used GenAI tools
- **Scope:** all client-facing consultants (~11k users); custom SIT for the firm's project-codename namespace (every active engagement codename plus 90-day decommissioned codenames) — list refreshed weekly from the engagement-management system; ChatGPT Enterprise + Copilot Chat under CAAC; SITs scoped narrow on purpose (codenames only) so the FP-management problem stayed bounded
- **Outcome:** custom-SIT refresh job became the operational dependency — when the codename feed failed silently for ~9 days the policy went stale; recovered with a freshness monitor; deal-related codename audit events ran ~30/week, ~95% legitimate (consultant working on the engagement using the codename in a prompt — coached not blocked); the ~5% TP were near-misses where a codename appeared in cross-engagement copy-paste

## Implementation pattern

Typical 12-week rollout for a tier-2 BFSI tenant new to CAAC session policies on GenAI apps. The DPIA cycle is the long pole — start it before W1 if you do not already have one covering clipboard inspection.

| Week | Activity | Output / gate |
|---|---|---|
| W-4 → W0 | DPIA / TIA for clipboard inspection on sanctioned AI tools; workforce-monitoring notice update; works-council / employee-representative consultation where applicable (EU); legal sign-off on the audit-log content-snippet retention | DPIA approved; workforce notice issued; legal-sign-off recorded |
| W1 | Verify ChatGPT Enterprise / Team SSO + SAML federation against the AI vendor; manual onboarding of the custom ChatGPT app in CAAC; Entra CA policy "Use Conditional Access App Control" matched to the right user group | App onboarded to CAAC; pilot CA policy live; end-to-end test session works |
| W2 | Define SIT mix — standard set (Credit Card + jurisdiction-specific PII) + custom SIT for internal project codenames / codebase paths / internal-secret formats; publish custom SITs in Purview; baseline a 7-day audit-mode dry run on a pilot ring of 50 users | SIT inventory documented; pilot-ring baseline event count; initial FP signature catalogue |
| W3-W4 | Expand pilot ring to 500 users; action = **Audit only**; SOC + DLP-ops triage queue stood up; coach-message draft tested with comms / HR | Audit-mode FP rate baseline; coach-message approved; SOC triage runbook v1 |
| W5-W6 | First tuning pass — raise SIT confidence thresholds where appropriate; allowlist internal-codename false-friend patterns (e.g. legitimate vendor product names that collide); add user-group exclusions for documented exception roles (legal eDiscovery, M&A team if scope demands) | FP rate at W6 typically 15-25%; exception-group register signed off |
| W7-W8 | Per-SIT promotion — flip the cleanest SITs (typically Credit Card + SSN) to Block; keep noisier SITs (custom codename, source-code-fragment) in Audit | First Block-action policy live; user-friction feedback channel open (typically a Teams channel + Power Automate ticket) |
| W9-W10 | Broaden user scope from pilot to all sanctioned-AI users on managed devices; monitor for typing-instead-of-pasting workaround behaviour; pair-policy with DSPM-for-AI prompt capture if not already deployed | Tenant-wide coverage on managed-device estate; DSPM-for-AI signal cross-correlation tested |
| W11-W12 | Promote remaining SITs to Block where FP rate allows; document quarterly review cadence; first audit-evidence pull tested (control-effectiveness evidence pack) | Production-ready handoff; quarterly tuning rhythm scheduled |
| W13+ | Steady-state — monthly SIT freshness check (especially custom-codename list); quarterly threshold tuning; semi-annual DPIA refresh; coordinated review with DSPM-for-AI signal | Audit-evidence package reusable; quarterly board-metric slot |

Skipping the W-4 DPIA cycle is the single most common procedural failure. Clipboard inspection is high-intrusion processing under GDPR Art. 35 and most equivalent regimes; pilots launched without it get pulled by Privacy.

## Action

- Primary: **block** with custom user-education message (per-SIT, after FP tuning)
- Phase-1 default: **audit** action for minimum 4 weeks to baseline FP rate before per-SIT flip-to-block
- Coach variant: **coach + log** for low-confidence SITs that should not block but should feed the behaviour-change metric

## Scope

- **Users:** all employees on managed devices, excluding documented exceptions (legal eDiscovery, M&A engagement teams where scope demands, executive assistants under named-individual exception)
- **Apps:** ChatGPT Enterprise / Team (SAML-federated only); Copilot Chat; other GenAI tools that support tenant SAML federation and have been manually onboarded to CAAC
- **Device posture:** managed (CAAC reverse-proxy only sees browser sessions on Entra-federated devices)
- **Network position:** any (browser session via Entra)
- **Traffic:** clipboard Paste (and optionally Cut / Copy / Sent message)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create Session policy. Plus matching Entra CA policy with "Use Conditional Access App Control" | Session control type = Block activities; App = Custom ChatGPT Enterprise (Manual onboarding); Activity type = Clipboard Paste (+ Sent message with content inspection); Inspection method = DCS; SIT = Credit Card + jurisdiction PII + custom codename SIT + source-code-fragment classifier; Action = Block with custom user-education message; start with Audit, escalate to Block per SIT after FP tuning | **ChatGPT must be onboarded to CAAC via SAML federation. Consumer ChatGPT (chat.openai.com with free or personal Plus account) does NOT support SAML.** Deployable ONLY against ChatGPT Enterprise or ChatGPT Team plan with SSO configured (verify against OpenAI's current SSO-by-plan page on day of deployment) `[VERIFY]` | Clipboard inspection is **high-intrusion processing** under GDPR Art. 35 / equivalent — DPIA before pilot. Audit log may retain truncated content snippets — itself regulated content. At 5k seats, clipboard inspection inflates Sentinel ingest by ~100 GB/month (sysint estimate). Policy only covers the browser session on CAAC-onboarded ChatGPT URL — opening ChatGPT in different browser / desktop app (Mac/Windows clients exist) / personal device / personal account = bypass |
| Netskope | `[unverified]` — Netskope inline GenAI policy with prompt-content inspection | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access inline GenAI controls | | | |
| Skyhigh | `[unverified]` — Skyhigh inline GenAI | | | |
| Zscaler | `[unverified]` — Zscaler ZIA + Data Protection inline GenAI | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant after FP-tuning is complete (W12+, per-SIT promotion done):

```yaml
policy:
  name: "Inline prompt DLP — ChatGPT Enterprise tenant-wide"
  type: SessionPolicy
  session_control: BlockActivities
  app_filter:
    - name: "ChatGPT Enterprise (Custom)"           # manual CAAC onboarding
      onboarding: SAML-federated-via-Entra
    - name: "Copilot Chat"
  activity_filters:
    activity_types:
      - ClipboardPaste
      - SentMessageWithContent                       # inspects the prompt body itself
  user_filter:
    include_groups:
      - "All-Sanctioned-AI-Users"
    exclude_groups:
      - "Legal-eDiscovery-Exception"
      - "M&A-Engagement-Active"
      - "Executive-Assistants-Named"
    require_device_compliance: true                  # managed devices only
  inspection:
    method: DataClassificationService
    sensitive_information_types:
      # promoted to Block after W8 tuning
      - id: "credit-card-number"
        confidence: 85
        minimum_violations: 1
        action: Block
      - id: "malaysia-nric"
        confidence: 85
        minimum_violations: 1
        action: Block
      - id: "iban"
        confidence: 85
        minimum_violations: 1
        action: Block
      # held at Audit pending further tuning (W12 state)
      - id: "custom-sit:internal-codename-active"    # refreshed weekly from engagement system
        confidence: 75
        minimum_violations: 1
        action: Audit
      - id: "custom-sit:source-code-fragment"        # internal repo path family
        confidence: 80
        minimum_violations: 1
        action: Audit
      - id: "custom-sit:api-key-regex-pack"          # AWS / GCP / GitHub PAT + internal secret-store
        confidence: 90
        minimum_violations: 1
        action: Block                                # narrow regex, safe to block
  governance:
    block_message:
      title: "This paste is blocked"
      body: |
        The content you tried to paste includes data classified as
        sensitive (e.g. cardholder data, personal identifiers, internal
        secrets). Please use the sanctioned-AI alternative documented at
        go/ai-safe-use, or contact the AI Risk team if you need help.
      help_link: "https://intranet.example.com/ai-safe-use"
    notify_user: true                                # the block IS the coaching moment
    notify_admin: false                              # avoid SOC fatigue from coach events
  audit_log:
    capture_truncated_snippet: true
    snippet_max_chars: 100                           # keep small — it is itself regulated
    snippet_retention_days: 30                       # tighter than the M365 default for regulated content
    snippet_access_control: ["dlp-admin-group"]
  sentinel_forward:
    enabled: true
    ingest_estimate_gb_per_month: 100                # ~5k seats; budget line, not just privacy
  custom_sit_refresh:
    internal_codename_feed: "engagement-management-export"
    cadence: "weekly"
    freshness_alert_days: 3                          # alert if feed has not refreshed
```

The `confidence: 85` floor on the productised SITs (Credit Card, NRIC, IBAN) is the typical post-tuning baseline for tenant-wide Block. Lower confidence = developer revolt within a week. The custom SITs sit on a different curve — they need a longer audit-only soak because the FP profile is org-specific rather than format-driven.

## Variants

### Industry-specific

- **BFSI:** PCI DSS-class SITs + jurisdiction PII + custom internal-secret regex; mandatory pairing with DSPM-for-AI for the typed-instead-of-pasted gap; FP fights centre on 16-digit customer reference numbers passing Luhn and internal codenames that resemble vendor product names; auto-block on cardholder data is the audit-defensible default
- **Healthcare:** PHI classifiers (US SSN, MRN format, ICD-10 codes, claim-ID family); HIPAA Privacy Officer co-owns the DPIA; in US tenants, state-level AI rules (California / Colorado / Texas, 2026) add a workforce-notice overlay; the screenshot-OCR-then-paste pattern is real — pair with image-handling guidance
- **Tech:** source-code-fragment classifier + API-key regex pack are the dominant SITs; productised PII classifiers are minor; the audit-mode phase doubles as a secrets-leakage discovery exercise (often surfaces unmanaged secrets in `.env` paste-overs); engineering pushback on Block is significant — coach + log for source-code, Block only for high-confidence secret formats
- **Retail:** loyalty-programme PII + cardholder data + customer transaction histories; the FP-tuning problem is high volume of legitimate-looking 16-digit identifiers (order numbers, loyalty IDs); custom-SIT exclusions for own-format identifiers required
- **Legal:** matter-coordinator and partner-secretary exception groups; client-matter codenames as the dominant custom SIT (very similar to consulting use case 4); privilege-aware audit-log access controls (the snippet itself may carry privileged content)
- **Public sector:** government-ID classifiers per jurisdiction; classification-marking SITs (Official / Sensitive / Secret family); some agencies prohibit consumer-AI use entirely — in those tenants this policy is the **enforcement** of an existing policy, not a discretionary control

### Maturity-based

- **Immature deployment:** one session policy, all SITs at vendor-default confidence (~50%), tenant-wide, Block on day one, no DPIA, no workforce notice update, no DSPM-for-AI pairing. Within 4 weeks: developer revolt, executive exception list growing weekly, a typing-instead-of-pasting workaround circulating internally. By month 6 the policy is rolled back to Audit-only or quietly disabled. The auditor finds the rollback and the programme loses credibility.

- **Mature deployment:** per-SIT confidence tuning complete; productised SITs (cardholder data, jurisdiction PII) at Block with confidence ≥85%; custom SITs (codenames, source-code, internal secrets) under quarterly refresh with documented FP rate; DPIA refreshed semi-annually; exception register published and signed off by Privacy + Legal; SOC triage runbook integrated with the broader DLP-ops queue; FP rate <15% steady-state; clipboard-inspection ingest line documented in the SIEM budget.

- **Advanced deployment:** per-user-group SIT mix (developers get source-code + secrets SITs; RMs get customer-PII SITs; legal gets matter-codename SITs); custom-SIT refresh fully automated from authoritative source (engagement system, secrets-store inventory, BIN-list system) with freshness monitoring; paired with DSPM-for-AI prompt capture for typed-content visibility; correlated with Purview IRM as an additional insider-risk signal; coach-message effectiveness measured (does the audit-event count for a coached SIT drop over the 90 days following first coach?); per-SIT TP rate published as a control-effectiveness metric on the GRC dashboard; integration with Edge for Business `gen-ai.app-control` browser-layer enforcement (2026 H2) on Edge-on-Windows estates removes the SAML-federation prerequisite for a subset of apps `[VERIFY against current MS roadmap]`.

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage + AI governance overlay](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition — RMiT does not yet name GenAI explicitly; mapping is via the broader data-leakage and technology-risk clauses]`
- PDPA MY 2024: personal-data protection at the prompt boundary; workforce-monitoring notice requirements
- HKMA SA-2 / MAS TRM: technology-risk and data-protection overlays `[VERIFY per current edition]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.11 (transparency to data subjects on monitoring)
- ISO/IEC 42001 (AI management system): governance of inputs to AI systems `[VERIFY clause]`
- NIST CSF 2.0 subcategory(ies): `PR.DS-05` (data leak protection), `DE.CM-09` (monitoring of unauthorized personnel) `[VERIFY]`
- NIST AI RMF 1.0: MAP / MEASURE functions on data-input controls `[VERIFY]`
- MITRE ATLAS: `AML.T0024` (Exfiltration via ML Inference API) — partial via clipboard inspection
- EU AI Act: high-risk-system input-governance overlay where the consumer-facing AI use is itself in scope `[VERIFY]`
- PCI DSS v4.0: where cardholder data is in the SIT mix, Req. 3 (protect cardholder data) and Req. 12.10 (incident response — prompt-leak runbook)

## False-positive risk

- Clipboard content inspection produces a lot of noise initially — code snippets, 16-digit identifiers, employee IDs all trigger Credit Card SIT
- Internal-project-codename SIT may fire on legitimate publications and external communications
- Productised PII SITs fire on synthetic / test data in developer-facing prompts (e.g. unit-test fixtures)
- Source-code-fragment classifier fires on documentation pastes and StackOverflow-style snippets that incidentally include public-library identifier strings

## Real-world FP experience

Typical FP-rate trajectory in a tier-2 BFSI tenant new to inline GenAI prompt DLP (assumes the recommended audit-then-block phasing):

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 40-60% | Vendor-default SIT confidence (~50%); custom SITs not yet tuned; no allowlists; new-policy noise across all SIT classes |
| W4 | 20-30% | After per-SIT confidence threshold raise (default ~50% → 85% for productised SITs); after first exclusion-group additions (legal eDiscovery, M&A) |
| W8 | 12-18% | After custom-codename allowlist for known false-friend patterns; after source-code classifier tuning to exclude common public-library identifiers |
| W12 | 8-15% | Mature per-SIT tuning; custom-SIT refresh cadence operating; per-user-group SIT mix applied |
| Steady-state | 5-10% | Quarterly tuning cycle; custom-SIT freshness monitor active; coach-message-driven behaviour change reducing event volume independently of FP rate |

Named FP scenarios encountered repeatedly across deployments:

| Scenario | Mitigation |
|---|---|
| 16-digit customer reference numbers / order IDs passing Luhn check fire as Credit Card | Raise Credit Card SIT confidence to ≥85%; add custom exclusion regex for the firm's own reference-number format |
| Internal project codename collides with a public vendor product name | Allowlist the specific collision string in the custom-codename SIT exclusion list; review weekly when the codename feed refreshes |
| Source-code-fragment classifier fires on documentation prose that contains public-library identifiers (e.g. `numpy.array`) | Tune source-code classifier to require minimum-violations ≥3 OR exclude prose-context detections; alternatively keep at Audit, never Block |
| API-key regex matches a UUID that resembles a token format | Tighten regex with prefix-anchor (e.g. AWS keys must start with `AKIA`); reject UUIDs by structural test |
| Developer pastes unit-test fixture containing fake PII (synthetic SSN like `123-45-6789`) | Treat as FP and exclude well-known synthetic-PII patterns; alternatively accept as TP because the behaviour itself is the leak (synthetic-vs-real distinction may not be obvious to the user) |
| OCR'd screenshot pasted — text is recognised by DCS as Credit Card | Often accepted as TP because real cardholder data has been pasted; the screenshot-as-paste vector is the threat |
| Legal eDiscovery / M&A team legitimately pasting client-matter content with codenames | Documented exception group with named-individual membership; quarterly re-attestation |
| Vendor-side false-friend — sanctioned-AI app changes URL behaviour after a vendor update, audit events spike | Re-verify CAAC onboarding monthly; subscribe to vendor change notifications |
| Productised SIT vendor update changes default detection signature mid-cycle | Pin SIT confidence threshold; alert on detection-volume step-change across all policies |

## Operational cost

- **Exception-handling load:** medium during the W4-W12 tuning ramp (10-20 exception requests per week typical for a 5k-seat pilot); low steady-state (3-8 per week)
- **Triage load:** high during audit phase (typical 200-400 audit events per day for a 5k-seat tenant at W1, falling to 30-80 at steady-state); medium ongoing after Block-action promotion (Block events themselves are usually TP, but the user-facing block requires a help-desk capacity for exception requests)
- **End-user friction:** medium-to-high during W1-W12; users see paste blocks; coaching message + sanctioned alternative essential. The first 30 days after Block-promotion typically produce the largest help-desk wave
- **SIEM ingest cost:** material — at 5k seats, clipboard inspection inflates Sentinel ingest by ~100 GB/month (illustrative estimate, validate against actual). Budget line, not just a privacy concern
- **Custom-SIT operational dependency:** the internal-codename feed (and equivalent custom-SIT data sources) becomes a production dependency — needs freshness monitoring, named owner, on-call alert if feed goes stale

Typical staffing: 0.5 FTE platform admin during the 12-week ramp; 0.3 FTE steady-state. DLP-ops analyst contributes another 0.3-0.5 FTE for triage during ramp, 0.1-0.2 FTE steady-state. AI Risk function (where one exists) commits ~0.1 FTE for the quarterly review.

## Privacy / data-protection considerations

- Clipboard inspection on the user's browser is **high-intrusion processing** — DPIA mandatory under GDPR Art. 35 / equivalent; works-council / employee-representative consultation in EU jurisdictions
- Workforce-notice posture: users must be informed in the AUP and onboarding materials that clipboard activity is inspected when accessing sanctioned AI tools — explicit, not buried
- Truncated content snippets in audit log = regulated content — itself subject to storage controls. Tighten retention (typical: 30 days) and access controls (DLP-admin group only) below the M365 defaults
- Cross-border: where DCS scanning happens vs where tenant data resides. For Malaysian / Singapore / Hong Kong tenants, Japan East primary-data region (2025 H2) `[VERIFY]` materially improved the cross-border posture
- DPIA scope must explicitly include: the SIT inventory, snippet retention and access, the typing-vs-pasting gap (transparency), the device-posture limitation (managed devices only — what happens on unmanaged is not seen), and the interaction with DSPM-for-AI prompt capture if paired
- HR / works-council sensitivity: clipboard inspection can be characterised as surveillance — frame as data-protection control, not productivity monitoring; explicit no-individual-performance-evaluation commitment in workforce notice
- PDPA MY 2024: workforce notice in employee handbook + AUP must cover SaaS-prompt-inspection scope explicitly

## Integration with broader programmes

- **PCI DSS audit cycle:** where Credit Card SIT is in the policy, the Block-rate + audit-event evidence feeds PCI DSS Req. 3 / Req. 12.10 attestation. Quarterly pull; annual auditor pull. The "data flowed to a third-party LLM" scenario is a Req. 4 (encryption in transit to third parties) consideration — coordinate with the CHD scoping team
- **AI risk register / NIST AI RMF deliverables:** the policy is a MEASURE-function control on data-input governance; effectiveness metrics (audit events / Block events / coached events / FP rate) feed the AI risk register
- **DPIA cycle:** annual refresh of the clipboard-inspection DPIA; semi-annual where the SIT mix changes materially or where the user-group scope expands
- **Vendor-risk programme:** the sanctioned-AI vendor (OpenAI for ChatGPT Enterprise, Microsoft for Copilot Chat, etc.) is a sub-processor under PDPA / GDPR; sub-processor inventory + DPA review + SOC 2 / ISO 27018 attestation pull. Microsoft is concurrently the CASB sub-processor on the inspection side
- **Board / executive reporting:** quarterly metric pair — (a) audit-event count + Block-event count by SIT family, trend; (b) measured behaviour change (audit-event volume for a given user group following first coach). Trend matters more than absolute number
- **Incident response runbook:** a confirmed prompt-leak event (Block triggered with high-confidence TP, OR detected post-hoc via DSPM-for-AI) feeds the data-loss IR playbook; preservation hold via Purview eDiscovery; vendor-side data-subject request (vendor data-deletion API where the user's prompts can be expunged from the AI side)
- **Insider Risk Management (Purview IRM or equivalent):** repeated coach-event-then-Block-event pattern by the same user is an IRM signal; correlate with departure signals (Use case 1 from `../detect/mass-download-alert.md` patterns)
- **Audit committee / Group Risk:** for BFSI, the policy is the audit-defensible answer to "what stops a developer pasting customer data into ChatGPT" — explicit slot in the FY control inventory; control-effectiveness evidence pulled quarterly

## Anti-patterns specific to this policy

1. **"Block on day one, tenant-wide, no audit-mode soak"** — developer revolt within a week; exception list explodes; programme rolled back; auditor finds the rollback. This is the single most common failure mode for inline prompt DLP rollouts
2. **"Skip the DPIA / works-council consultation because we already have a DPIA for CASB"** — clipboard inspection is materially more intrusive than the at-rest API-mode CASB controls the existing DPIA covers; Privacy pulls the policy mid-pilot
3. **"Use vendor-default SIT confidence (~50%) in production"** — produces the W1 40-60% FP rate; must raise to ≥85% on productised SITs for any production deployment
4. **"Block source-code-fragment matches"** — engineering pushback is severe and largely legitimate (Block on a documentation paste containing `numpy.array` is broken); keep source-code SITs at Coach + Audit, never Block
5. **"Treat the inline policy as the complete answer to GenAI data-loss"** — clipboard inspection sees pasted content only. Typing instead of pasting is a fundamental bypass. Pair with DSPM-for-AI prompt capture; document the gap in the AI risk register
6. **"Onboard consumer ChatGPT to CAAC"** — not possible. Consumer `chat.openai.com` does not support SAML. The policy applies only against ChatGPT Enterprise / Team plans with SSO. Treat consumer-AI access as a separate problem (Shadow-AI discovery + unsanction; see [`shadow-ai-discovery.md`](shadow-ai-discovery.md) and [`auto-unsanction-genai.md`](auto-unsanction-genai.md))
7. **"Forget the desktop ChatGPT app"** — Mac and Windows native ChatGPT clients exist; CAAC reverse-proxy does not see them. Either block the desktop-client install via endpoint management, or accept the gap and document
8. **"No exception process, no exception register"** — legitimate exceptions exist (legal eDiscovery, M&A team during active deal, named executive support roles). Without a documented exception path, users invent workarounds; you lose visibility on the legitimate exceptions and on the workaround behaviour
9. **"Notify admins on every audit event"** — SOC fatigue. The coach-message-to-user IS the signal during audit phase; admin notification should fire only on Block events or on repeated-by-same-user audit clusters
10. **"Let the custom-SIT feed (codenames / secrets / BIN list) go stale silently"** — the policy degrades into uselessness over weeks, and the auditor finds the gap. Freshness monitor + named owner on every custom-SIT data source
11. **"Treat Block-event count as the primary metric"** — Block events are a small fraction of value; the larger value is behaviour change driven by audit + coach events. Track audit-event-volume-by-SIT trend over time as the control-effectiveness measure, not Block-count alone

## Coverage gaps

- Image / screenshot / audio of the same data (DCS does not OCR / transcribe at session layer)
- **Typing instead of pasting (no paste event fires)** — fundamental bypass class. Pair with DSPM-for-AI prompt capture for the typed-content visibility
- Desktop ChatGPT apps (Mac / Windows clients exist) — CAAC reverse-proxy not in path
- Personal account on sanctioned-app domain (user signs in to ChatGPT with personal credentials instead of corporate SSO) — see [`shadow-ai-discovery.md`](shadow-ai-discovery.md)
- Non-Edge browsers without CAAC routing — depends on enterprise browser strategy
- Unmanaged devices — CAAC requires device-compliance; BYOD / hotspot / personal-laptop scenarios bypass the policy entirely
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) does not apply (this is session-mode) but the analogue is **typed-not-pasted** = no inspection event fires
- Slow-and-low prompt-by-prompt PII leak (one field at a time across many prompts) — neither inline DLP nor DSPM-for-AI catches the aggregate pattern reliably
- 2026 alternative: Microsoft Edge for Business `gen-ai.app-control` (H2 2026 GA) `[VERIFY against current MS roadmap]` — browser-layer enforcement without SAML-federation prerequisite, for Edge-on-Windows estates; partial replacement for this policy on Edge-only estates

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
