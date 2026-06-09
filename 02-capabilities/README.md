# 02-capabilities

The **cross-vendor capability comparison.** One canonical artefact.

## What this folder answers

> *"All five vendors claim they do DLP / OAuth governance / GenAI control. What does that claim actually look like per vendor, with what deployment-mode caveat, and where are the cells the vendor would rather you did not look?"*

If you are instead asking *"I have decided to enforce X — what does that policy look like across vendors"*, see [`../03-policies/`](../03-policies/) — that is the policy library.

- **Capability** = the tool *can* do this (vendor property; this folder).
- **Policy** = the firm has *decided to enforce* this (configuration decision; `03-policies/`).

The capability matrix tells you what to expect in an RFP demo. The policy library tells you what a deployment of that capability looks like in a real firm.

## Contents

| File | What it contains |
|---|---|
| [`capability-matrix.md`](capability-matrix.md) | The canonical cross-vendor capability matrix. Rows = capabilities (Shadow IT discovery, OAuth governance, inline DLP, etc.). Columns = the 5 vendors (MDA / NSK / PAN / SKY / ZS). Cells = Supported / Caveat / Not supported / Unverified, with caveat notes. Synthesis section at the end surfaces cross-vendor patterns |

## How to read the matrix

- Cells: **S** Supported · **C** Caveat (short note) · **N** Not supported · **U** Unverified (vendor doc did not confirm)
- `[unverified]` prefix = lens reviewers flagged this cell or its source claim as unreliable / vendor-marketing-grade
- Vendor codes: **MDA** Microsoft Defender for Cloud Apps · **NSK** Netskope One CASB · **PAN** Palo Alto Prisma Access (SaaS Security) · **SKY** Skyhigh CASB · **ZS** Zscaler ZIA CASB

The most useful read is the **"What the matrix reveals"** synthesis section at the bottom of [`capability-matrix.md`](capability-matrix.md). It surfaces cross-vendor convergence, divergence in deployment-mode posture, dead capabilities, and asymmetric depth — patterns the cell-by-cell view hides.

## Status

At v0.0, the matrix is the canonical artefact for this folder. **Per-capability deep-dive files (one file per capability) are not built** — the matrix is the synthesis and per-capability fragmentation is deferred to v0.2+ if/when cross-linking demand emerges.

## Cross-references

- [`../03-policies/`](../03-policies/) — the policy library (one file per policy, with vendor-implementation grid)
- [`../04-vendors/`](../04-vendors/) — per-vendor playbooks (deep configuration walkthrough)
- [`../README.md`](../README.md) — repo-level navigation
