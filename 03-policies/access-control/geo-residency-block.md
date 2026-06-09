# Geo-residency block

> Status: v0.0 — cross-vendor pattern; MDA implementation via location-filter + sensitivity-label filter; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [Data-residency / geo-policy enforcement + Inline DLP](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Hybrid (inline for block; API for residency-aware classification).
> MDA playbook reference: implicit via Location filter on session/access/activity policies; not a numbered Day 1/30/90 policy in MDA v1.

## Purpose

Block upload to (or download from) data regions outside the firm's permitted geographic perimeter for tagged regulated data. Enforces data-residency requirements imposed by regulator or contract — e.g. Malaysian-tagged data not allowed to leave MY-resident regions; EU-tagged data subject to GDPR Chapter V transfer rules. Reduces cross-border-transfer compliance exposure.

## What organisations use this for

Geo-residency block is one of the most misunderstood policies in the CASB catalogue. Practitioners deploy it expecting a single control that "stops data leaving the country"; what they actually deploy is a layered set — a CASB session/activity policy keyed on the *user's location*, plus a data-layer enforcement (Purview label policy / bucket-region rule / Workspace data-region setting) keyed on the *file's residency tag*. The CASB layer alone enforces *user-action-by-location*, not *data-by-region*. Conflating the two is the dominant misconfiguration on first rollout.

The policy is rarely demand-driven by an incident; it is demand-driven by an audit cycle, a regulator inquiry on cross-border data flows, or a customer-contract residency commitment that surfaces during sales due diligence. The deployment scope is typically narrow at first (one BU, one data class, one regulator's residency regime) and broadens as the firm builds a residency-labelling taxonomy.

### Use case 1 — Tier-1 ASEAN universal bank, BNM cross-border data-flow inquiry

- **Org type:** large universal bank, ~30k employees, M365 E5, BNM RMiT supervised, retail + corporate + Islamic banking
- **Trigger:** BNM supervisory inquiry on cross-border data flows following an industry-wide review of Malaysian FIs' use of foreign cloud regions; bank had to demonstrate per-data-class residency posture for customer KYC, AML transaction history, and credit-risk data within 90 days `[VERIFY against the specific BNM circular referenced]`
- **Scope:** Phase 1: KYC + AML BUs (~4k users); residency-tagged label family (`MY-Resident-KYC`, `MY-Resident-AML`); MDA session policy with country whitelist = MY only on download for files carrying these labels; parallel Purview label-policy preventing labelled files being saved to non-MY OneDrive geo
- **Outcome:** evidence pack accepted in the supervisor's follow-up; ~600 cross-border access attempts intercepted in the first 90 days, of which ~80% were legitimate travelling staff who needed an exception path, ~15% were misrouted Power BI dataset access, ~5% were genuine policy violations escalated to the data-protection officer. Tenant-region migration to Japan East (added 2025 H2) was a separate workstream needed to close the auditor's residual concern on log-data residency

### Use case 2 — Pan-EU asset manager under GDPR Chapter V

- **Org type:** EU-headquartered asset manager (~8k employees) with sub-advisory operations in US, UK, Singapore; client base spans EU retail + global institutional; M365 E5; in scope for GDPR + DORA (operational-resilience-driven cross-border-data review)
- **Trigger:** Schrems II follow-on review of US sub-processor flows; legal-and-compliance ask to enforce, not just document, the EEA-resident classification of EU-client personal data following the EDPB's renewed guidance on supplementary measures `[VERIFY against current EDPB guidance]`
- **Scope:** EU-client-facing BUs (~3k users); residency label `EEA-Personal-Data` applied via Purview DCS to client-onboarding, portfolio-statements, KYC folders; MDA session policy block on download outside EU country list (27 member states + UK as adequacy-decision exception); CAAC required to intercept session before download
- **Outcome:** documented control evidence for the firm's Chapter V Article 46 SCC programme; revealed undocumented data flow from EU-client folders to a US sub-advisor's read access (legitimate but not previously captured in the TIA); led to a TIA refresh and SCC re-execution for that sub-processor relationship

### Use case 3 — Multi-jurisdiction merchant acquirer, sovereign-data overlay

- **Org type:** payments processor operating across MY / SG / HK / TH / ID; in-scope for PCI DSS + BNM RMiT + MAS TRM + HKMA SA-2 + BoT supervisory expectations + OJK PERATURAN 11/2022 on personal data protection in finance `[VERIFY]`
- **Trigger:** simultaneous regulator queries arising from a peer-firm incident in 2025 that exposed cross-border CHD flows; board-level ask to demonstrate "per-jurisdiction residency for per-jurisdiction CHD"; coupled with Indonesia's sovereign-cloud trajectory
- **Scope:** residency-aware label taxonomy (`Conf-CHD-MY`, `Conf-CHD-SG`, `Conf-CHD-HK`, `Conf-CHD-TH`, `Conf-CHD-ID`); per-jurisdiction MDA session policy keyed on label + location; Purview label-publish scoping per jurisdiction-aligned user OUs; tenant primary-data region pulled and documented per regulator submission; sovereign-region commitment articulated against the firm's roadmap
- **Outcome:** per-jurisdiction residency-by-data-class evidence pack accepted by three of five regulators; remaining two requested supplementary in-country tenancy commitments that drove a Microsoft-tenant-region procurement decision separate from the CASB control. Ongoing cost of maintaining the 5-label taxonomy estimated at 0.4 FTE platform admin + 0.2 FTE compliance analyst

### Use case 4 — Sovereign healthcare programme, in-country processing mandate

- **Org type:** public-sector healthcare data-platform operator (~2k staff + 20k clinician users) under a sovereign in-country processing mandate codified in a national health-data circular; M365 G5-equivalent (public-sector tier)
- **Trigger:** national audit office review of cloud-hosted health data following a 2024 policy directive requiring "patient data shall be processed and stored in-country except where a documented exemption applies"; the audit office distinguished *storage* (in-country tenant) from *processing* (where the user accessing the data is located)
- **Scope:** all clinician users; label `Health-PHI-Domestic`; MDA session policy block on access from outside the country, even for read; documented telemedicine-from-abroad exception path through a named clinical-review committee; combined with conditional-access GeoIP block at Entra layer (defence in depth)
- **Outcome:** audit office accepted the layered control as meeting the directive's processing requirement; produced a quarterly metric "% of PHI access attempts from outside country" that became a reported KPI to the ministry. False-positive rate dominated by clinicians on training fellowships abroad — ~25% of alerts in the first quarter, fell to ~5% after the exemption path was operationalised

## Implementation pattern

Typical 12-week rollout sequence for a tier-2 BFSI tenant adding residency enforcement. This is longer than most CASB policies because the data-layer residency taxonomy is itself a deliverable.

| Week | Activity | Output / gate |
|---|---|---|
| W1-W2 | Residency obligation inventory — list every regulator, contract, and customer commitment requiring per-jurisdiction data treatment; identify the data classes in scope; map to existing Purview label taxonomy (or design new) | Residency obligation register; gap analysis against current label scheme |
| W3 | Tenant primary-data region verification — pull current region from M365 Admin Center; document against the regulator's expectation; identify whether tenant migration is needed (Japan East / India / EU as appropriate) | Region attestation document; tenant-migration decision (in or out of scope) |
| W4 | Label-taxonomy design (residency-tagged label family); per-jurisdiction publish scoping; label-application rules (auto-label via DCS on residency-anchor folders) | Labels created and published to scoped users |
| W5-W6 | DCS pilot on a single residency-anchored folder set (~100 files); validate end-to-end auto-labelling produces the expected residency tag | Label-application baseline; FP rate on classification step |
| W7 | MDA session policy design with Location filter (country whitelist) + File filter (label ∈ {residency-tagged}) + Action = Alert (NOT block on first pass) | Policy in monitor-only mode; CAAC backplane validated |
| W8 | Entra Conditional Access policy authoring the CAAC redirect for the affected user scope; break-glass exclusions documented; rollback plan written | CAAC path live for scoped users; rollback tested |
| W9 | First-tuning-pass on the alert mode — surface legitimate-travel patterns, VPN egress geo errors, null-location events, anonymising-proxy patterns | Exception path documented; named travel-exception group operationalised |
| W10 | Flip to Block action for the pilot BU; communications package to users explaining the residency policy and the exception process | Block live for pilot scope; help-desk runbook |
| W11 | Broaden to remaining in-scope BUs by phase | Per-BU rollout completion |
| W12 | Steady-state handoff; quarterly review cadence documented; first audit-evidence pull tested | Quarterly metric defined; audit-evidence package proven |
| W13+ | Quarterly tuning + annual residency-obligation refresh | Refreshed residency register annually; FP rate measured monthly |

The single most underestimated step is W7-W8 — the bilateral CAAC trap. Without both the Entra CA redirect and the MDA session policy, a session policy on its own is a silent no-op. Practitioners new to MDA frequently configure only the session policy, see no enforcement in testing, and assume a vendor bug rather than a missing CA policy.

## Action

- Primary: **block** upload/download to/from non-permitted region
- Alternative: **monitor** + alert (audit mode while baselining)

## Scope

- **Users:** all (or scoped to teams handling residency-tagged data — Finance, HR, KYC, Legal)
- **Apps:** sanctioned SaaS with region-aware storage (M365 multi-geo; AWS S3 with bucket-region tag; Google Workspace with data-location settings)
- **Device posture:** managed and BYOD (residency is data-attribute-driven, not device-driven)
- **Network position:** any
- **Traffic:** upload + download involving residency-tagged data

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Session/Activity policy with Location filter (country-level). Plus Purview sensitivity labels carrying residency metadata | Location filter = country whitelist; File filter = Sensitivity label ∈ {residency-tagged}; Action = Block | Country-level only — no sub-country granularity. M365 multi-geo "supported only for OneDrive" per docs; SharePoint multi-geo events not fully captured | Location filter on session policy fires for the user's *current location*, not the file's *origin region*. To enforce file-region-residency you need Purview Information Protection rules at the data layer in parallel — MDA enforces user-action-by-location, not data-by-region |
| Netskope | `[unverified]` — Netskope geo-fencing + DLP integration | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access geo-policy + SaaS Security | | | |
| Skyhigh | `[unverified]` — Skyhigh multi-mode geo control | | | |
| Zscaler | `[unverified]` — ZIA geo filtering + CASB | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised configuration values for a tier-2 ASEAN BFSI tenant enforcing MY-resident treatment of KYC data, after the W7-W9 tuning is complete:

```yaml
policy:
  name: "Geo-residency block — MY-Resident-KYC, KYC BU"
  type: SessionPolicy
  caac_required: true                      # Entra CA redirect must exist in parallel
  app_filter:
    - OneDrive for Business
    - SharePoint Online
    - Exchange Online (web)
  user_scope:
    include_groups:
      - "KYC-Operations"
      - "AML-Investigations"
      - "Customer-Onboarding-Tier1"
    exclude_groups:
      - "break-glass-soc"                  # break-glass exclusion (mandatory)
      - "break-glass-identity"
      - "executive-travel-approved"        # named exception group, justified
  location_filter:
    is_set: true                           # CRITICAL — without this, null-location events bypass
    country_whitelist:
      - MY
    treat_unknown_as: outside_whitelist    # null-location ≠ MY
  file_filter:
    sensitivity_labels:
      - "MY-Resident-KYC"
      - "MY-Resident-AML"
    file_classification_origin: PurviewDCS
  match_parameters:
    traffic_direction:
      - download
      - upload
    include_inline_view: true              # browser preview also intercepted
  governance:
    action: Block
    block_message: >
      Access to MY-Resident-KYC data is restricted to MY-located sessions.
      For exceptional access, contact data-protection@example.com.
    notify_user: true                      # transparency — user sees the block reason
    soc_alert: true
    siem_forward: true
  exception_path:
    workflow_owner: "data-protection-officer"
    approval_chain: ["line-manager", "DPO", "head-of-compliance"]
    documented_at: "intranet:/policies/cross-border-data-access"
    max_duration_days: 14                  # time-bounded exceptions only
  monitoring:
    review_cadence: monthly
    metrics:
      - blocks_per_week_by_country
      - exceptions_granted_by_BU
      - null_location_event_count
```

The `treat_unknown_as: outside_whitelist` setting is the single most important defensive tuning. Default behaviour in some configurations is to permit null-location events as "could not determine" — which is the bypass path attackers and VPN-on-iOS users routinely take.

## Variants

### Industry-specific

- **BFSI:** the dominant driver is regulator residency expectations (BNM RMiT, MAS TRM, HKMA, EU DORA + Chapter V); per-jurisdiction CHD residency overlays PCI DSS scoping; CAAC required because API-mode-only enforcement is post-event and gives no audit-credible prevention story. Common pitfall: treating customer-contractual residency commitments as a separate workstream from the regulator residency programme — they should share a label taxonomy
- **Healthcare:** PHI residency increasingly codified at national level (national health data acts; in-country processing mandates); the *processing* vs *storage* distinction matters more than in finance — auditors probe whether clinicians accessing PHI from abroad constitute extraterritorial processing. Telemedicine-from-abroad is a named exception class
- **Tech / SaaS:** customer-data residency promises in MSAs and DPAs drive enforcement; multi-tenant SaaS firms often need per-customer label families (one customer = EU only, another = APAC only); engineering cross-region debugging is a high-friction exception that needs a documented break-glass
- **Retail:** loyalty-programme PII residency under PDPA-type regimes; less granular than BFSI but higher volume; the FP fight is around legitimate cross-border vendor access (e.g. APAC analytics teams accessing EU customer data) which often required SCC + label-aware governance rather than a hard block
- **Public sector / sovereign:** in-country processing mandates frequently codified in legislation or ministerial directive; "exemption process" is itself a controlled artefact requiring named-committee approval; tenant region itself is often non-negotiable — drives sovereign-cloud procurement separately from the CASB control

### Maturity-based

- **Immature:** single tenant-wide policy, country whitelist with one country, no `is_set` clause (null-location bypass open), no exception path, action = Block on day one, no parallel data-layer enforcement (Purview label-policy missing). Result: legitimate travelling staff blocked indiscriminately, help-desk overwhelmed, policy rolled back within weeks. Or — the inverse — action = Alert with no follow-up triage, alerts pile up unread, no enforcement actually achieved
- **Mature:** per-BU policies scoped to residency-tagged label family; CAAC backplane validated; `is_set` clause configured; named travel-exception group with documented approval workflow + time-bounded exceptions; quarterly tuning cycle; parallel Purview label policy enforcing data-layer residency (so files cannot be saved into non-resident geo); FP rate <10%; quarterly audit-evidence pull tested
- **Advanced:** per-jurisdiction label taxonomy with per-jurisdiction CAAC policies; sovereign-tenant-region commitment articulated against the regulator's expectation; combined with Entra CA GeoIP block as defence in depth; integration with HR travel-notification system to pre-authorise short-term exception windows for known business travel; per-jurisdiction metric reported quarterly to the board; named-incident review for any cross-border access classed as policy violation; TIA refresh annually with the cross-border-flow evidence pack as input

## Control mappings

- BNM RMiT clause(s): [BNM RMiT cloud services + cross-border data transfer](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]`
- PDPA MY 2024 (Act A1709): cross-border-transfer regime `[VERIFY against gazetted text]`
- ISO 27017 control(s): [CLD.6.3.1 shared responsibility, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 A.10, A.11.1: use limitation, sub-processor disclosure relevant
- GDPR Chapter V (international transfers) for EU-tagged data; Article 46 SCC programme cross-reference; EDPB supplementary measures guidance `[VERIFY current]`
- DORA Article 28 (third-party risk) + ICT third-party register for residency-impacting sub-processors `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.DS-01` (data-at-rest), `PR.DS-02` (data-in-transit), `GV.SC-04` (supplier residency in supply-chain controls) `[VERIFY]`

## False-positive risk

- "Absence of clearly defined location" may identify risky activities — null-location events caught by Location filter without an `is set` companion filter. Mitigate with explicit `Location is set` clause
- Roaming employees legitimately accessing data from outside permitted region — name an exception group with documented justification
- VPN egress points geographically distant from user — may register as wrong country
- Anonymising-proxy / Tor egress — may register as a permitted country by IP geo but should be blocked separately
- Cloud-provider egress IPs (Azure / AWS / GCP) mis-attributed by GeoIP — common with PaaS-mediated access

## Real-world FP experience

Typical FP-rate trajectory in a tier-2 BFSI tenant new to residency enforcement:

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 (alert-only) | 50-70% | Mis-classified labels (DCS labelling under-tuned); null-location events; VPN egress GeoIP mismatch; legitimate business travel without exception process |
| W4 | 25-35% | After named travel-exception group operationalised + null-location handling tightened |
| W8 | 12-20% | After per-BU CAAC scoping + label taxonomy refinement + Power BI / Dynamics service-account exclusions |
| W12 (block live) | 6-12% | Mature exception workflow; integrated with HR travel-notification; quarterly tuning cycle running |
| Steady-state | 3-8% | Combined with Entra CA GeoIP for defence in depth catches the residual; remaining FPs are legitimate travel + cloud-provider IP geo errors |

Named FP scenarios encountered repeatedly across deployments:

| Scenario | Mitigation |
|---|---|
| Roaming employee accessing labelled data from a permitted-by-business but non-whitelist country (e.g. MY KYC analyst on overseas training) | Named travel-exception group with time-bounded membership; integrated with HR travel-notification system where one exists |
| VPN egress in a different country to the user (corporate VPN concentrator in SG for MY user) | Document corporate VPN egress IPs as a known-source enrichment; ideally relocate VPN egress to match residency expectation |
| Null-location event from mobile-app session, residential proxy, or browser-locale-suppressed connection | `is_set: true` filter + `treat_unknown_as: outside_whitelist` policy stance — the more defensive option |
| Power BI / Dynamics service-account access from non-whitelist Azure region | Service-account exclusion list; Power BI tenant-region enforcement as a separate control |
| Cross-tenant B2B guest access where guest's home tenant is in a different region | Per-tenant Cross-Tenant Access Settings carrying residency obligations into the partner relationship; do not rely solely on the CASB layer |
| Cloud-provider IPs mis-attributed by GeoIP (Azure / AWS / GCP edge nodes registered in a different country than where service runs) | Maintain a known-cloud-provider IP-range enrichment; periodically refresh against the providers' published IP-range datasets |
| Telemedicine / consultant-from-abroad scenarios in healthcare and consulting | Named clinical-review or matter-management exception process; time-bounded approvals |
| Legitimate cross-border collaboration with EEA adequacy-decision countries / FATF-aligned jurisdictions | Whitelist expansion to include the documented adequacy-decision list; reviewed annually |

## Operational cost

- **Exception-handling load:** high during business-travel periods; lower steady-state
- **Triage load:** medium — region anomalies often correlate with legitimate travel
- **End-user friction:** medium for roaming users; high for cross-border collaboration

Typical staffing: 0.3-0.5 FTE platform admin during the 12-week ramp (label-taxonomy work is the heavy step); 0.2 FTE steady-state. Compliance analyst commits ~0.15 FTE for the cross-border-flow evidence pack and quarterly audit-evidence pull. Help-desk Tier 1 absorbs the bulk of exception-request volume; expect 2-5x the steady-state ticket rate during business-travel-heavy quarters.

## Privacy / data-protection considerations

- Location enrichment uses IP + geo-lookup — workforce-location data; document under PDPA / GDPR Art. 88 workforce-monitoring posture
- Country-level location accuracy is Microsoft's claim; VPN endpoints, cloud-provider IPs, residential proxies all routinely produce wrong country
- The matching-content snippet on a blocked event may carry residency-tagged content — itself subject to the residency obligation that triggered the block. Where does the CASB store that snippet? If the tenant primary-data region is outside the obligation's geography, the snippet itself violates the policy. This is the classic recursive-residency trap; check tenant region against the strictest in-scope obligation
- DPIA / TIA trigger: any residency-enforcement programme touching personal data warrants a DPIA refresh; for EU personal data with Chapter V exposure, a TIA refresh is required to document the technical and organisational measures (the CASB control is one such measure)
- Workforce-monitoring notice: AUP and employee handbook must explicitly cover location-based access controls and the use of GeoIP enrichment

## Integration with broader programmes

- **Audit cycle (regulator-led residency):** quarterly metric — blocks-per-week-by-country + exceptions-granted-by-BU + null-location-event count — feeds the cross-border-flow evidence pack. For BNM RMiT supervisees, this becomes part of the annual self-assessment; for MAS TRM, the cross-border-flow disclosure schedule
- **DPIA / TIA refresh cycle:** the residency-block control is named as a technical measure in the TIA for any sub-processor flow involving EU personal data; refreshed annually or on material change to sub-processor relationships
- **Vendor / sub-processor risk register:** any sub-processor with residency-affecting access surfaces in the register; residency-enforcement control documented as one mitigation of the cross-border-transfer risk
- **Board / executive reporting:** quarterly metric — "% of cross-border access attempts blocked" trend; named-incident summary for policy violations escalated to DPO. The trend matters more than the absolute number — a sudden spike usually signals a misrouted application flow rather than a hostile actor
- **Incident response runbook:** a blocked event correlated with anomalous sign-in risk feeds an Identity-and-Access IR runbook (potential session hijack from outside the residency geography); a blocked event from a legitimate user pattern feeds a Data-Protection IR runbook (process review, not user discipline)
- **Contracts and customer obligations:** the residency-block control is the technical evidence behind customer-DPA residency commitments; should be named in customer-facing assurance reports (SOC 2 / customer security questionnaire responses)
- **Tenant region procurement:** the CASB control surfaces tenant-region inadequacy. For Malaysian / Singapore / Hong Kong FIs, the Japan East primary-data region (added 2025 H2) materially improves the cross-border story; for India, the H2 2026 roadmap matters. Verify the current state on the Microsoft Cloud regions page on the day of attestation
- **Business-continuity / DR:** residency-block must accommodate DR geography — if the DR site is in a different country, the policy needs a documented DR-window exception or the DR site must be in-residency

## Anti-patterns specific to this policy

1. **"The CASB location filter enforces data residency"** — it does not. It enforces user-action-by-location. Without a parallel data-layer enforcement (Purview label policy, bucket-region rule, Workspace data-region setting) you have not enforced data residency; you have enforced where the user can be when they touch the data
2. **"Country whitelist without `is set` clause"** — null-location events bypass the filter silently. The first attacker to use a VPN with location-spoof headers walks through; the first mobile-app session with location-suppression walks through. Always pair `country in {whitelist}` with `location is set = true`
3. **"Block from day 1 without a documented exception path"** — legitimate travelling staff are blocked indiscriminately; help-desk overwhelmed; executive-travel scenarios escalate to the CIO; policy rolled back within 2 weeks. Always start in monitor-only mode with the exception workflow live before flipping to block
4. **"Session policy without the corresponding Entra Conditional Access redirect"** — the bilateral CAAC trap. The session policy compiles, looks live in the console, generates no events. Operator assumes the policy is in audit mode or that "nothing matched", when in fact the session was never redirected through the CASB proxy
5. **"Treating customer-contractual residency as separate from regulator residency"** — leads to two parallel label taxonomies, two parallel CAAC policies, and a label-precedence conflict where a file is both `Customer-X-EU-Only` and `MY-Resident-KYC`. Unify the taxonomy or document the precedence rule explicitly
6. **"Ignoring the recursive-residency trap on the CASB itself"** — your CASB stores activity logs and policy-match snippets. If the tenant primary-data region is outside the obligation's geography, the CASB itself is the residency violation you deployed it to prevent. Verify tenant region before deployment, not after the auditor finds it
7. **"Country-level granularity is good enough"** — for ASEAN scenarios where KL ↔ SG ↔ JB is the legitimate-travel pattern, country-level is too coarse; for federal jurisdictions where state-level residency obligations exist (e.g. some US state laws, some EU member-state overlays), country-level under-enforces. Document the granularity limitation against the obligation
8. **"Forgetting Power BI / Dynamics service accounts"** — API-mode-only apps with their own service-account access patterns generate noise when they cross region boundaries on Microsoft's own backend; exclude documented service accounts or accept the noise
9. **"No HR travel-notification integration"** — every exception request becomes a manual workflow; the help desk becomes the bottleneck. Where a corporate HR travel-notification system exists, integrate it so pre-approved travel auto-applies the named travel-exception group for the trip window
10. **"Blocking the SOC and break-glass identities"** — incident responders need cross-border investigative access during an incident affecting offshore infrastructure. The break-glass exclusion group is mandatory and must be tested annually; the test result is itself audit evidence

## Coverage gaps

- Tenant primary-data region (where the CASB itself stores activity logs / DLP snippets) is separate from this policy's geo-block — the CASB log data residency is determined at tenant provisioning. For MDA: EU / US / UK / AU / Japan East (verify Trust Center on day of attestation)
- API-mode delays (Power BI / Dynamics 24-72hr) — region-block is post-event for non-inline traffic
- File-region-residency requires data-layer enforcement (Purview / GCP / AWS bucket-region rules) — MDA alone is user-action-by-location
- [Encrypted-upload bypass](../../08-failure-modes/encrypted-upload-bypass.md) — encrypted-by-user content cannot be tagged by content scan
- Sub-country granularity not available at the CASB layer — state / province / federal-territory residency obligations not directly enforceable
- Mobile-app and certificate-pinned native-app sessions bypass CAAC; residency enforcement on those flows requires Intune App Protection Policy or equivalent MAM control
- Cross-tenant B2B guest access where the guest's home tenant resides in a different geography — Cross-Tenant Access Settings + label-aware sharing controls needed; this policy alone insufficient
- GeoIP accuracy on cloud-provider egress IPs is poor; maintain a refreshed enrichment list

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
