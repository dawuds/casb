# Block unsanctioned app (with coach-and-redirect)

> Status: v0.0 — MDA column draft from playbook content; other vendors `[unverified]`.
> Depth: **Tier 2 — deep-dive**.
> Required capabilities: [Shadow IT discovery + Sanctioned-app inventory + risk-scoring](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Log-based discovery + endpoint enforcement (MDE Network Protection) or SWG block. Hybrid.
> MDA playbook reference: [Policy 1 (Shadow IT discovery)](../../04-vendors/microsoft-defender-for-cloud-apps.md) for the discovery feed; enforcement layer via SWG/MDE.

## Purpose

Block discovered SaaS apps tagged Unsanctioned (consumer-tier, high-risk, or unapproved business apps) and coach users toward the sanctioned alternative. Reduces uncontrolled SaaS adoption and data leakage to vendors outside the firm's risk-acceptance perimeter. Counters MITRE ATT&CK `T1567 Exfiltration Over Web Service` and `T1102 Web Service` C2 patterns; precondition for the broader Shadow IT control programme.

## What organisations use this for

Almost every CASB deployment ships discovery in Phase 1 (see [`../../07-implementation/phased-rollout.md`](../../07-implementation/phased-rollout.md)). The decision that follows — *which discovered apps to block* — is the first political moment of the programme. Block too aggressively and the help desk drowns; block too softly and the discovery dashboard becomes wallpaper. Most organisations land on a coaching-first stance — block plus redirect to the sanctioned alternative — because hard block without context generates the workaround culture documented in [`../../08-failure-modes/over-blocking-and-user-circumvention.md`](../../08-failure-modes/over-blocking-and-user-circumvention.md).

The hardest sub-decision is the **sanction taxonomy itself** — who decides which app is Sanctioned vs Tolerated vs Unsanctioned, on what cadence, with what business-owner sign-off. The technology is small; the governance is the work.

### Use case 1 — Mid-cap tech firm post-consumer-AI exfiltration incident

- **Org type:** B2B SaaS company, ~3k employees, M365 E5, engineering-heavy, low-regulatory environment (privacy + SOC 2 only)
- **Trigger:** internal review found 40+ employees had been pasting product code into personal ChatGPT Plus over three months. Board ask: "ensure this doesn't happen with the next consumer AI tool"
- **Scope:** all employees; SWG-layer block + MDE Network Protection for roaming; coaching message names ChatGPT Enterprise (the new sanctioned variant) + Cursor (sanctioned developer AI)
- **Outcome:** within 30 days, ~95% of consumer-AI traffic migrated to sanctioned variants; ~5% persistent attempts (BYOD off-network) handled through coaching + AUP escalation; 12 Shadow-AI vendors added to the Unsanctioned list in Q1; documented sanction-decision process became the template for the broader SaaS estate

### Use case 2 — Mid-size insurance carrier, SOC 2 + ISO 27001 audit prep

- **Org type:** insurance carrier, ~5k employees, M365 E5, BNM Insurance Act + MAS Insurance Act supervised
- **Trigger:** annual SOC 2 Type II + ISO 27001 surveillance audit; previous year's auditor finding was "no documented sanction taxonomy for cloud services; relying on tribal knowledge"
- **Scope:** top-100 discovered apps in CASB catalogue; documented business-owner per Sanctioned app; quarterly sanction-decision committee; auto-unsanction by risk-score for new discoveries
- **Outcome:** evidence pack for SOC 2 CC6.6 (logical access — boundary), ISO 27001 A.5.23 (cloud services information security) `[VERIFY against gazetted text]`; sanction taxonomy became annual review artefact; auditor accepted CASB Sanctioned/Tolerated/Unsanctioned tagging as control evidence

### Use case 3 — Regional healthcare provider, HIPAA + state-level governance

- **Org type:** multi-state US healthcare provider, ~12k employees, M365 E5 + ChatGPT Enterprise, HIPAA covered entity, state-level AI rules in Colorado / California / Texas
- **Trigger:** HIPAA Privacy Officer flagged near-miss — claims processor used a free consumer transcription service for a recorded clinical interview; PHI ended up in vendor's training corpus
- **Scope:** all clinical + claims staff (~7k users); coaching message names HIPAA-BAA-covered alternatives + the firm's intake form for new vendor BAA requests
- **Outcome:** ~30 BAA-required vendors surfaced in first quarter that had previously been used without one; sanction process tightened with explicit BAA-or-block gate; ~15% reduction in HIPAA Privacy near-misses by Q3

### Use case 4 — Federal-adjacent organisation, post-OIG controlled-software inquiry

- **Org type:** large not-for-profit handling federal-funded research, ~8k employees, M365 E5, IRS / GAO audit cadence, federal-grant-compliance regime
- **Trigger:** Office-of-Inspector-General inquiry into "what software is actually used to handle federal-grant data" surfaced gaps; the firm had a documented Approved Software List that had not been enforced
- **Scope:** all federal-grant-handling teams (~2k users initially); aggressive block posture (block-by-default for non-listed) with documented exception path; coaching message names Approved Software List portal
- **Outcome:** ~120 unsanctioned apps blocked in first month (mostly free-tier note-taking + scheduling tools); 40% of these had legitimate business need and went through the Approved-Software-List exception flow; sanction-decision throughput became the bottleneck (forced staffing of the committee); OIG follow-up satisfied

## Implementation pattern

Typical 8-12 week rollout for a tenant new to sanction-based enforcement:

| Week | Activity | Output / gate |
|---|---|---|
| W1-W2 | Stand up Shadow IT discovery (MDE feed + log collector); inventory top 200 discovered apps | Discovery dashboard populated with ≥30 days data |
| W3-W4 | Define sanction taxonomy criteria (risk-score threshold, BAA / DPA gate, business-owner requirement); convene sanction-decision committee | Sanction-taxonomy policy approved by exec sponsor |
| W5-W6 | Initial sanction pass — classify top 100 apps as Sanctioned / Tolerated / Unsanctioned; document business-owner per Sanctioned | Per-app sanction-decision recorded |
| W7-W8 | Configure block at SWG + MDE Network Protection; coaching message + sanctioned-alternative reference page; pilot with small user group | Audit-mode pilot live |
| W9 | Flip to block-mode for pilot; monitor help-desk tickets; refine coaching message + exception path | First-week block-rate baseline |
| W10-W12 | Phased rollout to remaining user populations; weekly sanction-decision-committee cadence | Steady-state operations |
| W13+ | Quarterly sanction-decision review; monthly Shadow IT discovery review for new apps | Sustainable cadence |

The committee cadence is the operational bottleneck. Without weekly cadence in months 1-3, the discovered-but-unclassified backlog grows faster than decisions can be made.

## Action

- Primary: **block** at endpoint (MDE Network Protection in block mode) and / or SWG layer
- Coaching layer: **block + custom message** naming the sanctioned alternative, exception-path link, business-justification form

## Scope

- **Users:** all (excluding documented break-glass and exception groups)
- **Apps:** apps tagged `Unsanctioned` in the CASB catalogue
- **Device posture:** managed (block works at endpoint); BYOD (block works at SWG; off-network bypass)
- **Network position:** corporate network + roaming via endpoint agent
- **Traffic:** all (HTTP/HTTPS to the unsanctioned app domain)

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Cloud discovery → tag apps as Unsanctioned; pair with `Cloud apps → Policies → Shadow IT → App discovery policy` for auto-tagging | Tag = Unsanctioned; auto-tag policy with `Risk score ≤ 5` + `Users per day > 10` thresholds; MDE Network Protection in **block mode** (not audit) | Endpoint enforcement requires Defender for Endpoint deployed with Network Protection in **block** mode. SWG block-script export is CDN-fronted — partial fail | Auto-unsanction propagates only to MDE-managed endpoints. BYOD-mobile + off-network + non-MDE = uncovered. "Generate block script for SWG" produces flat IP/domain list — modern GenAI vendors use rotating CDNs |
| Netskope | `[unverified]` — Netskope CCI tagging + URL category block; native SWG enforcement available | | Native SWG path is structurally tighter than MDA→MDE chain | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security tagging + Prisma Access URL-filtering block | | | |
| Skyhigh | `[unverified]` — Skyhigh CCI + multi-mode enforcement | | | |
| Zscaler | `[unverified]` — ZIA URL filtering + Cloud App Control | | | |

## Worked configuration example (tier-2 BFSI baseline)

Anonymised values for a mid-size BFSI tenant after FP-tuning:

```yaml
sanction_taxonomy:
  decision_committee:
    chair: "Head of IT Security"
    members: ["CIO delegate", "Legal", "DPO", "Procurement", "BU representatives"]
    cadence: weekly
  criteria:
    sanctioned:
      requires:
        - "Documented business owner"
        - "BAA or DPA signed (if PII / regulated data in scope)"
        - "Risk score >= 7 OR exception-approved"
        - "Reviewed by Privacy + Legal"
    tolerated:
      requires:
        - "Risk score >= 5"
        - "No regulated data in scope"
        - "Quarterly review for promotion or demotion"
    unsanctioned:
      auto_tag_policy:
        category: "any"
        risk_score: "<= 5"
        users_per_day: "> 10"
        action: "Tag as Unsanctioned + Alert"

enforcement:
  endpoint:
    mde_network_protection: block_mode
    fallback_for_non_mde: "swg_url_filtering"
  swg:
    block_action: "user-facing-message"
    block_message_template: |
      This application is not on the company-approved list.
      Sanctioned alternative: {sanctioned_alt}
      Exception request: {exception_form_url}
      Contact: it-security@example.com

allowlist:
  exception_group: "approved-app-trial"
  expiry_days: 90
  re_review_required: true
```

## Variants

### Industry-specific

- **BFSI:** BAA / DPA gate is non-negotiable for any app touching regulated data; Risk-score floor for sanction is typically higher (7+); regulator sensitivity (BNM RMiT third-party / MAS Outsourcing) drives faster blocking of new high-risk discoveries
- **Healthcare:** HIPAA BAA is the gate; PHI-handling categories blocked aggressively even at high risk score; state-level AI rules add per-state nuance
- **Tech:** lower risk-score floor (5-6) acceptable; faster sanction throughput; consumer-AI category most-fought
- **Retail:** lower regulatory pressure; loyalty-data leakage is the main concern; longer tail of department-specific tools to triage
- **Public sector:** Approved-Software-List + central procurement is often the gate; sanction decisions tied to procurement record

### Maturity-based

- **Immature:** tenant-wide enforcement against a flat Unsanctioned list; no taxonomy; no committee; no coaching; help-desk flood at month 1; programme rolled back to audit-only by month 3
- **Mature:** documented taxonomy + weekly committee + per-BU sanction decisions; coaching message with named alternative; exception process documented; FP rate / overblock-rate measured monthly; tenant-wide block-coverage trends published quarterly
- **Advanced:** sanction-decision committee integrated with procurement workflow; new-vendor onboarding triggers sanction-review automatically; allowlist managed-as-code (Terraform-style + audit log); Adaptive Protection-integrated for risk-based per-user scoping; SaaS-admin-of-the-month rotation across BUs to maintain governance presence

## Control mappings

- BNM RMiT clause(s): [BNM RMiT third-party / outsourcing + cloud services](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY against current edition]`
- ISO 27017 control(s): [CLD.6.3.1 shared responsibility, CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `GV.SC` (cybersecurity supply chain), `ID.AM-04` (services from suppliers), `PR.AA-05` (access permissions), `DE.CM-09` `[VERIFY]`
- SOC 2 Trust Services Criteria: CC6.6 (logical access — boundary), CC9.2 (vendor management)

## False-positive risk

- Approved-by-business AI vendor scoring ≤5 due to "SOC 2 = No" in catalogue but enterprise contract exists — exempt by app ID
- CDN-fronted domains shared across legitimate and unsanctioned apps — block-script granularity too coarse
- New SaaS with no catalogue entry yet — classified Unknown, may be blocked during legitimate trial
- Vendor reorganisation / rebrand where the catalogue still tracks the old name

## Real-world FP experience

| Week | Typical FP rate | Dominant cause |
|---|---|---|
| W1 | 40-60% | Mis-classified Tolerated apps blocked as Unsanctioned; new vendors in trial blocked while sanction-decision pending |
| W4 | 20-30% | After first sanction-committee pass on flagged FPs |
| W8 | 10-15% | After allowlist-by-app-id for known-approved exceptions |
| W12 | 5-10% | Mature sanction-decision throughput closes gaps before they grow |
| Steady-state | 3-6% | Quarterly review cycle catches new mis-classifications before user impact |

Named FP scenarios:

| Scenario | Mitigation |
|---|---|
| Procurement signed a vendor but Security wasn't notified | Joint procurement-security workflow; new-vendor sanction-decision auto-triggered |
| Vendor scores poorly in catalogue but firm has enterprise contract / DPA | Allowlist by app ID; document the override |
| Free-tier trial of a high-risk app for legitimate evaluation | Time-bounded exception (30 days) + review |
| Engineering team adopts a new dev tool (e.g. Cursor) faster than sanction throughput | Pre-approved category lists; engineering sanction sub-committee |

## Operational cost

- **Exception-handling load:** high during W1-W12 ramp (15-30 exceptions per week); medium steady-state (5-10 per week)
- **Triage load:** low — block events are silent at endpoint; SOC sees alerts only for high-priority categories
- **End-user friction:** high in first month; declining as coaching messages improve and sanction-decision throughput catches up

Typical staffing: 0.2 FTE security analyst (sanction decisions + exception triage); committee participation ~2 hours / week across 5 members; help-desk impact ~10-20 tickets per week in month 1, declining.

## Privacy / data-protection considerations

- Discovery feed includes user identity tied to app access patterns — workforce monitoring; document under PDPA MY 2024 / GDPR Art. 88 workforce-monitoring posture
- Block at endpoint does not require SSL inspection — lower privacy footprint than inline DLP
- Coaching message text reviewed by Privacy + Legal — must not imply more inspection than is occurring
- Allowlist contents are governance records — access-controlled to sanction-committee members

## Integration with broader programmes

- **Procurement workflow:** new-vendor onboarding triggers sanction-review automatically; procurement sign-off requires sanction-decision record
- **Vendor risk management:** sanction-decision feeds the vendor inventory + DPA register; quarterly attestation cycle uses sanctioned-app list as the in-scope set
- **Annual audit:** sanction taxonomy + committee minutes + block-coverage metric feed SOC 2 / ISO 27001 attestation; auditor sample-tests the sanction-decision evidence trail
- **Board reporting:** quarterly metric — Shadow IT count (declining trend), Sanctioned-app count (stable), Block-action count (declining trend)
- **AUP enforcement:** repeated personal-AI attempts after coaching → HR escalation; documented escalation tiers

## Anti-patterns specific to this policy

1. **"Block everything not-Sanctioned"** — without coaching layer, drives users to workarounds (personal devices, personal accounts). Always pair with named alternative + exception path
2. **"Run sanction committee monthly"** — too slow; discovered-but-unclassified backlog grows faster than decisions. Weekly cadence in months 1-3 minimum
3. **"Single sanction committee covers the whole org"** — different BUs have different SaaS needs; central committee becomes bottleneck. Federate by BU (engineering, finance, marketing each with sub-committee feeding central)
4. **"Coaching message lists generic alternative"** — "use an approved tool" is unhelpful; name the specific app + how to access it
5. **"Auto-unsanction by risk score alone"** — Microsoft / vendor opaque scoring overrides business reality; pair with human-in-the-loop review for any new auto-unsanction
6. **"No exception path"** — exceptions happen anyway, off-the-record. Documented path with documented expiry is the discipline
7. **"Skip the BAA / DPA check"** — sanction-decision committee that doesn't include Legal + Privacy creates downstream compliance exposure
8. **"Treat Tolerated and Unsanctioned the same operationally"** — Tolerated should be Monitor-only; Unsanctioned blocked; conflating produces over-blocking and audit-evidence muddle

## Coverage gaps

- [BYOD and unmanaged endpoint coverage gap](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — personal devices without MDE/MDM are uncovered
- [Over-blocking and user circumvention](../../08-failure-modes/over-blocking-and-user-circumvention.md) — block-without-coach drives users to personal devices / personal accounts; the move-to-the-shadow class
- Personal-account sign-in to consumer SaaS on corp device — Entra and CASB never see it
- Encrypted DNS-over-HTTPS bypassing SWG — emerging gap
- Mobile native apps with own auth — pair with Intune MAM

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
