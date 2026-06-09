# Restructure specification

Scope: navigation + `02-capabilities/` + `03-policies/` only. Canonical content in `04-vendors/microsoft-defender-for-cloud-apps.md`, `07-implementation/`, `06-compliance/`, `08-failure-modes/`, and `CLAUDE.md` is preserved.

## Diagnosis summary

- **[Consensus, all 4 reviewers]** `02-capabilities/_capability-matrix.md` and `03-policies/_policy-archetypes.md` are the same cross-vendor data cut twice. Both have vendors as columns, capabilities/archetypes as rows, the same `[unverified]` flags, the same source drafts. Neither is a policy library; both are matrices.
- **[Consensus, all 4 reviewers]** The README "How to navigate" table fails as a scanning surface — mixed axis of organisation (reader goal / content type / cross-cutting view / failure framing all in one column), heterogeneous granularity in the destination column (folder vs single file vs 600-line page), 9-11 rows where a first-time reader sees no row that says "start here".
- **[Consensus, 3 of 4 reviewers — ms-casb-implementor, info-architect, first-time-reader]** `03-policies/_policy-archetypes.md` overpromises: it is named "policy" but contains zero policy entries matching the schema declared in `03-policies/_index.md`. The schema fields (Scope / Action / Required capabilities / Control mappings / False-positive risk / Operational cost / Privacy considerations) are declared and never populated. The only real policy library in the repo today is inside `04-vendors/microsoft-defender-for-cloud-apps.md` (13 policies, Day 1 / 30 / 90).
- **[Consensus, all 4 reviewers]** The MDA vendor file is the gold-standard artefact in the repo and is underweighted in the README. It is row 4 of 11 in the navigation table; its Day 1 / 30 / 90 cadence and policy schema (Risk reduced → What it does → ATT&CK → Prereq → Console path → Key config → Trap → Defeated by → Evidence → SIEM artefact) is the right shape for a practitioner reference. Other four vendor files are explicitly draft-quality per the promotion log — the README understates this gap.
- **[Consensus, 2 of 4 — info-architect, first-time-reader]** The thesis sentence at README line 5 is an author-research artefact, not a reader artefact. It belongs in `00-meta/thesis.md` (where it is already linked), not in the second paragraph of the README.

## Resolved decisions

| Decision | Surfaced by | Resolution |
|---|---|---|
| **02/03 split: keep, merge, or reframe?** | ms-casb-implementor (delete 03), info-architect (rename to 03-policy-patterns), first-time-reader (keep+reframe), multi-vendor-practitioner (replace with proper policy library) | **Keep both folders, with hard-edged purpose split**: 02 is cross-vendor capability comparison (one matrix file, canonical), 03 is the vendor-neutral policy library (one file per policy, schema-driven, sub-foldered by control family). The current `_policy-archetypes.md` is downgraded from canonical to a research artefact and renamed/moved. Rationale: the locked thesis ("what policies can a practitioner actually configure") names "policies" as the unit of output. Deleting `03-policies/` (ms-casb-implementor Option A) surrenders the locked deliverable. Renaming (info-architect) is cosmetic. The multi-vendor-practitioner is correct that the right shape is a policy library with vendor-implementation grids. |
| **Policy = what the firm decides, or = the configurable thing in a console?** | ms-casb-implementor flags the word "policy" carries two meanings on adjacent README lines | **Policy = the configurable thing.** The capability/policy fence in `CLAUDE.md` line 109-110 ("capability = the tool can…; policy = the firm has decided to…") is preserved as editorial discipline, but the unit of output in `03-policies/` is the *configurable, vendor-neutral policy spec with a per-vendor implementation grid* (MDA Policy 5 shape, abstracted across vendors). The "archetype" framing from `_policy-archetypes.md` is dropped — archetypes were policy families, not policies. |
| **Day 1 / 30 / 90 axis in 03-policies/?** | multi-vendor-practitioner argues no (cadence is per-firm, not per-policy) | **No.** Day 1/30/90 is a vendor-specific rollout cadence and stays inside `04-vendors/<vendor>.md` (where it already is) and in `07-implementation/phased-rollout.md` (vendor-agnostic). `03-policies/` is organised by control family (access / DLP / detect / OAuth / GenAI / posture). The README signposts the cadence as a vendor-file property. |
| **Per-capability files in 02-capabilities/?** | ms-casb-implementor says no (drop the 14-entry plan; matrix is the canonical artefact). multi-vendor-practitioner says yes (split the matrix). info-architect says only if there is content that does not belong in the matrix. first-time-reader says no. | **No** at v0.1. The matrix is the canonical synthesis. The 14-entry per-capability plan in `_index.md` lines 22-37 is removed. Per-capability files are a v0.2+ deliverable if cross-linking demand grows; not built now. Reviewer majority (3 of 4) plus promotion-log discipline ("no speculative files") agree. |
| **README pattern: file-led vs reader-led?** | all 4 reviewers | **Reader-led.** Replace the 9-row "How to navigate" table with a 3-row "Start here" table keyed off reader state (new-to-CASB / running MDA / planning a deployment). Existing 9-row table is folded into a single "Folder map" block below. |
| **Thesis sentence in README front matter?** | info-architect + first-time-reader | **Moved.** Out of README line 5; into `00-meta/thesis.md` (already linked) and surfaced only in CLAUDE.md / promotion log. README opens with a 2-sentence "what this is for the reader" paragraph. |
| **Promote MDA file visibility in README?** | all 4 reviewers | **Yes.** MDA file is the second row of the "Start here" table, not the fourth. Its Day 1 / 30 / 90 shape is named in the row. The README states the depth asymmetry explicitly (MDA = lens-reviewed v1; other 4 = drafts at ~10% of MDA depth). |
| **Capability/policy framing visible to public reader?** | info-architect + first-time-reader | **Yes.** Move the cleanest framing (capability = the tool *can*…; policy = the firm has *decided to*…) from `CLAUDE.md` to the public `_index.md` of both folders, phrased as a navigation principle the reader needs ("if you are asking what the tool can do, go to 02; if you are asking what a policy looks like, go to 03"). |
| **Component model duplication?** | info-architect + first-time-reader | **Collapsed.** The "How to navigate" table and the "Component model" code-block describe the same folder list. Keep one block ("Folder map"); drop the duplicate. |
| **Where do per-vendor implementation details live: in 03-policies/ or 04-vendors/ or both?** | multi-vendor-practitioner | **Both, with cross-links.** `03-policies/<family>/<policy>.md` carries the vendor-neutral spec and a 5-row vendor-implementation summary grid (console path, key config, deployment-mode caveat, known trap per vendor — one cell each). `04-vendors/<vendor>.md` carries the full configuration walkthrough (the existing MDA shape). Grid links cell-to-section both ways. |

