# 04-vendors

Per-vendor playbooks. The deep-configuration walkthrough for each CASB / SSE product in scope.

## Contents

| File | Depth | What it contains |
|---|---|---|
| [`microsoft-defender-for-cloud-apps.md`](microsoft-defender-for-cloud-apps.md) | **v1 lens-reviewed** | The gold-standard playbook. Day 0 preconditions + Gaps-and-compensating-controls table + 13 numbered policies (Day 1 / 30 / 90) + 7 appendices (SCIM cascade, slow-and-low KQL, CAAC deprovisioning runbook, DR / fail-mode matrix, ATT&CK + ATLAS coverage, threat-actor profiles, glossary). ~620 lines |
| [`netskope.md`](netskope.md) | Draft | Vendor capability set, deployment modes, configurable policies, limitations. Not yet at MDA depth |
| [`zscaler.md`](zscaler.md) | Draft | Same shape; not yet at MDA depth |
| [`palo-alto-prisma-access.md`](palo-alto-prisma-access.md) | Draft | Same shape; not yet at MDA depth |
| [`skyhigh-security.md`](skyhigh-security.md) | Draft | Same shape; not yet at MDA depth |

## Reading order

1. If you have one specific vendor in mind, go directly to that file.
2. If you're new to CASB, start with [`microsoft-defender-for-cloud-apps.md`](microsoft-defender-for-cloud-apps.md) — it is the most thoroughly worked example, and the patterns (CAAC bilateral trap, API-vs-inline split, DSPM-for-AI prompt-capture depth, concentration risk, ATT&CK coverage matrix) apply across vendors with vendor-specific implementation detail.
3. For cross-vendor comparison see [`../02-capabilities/capability-matrix.md`](../02-capabilities/capability-matrix.md) — that is the canonical cross-vendor view.

## What each vendor file contains (canonical structure)

Following the MDA v1 shape:

- **Status + read-this-if / skip-this-if**
- **One-page summary card** — licensing, dependencies, key traps, policy index
- **Day 0 precondition checklist** — what must be true before any policy ships
- **What MDA / vendor is for** — neutral framing in 4 capability sets
- **Gaps and compensating controls** — single canonical "what this vendor cannot do" table
- **Concentration risk** — top-of-stack callout
- **Day 1 / 30 / 90 policy detail** — each policy with: Risk reduced | What it does | ATT&CK | Critical prerequisite | Console path | Key configuration | Trap to avoid | Defeated by | Auditable evidence | SIEM artefact + connector
- **Appendices** — SCIM cascade, slow-and-low KQL, deprovisioning runbook, DR matrix, ATT&CK matrix, threat-actor profiles, glossary

Only the MDA file has all of the above filled in at v0.0. Other vendors will catch up in v0.2.

## Status caveat

**The 5-vendor scope is locked** (per [`../CLAUDE.md`](../CLAUDE.md)). Cisco Cloudlock / Forcepoint ONE / Lookout / iboss are added only if a specific wedge demands. Same for Symantec / Broadcom — covered conceptually in [`../01-foundations/market-motion.md`](../01-foundations/market-motion.md) but not as standalone vendor files.

## Vendor lineage / renaming watch-list

| Lineage | Current 2026 name |
|---|---|
| Adallom → MCAS | Microsoft Defender for Cloud Apps |
| Skyhigh Networks → McAfee MVISION Cloud → Trellix CASB | Skyhigh Security (separate company spun out 2022) |
| CirroSecure → Aperture | Palo Alto SaaS Security (inside Prisma Access) |
| Symantec CloudSOC → Broadcom Symantec Cloud SWG | (CASB folded into SWG; not in current scope) |
| Bitglass → Forcepoint ONE | (not in current scope) |
| Oracle CASB | Discontinued |
| CipherCloud → Lookout | (not in current scope) |

## Cross-references

- [`../02-capabilities/capability-matrix.md`](../02-capabilities/capability-matrix.md) — cross-vendor capability comparison
- [`../03-policies/`](../03-policies/) — vendor-neutral policy library (each per-vendor file's policies feed the policy-library vendor-implementation grids)
- [`../05-architecture/`](../05-architecture/) — reference architectures
- [`../README.md`](../README.md) — repo-level navigation
