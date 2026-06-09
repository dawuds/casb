# Sanctioned-tenant pinning for GenAI

> Status: v0.0 — cross-vendor pattern; MDA implementation via tenant-restriction headers + CAAC; other vendors `[unverified]`.
> Required capabilities: [Risk-based session policy + GenAI app governance + tenant-restriction header injection](../../02-capabilities/capability-matrix.md).
> Deployment-mode requirement: Reverse-proxy (CAAC) + SWG header injection. Hybrid.
> MDA playbook reference: companion to [Policy 9](../../04-vendors/microsoft-defender-for-cloud-apps.md) (clipboard paste block).

## Purpose

For sanctioned GenAI tools (ChatGPT Enterprise, Copilot, Gemini for Workspace) — pin sessions to the corporate workspace and block consumer-tenant or personal-account variants of the same SaaS family. Reduces the "user signs into ChatGPT with personal Gmail on a corporate device → exfiltrates to personal account" pattern. Counters MITRE ATT&CK `T1078.004 Cloud Accounts` (personal-account variant) and the personal-tenant variant of `T1567`.

## Action

- Primary: **block** sign-in to personal-tenant variant of sanctioned GenAI from corporate network or managed device
- Companion: **monitor** + alert on attempted personal-account sign-in

## Scope

- **Users:** all employees + contractors
- **Apps:** ChatGPT (Enterprise vs personal Plus / Free), Microsoft Copilot (work vs consumer Copilot), Gemini (Workspace vs personal), Anthropic Claude (claude.ai Enterprise vs consumer)
- **Device posture:** managed primarily; BYOD via MAM where supported
- **Network position:** corporate + roaming via endpoint agent
- **Traffic:** sign-in flow

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | Tenant-restriction headers (`Restrict-Access-To-Tenants`, `Restrict-Access-Context`) injected by SWG/proxy. MDA itself does not inject — requires GSA or third-party SWG. Plus CAAC session policy on the sanctioned variant | Header values = allowed tenant IDs; CAAC policy onboards the sanctioned variant; Entra CA blocks sign-in from non-listed tenants | Tenant-restriction header support is per-vendor on the AI side — ChatGPT supports tenant pinning for Enterprise/Team; consumer ChatGPT does not. Verify per-vendor | Personal account on personal device on personal network = entirely uncovered. AI vendors without tenant-restriction-header support fall back to SWG URL filtering, which is coarser |
| Netskope | `[unverified]` — Netskope SWG with header injection + CASB integration | | | |
| Palo Alto Prisma Access | `[unverified]` — Prisma Access SWG | | | |
| Skyhigh | `[unverified]` — Skyhigh multi-mode | | | |
| Zscaler | `[unverified]` — ZIA URL filtering + header injection | | | |

## Control mappings

- BNM RMiT clause(s): [BNM RMiT IAM + DLP + AI governance overlay](../../06-compliance/malaysia/bnm-rmit.md) `[VERIFY]`
- ISO 27017 control(s): [CLD.9.x access control overlay](../../06-compliance/iso-27017.md) `[VERIFY]`
- NIST CSF 2.0 subcategory(ies): `PR.AA-05` (access permissions), `GV.SC` (supply chain) `[VERIFY]`

## False-positive risk

- Legitimate cross-tenant collaboration with partner AI workspaces — name an exception group
- Users with both personal-and-business Microsoft accounts on the same browser session — confusion at sign-in

## Operational cost

- **Exception-handling load:** low — sanctioned-tenant list is small and stable
- **Triage load:** low — block is silent at IdP / SWG layer
- **End-user friction:** medium — users who routinely sign into personal AI accounts at work will see blocks

## Privacy / data-protection considerations

- Block does not inspect personal traffic content; inspects only sign-in tenant claim
- Lower privacy footprint than SSL inspection

## Coverage gaps

- Off-network personal-device personal-account = entirely uncovered
- AI vendors without tenant-restriction-header support — only SWG URL filtering available (coarser, doesn't distinguish tenants on shared domains)
- Mobile native AI apps with own auth flows — MAM compensating

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
