# Sanctioned-tenant pinning for GenAI

> Status: v0.0 — cross-vendor pattern; MDA implementation via tenant-restriction headers + CAAC; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [Risk-based session policy + GenAI app governance + tenant-restriction header injection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC) + SWG header injection. Hybrid.
> MDA playbook reference: companion to [Policy 9](../../04-vendors/microsoft-defender-for-cloud-apps.md) (clipboard paste block).

## Purpose

For sanctioned GenAI tools (ChatGPT Enterprise, Copilot, Gemini for Workspace, Claude Enterprise) — pin sessions to the corporate workspace and block consumer-tenant or personal-account variants of the same SaaS family. Reduces the "user signs into ChatGPT with personal Gmail on a corporate device → exfiltrates to personal account" pattern. Counters MITRE ATT&CK `T1078.004 Cloud Accounts` (personal-account variant) and the personal-tenant variant of `T1567`. Adjacent to MITRE ATLAS `AML.T0010` (ML supply chain) where the personal-account variant becomes the data-egress channel into an unmanaged vendor relationship.

## What organisations use this for

This policy sits at the intersection of GenAI adoption and identity-perimeter discipline. Organisations sanction a corporate GenAI tenant (typically ChatGPT Enterprise, Copilot, or Gemini for Workspace) precisely so they can negotiate enterprise data-handling terms — no training on prompts, audit logging, DPA, residency. The control that makes those negotiated terms enforceable is the **denial of the consumer-tier path on corporate surfaces**. Without tenant pinning, the enterprise contract becomes a piece of paper that the workforce routes around in three clicks: sign out of corporate ChatGPT, sign back in with personal Gmail, paste the same prompt. Audit cannot tell the two flows apart from the network logs alone — both hit the same `chat.openai.com` domain.

The policy is rarely the first GenAI control deployed — Discovery (MDA Policy 1) and auto-unsanction (MDA Policy 8) usually come first. Tenant pinning is the second-wave control once a sanctioned tenant exists. The hardest part is not the technology — `Restrict-Access-To-Tenants` headers are well-documented — it is the cross-vendor consistency: ChatGPT supports SAML tenant-restriction for Enterprise / Team; Gemini for Workspace uses Google's own `X-GoogApps-Allowed-Domains`; Anthropic Claude added SCIM + SAML in 2025; Copilot is implicitly tenant-pinned via Entra. Five vendors, five header conventions, three sign-in surfaces (browser, native desktop, mobile) — every gap is an exfiltration channel.

### Use case 1 — Tier-1 ASEAN universal bank, post personal-account ChatGPT exfil incident

- **Org type:** large universal bank, ~30k employees, M365 E5 + ChatGPT Enterprise, BNM RMiT supervised, MAS TRM supervised (SG branch)
- **Trigger:** internal investigation found a credit analyst had pasted a client's full restructuring deck into personal ChatGPT Plus over six weeks before resigning. The corporate ChatGPT Enterprise tenant had been live for four months; the bank had assumed users would prefer the enterprise variant because it was "better". DLP at email + DSPM-for-AI prompt capture (MDA shallow column for ChatGPT Enterprise had not yet been operational) missed the personal-account flow entirely — Entra had no visibility because the user signed in with personal Gmail
- **Scope:** all employees + contractors (~30k); tenant pinning via `Restrict-Access-To-Tenants` injected at GSA; CAAC session policy on ChatGPT Enterprise; complementary block at MDE Network Protection on `chat.openai.com` for any session NOT carrying the enterprise tenant claim
- **Outcome:** ~400 personal-account ChatGPT sign-in attempts blocked per week in the first month, ~80 per week at steady state. Three users escalated to HR for repeated attempts after the block message. Board cyber report carried a "GenAI tenant-pinning coverage" KPI in the quarter following deployment. Residual gap acknowledged: BYOD off-corp-network personal-account ChatGPT remains uncovered; compensating control = workforce AUP + DPIA-driven training

### Use case 2 — Digital-native challenger bank, post-Storm-2372 OAuth cleanup

