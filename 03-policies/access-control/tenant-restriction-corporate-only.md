# Tenant restriction — corporate-only

> Status: v0.0 — partial (cross-vendor pattern; MDA implementation via CAAC + Entra tenant-restriction-v2 + SWG header injection); other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive (Microsoft-standards QA applied 2026-06-10)**.
> Required capabilities: [Risk-based session policy + Identity integration + SWG header injection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (IdP-mediated) + browser headers via SWG (Global Secure Access / Zscaler / Netskope / Prisma Access).
> Internal playbook reference (not Microsoft documentation): implicit (CAAC + Entra Cross-Tenant Access Settings + tenant-restriction-v2 headers injected at SWG); not numbered separately in the internal MDA Day 1/30/90 playbook — sits across Day 0 (Entra CTAS) + Day 30 (CAAC scoping) + Day 90 (Policy 9 ChatGPT clipboard, the corollary control). The "MDA Policy N" numbering here is internal playbook nomenclature, not Microsoft documentation.

## Microsoft architectural attribution (Tenant Restrictions v2)

Tenant Restrictions v2 (TRv2) is an **Entra + Global Secure Access (GSA) control** under the Identity Zero Trust pillar (Initial Deployment Objective IV — "Restrict user consent / Restrict access to applications"). The policy ownership and enforcement live in Entra ID's Cross-Tenant Access Settings (CTAS) and in the SWG / GSA layer that injects the `Restrict-Access-To-Tenants` / `Restrict-Access-Context` headers into outbound traffic.

**Microsoft Defender for Cloud Apps (MDA) is not the architectural owner of this control.** MDA's role is limited to the in-session Conditional Access App Control (CAAC) supplement for the subset of cloud apps where in-session restriction adds value beyond the sign-in-layer block (e.g. ChatGPT Enterprise paste-block once a user is legitimately signed in to a corporate-allowlisted tenant). Practitioners must keep this attribution straight in audit-evidence packages — confusing MDA-as-signal-source with MDA-as-policy-owner is a routine finding in internal audit.

Citations:

- Entra Tenant Restrictions v2 reference: [Microsoft Learn: https://learn.microsoft.com/en-us/entra/external-id/tenant-restrictions-v2]
- Identity Zero Trust deployment objective IV: [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity]
- MDA Conditional Access App Control intro (for the in-session CAAC supplement only): [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad]

## Purpose

Block sign-in to M365 / Google Workspace / Salesforce / ChatGPT Enterprise / other federated SaaS using personal accounts or non-corporate tenants from corporate networks and managed devices. Stops the dominant 2024-2026 data-leak pattern of "user signs into ChatGPT (or any SaaS) with personal Gmail on a corporate device, then exfiltrates corporate data to a personal account by pasting / uploading content the corporate DLP path never sees". Counters MITRE ATT&CK `T1078.004 Cloud Accounts` (personal-account variant) and the personal-tenant variant of `T1567 Exfiltration Over Web Service`. Also closes the trusted-relationship-abuse surface (`T1199`) exploited by Storm-2372 / Storm-0539 consent-phishing chains where the attacker uses a partner or attacker-controlled tenant as the inbound vector. Threat-actor names follow Microsoft canonical taxonomy [Microsoft Learn: https://learn.microsoft.com/en-us/unified-secops/microsoft-threat-actor-naming].

## What organisations use this for

Tenant restriction is rarely the *first* control a CASB programme deploys. It surfaces on the roadmap after a named event — almost always a post-incident review where the forensic timeline traces data out via a personal account on a corporate device, and the DLP team realises the corporate session was never the egress channel. Once that pattern is named, tenant restriction moves up the priority list because it closes the channel architecturally rather than chasing detection signatures.

The dominant 2024-2026 trigger is the personal-ChatGPT exfiltration pattern: an employee copies regulated content from a sanctioned source, opens a personal-Gmail-authenticated tab to consumer ChatGPT, pastes the content, and the corporate DLP path sees nothing because the OAuth session never touches the IdP. Tenant restriction breaks that flow at the sign-in layer — the personal-Gmail authentication itself is refused when the egress traffic carries the corporate `Restrict-Access-To-Tenants` header. The control is unusual in that it lives in three places at once: Entra Cross-Tenant Access Settings (CTAS) define the policy, the SWG injects the headers into outbound traffic, and CAAC supplements with in-session restriction for users who slip the network-layer net via Wi-Fi tethering or off-corp roaming. None of the three alone is sufficient.

### Use case 1 — Tier-1 ASEAN bank post-personal-ChatGPT-incident response

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, ChatGPT Enterprise rolled out tenant-wide as the approved AI tool
- **Trigger:** post-incident review of a near-miss in Q4 2025 — internal-audit analyst pasted a draft Audit Committee memo (containing customer-segment loss provisions) into consumer ChatGPT on a corporate laptop using a personal Gmail account; the paste was caught only because the analyst self-reported. CASB clipboard policy was scoped to ChatGPT Enterprise only and never fired against the consumer surface. The board asked the CISO "why didn't anything block this", and tenant restriction was the architectural answer
- **Scope:** all employees + managed contractors; corporate network + GSA-tunnelled roaming; consumer Google + consumer Microsoft + ChatGPT personal-plan sign-ins blocked; ChatGPT Enterprise tenant ID in the allowed list; phased rollout starting with the audit + risk + compliance functions where the incident originated
- **Outcome:** ~600 blocked personal-account sign-in attempts per week steady-state across the workforce (most legitimate-but-policy-violating, e.g. employees doing personal Gmail at lunch). Six weeks in, one confirmed exfiltration attempt blocked at the header-injection layer (employee under disciplinary review for unrelated reasons attempted to upload deal-pipeline material to personal Google Drive). Friction-driven exception requests dropped from ~40/week in W2 to ~5/week by W8 once personal-traffic-on-personal-device guidance landed. **[VERIFY:** BNM RMiT clause on third-party identity provider acceptance — likely 10.x access-management section, confirm against current edition**]** (regulatory clauses are illustrative only — not regulatory advice)

### Use case 2 — Merchant acquirer hardening Cross-Tenant Access after Storm-2372 OAuth campaign

- **Org type:** payments processor operating across MY / SG / HK / TH, ~4k employees, in-scope for PCI DSS + BNM RMiT + MAS TRM, M365 E5, multiple B2B partner tenants (issuing banks, scheme operators, settlement bureaus)
- **Trigger:** internal threat-intel team identified two failed consent-phishing attempts originating from a tenant that mimicked a known partner's display name (Storm-2372-pattern device-code phishing against the cards-operations team). Cross-Tenant Access Settings were in default permissive state ("trust all external Entra tenants"). Audit Committee asked for partner-tenant allowlisting; CTAS hardening pulled tenant restriction into scope as the broader control
- **Scope:** all employees; partner allowlist = 47 named partner tenants (each with documented business owner, scheme-operator contract reference, annual review date); consumer accounts entirely blocked; CAAC supplements for the subset of partner sessions where in-tenant download of cardholder-data-class material occurs
- **Outcome:** documented partner-tenant inventory replaced the implicit "trust all" posture as control evidence for PCI DSS Req. 7 + Req. 12.8 (third-party access). Six partner tenants identified as orphaned (no live business relationship) and removed. Auditor accepted the CTAS configuration export as primary access-control evidence over screenshots. Quarterly partner-review cadence became a standing item on the third-party-risk forum. **[VERIFY:** PCI DSS v4.0 Req. 12.8 third-party access maps — verify current sub-requirement numbering**]** (regulatory clauses are illustrative only — not regulatory advice)

### Use case 3 — Digital-native challenger bank rationalising B2B-partner workspace overlap

- **Org type:** neobank, ~600 employees, mixed Google Workspace + M365 estate (engineering on Workspace, finance + compliance on M365), high B2B-partner volume because of card-issuer + KYC-provider + open-banking-aggregator integrations
- **Trigger:** internal IAM review surfaced that ~30% of staff had personal Workspace accounts + B2B-guest accounts to partner workspaces that overlapped semantically with corporate accounts (e.g. an engineer with personal `firstname.lastname@gmail.com`, corporate `firstname@<bank>.com`, and a B2B guest in a vendor's Workspace as `firstname.lastname@<vendor>.com`). The overlap meant DLP signals could not reliably attribute activity to the corporate identity. Cross-Tenant Access Settings inconsistent between the Workspace and M365 sides
- **Scope:** Workspace tenant restriction headers via the SWG for Google sign-in; Entra CTAS hardening for M365 partner allowlist; explicit B2B-guest allowlist of ~15 named partner tenants on each side; consumer Google + consumer Microsoft sign-ins blocked tenant-wide
- **Outcome:** identity-attribution reliability for DLP signals improved measurably (FP rate on the mass-download policy dropped from ~22% to ~12% once misattributed B2B-guest events were filtered out — practitioner observation; not Microsoft-documented). One unanticipated outcome: three legitimate workflows broke at rollout because external auditors had been signing in via consumer-Google accounts to view shared statements — the auditor relationship had to be re-papered through proper B2B-guest provisioning. **[VERIFY:** the FP-rate delta is a specific-deployment observation, not a general claim**]**

### Use case 4 — Regional consulting firm responding to client contractual access-control requirements

- **Org type:** professional-services firm, ~8k employees, M365 E5, ChatGPT Enterprise tenant-wide; serves regulated FS clients across ASEAN
- **Trigger:** three Tier-1 BFSI clients added clauses to their MSAs in 2025 requiring the consulting firm to attest that staff working on the client account cannot sign into personal-account SaaS from devices used for client work. The clause appeared in the MAS TRM-aligned and BNM RMiT-aligned MSA templates simultaneously. CISO asked Architecture for the cheapest path to the attestation; tenant restriction emerged as the answer
- **Scope:** all employees on client-account engagements; phased by practice area (FS practice first); allowlist = consulting firm tenant + named partner tenants (KPMG audit references, tax-bureau filing portals) + ChatGPT Enterprise; consumer accounts blocked
- **Outcome:** documented control evidence package for client-attestation cycles. Three client procurement teams accepted the evidence pack as sufficient; one (a Singapore bank) requested supplemental quarterly evidence pulls. Operational friction surfaced around personal device-use during travel (consultants overseas were tunnelling through hotel Wi-Fi to corporate SWG, then trying to do personal email — silent block fired and frustrated users until the on-corp / off-corp scoping was documented). **[VERIFY:** MAS TRM clause on access control and third-party staff — verify against current TRM Guidelines edition**]** (regulatory clauses are illustrative only — not regulatory advice)

## Implementation pattern

Typical 10-week rollout for a tier-2 BFSI tenant with an existing SWG (GSA / Zscaler / Netskope / Prisma Access) and Entra-as-IdP:

| Week | Activity | Output / gate |
|---|---|---|
| W1 | Discovery: pull 90-day SWG logs of consumer-account sign-in destinations (Google, Microsoft consumer, ChatGPT personal, Slack consumer, LinkedIn consumer); identify high-volume legitimate-personal use; document the "Wi-Fi tethering / off-corp" coverage edge | Personal-traffic baseline; rollout-comms draft |
| W2 | Stakeholder alignment: Legal + HR + IR review the workforce-notice posture; PDPA / GDPR DPIA scope decision; AUP refresh language drafted | Workforce notice signed off; DPIA pre-assessment recorded |
| W3 | Entra Cross-Tenant Access Settings: document the existing partner-tenant inventory (`Get-MgPolicyCrossTenantAccessPolicyPartner` or portal export); identify orphaned partner tenants; circulate to business owners for confirmation | Documented CTAS partner list with business owners |
| W4 | Pilot scope: 1 BU (~50-200 users); tenant-restriction-v2 headers injected by SWG for that user group only; allowlist = corporate tenant + ChatGPT Enterprise + named B2B partners | Pilot policy live in monitor-mode (header injection logs only, no block) |
| W5 | Monitor-mode review: classify attempted-blocked sign-ins (legitimate-personal-but-policy-violating vs B2B-partner-not-on-allowlist vs unknown); add partners to allowlist where business case stands | Exception-handling runbook v1; allowlist updated |
| W6 | Flip pilot BU to block-mode; SOC alert routing for block events configured; user-facing block message customised with self-service exception-request link | Pilot in block-mode; first-week ticket volume measured |
| W7-W8 | Expand to next 2-3 BUs; refine per-BU partner allowlists; document the off-corp / roaming coverage path via GSA tunnel (or third-party SWG roaming agent) | Expanded scope; coverage gap (personal-device-personal-network) explicitly documented as residual risk |
| W9 | Tenant-wide rollout; CAAC supplements deployed for the subset of high-risk apps where in-session controls add value (ChatGPT Enterprise paste-block, M365 download-block) | Tenant-wide block-mode live |
| W10 | Documentation: control-evidence package for audit cycle; quarterly review cadence on partner allowlist + exception list; integration into IRM signal feed | Audit-evidence pack v1; quarterly review scheduled |
| W11+ | Steady-state: quarterly partner allowlist review; monthly exception-list pruning; correlation of block events with insider-risk signals | Quarterly metric on block events + exceptions |

The pilot phase is the work. Rolling tenant-wide on day one without the W4-W5 monitor-mode pass guarantees the help desk is overwhelmed by partner-allowlist exceptions on day one of block-mode.

## Action

- Primary: **block** sign-in from non-allowlisted tenant at the SWG header-injection layer + Entra CTAS layer (Entra + GSA owned per Identity ZT Objective IV)
- Secondary: **monitor + alert** on attempted personal-tenant sign-in from corporate device (feeds insider-risk signal and named-individual pattern detection)
- Tertiary: **CAAC supplement** — in-session restriction on the subset of apps where tenant restriction alone is insufficient (ChatGPT Enterprise where users have legitimate access but may attempt cross-tenant data movement). This is MDA's narrow role in this control class — supplement, not owner

## Scope

- **Users:** all employees + managed contractors. Service accounts excluded by UPN regex (they sign in via client-credentials, not user flows, but excluding explicitly avoids ambiguity)
- **Apps:** M365, Google Workspace, ChatGPT Enterprise, Slack, Salesforce, GitHub Enterprise, Atlassian Cloud, Anthropic Claude (where SAML-federated), any SaaS supporting tenant-restriction-v2 headers or equivalent
- **Device posture:** managed devices primarily (Intune-compliant or Entra-hybrid-joined); BYOD-mobile via Intune MAM (App Protection Policies) where supported; unmanaged BYOD on personal network = out of scope (documented residual risk)
- **Network position:** corporate network + roaming via GSA / SWG-roaming-agent. Personal device on personal network = entirely uncovered (architectural)
- **Traffic:** sign-in flow + OAuth consent flow

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| Microsoft stack (Entra + GSA; MDA in supplement role only) | Entra portal → External identities → Cross-tenant access settings (CTAS) + Conditional Access policy + SWG injection of `Restrict-Access-To-Tenants` and `Restrict-Access-Context` headers via GSA or third-party. **The policy owner is Entra (Identity ZT Objective IV); the header-injection layer is the SWG / GSA. MDA does not own or enforce this control** — MDA's role is the in-session CAAC supplement only [Microsoft Learn: https://learn.microsoft.com/en-us/entra/external-id/tenant-restrictions-v2; https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad] | CTAS default = block all external; allowlist named partner tenants only. Headers = tenant-restriction-v2 (`Restrict-Access-To-Tenants: tenant-id-1,tenant-id-2,...` + `Restrict-Access-Context: <home-tenant-id>`). CA policy = block sign-in for users on managed-device when home tenant is not in allowlist | Tenant-restriction-v2 requires the SWG to inject headers in user outbound traffic — pure-MDA does not do this. GSA does it natively; third-party SWGs need v2 support specifically. v1 (`Restrict-Access-To-Tenants` original form) was app-class-only and is being deprecated in favour of v2 [Microsoft Learn: https://learn.microsoft.com/en-us/entra/external-id/tenant-restrictions-v2 — see "What's the difference between v1 and v2" + v1 deprecation guidance; **[VERIFY: precise v1 deprecation date against the Microsoft Learn page on the day of attestation]**]. Personal account on personal device on personal network = entirely uncovered architecturally | tenant-restriction-v2 supports per-tenant context — v1 was app-class-only. Verify which version your SWG injects. **Silent fail on cert-pinned native mobile apps that bypass the SWG** (e.g. Outlook Mobile, Teams Mobile, Gmail iOS — header injection requires the traffic to flow through the proxy). **CAE-induced session re-establishment may not re-evaluate the header constraint** — pair with Entra Token Protection (verify GA status) |
| Netskope | `[unverified]` — Netskope NewEdge SWG supports tenant-restriction-v2 header injection per Netskope docs; CASB tagging supports complementary in-session policy | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access SWG supports header injection; SaaS Security adds in-session controls | | | |
| Skyhigh | `[unverified]` — Skyhigh SWG header injection + UCE policy | | | |
| Zscaler | `[unverified]` — ZIA URL filtering + tenant-restriction header injection (verify v2 support against current ZIA release) | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration for a tier-2 ASEAN BFSI tenant after the W4-W8 pilot tuning is complete. Property-list format; not vendor literal syntax:

```yaml
policy:
  name: "Tenant restriction — corporate-only, tenant-wide"
  type: TenantRestrictionV2
  enforcement_layers:
    - layer: SWG_HeaderInjection
      vendor: GlobalSecureAccess        # or Netskope / Zscaler / Prisma / Skyhigh
      headers_injected:
        Restrict-Access-To-Tenants:
          - "11111111-aaaa-bbbb-cccc-222222222222"  # corporate Entra tenant
          - "33333333-dddd-eeee-ffff-444444444444"  # ChatGPT Enterprise tenant
          - "55555555-1111-2222-3333-666666666666"  # named partner — scheme operator
          - "77777777-7777-8888-9999-aaaaaaaaaaaa"  # named partner — KYC provider
          # ... 43 additional named partner tenants
        Restrict-Access-Context: "11111111-aaaa-bbbb-cccc-222222222222"
      injection_scope:
        - on_corporate_network: true
        - via_gsa_tunnel: true
        - roaming_agent_active: true
      exclude_traffic:
        - destination_domains:
            - "*.bank-personal-banking.example.com"   # legitimate personal-banking portal
            - "*.gov.my"                              # tax / regulator portals
            - "*.hr-self-service-vendor.example.com"  # employee SSO carve-out
    - layer: Entra_CrossTenantAccessSettings
      default_inbound: Block
      default_outbound: Block
      partner_allowlist:
        - tenant_id: "55555555-1111-2222-3333-666666666666"
          partner_name: "Scheme Operator A"
          business_owner: "head-of-cards-ops@example.com"
          scoped_users: B2B_DirectFederation
          allowed_apps: ["Office 365"]
          review_cadence_months: 6
          contract_reference: "MSA-2024-CARDS-001"
        # ... per-partner detail rows
    - layer: ConditionalAccess
      ca_policy_name: "CA-Block-non-corporate-tenant-on-managed-device"
      assignments:
        users_and_groups:
          include: All
          exclude: ["BreakGlass-Admin", "Service-Accounts"]
        cloud_apps:
          include: All
        conditions:
          client_app: Browser + ModernAuth
          device_platforms: ["Windows", "macOS"]
          device_state:
            include: ["IntuneCompliant", "MicrosoftEntraHybridJoined"]
      access_controls:
        grant: Block
        when: HomeTenant_Not_In_Allowlist
  alerting:
    monitor_mode_first_weeks: 2
    soc_alert_routing:
      destination: "soc-l1@example.com"
      severity: Medium
      sentinel_forward: true
    daily_alert_limit: 200
  user_experience:
    block_message: "Sign-in to non-corporate accounts is restricted on this device. For business exceptions raise a ticket via <self-service-link>. For personal use, switch to your personal device on a personal network."
    self_service_exception_link: "https://servicedesk.example.com/tenant-restriction-exception"
  governance:
    quarterly_partner_review_owner: "third-party-risk@example.com"
    monthly_exception_pruning_owner: "iam-platform@example.com"
    audit_evidence_pull_cadence: Quarterly
```

The 47-partner allowlist size is typical of a mid-size BFSI tenant after the W3 partner-inventory exercise (**practitioner observation; not Microsoft-documented**). Sub-20 allowlists are unrealistic and usually indicate orphaned partners were not surfaced; >100 usually indicates the inventory included one-off project tenants that should have been time-bounded.

## Variants

### Industry-specific

- **BFSI:** the partner allowlist is dominated by scheme operators, settlement bureaus, KYC providers, and shared utilities (CCRIS / CTOS equivalents). PCI DSS Req. 12.8 third-party access framing supplies the audit-evidence story. The dominant FP fight is the cards-operations workflow that legitimately spans 6-12 partner tenants — needs careful per-BU allowlist scoping rather than a single tenant-wide list.
- **Healthcare:** partner allowlist dominated by clinical-system integrators, lab partners, claims processors. HIPAA Business Associate Agreement (BAA) maps to the allowlist row (each partner has a BAA reference). The dominant FP fight is the patient-portal vendor sign-in that uses consumer-Google-as-IdP — almost always requires an exception.
- **Tech:** smaller partner allowlist; the dominant variant is **engineering use of personal GitHub** for OSS contributions — needs a documented carve-out (typically GitHub.com personal-account sign-in allowed only via a specific contributor-allowlist UPN group, not tenant-wide). The personal-ChatGPT exfil risk is acute because engineers paste code into AI tools by reflex.
- **Retail:** smaller partner allowlist; the dominant variant is supplier and logistics-partner tenants. Consumer-account use among store-level staff is much higher than in BFSI — needs heavier pre-rollout comms and longer monitor-mode phase. POS-terminal accounts often excluded entirely.
- **Public sector / Legal:** partner allowlist dominated by court systems, regulator portals, opposing-counsel-firm tenants (legal). Often paired with a stricter posture — block all consumer accounts categorically, no exceptions on managed devices.

### Maturity-based

- **Immature:** Entra CTAS in default "trust all external tenants"; no SWG header injection; no partner inventory; allowlist not documented; personal-account sign-in on corporate device entirely possible. Common at 6-12 months post-CASB-deployment in programmes that prioritised DLP detection over identity-layer hardening. Audit finding waiting to happen.
- **Mature:** CTAS hardened with documented partner allowlist; SWG header injection live for corporate-network users; monitor-mode pilot completed; CA policy blocking non-allowlisted home tenants on managed devices; quarterly partner-review cadence operating; SOC alerting on block events. The control is real; exceptions are managed; audit-evidence package is reusable.
- **Advanced:** per-BU partner allowlists (cards-ops sees scheme operators, treasury sees liquidity-pool partners, etc.); GSA roaming agent extends coverage off-corp; CAAC supplements layered on for high-risk apps; integration with Purview Insider Risk Management — block events boost IRM risk score; correlation with HR pre-departure watchlist; per-partner-tenant token-binding (Entra Token Protection) where supported; automated partner-allowlist drift detection (any orphan tenant unused for 90 days flagged for removal).

## Control mappings

- **CIS Microsoft 365 Foundations Benchmark v5.0.0** (30 April 2025):
  - **CIS 5.1.6.2 (L1)** — Ensure that 'Guest invite restrictions' is set to "Only users assigned to specific admin roles can invite guest users"
  - **CIS 5.1.6.1 (L2)** — Ensure that collaboration invitations are sent only to allowed domains
  - **CIS 5.2.2.9 (L1)** — Ensure 'sign-in frequency' is enabled and browser sessions are not persistent for administrative users (Conditional Access framing applies to the CA layer of this control)
  - **CIS 5.2.2.12 (L1)** — Ensure 'Restrict access to Microsoft Entra admin center' is enabled (companion control limiting personal-tenant access into Entra admin)
  - Direct cite of the CIS control that addresses the Storm-2372 device-code-phishing pattern named extensively in this policy
- **Microsoft Secure Score** — Identity group + Apps group improvement actions; specifically "Enable Conditional Access policies" and the Cross-Tenant Access Settings hardening actions [Microsoft Learn: https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score; https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score-improvement-actions]
- **Microsoft Entra Tenant Restrictions v2** primary control reference [Microsoft Learn: https://learn.microsoft.com/en-us/entra/external-id/tenant-restrictions-v2]
- **Microsoft Zero Trust Identity pillar — Initial Deployment Objective IV** ("Restrict user consent / Restrict access to applications") [Microsoft Learn: https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity]
- **MDA Conditional Access App Control intro** (for the supplemental CAAC role only) [Microsoft Learn: https://learn.microsoft.com/en-us/defender-cloud-apps/proxy-intro-aad]
- BNM RMiT clause(s): [BNM RMiT 10.x identity + access management; 11.x third-party / outsourcing access](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]` (illustrative only — not regulatory advice)
- MAS TRM Guidelines: third-party access + identity-and-access-management chapters `[VERIFY]` (illustrative only — not regulatory advice)
- ISO 27017 control(s): [CLD.9.x access-control overlay; CLD.6.3 administrative access](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.9 user access management `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-01` (identities and credentials issued / managed), `PR.AA-05` (access permissions / authorisations / managed), `PR.IR-01` (networks and environments protected from unauthorised logical access) `[VERIFY]`
- PCI DSS v4.0 Req. 7 (need-to-know), Req. 8 (identity), Req. 12.8 (third-party access management) `[VERIFY current sub-requirement numbering]` (illustrative only — not regulatory advice)

## False-positive risk

- Legitimate B2B-guest access to partner-tenant apps where the partner was not yet on the allowlist — exempt by adding the named partner tenant ID after business-owner confirmation
- User accessing legitimate personal services that share a SaaS family with corporate (e.g. personal LinkedIn alongside LinkedIn for Business; personal Microsoft account for an HMRC / IRAS / LHDN personal tax filing) — destination-domain carve-out at SWG
- Auditor / consultant who legitimately holds a consumer-account identity used in a documented business engagement — re-paper as B2B-guest provisioning; do not extend the consumer-account exception
- Mobile native apps where SWG cannot inject headers (cert-pinned) — accept the residual risk and document as a coverage gap; pair with Intune MAM where supported
- M&A / divestiture transitional period where two corporate tenants legitimately overlap — time-bounded transitional allowlist; documented end date

## Real-world FP experience

> **Practitioner-observed ranges; not from Microsoft documentation. Validate against tenant baseline.**

Typical FP-rate trajectory in a tier-2 BFSI tenant new to tenant restriction. FP here = "block fired against a sign-in that turned out to be legitimate-and-policy-compliant once exception reviewed":

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 (monitor-mode) | 35-50% | Pre-rollout — partner allowlist incomplete; orphaned but live partner tenants surface; personal-banking / regulator portals flagged because shared SaaS family |
| W4 | 18-28% | After W3 partner-inventory exercise + first allowlist additions + destination-domain carve-outs for personal-banking / gov portals |
| W8 | 8-15% | After monitor-mode pilot in 2-3 BUs; per-BU allowlist deltas surfaced; cards-ops scheme-operator allowlist mature; auditor-relationship re-papering complete |
| W12 | 4-9% | Tenant-wide block-mode steady; exceptions defined; off-corp roaming coverage documented; AUP refresh communicated; second tier of partner-onboarding flow integrated |
| Steady-state | 2-5% | Quarterly partner review + monthly exception pruning catch drift before it grows; new partner onboarding goes through CTAS-allowlist gate as part of contract-execution |

Named FP scenarios encountered repeatedly:

| Scenario | Mitigation |
|---|---|
| New partner tenant signed by Procurement; live business need; not yet on CTAS allowlist | Embed CTAS-allowlist row creation into the partner-onboarding workflow; SLA <48 hours from contract execution |
| Consumer-Google sign-in to LHDN / IRAS / HMRC personal-tax portal from corporate laptop during tax season | Destination-domain carve-out at SWG for named government portals; communicate via AUP refresh |
| Engineering staff signing into personal GitHub for OSS contributions | Documented per-UPN allowlist for contributor group; tenant-wide block remains; HR signs off on the carve-out as a compensating-control documented case |
| Auditor / external consultant signing in with consumer-Google to access shared documents | Re-paper as proper B2B-guest provisioning; the consumer-account workflow is itself an access-control gap |
| Off-corp roaming user with broken GSA tunnel hitting personal account; SWG header injection silently inactive | GSA tunnel-health monitoring; user-facing notification when tunnel down; documented degraded-mode posture |
| Cert-pinned mobile native app (Outlook Mobile, Gmail iOS) bypassing SWG header injection silently | Intune MAM (App Protection Policies) at the app layer; Entra CA requiring approved-client-app |
| M&A transitional period — two corporate Entra tenants both legitimate, neither on each other's allowlist | Time-bounded transitional allowlist with documented end date; review at 30 / 60 / 90 days |
| Conference / workshop venue Wi-Fi captive portal requiring consumer-account sign-in | Pre-trip exception window OR mobile hotspot guidance; not a standing exception |

## Operational cost

- **Exception-handling load:** medium during W4-W12 ramp (20-40 exception requests per week typical for a 5k-employee tenant — practitioner observation; not Microsoft-documented); low steady-state (5-10 per week, dominated by new-partner-onboarding rather than user-friction)
- **Triage load:** low ongoing — most blocks are silent at the IdP layer; SOC sees an alert only on suspicious patterns (named-individual repeated blocks, blocks from unusual home tenants)
- **End-user friction:** initially high — users who routinely sign into personal accounts on work devices see blocks immediately. Pre-rollout AUP refresh + clear self-service exception path absolutely critical. After 4-6 weeks of operating, friction drops sharply as users internalise the corporate / personal device boundary

Typical staffing: 0.3 FTE platform admin (IAM team) during the 10-week ramp; 0.1 FTE steady-state. Third-party-risk programme analyst commits ~0.1 FTE for the quarterly partner-review cycle. SOC contributes ~0.05 FTE for alert triage at the named-individual pattern level. (Staffing figures are practitioner observation; not Microsoft-documented.)

## Privacy / data-protection considerations

- The control does **not** inspect personal traffic content — it inspects only the sign-in tenant claim. Materially lower privacy footprint than SSL inspection of personal browser sessions
- Workforce-notice posture still applies — users must be informed which sign-in patterns are blocked. PDPA MY 2024 + GDPR Art. 88 / equivalent: workforce notice in employee handbook + AUP refresh must cover sign-in monitoring scope (illustrative only — not regulatory advice)
- Block-event logs contain user identity + attempted-home-tenant ID + timestamp + device — workforce-monitoring data. Document retention + access governance (typically: 13 months retention; access restricted to IAM admin + SOC L2; quarterly access-log review)
- DPIA pre-assessment: typically passes as low-impact (no content inspection); some EU works councils have raised questions on the "corporate device used during personal time" boundary — document the device-use policy interaction
- Cross-border: block events surface in the IdP / SWG region; document where the log lands and which jurisdiction governs

## Integration with broader programmes

- **Audit cycle:** Entra CTAS partner-allowlist export + quarterly partner-review meeting minutes + block-event audit log feed PCI DSS Req. 7 + Req. 12.8, ISO 27017 CLD.6.3, NIST CSF PR.AA-05 attestation evidence. Annual auditor pull; quarterly internal audit pull. **[VERIFY** clause numbers against current standard editions**]** (illustrative only — not regulatory advice)
- **Board reporting:** quarterly metric — "block events per 1k employees" + "partner-tenant allowlist count + quarterly delta" + named-incident summary (sanitised). Trend matters more than absolute number — sharp rises signal control drift or attacker probing
- **Incident response runbook:** repeated block events against a named individual feed insider-risk triage (correlate with HR pre-departure watchlist, Purview IRM, mass-download alert). Block events from unusual home tenants feed external-threat triage (Storm-2372-class consent-phishing patterns). Cross-references mass-download-alert IR runbook for the post-departure exfil case
- **Third-party-risk programme:** partner-tenant allowlist row IS a third-party access record. Annual partner-tenant review piggybacks on the third-party-risk review cycle. Removal of a partner from the allowlist follows contract-termination
- **Insider Risk Management:** block events from corporate users attempting personal-tenant sign-in feed Purview IRM as a risk-score booster. Standalone the signal is weak (most blocks are legitimate-but-policy-violating); correlated with departure notice / mass-download / anomalous-paste the signal is strong
- **DPIA refresh:** annual DPIA cycle includes review of block-event log retention + workforce-notice posture + scoping of the control to managed-device-on-corporate-network
- **Vendor-risk programme:** the SWG side is a sub-processor relationship; SOC 2 + ISO 27018 attestation pull; DPA review. GSA-specifically inherits the Microsoft sub-processor list (already in scope for the broader Microsoft stack)

## Anti-patterns specific to this policy

1. **"Inject the headers without first inventorying partner tenants"** — guarantees mass-breakage on day one of block-mode. Partner-allowlist work is the prerequisite; header-injection without it is a self-DOS
2. **"Trust the Entra CTAS 'default — trust all' setting"** — the implicit-trust default is itself the gap. Storm-2372-class consent phishing relies on the attacker tenant looking like a legitimate partner. Default must flip to "block all external, allowlist named partners"
3. **"Skip the monitor-mode phase"** — partner inventories are always incomplete on first pass; orphaned-but-live tenants surface only when traffic is observed against them. Going straight to block-mode generates an exception backlog that destroys help-desk credibility
4. **"Allowlist tenant-wide instead of per-BU"** — cards-ops sees scheme operators; treasury sees liquidity pools; finance sees auditors. A single tenant-wide allowlist becomes the union of all BU needs and is over-permissive. Per-BU scoping where the volume justifies the operational cost
5. **"Ignore the cert-pinned mobile native bypass"** — Outlook Mobile, Gmail iOS, Teams Mobile bypass SWG header injection silently. Without Intune MAM (App Protection Policies) the mobile surface is uncovered; tenant-restriction-on-desktop-only is a partial control and must be documented as such
6. **"Treat ChatGPT-personal-plan as 'same as ChatGPT Enterprise + tenant restriction'"** — personal-plan ChatGPT does not support SAML and is not bound by Entra CTAS. The personal-ChatGPT-on-personal-Gmail exfil pattern requires the SWG-header-injection layer to refuse the consumer-Google sign-in itself; CAAC paste-block alone (internal-numbering "Policy 9" — internal playbook nomenclature, not Microsoft documentation) does not cover it
7. **"Allowlist by tenant name string match instead of tenant ID"** — display names are spoofable; tenant IDs (GUIDs) are not. Storm-2372-class attacks specifically rely on display-name impersonation. Allowlist by GUID only
8. **"Set quarterly partner review and forget the orphan-detection automation"** — quarterly review catches drift slowly; partners are added between reviews and never removed. Automated "no traffic against this allowlisted tenant in 90 days" flagging closes the gap
9. **"No off-corp roaming coverage"** — corporate-network-only injection means users on hotel Wi-Fi / home Wi-Fi without GSA tunnel are uncovered. Coverage gap must be either closed (mandatory GSA tunnel) or accepted (documented residual risk)
10. **"No SOC alerting on block-event patterns"** — block events are not just access-control noise; repeated blocks against a named user signal an insider pattern, blocks from unusual home tenants signal external probing. Without SOC routing, the signal is lost
11. **"Conflate MDA with policy ownership"** — MDA is not the architectural owner of tenant restriction. Entra (Identity ZT Objective IV) owns the policy; GSA / SWG enforces the header injection; MDA's role is the in-session CAAC supplement only. Audit-evidence packages that present MDA as the owner mislead the reviewer

## Coverage gaps

- Personal device on personal network = entirely uncovered (architectural — no SWG in the path, no CTAS evaluation, no CAAC interception). Document as residual risk; pair with workforce-notice + AUP
- SaaS apps that do not support tenant-restriction-v2 headers (most consumer-grade apps; some long-tail enterprise SaaS) — only blockable via SWG URL-filtering, not at the sign-in layer
- Cert-pinned mobile native apps bypassing SWG header injection — pair with [Intune MAM coverage of BYOD](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md)
- [Continuous Access Evaluation session re-establishment](../../08-failure-modes/cae-session-bypass.md) — when CAE revokes a session mid-flight, the re-established session may not re-evaluate the tenant-restriction header constraint. Pair with Entra Token Protection
- Encrypted DNS-over-HTTPS bypassing the SWG — discovery and header-injection both blind. Browser-level DoH must be controlled via Edge / Chrome policy
- M&A / divestiture transitional periods where the partner-allowlist model strains — documented time-bounded posture rather than a structural fix
- Personal-account-on-corporate-device-on-personal-Wi-Fi (e.g. employee using corporate laptop on home Wi-Fi without VPN/GSA active) — partial gap; depends on whether GSA roaming agent is enforced always-on or user-initiated
- Service-principal / workload-identity flows (client-credentials) — out of scope of this control (user-flow only); covered by [Entra Workload Identities + Conditional Access for Workload Identities](../../08-failure-modes/oauth-application-permissions-gap.md)

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
