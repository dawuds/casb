# Block unsanctioned app (with coach-and-redirect)

> Status: v0.0 — MDA column draft from playbook content; other vendors `[unverified]`.
> Required capabilities: [Shadow IT discovery + Sanctioned-app inventory + risk-scoring](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Log-based discovery + endpoint enforcement (MDE Network Protection) or SWG block. Hybrid.
> MDA playbook reference: [Policy 1 (Shadow IT discovery)](../../04-vendors/microsoft-defender-for-cloud-apps.md) for the discovery feed; enforcement layer via SWG/MDE.

## Purpose

Block discovered SaaS apps tagged Unsanctioned (consumer-tier, high-risk, or unapproved business apps) and coach users toward the sanctioned alternative. Reduces uncontrolled SaaS adoption and data leakage to vendors outside the firm's risk-acceptance perimeter. Counters MITRE ATT&CK `T1567 Exfiltration Over Web Service` and `T1102 Web Service` C2 patterns.

## Action

- Primary: **block** at endpoint (MDE Network Protection) or SWG layer
- Secondary: **coach** with end-user message naming the sanctioned alternative + how to request an exception

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
| Netskope | `[unverified]` — Netskope CCI tagging + URL category block; native SWG enforcement available; pending lens review | | Native SWG path is structurally tighter than the MDA→MDE chain | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security tagging + Prisma Access URL-filtering block | | | |
| Skyhigh | `[unverified]` — Skyhigh CCI + multi-mode enforcement | | | |
| Zscaler | `[unverified]` — ZIA URL filtering + Cloud App Control | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT cloud services + data leakage](../../06-compliance/malaysia/bnm-rmit.md) — `[VERIFY]` against current edition
- ISO 27017 control(s): [CLD.12.4.5 monitoring of cloud services](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions), `DE.CM-09` (computing hardware and software monitored) `[VERIFY]`

## False-positive risk

- Approved-by-business AI vendor with `SOC 2 = No` in Microsoft's opaque catalogue scoring — false-positive block on legitimately-sanctioned vendors. Mitigate with documented allowlist by app ID.
- CDN-fronted domains shared across legitimate and unsanctioned apps — block-script granularity too coarse.
- New SaaS with no catalogue entry yet — classified Unknown, may be blocked while in legitimate trial.

## Operational cost

- **Exception-handling load:** medium — every new business-approved SaaS needs allowlist update; weekly catalogue review during rollout, monthly steady-state.
- **Triage load:** low — block actions are silent at endpoint; no per-event SOC triage required.
- **End-user friction:** medium-to-high — users who relied on consumer-tier alternatives need to find sanctioned replacements; coaching message + alternative-app reference critical.

## Privacy / data-protection considerations

- Discovery feed includes user identity (UPN) tied to app access patterns — treat as workforce monitoring; document under PDPA MY 2024 / GDPR Art. 88 workforce-monitoring posture.
- Block at endpoint does not require SSL inspection; lower privacy footprint than inline DLP.

## Coverage gaps

- [BYOD and unmanaged endpoint coverage gap](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — personal devices without MDE/MDM are uncovered.
- [Over-blocking and user circumvention](../../08-failure-modes/over-blocking-and-user-circumvention.md) — block-without-coach drives users to personal devices / personal accounts; the move-to-the-shadow class.
- Personal-account sign-in to consumer SaaS on corp device — Entra and CASB never see it.

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