## New README

The following is the full proposed README content. Paste verbatim into `/Users/dawud/claude/casb/README.md`.

````markdown
# casb

Practitioner reference for **Cloud Access Security Broker (CASB)** controls in 2026 — capability set, configurable policies, vendor coverage, regulator mapping, and where the tooling demonstrably fails. Anchored Malaysia (BNM RMiT) with MAS / HKMA / EU / ISO as comparators.

**Status:** v0.0 — canonical content populated 2026-06-10. **Microsoft Defender for Cloud Apps is the only vendor at v1 lens-reviewed depth**; Netskope / Zscaler / Palo Alto Prisma Access / Skyhigh are draft-quality only. See [`00-meta/promotion-log.md`](00-meta/promotion-log.md). Outstanding items before v0.1 listed there.

## Start here

Three reading paths. Pick yours.

| If you are | Read in this order |
|---|---|
| **New to CASB** | [`01-foundations/what-casb-is-and-isnt.md`](01-foundations/what-casb-is-and-isnt.md) → [`01-foundations/deployment-modes.md`](01-foundations/deployment-modes.md) → [`07-implementation/programme-readiness.md`](07-implementation/programme-readiness.md) |
| **Already running Microsoft Defender for Cloud Apps** and need to know what to switch on | [`04-vendors/microsoft-defender-for-cloud-apps.md`](04-vendors/microsoft-defender-for-cloud-apps.md) — Day 0 preconditions → Gaps table → 13 numbered policies (Day 1 / 30 / 90), each with console path, trap, defeated-by, audit evidence, SIEM artefact |
| **Planning a CASB programme** (any vendor) | [`07-implementation/programme-readiness.md`](07-implementation/programme-readiness.md) → [`07-implementation/phased-rollout.md`](07-implementation/phased-rollout.md) → [`04-vendors/<your-vendor>.md`](04-vendors/) → [`08-failure-modes/`](08-failure-modes/) |

Everything else is reference; see "Folder map" below.

## How the content is split

- **Capability** = what the tool *can* do. Lives in [`02-capabilities/`](02-capabilities/) (one cross-vendor matrix).
- **Policy** = what the firm has *decided to enforce*, expressed as a configurable, vendor-neutral spec with a per-vendor implementation grid. Lives in [`03-policies/`](03-policies/) (one file per policy, sub-foldered by control family).
- **Vendor playbook** = the full per-vendor configuration walkthrough (Day 1 / 30 / 90 rollout cadence, console paths, traps). Lives in [`04-vendors/<vendor>.md`](04-vendors/).

The per-vendor file is canonical for "how do I configure this". The policy file is canonical for "what does this policy look like across vendors". The capability matrix is canonical for "which vendors can do this at all".

