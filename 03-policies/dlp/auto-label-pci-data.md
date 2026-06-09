# Auto-label PCI data

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Required capabilities: [API-mode DLP + sensitivity-labelling integration](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: API connector + Purview (or equivalent labelling platform).
> MDA playbook reference: [Policy 6](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Auto-label PCI files in OneDrive/SharePoint).

## Purpose

Continuously scan sanctioned SaaS storage for files containing PCI cardholder data; auto-apply a Confidential-Finance sensitivity label that carries encryption + access restrictions via the labelling platform. Reduces cardholder-data sprawl. Precondition for downstream encryption controls and the inline download-block policy.

## Action

- Primary: **apply sensitivity label** with encryption + permissions on detection
- Phase-1 alternative: **alert-only** for one week to baseline FP rate before flipping to label-apply

## Scope

- **Users:** all OneDrive / SharePoint owners
- **Apps:** OneDrive for Business + SharePoint Online (primary); Google Drive where Workspace + Google labels are integrated
- **Device posture:** N/A (at-rest scan)
- **Network position:** N/A
- **Traffic:** at-rest

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Information protection → Create File policy. Plus Purview label configured at Purview portal → Information Protection → Labels | App filter = OneDrive + SharePoint; Parent folder = scoped to finance / cardholder-data folders on rollout; Inspection method = Data Classification Service; SIT = Credit Card Number; Minimum violations = 5; Governance action = Apply Microsoft Purview sensitivity label → Confidential — Finance; Alert + email per match | API-mode — post-upload labelling, not pre-upload prevention. Label-encryption does NOT propagate to existing co-author sessions until they reopen | The 100-character matching-content preview is itself cardholder data — lives in Defender's incident metadata, subject to PCI DSS 3.4 / 3.5 storage controls. Label silently fails if Purview label is not published to affected users OR if DCS is not onboarded to the SharePoint sites. Validate end-to-end with a known-positive test file before broad rollout |
| Netskope | `[unverified]` — Netskope CASB API + native labelling integration | | | |
| Palo Alto Prisma Access | `[unverified]` — SaaS Security with sensitivity-label-apply action | | | |
| Skyhigh | `[unverified]` — Skyhigh CASB with labelling | | | |
| Zscaler | `[unverified]` — Zscaler SaaS Security with labelling integration | | | |

## Control mappings

- PCI DSS Req. 3 (protect cardholder data); Req. 4 (encryption); Req. 7 (need-to-know) — **direct mapping**
- BNM RMiT clause(s): [BNM RMiT data classification + DLP](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 monitoring](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): A.10 (use limitation), A.12 (information security)
- NIST CSF 2.0 subcategory(ies): `PR.DS-01` (data-at-rest), `PR.DS-05` (data leak protection) `[VERIFY]`

## False-positive risk

- DCS hits on test cards (4111-1111-1111-1111 family), mock data, expired/cancelled cards
- Excel spreadsheets with numeric columns matching credit-card format (employee IDs, order numbers)
- PDF receipts where the cardholder number is partially masked

## Operational cost

- **Exception-handling load:** medium during week-1 alert-only baseline; lower steady-state once tuning closes FP gap
- **Triage load:** high during baseline; medium steady-state
- **End-user friction:** initially high — files become encrypted, downstream recipients may not be able to open if outside the label's access scope. Communicate label semantics upfront

## Privacy / data-protection considerations

- The matching-content snippet contains PCI cardholder data — itself regulated content under PCI DSS 3.4 / 3.5
- Document where the snippet is stored, who can access it, retention. The auditor will probe
- For OneDrive personal-area folders, some tenants treat OneDrive as personal scope — content inspection has PDPA / GDPR implications

## Coverage gaps

- Image-format card numbers (photo of a card, scan of a receipt) — DCS does not OCR at scan layer
- Files larger than DCS content-inspection limit — unscannable
- [API mode is not prevention](../../08-failure-modes/api-mode-is-not-prevention.md) — file existed in tenant before labelling completes
- File-policy ceiling = 200 per tenant; cardholder-data scope often consumes 3-5 policies (per app × per label family)

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
