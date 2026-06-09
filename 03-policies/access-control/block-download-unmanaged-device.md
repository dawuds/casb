# Block download to unmanaged device

> Status: v0.0 — MDA column lens-reviewed from playbook v1; other vendors `[unverified]`.
> Required capabilities: [Inline DLP for sanctioned SaaS + Risk-based session policy](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC) or forward-proxy. Entra ID P1 required for the MDA path.
> MDA playbook reference: [Policy 5](../../04-vendors/microsoft-defender-for-cloud-apps.md) (Block download of Confidential-labelled files to unmanaged browsers).

## Purpose

Prevent download of files labelled Confidential or higher from sanctioned SaaS to devices that are not Entra-hybrid-joined or Intune-compliant (BYOD / personal). Reduces sensitive-document exfiltration via personal device. Counters MITRE ATT&CK `T1530 Data from Cloud Storage Object` via untrusted endpoint (Prevent for the download leg only).

## Action

- Primary: **block** download
- Alternative (BYOD soft-mode): **allow view + edit in-session**, block download + copy + paste + print + share

## Scope

- **Users:** all employees (excluding documented break-glass)
- **Apps:** Microsoft 365 bundle (SharePoint, OneDrive, Exchange Online, Teams host app); extend to other CAAC-onboarded sanctioned SaaS as onboarding completes
- **Device posture:** **unmanaged** (Device tag ≠ MicrosoftEntraHybridJoined AND ≠ IntuneCompliant)
- **Network position:** any (browser session)
- **Traffic:** download leg only

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Defender portal → Cloud apps → Policies → Policy management → Conditional Access → Create Session policy. **Plus** Entra CA policy with "Use Conditional Access App Control" session control on same scope | Session control type = Control file download (with inspection); App = Office 365 bundle (not individual apps); Client app = Browser; Device filter = Device tag ≠ hybrid AND ≠ compliant; File filter = Sensitivity label ∈ {Confidential, Highly Confidential}; Action = Block; `Always apply if data cannot be scanned` = false | Reverse-proxy CAAC only; browser sessions only. Native desktop / mobile apps bypass. Requires Entra ID P1 | **Bilateral CAAC trap** — needs CA policy AND session policy (two consoles, two objects). Inverse trap: CA policy without MDA policy = silent no-op proxy redirect. `*.mcas.ms` URL rewrite breaks `window.parent.postMessage` apps (Salesforce Lightning, ServiceNow Polaris, Workday Extend) — regression test before scoping broadly |
| Netskope | `[unverified]` — Netskope CASB Inline supports per-app session policy with device-classification | | Native forward-proxy + reverse-proxy; broader app support than MDA's CAAC | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access SaaS Security inline | | | |
| Skyhigh | `[unverified]` — Skyhigh reverse-proxy with device-classification | | | |
| Zscaler | `[unverified]` — ZIA inline CASB with device posture check via ZCC | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT data leakage prevention](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.12.4.5 + ISO 27002 A.5.18 access rights](../../06-compliance/iso-27017.md) `[VERIFY]`
- ISO 27018 control(s): [A.10 (use, retention and disclosure limitation)](../../06-compliance/iso-27018.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05`, `PR.DS-01` (data-at-rest) `[VERIFY]`
- PCI DSS Req. 7 (restrict access by need-to-know) where PCI data in scope

## False-positive risk

- Finance user on BYO laptop urgently needing a quarterly report — blocked. Mitigate by scoping initial rollout to pilot group; ensure exception path documented
- Senior exec on iPad-only setup — blocked; common cause of week-one rollback. Pair with read-only session policy
- Device tag empty due to CA evaluation not running on session-token-issuance branch — fails open or closed depending on `Always apply` setting

## Operational cost

- **Exception-handling load:** medium initially (week 1-2); declining as enrolment campaigns close gaps
- **Triage load:** low — most events are silent blocks
- **End-user friction:** **high** for BYOD population unless paired with read-only session policy fallback

## Privacy / data-protection considerations

- CAAC reverse-proxy of employee browser sessions = high-risk processing under GDPR Art. 35; DPIA mandatory in EU/UK/equivalent regimes
- Audit log of the block decision lives in `CloudAppEvents` — itself regulated content (contains user identity, file metadata, decision verdict)
- PDPA MY 2024 / HKPCPD workforce-monitoring notice required

## Coverage gaps

- [BYOD and unmanaged endpoint coverage gap](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — native mobile apps bypass entirely
- [SSL/TLS inspection breakage](../../08-failure-modes/ssl-tls-inspection-breakage.md) — cert-pinned managed-device profiles may reject `*.mcas.ms` chain, fail closed at TLS but appear fail-open at app layer
- T1550.001 Application Access Token replay — bypasses device-tag filter entirely (token issued legitimately, replayed elsewhere). Pair with Entra Token Protection
- T1213 in-session screenshot — no technical control; visible-marking labels deter only

## Three-lens sign-off

- **Architect:** _pending — to incorporate findings from MDA lens reviewers_
- **Product:** _pending_
- **Compliance:** _pending_