## Folder map

| Folder | One-line role |
|---|---|
| [`00-meta/`](00-meta/) | Thesis lock, promotion log, three-lens review template, glossary |
| [`01-foundations/`](01-foundations/) | What CASB is and isn't; deployment modes; where it sits in SSE; market motion |
| [`02-capabilities/`](02-capabilities/) | Cross-vendor capability matrix — what each tool can do, cell-by-cell with caveats |
| [`03-policies/`](03-policies/) | Vendor-neutral policy library — one file per policy, with a 5-row vendor-implementation grid |
| [`04-vendors/`](04-vendors/) | Per-vendor playbooks — MDA at v1 lens-reviewed depth; Netskope / Zscaler / Palo Alto / Skyhigh as drafts |
| [`05-architecture/`](05-architecture/) | Reference architectures — proxy / API / hybrid; SSE stack; BYOD; GenAI overlay |
| [`06-compliance/`](06-compliance/) | Control mapping framework + BNM RMiT / ISO 27017 / ISO 27018 framework pages |
| [`07-implementation/`](07-implementation/) | Programme readiness, phased rollout, SOC integration, metrics, change management, cost of ownership |
| [`08-failure-modes/`](08-failure-modes/) | Where CASB demonstrably fails — BYOD, OAuth, SSL/TLS, API-mode, encrypted upload, over-blocking, lock-in, cost |
| [`_research/`](_research/) | Working notes, multi-agent workflow outputs, lens reviews — not citation-gated |
| [`_sources/`](_sources/) | Primary-source library (in development) |

## What this repo does NOT cover

- **Vendor evaluation / RFP scoring** — the per-vendor files are practitioner references, not buying guides
- **Cross-vendor consulting deliverables** — the syntheses are draft-quality, not productized
- **Full regulator clause mapping** — `06-compliance/` has framework pages; clause-by-clause mapping is a separate deliverable
- **Third-party attestation citations** — pulls from Trust Center / Service Trust Portal still required before treating any vendor claim as audit-evidenced
- **DPIA / TIA documents** — CAAC reverse-proxy and DSPM-for-AI prompt capture both require formal data-protection impact assessments before pilot

## Caveats

- **Material on regulatory clauses is illustrative; not legal/regulatory advice.** Confirm against gazetted text and your assurance lead.
- **Vendor capability claims are vendor-controlled.** Cited from vendor documentation; corroborate before treating as audit evidence.
- **The 2026 CASB / SSE landscape is moving fast.** GenAI-related capabilities especially are shifting quarterly. Verify the current-quarter state before relying on a 2026-H1 capability claim in production.

## For collaborators

See [`CLAUDE.md`](CLAUDE.md) for citation discipline, sanitisation rules, file-creation discipline, the three-lens review pattern, and the workflow. The thesis lock and rationale are at [`00-meta/thesis.md`](00-meta/thesis.md). The promotion log at [`00-meta/promotion-log.md`](00-meta/promotion-log.md) records what was promoted from `_research/` and what is outstanding before v0.1.
````

## 03-policies/ restructure

### New `_index.md` content (full)

````markdown
# 03-policies

The **policy library.** One file per policy, schema-driven, sub-foldered by control family.

## What this folder answers

> *"I have decided to enforce X. What does that policy look like? Which capabilities does it depend on? Which vendor in my estate can do it, and where is the trap?"*

If you are instead asking *"what can the tool do"*, see [`../02-capabilities/`](../02-capabilities/) — that is the capability matrix. The two folders are deliberately distinct:

- **Capability** = the tool *can* do this (vendor property).
- **Policy** = the firm has *decided to enforce* this (configuration decision).

The capability matrix tells you what to expect in an RFP demo. The policy library tells you what decisions you have to make in a design workshop, and what each decision costs you per vendor.

## What a policy entry looks like

Each file in this folder is structured identically:

- **Name** — canonical, vendor-neutral.
- **Purpose** — one sentence; which named risk does this reduce.
- **Action** — block / coach / monitor / quarantine / encrypt / step-up / label / DLP-redact.
- **Scope** — which users (group / OU / role), which apps (sanctioned / tolerated / unsanctioned / specific app), which device posture (managed / unmanaged / BYOD), which network position, which traffic (upload / download / share / API call).
- **Required capabilities** — links to `02-capabilities/` matrix rows the policy depends on.
- **Deployment-mode requirement** — forward-proxy / reverse-proxy / API / hybrid. Collapses the candidate-vendor field.
- **Vendor implementation grid** — one row per vendor (MDA / Netskope / Palo Alto Prisma / Skyhigh / Zscaler): console path, key configuration values, deployment-mode caveat, known trap. Cells link to the corresponding section of `04-vendors/<vendor>.md` for the full configuration walkthrough.
- **Control mappings** — links to `06-compliance/` entries this policy generates evidence for (BNM RMiT clause, ISO 27017 control, NIST CSF subcategory).
- **False-positive risk** — named scenarios, not a rating.
- **Operational cost** — exception load, triage load, end-user friction.
- **Privacy / data-protection considerations** — SSL inspection on personal traffic? PII surfaced to admins? DPIA trigger?
- **Coverage gaps** — what this policy provably misses (mobile native, unmanaged endpoints, certificate-pinned apps, encrypted payloads). Cross-link to `08-failure-modes/`.
- **Three-lens sign-off** — Architect / Product / Compliance.