- **Org type:** neobank, ~800 employees, M365 E5 + Copilot, Google Workspace for engineering, high-velocity product team
- **Trigger:** Storm-2372 device-code-phishing campaign (Microsoft Threat Intel, 2025) tricked three engineers into consenting to malicious OAuth apps that masqueraded as AI plugins. Forensic review surfaced a broader pattern — engineers had also been signing into personal ChatGPT / Claude with personal Google accounts on corporate Macs, sometimes pasting production code. The board treated both as a single class — uncontrolled AI-vendor identity sprawl — and asked for a single answer
- **Scope:** all engineering + product (~400 users initially); Copilot tenant-pinned via Entra (native); ChatGPT Enterprise tenant-pinned via SWG header injection; Claude Enterprise tenant-pinned via SAML SSO post-rollout; Gemini blocked entirely until Workspace-tier decision made
- **Outcome:** 100% of in-scope users moved to enterprise variants within 8 weeks. Two engineers raised support tickets — both legitimate cross-team consulting work, resolved by adding a documented partner-tenant entry. Policy became a precondition for the firm's later DSPM-for-AI rollout (could not capture prompts on a tenant they didn't control). Personal-account attempts dropped to <5/week after month 3

### Use case 3 — Merchant acquirer with multi-jurisdiction CHD, GenAI-residency cleanup

