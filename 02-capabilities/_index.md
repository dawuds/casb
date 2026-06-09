# 02-capabilities

The **cross-vendor capability comparison.** One canonical artefact: [`capability-matrix.md`](capability-matrix.md).

## What this folder answers

> *"All five vendors claim they do DLP / OAuth governance / GenAI control. What does that claim actually look like per vendor, with what deployment-mode caveat, and where are the cells the vendor would rather you did not look?"*

If you are instead asking *"I have decided to enforce X — what does that policy look like across vendors"*, see [`../03-policies/`](../03-policies/) — that is the policy library.

- **Capability** = the tool *can* do this (vendor property; this folder).
- **Policy** = the firm has *decided to enforce* this (configuration decision; `03-policies/`).

The capability matrix tells you what to expect in an RFP demo. The policy library tells you what a deployment of that capability looks like in a real firm.

## How to read the matrix

- Cells: **S** Supported · **C** Caveat (short note) · **N** Not supported · **U** Unverified (vendor doc did not confirm).
- `[unverified]` prefix = the lens reviewers flagged this cell or its source claim as unreliable / vendor-marketing-grade in the underlying draft.
- Vendor codes: **MDA** Microsoft Defender for Cloud Apps · **NSK** Netskope One CASB · **PAN** Palo Alto Prisma Access (SaaS Security) · **SKY** Skyhigh CASB · **ZS** Zscaler ZIA CASB.

The most useful read is the "What the matrix reveals" synthesis section at the bottom of [`capability-matrix.md`](capability-matrix.md). It surfaces the cross-vendor patterns (convergence below the floor, divergence in deployment-mode posture, dead capabilities, asymmetric depth) that the cell-by-cell view hides.

## Status

The matrix is the canonical artefact for this folder. Per-capability deep-dive files (one file per capability) are **not** built at v0.0 — the matrix is the synthesis and per-capability fragmentation is deferred to v0.2+ if/when cross-linking demand emerges from policy or vendor pages.