## Organisation

Files are sub-foldered by control family, not by Day 1/30/90 (cadence is a per-firm and per-vendor rollout property, not a policy property):

```
03-policies/
  _index.md
  _schema.md                                              <- this folder's schema (extracted from _index.md)
  _archetype-index.md                                     <- cross-vendor archetype index (was _policy-archetypes.md; downgraded from canonical to navigation aid)
  access-control/
    block-unsanctioned-app-with-coach.md
    tenant-restriction-corporate-only.md
    block-download-unmanaged-device.md
    geo-residency-block.md
  dlp/
    inline-upload-block-regulated-data.md
    api-data-at-rest-quarantine.md
    external-share-link-quarantine.md
    auto-label-pci-data.md
  detect/
    mass-download-alert.md
    impossible-travel-alert.md
    mass-delete-anomaly.md
    terminated-user-cross-saas.md
    b2b-partner-exfil-alert.md
  oauth/
    post-consent-cleanup-and-ban.md
    high-scope-grant-alert.md
  genai/
    discovery-and-auto-unsanction.md
    inline-prompt-dlp.md
    sanctioned-tenant-pinning.md
  posture/
    sspm-tenant-misconfig-drift.md
```

## Status

At v0.0, only the **MDA-tier policies** (those backed by the lens-reviewed MDA v1 playbook) have enough corroboration to populate a full vendor-implementation grid. Other vendor cells in the grid will carry `[unverified]` until those vendor playbooks reach MDA depth (per outstanding items in [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md)).

Until then, treat per-policy files as **partial** — the MDA column is reliable; the other four columns inherit the source-quality flags from the underlying vendor drafts.

The legacy cross-vendor archetype index ([`_archetype-index.md`](_archetype-index.md), formerly `_policy-archetypes.md`) is preserved as a navigation aid — it maps each archetype to the per-policy file(s) under it — but is no longer the canonical synthesis. The canonical synthesis is the per-policy file.
````

### Files to create

Initial pass: 19 policy files, mapped to MDA Day 1 / 30 / 90 cadence and the existing archetype list. Each carries the schema from `_index.md`.