- **Org type:** payments processor operating across MY / SG / HK / TH; in-scope for PCI DSS v4.0 + BNM RMiT + MAS TRM + HKMA SA-2
- **Trigger:** PCI DSS v4.0 QSA pre-assessment finding — "use of unsanctioned generative AI tools by personnel handling cardholder data poses an undocumented data-flow risk; need control evidence that CHD-handling staff cannot route CHD-class content to consumer AI tenants whose residency / training posture is undocumented" (illustrative finding wording — actual QSA reports vary)
- **Scope:** Cards + Acquiring + Risk + Customer Service business units (~2.5k users initially) handling CHD; tenant pinning on ChatGPT, Gemini, Claude, Copilot; complementary clipboard-paste block on ChatGPT Enterprise (MDA Policy 9) to stop CHD-pattern paste into prompts even on the enterprise variant
- **Outcome:** documented control evidence for PCI DSS Req. 7 + Req. 12.10 (incident response — AI-vendor scope); residency story per-jurisdiction (each business unit's sanctioned AI tenant region documented in DPIA refresh); residual risk = mobile native ChatGPT app (Mac/iOS desktop) bypass partially closed by MAM, partially accepted

### Use case 4 — Tier-1 European bank under EU AI Act + DORA dual posture

- **Org type:** tier-1 European universal bank, ~80k employees, M365 E5 + ChatGPT Enterprise + Copilot, in-scope for EU AI Act provider/deployer obligations and DORA ICT third-party concentration risk
- **Trigger:** EU AI Act provider/deployer obligations bringing into scope every AI tool employees use to process customer data; legal asked "can we evidence that customers' personal data does not leave the firm via personal-account AI?"; in parallel, DORA Art. 28-30 third-party concentration analysis flagged unsanctioned-AI as an undocumented dependency cluster
- **Scope:** entire workforce (~80k); tenant pinning across all four sanctioned vendors; integrated with Workforce IAM joiner-mover-leaver flow such that leavers' enterprise AI access revokes within 24h
- **Outcome:** legal got the evidence pack (sign-in event logs proving enterprise-only access on managed devices); EU AI Act deployer documentation includes the tenant-pinning control as a primary technical safeguard; DORA concentration analysis updated to treat enterprise AI vendors as in-scope ICT third parties with full vendor-risk attestation cycle. Operational cost has been higher than expected — vendor sprawl management is now a permanent 0.5 FTE role

## Implementation pattern

Typical 10-week rollout sequence for a tier-2 BFSI tenant new to enterprise GenAI + tenant-pinning:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Inventory of sanctioned AI vendors; map each to tenant-pinning mechanism (Entra-native for Copilot; SAML + tenant-restriction header for ChatGPT Enterprise; SAML for Claude; `X-GoogApps-Allowed-Domains` for Gemini); document workforce AUP language | Sanctioned-AI register; pinning-mechanism matrix per vendor |
| W2 | Identify personal-account-AI usage baseline — pull SWG / MDE network telemetry for `chat.openai.com`, `gemini.google.com`, `claude.ai`, `copilot.microsoft.com` egress by user over 30 days; classify as enterprise (carries tenant claim) vs personal | Baseline volume of personal-account AI sessions; user-segment view (who, how often, what time of day) |
| W3 | Configure tenant-restriction headers at SWG / GSA — `Restrict-Access-To-Tenants` for Microsoft endpoints, `X-GoogApps-Allowed-Domains` for Google, vendor-specific equivalents for OpenAI / Anthropic. Audit-only mode | Headers injecting; audit telemetry showing the inject is hitting the right traffic |
| W4 | Configure CAAC session policy on each sanctioned-tenant variant; matching Entra Conditional Access policy with "Use Conditional Access App Control" — bilateral CAAC trap closed | Session policies live in audit-only; verify both halves with named test users |
| W5 | Pilot enforcement on a single BU (e.g. Compliance, ~200 users) — flip headers + CAAC to block mode; monitor exception queue daily | Pilot exception queue baseline; tuning of partner-tenant allowlist; refinement of user-facing block-page message |
| W6-W7 | Phased rollout to remaining BUs; close mobile-native gap via Intune MAM (managed-app restriction on ChatGPT iOS / Android apps); document break-glass for incident-response use | Per-BU enforcement live; MAM policies enforced on managed mobile; named break-glass account documented |
| W8 | Integration with offboarding — leavers' enterprise AI access revoked within 24h via SCIM (where supported) or manual cycle; integration with DSPM-for-AI (MDA prompt capture) on the sanctioned tenants | Joiner-mover-leaver flow includes AI vendor revocation step; DSPM-for-AI live on captured surfaces |
| W9-W10 | Steady-state ops — quarterly review of sanctioned-AI register; weekly review of personal-account-attempt alerts; monthly review of partner-tenant allowlist; first audit-evidence pull tested | Audit-evidence package; quarterly attestation cadence agreed; KPI committed to board cyber report |
| W11+ | Quarterly tuning; annual DPIA refresh; vendor-risk attestation cycle (each sanctioned AI vendor in the ICT third-party register) | Steady-state operating model |

The W2 baseline pull is the load-bearing step. Without it, the rollout flies blind into the user-friction question — how many people will this block on day one. With it, the firm knows the population in advance and can stagger communications.

## Action

- Primary: **block** sign-in to personal-tenant variant of sanctioned GenAI from corporate network or managed device
- Companion: **monitor** + alert on attempted personal-account sign-in (SOC ticket on repeated attempts)
- Sub-action: **block** custom CAAC session policy on the sanctioned variant — prevents in-session pivot to personal account via account-switcher
- Off-corp escalation: **MDE Network Protection** block of `chat.openai.com` etc. for any session NOT carrying the enterprise tenant claim header (Edge-on-Windows estates)

## Scope

- **Users:** all employees + contractors
- **Apps:** ChatGPT (Enterprise / Team vs personal Plus / Free), Microsoft Copilot (work vs consumer Copilot), Gemini (Workspace vs personal), Anthropic Claude (claude.ai Enterprise vs consumer), Perplexity Enterprise vs consumer
- **Device posture:** managed primarily; BYOD via MAM where supported
- **Network position:** corporate + roaming via endpoint agent / GSA; **off-network personal-device personal-account = uncovered (see Coverage gaps)**
- **Traffic:** sign-in flow (header inspection on auth flow); session continuation (CAAC); endpoint egress (MDE NP fallback)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Tenant-restriction headers (`Restrict-Access-To-Tenants`, `Restrict-Access-Context`) injected by SWG / Global Secure Access / third-party proxy. MDA itself does not inject — requires GSA or third-party SWG. Plus CAAC session policy on the sanctioned variant. Plus MDE Network Protection for off-corp endpoint enforcement | Header values = comma-separated allowed Entra tenant IDs (your tenant only for the inbound block); CAAC session policy onboards the sanctioned-vendor app; Entra CA "Use Conditional Access App Control" wired in; for non-Microsoft sanctioned variants (ChatGPT, Claude, Gemini) tenant pinning relies on the vendor's own SAML / header conventions | Tenant-restriction header support is per-vendor on the AI side — ChatGPT supports tenant pinning for Enterprise / Team via SAML SSO; consumer ChatGPT does NOT. Claude Enterprise (post-2025) supports SAML + SCIM. Gemini uses `X-GoogApps-Allowed-Domains`. Each must be configured at its own surface. The Microsoft-only `Restrict-Access-To-Tenants` header covers Microsoft auth endpoints, NOT third-party AI vendor auth endpoints | Personal account on personal device on personal network = entirely uncovered (architectural gap, accept and document). AI vendors without tenant-restriction-header support fall back to SWG URL filtering, which is coarser. Mobile native ChatGPT app uses its own auth flow — MAM compensating control required. Cert-pinned native apps may bypass SWG header injection (TLS pinning fails the proxy chain). Edge for Business `gen-ai.app-control` (2026 H2) may supersede part of this for Edge-on-Windows estates |
| Netskope | `[unverified]` — Netskope SWG with header injection + CASB integration; reportedly supports per-vendor tenant pinning via custom app policy + decryption + header rewrite | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access SWG with custom URL category + header insertion via decryption policy | | | |
| Skyhigh | `[unverified]` — Skyhigh multi-mode (reverse-proxy + forward-proxy) with custom app policy | | | |
| Zscaler | `[unverified]` — ZIA URL filtering + ZIA header injection (per Zscaler docs); per-tenant pin requires SSL inspection enabled on the AI vendor domains | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration for a tier-2 ASEAN BFSI tenant with ChatGPT Enterprise + Copilot + Claude Enterprise sanctioned, Gemini blocked:

```yaml
policy_set:
  name: "GenAI sanctioned-tenant pinning — tenant-wide"

  # Layer 1 — Microsoft endpoints (Entra-native)
  microsoft_endpoints:
    method: TenantRestrictionV2
    restrict_access_to_tenants:
      - "11111111-2222-3333-4444-555555555555"   # this tenant's Entra ID
    restrict_access_context: "11111111-2222-3333-4444-555555555555"
    header_injection_point: GlobalSecureAccess   # or third-party SWG
    block_message: "Sign in with your corporate Microsoft account."
    scope:
      include_users: AllEmployeesAndContractors
      exclude_groups:
        - "Break-Glass-Admins"                   # documented break-glass
        - "Microsoft-B2B-Approved-Partners"      # documented partner allowlist

  # Layer 2 — ChatGPT Enterprise (SAML SSO + per-OpenAI tenant binding)
  chatgpt:
    sanctioned_variant: ChatGPTEnterprise
    sso_provider: Entra
    saml_audience: "https://chat.openai.com/sso/<workspace-id>"
    enforce_login_via_sso: true
    block_personal_signin:
      method: SWGUrlFilter
      url_patterns:
        - "chat.openai.com/auth/login"           # consumer login flow
        - "platform.openai.com/login"            # API console personal flow
      bypass_if_header_present:
        header_name: "X-OpenAI-Org"
        header_value_regex: "^org-<enterprise-org-id>$"
    caac_session_policy:
      block_account_switch: true                 # prevent in-session pivot
      block_signout_relogin: true
    mde_network_protection:
      mode: Block
      domain_block: "chat.openai.com"
      domain_block_exception_if_session_carries: "X-OpenAI-Org=org-<enterprise-org-id>"

  # Layer 3 — Anthropic Claude Enterprise
  claude:
    sanctioned_variant: ClaudeEnterprise
    sso_provider: Entra
    saml_audience: "https://claude.ai/sso/<workspace-id>"
    enforce_login_via_sso: true
    block_personal_signin:
      method: SWGUrlFilter
      url_patterns:
        - "claude.ai/login"                      # consumer login flow
      bypass_if_session_in_workspace: "<workspace-id>"

  # Layer 4 — Google Gemini (blocked entirely — no Workspace tier sanctioned)
  gemini:
    sanctioned_variant: None
    action: Block
    method: SWGUrlBlock
    url_patterns:
      - "gemini.google.com"
      - "bard.google.com"
    block_message: "Gemini is not a sanctioned AI tool. Use ChatGPT Enterprise or Copilot."

  # Layer 5 — Mobile native apps (MAM compensating control)
  mobile_native:
    intune_app_protection_policies:
      - app: "OpenAI ChatGPT (iOS / Android)"
        managed_only: true                       # block on unmanaged
        copy_paste_restriction: PolicyManaged    # cannot paste from managed
        save_to_personal_cloud: Block
      - app: "Anthropic Claude (iOS / Android)"
        managed_only: true
        copy_paste_restriction: PolicyManaged

  # Monitoring
  alerts:
    personal_signin_attempt:
      severity: Medium
      destination: soc-l1@example.com
      repeat_threshold: 3                        # alert HR after 3 attempts/user/week
    daily_alert_limit: 200

  # Audit retention
  audit:
    sign_in_event_retention_days: 365            # workforce monitoring evidence
    cross_correlate_with_irm: true               # feeds Purview IRM
```

The `Restrict-Access-To-Tenants` value is your own Entra tenant ID — that is the deny-all-other-Microsoft-tenants posture. Adding additional tenant IDs to that list onboards B2B partners. The `bypass_if_header_present` pattern is the load-bearing trick — it lets enterprise sessions through while blocking consumer sessions on the SAME domain (e.g. `chat.openai.com`), which URL filtering alone cannot do.

## Variants

### Industry-specific

- **BFSI:** all four major sanctioned AI vendors pinned; PCI DSS / BNM RMiT mapping referenced as control evidence; mobile-native exception path requires explicit DPIA; partner-tenant allowlist heavily scrutinised because banking partners often have their own AI tenants. EU DORA-supervised firms additionally treat each sanctioned AI vendor as an in-scope ICT third party
- **Healthcare:** PHI exposure risk dominates — personal-account ChatGPT containing PHI is a HIPAA / regional health-data reportable event. Sanctioned-tenant pinning often paired with a hard block on `chat.openai.com` consumer variant entirely (no exception). Telemedicine / clinical-decision-support tools may complicate the sanctioned register (specialised medical-AI vendors with their own tenancy model)
- **Tech / SaaS:** developer-tool overlap — GitHub Copilot, Cursor, Codeium, etc. each have their own tenant model; sanctioned register can run to 8-12 vendors. Personal-account use is culturally entrenched; rollout requires more communication and product-team alignment than in BFSI
- **Retail:** lower volume of sensitive-data exposure but customer PII via loyalty / marketing analytics flows is a sleeper risk. Often a lighter posture — sanctioned ChatGPT Enterprise only, Gemini / Claude blocked, Copilot if M365 estate
- **Legal:** privileged-content exfil risk + ethical-walls posture — many firms maintain matter-specific AI tenants (a separate Claude Enterprise workspace per major client matter) so the tenant-pinning policy must support multi-tenant per-user authorisation; the partner-tenant allowlist becomes a dynamic, matter-scoped object
- **Public sector:** sovereign-AI residency overlay — sanctioned AI vendor must have in-jurisdiction data residency; consumer variants often blocked entirely regardless of corporate-managed-device status. The personal-device personal-account gap is closed by an absolute device policy: no personal device used for work, period

### Maturity-based

- **Immature deployment** looks like: ChatGPT Enterprise tenant procured; users told "please use the enterprise version"; no technical enforcement; SWG has `chat.openai.com` allowed; CAAC not configured; ~30-50% of GenAI usage routes through personal accounts and the firm cannot tell which. Audit evidence = procurement contract only. Common at month 3-6 post-procurement for under-resourced programmes.
- **Mature deployment** looks like: tenant-restriction headers injected at SWG / GSA for Microsoft endpoints; SAML SSO enforced for ChatGPT Enterprise + Claude Enterprise; CAAC session policy live; MDE NP block as fallback; mobile MAM on the sanctioned-vendor native apps; documented exception path for cross-tenant collaboration; quarterly review of the sanctioned-AI register and the partner-tenant allowlist; first-line audit evidence pull tested. FP rate stable, weekly personal-attempt alerts <50/1k users.
- **Advanced deployment** looks like: every sanctioned AI vendor in the ICT third-party register with full vendor-risk attestation cycle (SOC 2, ISO 27018, EU AI Act provider obligations where applicable, DORA contract clauses); DSPM-for-AI prompt capture on the sanctioned tenants feeds Purview IRM as a signal; joiner-mover-leaver flow revokes enterprise AI access in <24h; tenant-pinning posture per-jurisdiction (different sanctioned-AI register per region); board cyber report carries "% of GenAI sessions on enterprise tenant" as a steady-state KPI; gap analysis against EU AI Act deployer obligations + ASEAN equivalents documented.

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + DLP + AI governance overlay](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]`
- ISO 27017 control(s): [CLD.9.x access control overlay](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation) — personal-account variant routes PII outside the contracted CSP
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions enforcement), `GV.SC-04` (suppliers known), `GV.SC-06` (planning + due diligence on suppliers), `PR.DS-02` (data-in-transit) `[VERIFY]`
- NIST AI RMF 1.0: `GOVERN 3.2` (sanctioned-AI register), `MAP 4.1` (third-party AI dependencies catalogued)
- EU AI Act: deployer obligations Art. 4 / Art. 26 (literacy + accountable use) — tenant pinning evidences the accountable-use control where Personal-account use would constitute unaccountable use `[VERIFY against final text]`
- EU DORA: Art. 28-30 (ICT third-party risk) — each sanctioned AI vendor in scope

## False-positive risk

- Legitimate cross-tenant collaboration with partner AI workspaces — name an exception group and document the partner-tenant allowlist
- Users with both personal and business Microsoft accounts on the same browser session — confusion at sign-in; user assumes the work account is active, ends up on personal; block message must be clear about which account
- Joint ventures or consulting engagements where the contracting entity uses a different sanctioned-AI tenant — case-by-case allowlist
- Users authenticating to a third-party SaaS that embeds Copilot iframes — the embedded sign-in may carry a different tenant claim than expected
- Migration windows when moving from one sanctioned-AI vendor to another — temporary dual-allow window must be time-boxed

## Real-world FP experience

Typical FP-rate trajectory for tenant pinning in a tier-2 BFSI tenant new to enterprise GenAI:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 30-50% | Users with legitimate dual-tenant relationships (joint ventures, consulting); new-hire onboarding with not-yet-provisioned enterprise account; users routing through legacy personal accounts they used to test the tool pre-procurement |
| W4 | 15-25% | After partner-tenant allowlist established; after onboarding flow updated to provision enterprise AI access on day one |
| W8 | 8-15% | After common exception patterns documented (consulting engagements, training-vendor demos); after user-comms on which account to use |
| W12 | 5-10% | Mature allowlist; documented exception process; user behaviour shifted to enterprise-default |
| Steady-state | 3-8% | Quarterly review of partner-tenant allowlist; FP residual mostly cross-tenant collaboration legitimately changing |

Named FP scenarios encountered repeatedly in production deployments:

| Scenario | Mitigation |
|---|---|
| User has personal Gmail signed in on Chrome alongside corporate; account-switcher selects personal at sign-in; block fires | User-comms: keep work browser profile separate; block-page message explicitly names "you signed in with personal Gmail" rather than generic "access denied" |
| Joint-venture partner shares a Claude workspace with the firm; partner-tenant blocked initially | Partner-tenant allowlist; documented owner, expiry date, scope (which users / which JV) |
| Consultant on engagement at the firm has their own ChatGPT Enterprise tenant for their consulting practice; needs to sign in from corporate device | Named individual allowlist OR consulting firm AUP explicitly states "use your own devices for your own AI tenancy" |
| New hire signed up for personal ChatGPT during onboarding training; later tries to use corporate variant from same browser; cookies / cached creds collide | Onboarding instruction: sign out of all personal AI accounts before first enterprise sign-in; help desk runbook for cookie-clear |
| Marketing team using a personal Gemini demo account during vendor evaluation pre-procurement; blocked unexpectedly | Pre-procurement: documented temporary allowlist for evaluation tenant; time-boxed |
| Training-vendor session using their own AI sandbox tenant for a live demo | Training-vendor allowlist entry; revoked after training cycle |
| Engineer on a personal device personal-network during off-hours hits the corporate-network block from a misconfigured VPN routing | Diagnose the VPN routing; not actually a policy FP — the user is on the corporate posture inadvertently |

The FP profile is materially lower than DLP-class policies (auto-label PCI runs W1 60-80%) because tenant pinning is identity-based, not content-based — it either is or is not the sanctioned tenant claim. The FP is overwhelmingly about *human* misconfiguration (which account is active in the browser), not classifier noise.

## Operational cost

- **Exception-handling load:** low ongoing — sanctioned-tenant list is small (4-6 vendors typical) and stable; partner-tenant allowlist sees ~2-5 changes per quarter in mature deployments
- **Triage load:** low — block is silent at IdP / SWG layer; alerts only on repeated-attempt patterns or SOC pivot from another signal
- **End-user friction:** medium during rollout — users who routinely sign into personal AI accounts at work see blocks; communications + clear block-page messaging mitigate. After 2-3 months of operating, friction drops sharply as users internalise the "use the enterprise one" norm
- **Vendor management cost:** **the under-counted item** — every sanctioned AI vendor pinned is now a managed vendor relationship with ongoing attestation, contract review, DPA refresh. At 4-6 sanctioned vendors, this is 0.3-0.5 FTE permanent in vendor management

Typical staffing: 0.2 FTE platform admin during 10-week rollout; 0.1 FTE steady-state; 0.3-0.5 FTE vendor management (ongoing); 0.1 FTE compliance analyst for audit-evidence + DPIA-refresh cycles.

## Privacy / data-protection considerations

- Block does not inspect personal traffic content; inspects only sign-in tenant claim / SAML audience / header value. Lower privacy footprint than SSL inspection of content
- However: the **sign-in event log itself is workforce monitoring data** — who tried to sign into personal ChatGPT, when, how often. Under PDPA MY 2024 / GDPR Art. 88 / equivalent, this is workforce-data that requires documented purpose, retention, access governance
- DPIA scope: covers the sign-in monitoring posture + repeat-attempt-to-HR escalation path. Repeated personal-account attempts triggering HR review is high-impact under GDPR — must be documented in the AUP / employee handbook upfront
- The MDE NP block events at endpoint level are also workforce-data — same DPIA scope
- Cross-border transfer: when enterprise AI vendor's data residency is different from workforce-data residency, the audit-evidence pull crosses borders — DPA / SCC review required
- Mobile MAM telemetry adds another workforce-monitoring layer — block-event records on personal mobile under MAM are sensitive; document the workforce notice carefully
- For EU AI Act: sign-in monitoring posture must be documented under the deployer transparency obligations (illustrative — verify against final text)

This material is illustrative — not legal / regulatory advice.

## Integration with broader programmes

- **Audit cycle:** sanctioned-AI register + tenant-pinning enforcement evidence + sign-in event logs feed annual BNM RMiT / MAS TRM / HKMA cyber attestation; PCI DSS Req. 7 + Req. 12.10 evidence; ISO 27017 CLD.9 evidence; quarterly cadence for register review, annual for full audit pull
- **Board reporting:** quarterly KPI — "% of GenAI sessions on sanctioned-enterprise tenant" + "personal-account block events trend"; named-incident escalations (HR referrals) summarised as control-effectiveness signal
- **Incident response runbook:** repeated personal-account attempts feed Insider Risk programme — three attempts in a week to an SOC ticket, five to HR; integrated with Purview IRM as a boosted indicator; correlation with mass-download (MDA Policy 3) + DSPM-for-AI prompt-capture signals — a user paste-of-CHD into enterprise ChatGPT followed by a personal-account attempt is a different threat class than either alone
- **DPIA refresh:** annual DPIA cycle covers the sign-in monitoring scope, the repeat-attempt-to-HR escalation, the workforce-notice posture; cross-border transfer review per sanctioned AI vendor
- **Vendor-risk programme:** every sanctioned AI vendor is an in-scope ICT third party — SOC 2 attestation pull, ISO 27018, DPA review, EU AI Act provider documentation (where applicable), DORA contract clauses (where applicable); annual cycle; concentration-risk register entry
- **Workforce AUP:** explicitly states that personal-account use of AI tools at work is prohibited; explicitly references the technical control; user-onboarding pack includes the sanctioned-AI register + which account to use for what
- **Joiner-mover-leaver:** leavers' enterprise AI access revoked within 24h per the SCIM / manual process; movers' access updated when changing BU (some BUs may have BU-specific sanctioned tenants)

## Anti-patterns specific to this policy

1. **"Procure ChatGPT Enterprise and tell users to use it"** — without technical enforcement, ~30-50% of GenAI usage routes through personal accounts anyway. Pure policy-without-control. The enterprise contract becomes audit theatre
2. **"Block the consumer domain at SWG and call it done"** — blocks the enterprise variant too (both live on `chat.openai.com`); user backlash, rollback, programme credibility damaged. Must use header-based bypass
3. **"Pin Microsoft endpoints only because Restrict-Access-To-Tenants is easy"** — covers ~20-30% of the AI exfil surface; ChatGPT, Claude, Gemini remain personal-account-accessible; programme has the appearance of coverage with material gaps
4. **"Skip the bilateral CAAC trap"** — Entra CA "Use Conditional Access App Control" without MDA session policy = no-op redirect; MDA session policy without Entra CA = never fires. Both silent failures. Verify both halves with named test users
5. **"Don't bother with the mobile-native gap"** — mobile native ChatGPT / Claude apps have own auth flows that bypass SWG header injection; MAM compensating control is essential for managed mobile; leaving the gap means tenant pinning works only on browser sessions
6. **"Treat the partner-tenant allowlist as a one-time exercise"** — partners come and go; allowlist drifts; stale entries become attack surface (T1199 Trusted Relationship). Quarterly review cadence required
7. **"Forget the consumer-tier still has the data after enterprise rollout"** — past personal-account use accumulated prompts / outputs in personal ChatGPT history; pinning forward doesn't remediate the historical data. Workforce comms should ask users to delete their personal-account history; audit may want evidence of the request being made
8. **"Use the same allowed-tenant list across all sanctioned AI vendors"** — each vendor's auth surface is independent; ChatGPT partner-tenant allowlist is different from Claude is different from Copilot is different from Gemini. Document per-vendor
9. **"Notify the user via personal email when blocked"** — defeats the privacy posture; some firms misconfigure the block-page contact info. Block message should reference the corporate help desk and AUP only
10. **"Allow the break-glass-admin account to bypass tenant pinning permanently"** — break-glass = emergency use only; permanent bypass on a privileged account is itself the attack surface (Storm-0501-class persistence). Time-box break-glass; alert on use; rotate quarterly
11. **"Wire repeat-attempt alerts to HR before workforce notice is updated"** — three attempts goes to HR ticket; workforce hasn't been told this monitoring exists; GDPR / PDPA complaint. Workforce notice + AUP must precede the HR escalation path going live
12. **"Tenant-pin ChatGPT and assume Claude / Gemini will follow vendor parity"** — vendor SAML / SCIM capabilities are not uniform; Claude added SAML in 2025, Gemini uses different headers, each pin is independent work

## Coverage gaps

- Off-network personal-device personal-account = entirely uncovered (architectural — no corporate visibility into the session). Compensating: workforce AUP + culture; not a technical control
- AI vendors without tenant-restriction-header support and without SAML — only SWG URL filtering available (coarser; cannot distinguish enterprise vs consumer on shared domains)
- Mobile native AI apps with own auth flows — MAM compensating; managed-mobile-only posture; BYOD-mobile remains a gap
- Cert-pinned native desktop apps (ChatGPT for Mac, Claude desktop) — TLS pinning fails the SWG decryption; header injection silently bypassed. Endpoint AV / EDR-policy block on the binary is the compensating control
- New AI vendors emerging faster than the sanctioned register can keep pace (a new entrant goes from launch to 1k corporate users in weeks) — Shadow IT discovery (MDA Policy 1) feeds the sanction-or-block decision but there is a 2-4 week response lag
- CAE-induced session re-establishment may bypass session-policy re-evaluation — see [`../../08-failure-modes/cae-bypass.md`](../../08-failure-modes/cae-bypass.md) `[VERIFY path]`
- Personal-account variant of vendor with no enterprise tier — some specialised AI tools have no enterprise SKU; block-or-accept decision is binary
- See also [`../../08-failure-modes/api-mode-is-not-prevention.md`](../../08-failure-modes/api-mode-is-not-prevention.md) — session-policy variant where the sanctioned-tenant pin is API-detection only, not prevention

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