| Path (from repo root) | One-line description | Tier | MDA playbook policy # |
|---|---|---|---|
| `03-policies/_schema.md` | Policy entry schema — extracted from `_index.md` for stable cross-referencing | n/a | n/a |
| `03-policies/_archetype-index.md` | **Move from** `_policy-archetypes.md` — cross-vendor archetype index, now a navigation aid pointing to per-policy files | n/a | n/a |
| `03-policies/access-control/block-unsanctioned-app-with-coach.md` | Block / coach high-risk Shadow-IT apps with redirect-to-sanctioned variant | Day 1 | MDA #7 (Auto-unsanction GenAI category — Day 90 in MDA; access-control generally Day 1) |
| `03-policies/access-control/tenant-restriction-corporate-only.md` | Allow only corporate M365/Workspace tenant; block personal/other-org tenants | Day 30 | (MDA implicit via CAAC; cross-vendor Archetype #3) |
| `03-policies/access-control/block-download-unmanaged-device.md` | Block file download from sanctioned SaaS to BYOD/unmanaged endpoints | Day 30 | MDA #1, #5 |
| `03-policies/access-control/geo-residency-block.md` | Block uploads to non-MY data regions for tagged regulated data | Day 90 | (cross-vendor; MDA via geo condition) |
| `03-policies/dlp/inline-upload-block-regulated-data.md` | Inline DLP — pre-upload block of PCI / PII / source / secrets | Day 30 | MDA #3 |
| `03-policies/dlp/api-data-at-rest-quarantine.md` | API-mode DLP — post-upload scan, quarantine to user-root tombstone | Day 30 | MDA #2 |
| `03-policies/dlp/external-share-link-quarantine.md` | Quarantine external/anonymous-link shares on sanctioned storage; two-stage user-notify | Day 30 | MDA #10 |
| `03-policies/dlp/auto-label-pci-data.md` | Auto-apply Purview Confidential-PCI label on detection in sanctioned SaaS | Day 30 | MDA #6 (Auto-label PCI) |
| `03-policies/detect/mass-download-alert.md` | UEBA alert on volumetric / off-hours mass download | Day 1 | MDA #4 |
| `03-policies/detect/impossible-travel-alert.md` | Compromised-account detection — two geo-distant sign-ins in implausible interval | Day 1 | MDA #9 |
| `03-policies/detect/mass-delete-anomaly.md` | Anomaly detection on bulk-delete operations against sanctioned storage | Day 30 | MDA (Day 30) |
| `03-policies/detect/terminated-user-cross-saas.md` | Cross-SaaS account activity check on every termination | Day 90 | MDA #10 (Terminated-user) |
| `03-policies/detect/b2b-partner-exfil-alert.md` | Alert on B2B-partner exfiltration patterns from sanctioned tenant | Day 30 | MDA (Day 30) |
| `03-policies/oauth/post-consent-cleanup-and-ban.md` | Day 1 OAuth grant inventory cleanup + ban malicious/misleading-publisher consents | Day 1 | MDA #2 (Day 1 OAuth) |
| `03-policies/oauth/high-scope-grant-alert.md` | Alert on high-permission, low-community-use OAuth grants | Day 30 | MDA App Governance Predictive Risk (Day 30) |
| `03-policies/genai/discovery-and-auto-unsanction.md` | GenAI app discovery + auto-unsanction propagated to endpoint enforcement | Day 90 | MDA #7 |
| `03-policies/genai/inline-prompt-dlp.md` | Inline prompt-content DLP for GenAI (block clipboard-paste of regulated data) | Day 90 | MDA #8 |
| `03-policies/genai/sanctioned-tenant-pinning.md` | Pin GenAI app sessions to corporate workspace (block consumer-tenant variants) | Day 90 | MDA #8 (paired) |
| `03-policies/posture/sspm-tenant-misconfig-drift.md` | SSPM — continuous assessment of connected SaaS tenant settings against posture-rule library | Day 90 | (cross-vendor; MDA App Governance posture) |

That is **21 new files to create** in `03-policies/` (1 schema + 1 renamed/moved index + 19 policy files).

### Schema for each new policy file

Each `03-policies/<family>/<policy>.md` file carries the following sections in order. The MDA column of the vendor-implementation grid is the only column that can be populated with reliable detail at v0.0; other vendor columns carry `[unverified]` until their drafts reach MDA depth.

```markdown
# <Policy name>

> Status: <v0.0 — MDA-tier corroborated only / draft / etc.>
> Required capabilities: link to `02-capabilities/_capability-matrix.md` row(s)
> Deployment-mode requirement: <forward-proxy / reverse-proxy / API / hybrid>

## Purpose
One sentence. Which named risk does this reduce. Cite ATT&CK technique where applicable.

## Action
<block / coach / monitor / quarantine / encrypt / step-up auth / require labelling / DLP-redact>

## Scope
- **Users:** which group / OU / role
- **Apps:** sanctioned / tolerated / unsanctioned / specific app
- **Device posture:** managed / unmanaged / BYOD
- **Network position:** on-prem / remote / mobile
- **Traffic:** upload / download / share / API call

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | <Defender portal → Cloud apps → Policies → ...> | <key fields> | <e.g. browser-only via CAAC; requires Entra ID P1> | <e.g. bilateral CAAC trap — needs CA policy + session policy> |
| Netskope | `[unverified]` until lens-reviewed | | | |
| Palo Alto Prisma Access | `[unverified]` | | | |
| Skyhigh | `[unverified]` | | | |
| Zscaler | `[unverified]` | | | |

Each cell links to the corresponding section of `04-vendors/<vendor>.md` for the full configuration walkthrough.

## Control mappings
- BNM RMiT clause(s): <link to `06-compliance/malaysia/bnm-rmit.md`>
- ISO 27017 control(s): <link to `06-compliance/iso-27017.md`>
- ISO 27018 control(s): <link to `06-compliance/iso-27018.md`>
- NIST CSF 2.0 subcategory(ies): <reference>

## False-positive risk
Named scenarios — not a rating. e.g. "Regex on PCI/SSN fires on test fixtures, employee IDs, order numbers". Cite source.

## Operational cost
- Exception-handling load: <named scenarios>
- Triage load: <alerts/day estimate where possible>
- End-user friction: <named UX consequences>

## Privacy / data-protection considerations
- SSL inspection on personal traffic? PDPA / GDPR implications.
- PII surfaced to admins? Access governance on the policy's evidence record.
- DPIA / TIA trigger?

## Coverage gaps
Cross-link to `08-failure-modes/<mode>.md` entries. What this policy demonstrably does not catch (mobile native, certificate-pinned apps, encrypted payloads, third-party-to-third-party API flows).

## Three-lens sign-off
- **Architect:** <findings + resolution>
- **Product:** <findings + resolution>
- **Compliance:** <findings + resolution>
```

Per-section content guidance:

- **Purpose** — one sentence, no marketing language. Cite the ATT&CK / ATLAS technique countered.
- **Vendor implementation grid** — MDA column is the one to populate fully at v0.0. Other columns carry `[unverified]` placeholders explicitly tied to the vendor draft's source-quality flags. Each cell ≤3 lines; the full walkthrough lives in `04-vendors/<vendor>.md`.
- **Control mappings** — at v0.0, the framework pages exist but the clause-by-clause mapping does not. Link to the framework page and add a `[VERIFY]` marker; the full clause map is deferred per `00-meta/promotion-log.md` outstanding items.
- **False-positive risk** — pulled from the MDA playbook's "Trap to avoid" and "Defeated by" sections where applicable.
- **Three-lens sign-off** — empty block at v0.0; populated when the policy passes the three-lens review per `CLAUDE.md`.

## 02-capabilities/ adjustment

### Rename

| From | To |
|---|---|
| `02-capabilities/_capability-matrix.md` | `02-capabilities/capability-matrix.md` (drop the underscore — the file is canonical, not scaffolding) |

### `_index.md` rewrite (full proposed content)

````markdown
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
````

### `_capability-matrix.md` → `capability-matrix.md` changes

- Promote the "What the matrix reveals" section (current lines 78-96) above the cell-by-cell tables. The synthesis is the read; the cells are the lookup. Today the synthesis is at the bottom, behind the wall of cells.
- Re-section by **control family** (access / DLP / detect / OAuth+posture / data-protection / GenAI / data-residency) rather than the current ad-hoc grouping ("Identity / OAuth / SSPM" crams three different things; "Data residency / control plane" mixes two). Cross-reference the section headings to the policy sub-folders in `03-policies/` so a reader can move from a matrix row to the policy library entry that depends on it.
- For each matrix row that has a corresponding policy file in `03-policies/`, add a "Used by policies" pointer column or end-of-row link.

## Cross-reference updates needed

Files that reference `02-capabilities/_capability-matrix.md`, `03-policies/_policy-archetypes.md`, or carry the capability/policy framing in a way that will drift after the restructure:

| File | What changes |
|---|---|
| `/Users/dawud/claude/casb/README.md` | Wholesale replacement per the "New README" section above |
| `/Users/dawud/claude/casb/CLAUDE.md` line 13 | Update "Synthesis pages (`02-capabilities/`, `03-policies/`, `08-failure-modes/`)" — clarify that `03-policies/` is the policy library, not a synthesis page |
| `/Users/dawud/claude/casb/CLAUDE.md` line 22 (component model) | Replace "Capability catalogue — one entry per capability" with "Cross-vendor capability matrix (one canonical file)" |
| `/Users/dawud/claude/casb/CLAUDE.md` line 25 (component model) | Replace "Policy catalogue — one entry per policy archetype" with "Policy library — one file per policy, sub-foldered by control family (access / DLP / detect / OAuth / GenAI / posture). Each carries a vendor-neutral spec and a 5-row vendor-implementation grid linking to `04-vendors/<vendor>.md`." |
| `/Users/dawud/claude/casb/CLAUDE.md` line 45 ("Capability vs Policy separation" locked decision) | No change to the rule itself. The discipline holds. Update the "Strict — capabilities in `02-`, policies in `03-`, never inline together" wording with a clarifying sentence: "The split is also surfaced as a public navigation principle in `02-capabilities/_index.md` and `03-policies/_index.md`." |
| `/Users/dawud/claude/casb/CLAUDE.md` line 78 (style — schemas) | Already says "Each policy entry follows `03-policies/_schema.md`." Now true once `_schema.md` is created. No change needed beyond ensuring the file exists. |
| `/Users/dawud/claude/casb/00-meta/promotion-log.md` | Add a new section "2026-06-1X navigation restructure event" recording: (a) `03-policies/_policy-archetypes.md` renamed to `_archetype-index.md` and downgraded from canonical to navigation aid, (b) `_capability-matrix.md` renamed to `capability-matrix.md`, (c) initial 19 per-policy files created at v0.0 partial-grid status, (d) README replaced. Honest record of what changed. |
| `/Users/dawud/claude/casb/02-capabilities/_capability-matrix.md` (becoming `capability-matrix.md`) | Internal cross-refs in cells that point to `03-policies/_policy-archetypes.md` → update to point at the relevant per-policy file under `03-policies/<family>/<policy>.md` |
| `/Users/dawud/claude/casb/04-vendors/microsoft-defender-for-cloud-apps.md` line 17 (currently `For vendor comparison, see synthesis/capability-matrix.md`) | Update path to `02-capabilities/capability-matrix.md` |
| `/Users/dawud/claude/casb/04-vendors/microsoft-defender-for-cloud-apps.md` per-policy entries (1-13) | Add a back-link at the top of each policy: "Vendor-neutral spec: [`03-policies/<family>/<policy>.md`](../03-policies/<family>/<policy>.md)" |
| `/Users/dawud/claude/casb/04-vendors/netskope.md`, `zscaler.md`, `palo-alto-prisma-access.md`, `skyhigh-security.md` | Add equivalent back-links in their policy sections (where they exist); for cells flagged `[unverified]`, surface the flag in the corresponding `03-policies/<policy>.md` grid cell |
| Folder `_index.md` files in `01-foundations/`, `05-architecture/`, `06-compliance/`, `07-implementation/`, `08-failure-modes/` (where they exist) | Check for any references to "policy archetypes" or "capability catalogue" framing — replace with "policy library" / "capability matrix" terminology |

Audit cross-references with `grep -rn '_policy-archetypes\|_capability-matrix\|policy archetype\|capability catalogue\|policy catalogue' /Users/dawud/claude/casb/` after the restructure and patch the hits.

## Folder-by-folder navigation strapline (for README "Folder map")

| Folder | Strapline (one sentence) |
|---|---|
| `00-meta/` | Thesis lock, promotion log, three-lens review template, glossary. |
| `01-foundations/` | What CASB is and isn't, deployment modes, where CASB sits in SSE/SASE, market motion. |
| `02-capabilities/` | Cross-vendor capability matrix — what each tool can do, cell-by-cell with caveats and `[unverified]` flags. |
| `03-policies/` | Vendor-neutral policy library — one file per policy, sub-foldered by control family, each with a 5-row vendor-implementation grid. |
| `04-vendors/` | Per-vendor playbooks — Microsoft Defender for Cloud Apps at v1 lens-reviewed depth; Netskope / Zscaler / Palo Alto Prisma Access / Skyhigh as drafts. |
| `05-architecture/` | Reference architectures — proxy-only, API-only, hybrid; SSE stack integration; BYOD; GenAI overlay. |
| `06-compliance/` | Control mapping framework plus framework pages for BNM RMiT, ISO 27017, ISO 27018; clause-by-clause maps deferred. |
| `07-implementation/` | Programme readiness, phased rollout, SOC integration, success metrics, change management, cost of ownership — vendor-agnostic. |
| `08-failure-modes/` | Where CASB demonstrably fails — BYOD, OAuth blind spot, SSL/TLS breakage, API-mode-is-not-prevention, encrypted-upload bypass, over-blocking, vendor lock-in, cost blowout. |

## Disputes between reviewers

| Dispute | Reviewer A position | Reviewer B position | Resolution + rationale |
|---|---|---|---|
| **Whether to delete `03-policies/` entirely** | **ms-casb-implementor Option A**: delete the folder; move archetypes to `02-capabilities/use-case-grouped-matrix.md`; the real policy library is inside the MDA vendor file. | **multi-vendor-practitioner**: keep `03-policies/` and replace the archetype file with a proper vendor-neutral policy library, sub-foldered by control family. Also info-architect (keep, rename to `03-policy-patterns/`) and first-time-reader (keep, reframe). | **Adopt multi-vendor-practitioner's policy-library shape.** ms-casb-implementor is right that today's `03-policies/_policy-archetypes.md` is duplicative of `02-capabilities/_capability-matrix.md` — but the right fix is to *build* the policy library, not to surrender it. The locked thesis names "policies" as the unit of output. Deleting the folder concedes the thesis. The honest constraint ms-casb-implementor raises (only MDA is lens-reviewed; the other four vendor columns would be `[unverified]` placeholders) is solved by being explicit in `_index.md` about v0.0 status and partial-grid: the MDA column is reliable, other columns inherit source-quality flags. |
| **Whether to rename `03-policies/` to `03-policy-patterns/` or `03-policy-archetypes/`** | **info-architect**: rename to match the artefact (`03-policy-patterns/` or `03-policy-archetypes/`). | **multi-vendor-practitioner**: keep `03-policies/` and write proper policy entries that match the existing schema. | **Keep `03-policies/`.** Once the folder contains actual policy entries (per-file, schema-driven, with vendor-implementation grids), the name is accurate. Renaming to "patterns" or "archetypes" bakes in the current (broken) state. Fix the content; the name then matches. |
| **Whether to split `02-capabilities/_capability-matrix.md` into per-capability files** | **multi-vendor-practitioner**: yes — split into per-capability files, keep the matrix as a verdict-only summary, link out for cells. | **ms-casb-implementor / info-architect / first-time-reader**: no — the matrix is the canonical synthesis; the 14-entry per-capability plan in `_index.md` is aspirational and should be dropped. | **Do not split at v0.1.** The matrix is dense, well-cited, and lens-flagged. Per-capability files add maintenance cost without solving the user-named confusion. If cross-linking demand from policy or vendor pages grows, revisit at v0.2. Reviewer majority (3 of 4) plus `CLAUDE.md` file-creation discipline ("no speculative files"). |
| **Whether to keep the thesis sentence in the README front matter** | **ms-casb-implementor / multi-vendor-practitioner**: keep — it is load-bearing and the practitioner-reference framing is the right one. | **info-architect / first-time-reader**: move out of the README; it is an author-research artefact, belongs in `00-meta/thesis.md` (already linked). | **Move out of README.** A first-time reader is not here to validate a thesis. The lock + rationale stay in `00-meta/thesis.md` and `CLAUDE.md`. The README opens with one paragraph of "what this is" + a 3-row "Start here" table. Reviewer majority (2 of 4) on the move; the other two reviewers' concern (don't lose the practitioner-reference framing) is preserved by making the MDA file the second row of the "Start here" table and naming its Day 1/30/90 shape there. |
| **Whether to expose the capability/policy framing publicly** | **info-architect / first-time-reader**: yes — move the CLAUDE.md framing into the public `_index.md` of both folders. | (ms-casb-implementor argues the fence is academic — but does not contradict making it public.) | **Yes — move it.** Both `_index.md` files now lead with the two-sentence framing phrased as a navigation principle. CLAUDE.md retains the rule as editorial discipline; the public files mirror it as a reader signpost. |
| **Day 1/30/90 axis in `03-policies/` files** | **ms-casb-implementor Option B / first-time-reader** (implicitly): the MDA Day 1/30/90 cadence is a natural axis. | **multi-vendor-practitioner**: no — cadence is a per-firm rollout property, not a policy property. Organise by control family. | **By control family.** Cadence is a property of the rollout, not the policy. A bank with mature endpoint DLP defers inline DLP to Day 90; a greenfield firm needs impossible-travel on Day 1. Hard-coding Day 1/30/90 in the policy library forces one firm's rollout opinion into the canonical reference. Cadence lives in `04-vendors/<vendor>.md` (vendor-specific) and `07-implementation/phased-rollout.md` (vendor-agnostic). The per-policy file lists *which Day-tier the MDA playbook places it in* as metadata in its status line, not as the file's organising axis. |

## Out of scope

The following are deliberately not in this restructure pass:

- **No content changes to `04-vendors/microsoft-defender-for-cloud-apps.md`** beyond adding a back-link pointer in each of the 13 policy sections to the corresponding `03-policies/<family>/<policy>.md`. The Day 0 / Gaps / Day 1 / 30 / 90 / Appendices structure is preserved verbatim. It is the gold-standard shape.
- **No promotion of Netskope / Zscaler / Palo Alto Prisma Access / Skyhigh** to MDA-tier depth. Their drafts are unchanged. The vendor-implementation grid cells in the new `03-policies/` files inherit their existing `[unverified]` flags. Promoting those four vendors to MDA depth is the v0.2 work tracked in `00-meta/promotion-log.md` outstanding items.
- **No clause-by-clause regulator mapping** in `06-compliance/`. The framework pages exist; the full BNM RMiT / MAS TRM / ISO 27017 / ISO 27018 / NIST CSF / PCI DSS clause-by-clause mapping is a separate deliverable per the promotion log.
- **No changes to `CLAUDE.md`'s** citation discipline, sanitisation rules, three-lens review, locked decisions, or common pitfalls. Only the component-model paragraph and one line in "Capability vs Policy separation" are touched, both for terminology consistency with the new public `_index.md` files.
- **No changes to `07-implementation/`**. Six files, clean axis, vendor-agnostic — preserved verbatim. (Cross-walk between Day 1/30/90 and Phase 1-5, suggested by multi-vendor-practitioner, is a v0.2 nice-to-have; not in this pass.)
- **No changes to `08-failure-modes/`**. Eight named failure modes, public-wedge candidate — preserved verbatim. (Cross-link to `04-vendors/<vendor>.md` Gaps tables, suggested by multi-vendor-practitioner, is a v0.2 nice-to-have.)
- **No changes to `01-foundations/` or `05-architecture/`** beyond cross-reference path patches if any internal links break.
- **No new per-capability files** in `02-capabilities/`. The 14-entry plan is dropped from `_index.md` as a deferred-or-cancelled item; per-capability files remain a v0.2+ option.
- **No `_research/` cleanup.** The lens-review outputs, raw drafts, and v0 synthesis files stay where they are. The restructure operates on the canonical `01-08` tree only.
- **No introduction of new spine folders** (no `09-policy-library/`, no `09-decision-frames/`). The 00-08 skeleton is preserved.
