# Market motion

> Pure-play → SSE-feature timeline; acquisitions; what this means for buyers. Vendor-agnostic. Cites analyst material as opinion, not measurement.

## Timeline of consolidation

| Year | Event | Implication |
|---|---|---|
| **2012-2013** | First-gen CASB founded — Adallom, Bitglass, Netskope, Skyhigh Networks, CipherCloud | Pure-play category exists; standalone product |
| **2015** | **Microsoft acquires Adallom** | First major vendor consolidation. Adallom becomes Microsoft Cloud App Security (MCAS) |
| **2015** | Palo Alto acquires CirroSecure (later Aperture API-CASB) | NGFW vendor adds API-CASB |
| **2016** | Cisco acquires CloudLock | Cisco SSE positioning |
| **2017** | **McAfee acquires Skyhigh Networks** | Skyhigh becomes MVISION Cloud inside McAfee |
| **2018** | Symantec acquires CloudSOC; Forcepoint acquires Bitglass derivative | Two pure-plays absorbed |
| **2019** | Gartner coins **SASE** | Category-naming makes "platform vendor" the aspiration |
| **2019** | Symantec sold to Broadcom (enterprise unit); Symantec CloudSOC absorbed into Broadcom Symantec Cloud SWG | Pure-play SaaS-security capability moves further into a SWG platform |
| **2021** | Microsoft renames MCAS to **Microsoft Defender for Cloud Apps**; folds into Defender XDR | CASB framed as part of XDR, not standalone |
| **2021** | Gartner coins **SSE** as the security subset of SASE | Sharpens the conversation; SSE-leader is the new "Magic Quadrant" position |
| **2021** | **Lookout acquires CipherCloud** | Another pure-play absorbed |
| **2022** | **McAfee enterprise unit splits** — consumer goes to McAfee Inc; enterprise security goes to Trellix; SSE / CASB portion spins out as **Skyhigh Security** as a separate company | Skyhigh becomes one of the few remaining stand-alone CASB-lineage names |
| **2022** | **Forcepoint renames Bitglass-lineage product to Forcepoint ONE** | SSE-platform branding |
| **2023-2024** | SSE-leader narrative dominates analyst commentary | Netskope, Zscaler, Palo Alto, Cisco, Broadcom Symantec, Forcepoint, iboss, Lookout all positioned as SSE platforms |
| **2024-2025** | First wave of **GenAI-app discovery and governance** added to SSE platforms | CASB capability set expands beyond the original Gartner four pillars |
| **2025-2026** | **DSPM-for-AI / prompt-content visibility** emerges as a CASB+ feature wave; Microsoft Edge for Business adds native AI controls; GSA / equivalent platforms add native AI category enforcement | The "what's CASB and what's adjacent" question gets harder again |

## What this means for buyers in 2026

### "Buying CASB" is mostly "buying SSE"

The procurement category in 2026 is SSE, not CASB. If a vendor proposes "CASB" as a standalone product, the more useful question is: what SSE bundle are you on, and what does the CASB sub-feature look like inside it?

Skyhigh remains the meaningful exception — a stand-alone-CASB-lineage vendor in the 2026 market — but even Skyhigh sells primarily as SSE.

### Vendor renames are routine — name discipline matters

The same product line has often had three or four names within a decade. For the OneDrive of "what is the product actually called this quarter":

| Lineage | 2026 name (and the prior names) |
|---|---|
| Adallom | Microsoft Defender for Cloud Apps (was Microsoft Cloud App Security / MCAS) |
| Skyhigh Networks | Skyhigh Security (was MVISION Cloud, was McAfee MVISION Cloud, briefly Trellix CASB) |
| CirroSecure → Aperture | Palo Alto SaaS Security (sometimes still called Inline CASB inside Prisma Access) |
| Symantec CloudSOC | Broadcom Symantec Cloud SWG (CASB folded in) |
| Bitglass | Forcepoint ONE |
| Oracle CASB | Discontinued |
| CipherCloud | Folded into Lookout Cloud Security |

This matters because vendor procurement and partner-channel staff use whichever name the regional team is comfortable with. Documentation, audit evidence, and architecture diagrams should use the current canonical name and reference the lineage.

### Pure-play extinction has consequences

Pure-play CASB vendors were narrow but deep. SSE-feature CASB is broader (more components, more bundling) but each component is shallower than the pure-play it replaced.

A buyer in 2026 typically picks one of three patterns:

| Pattern | Trade-off |
|---|---|
| **SSE-platform bundle (Netskope / Zscaler / Palo Alto)** | One vendor, one platform; CASB depth varies by vendor; integration with non-Microsoft IdP is variable |
| **Microsoft-stack consolidated (M365 E5 + GSA + Purview)** | Deepest integration if already on Microsoft; severe concentration risk; no forward-proxy CASB |
| **Stand-alone CASB-lineage (Skyhigh)** | Multi-mode (proxy + API + log-based) heritage; less deep on the broader SSE components (SWG / ZTNA) |

There is no "best" choice — each has named trade-offs.

### The GenAI wave reshaped the procurement criteria

In 2023 the CASB checklist did not include GenAI discovery. In 2026 it does. SSE vendors that didn't ship GenAI capabilities in 2024-2025 are catching up; the differentiation in 2026 procurement is on:

- GenAI sub-categorisation (LLM vs image vs video vs voice/avatar)
- Prompt-content DLP inspection (and the SAML-federation prerequisite for the inspection)
- Approved-AI-vendor allowlist management
- DSPM-for-AI prompt visibility (Microsoft Purview integration or equivalent)

This is changing fast. A 2026 H1 procurement document will need updating by 2026 H2.

### The concentration-risk conversation is getting louder

A regulated FI that picks an SSE platform also picks an IdP integration, a SIEM integration, a DLP integration, and often an EDR integration. The concentration accumulates.

Regulators are beginning to ask about this — BNM RMiT third-party / outsourcing sections, MAS Outsourcing, EU DORA concentration-risk recitals — verify against the current edition of each. Procurement that doesn't name concentration risk early gets it raised at the regulator conversation late.

## What changed that the original Gartner formulation didn't anticipate

| Original four-pillar model (2012) | What 2026 needs |
|---|---|
| Visibility, Data Security, Threat Protection, Compliance | + Generative-AI app governance (~half a pillar of its own) |
| | + AI-prompt content DLP and visibility |
| | + Cross-border data-flow governance (sharper since 2022) |
| | + Identity-platform integration depth (was assumed; now decisive) |
| | + Audit-evidence quality (was a side-note; now front of the procurement model) |

A buyer who runs CASB selection on the original 2012-framing checklist will mis-prioritise.

## Analyst material caveat

| Caveat | Why |
|---|---|
| **Magic Quadrant position is analyst opinion, not measurement.** | Position reflects execution + vision against analyst-defined criteria. It is not market share, capability test, or buyer satisfaction. Cite as "Gartner [year, report ID] positioned X as Leader" — never as a capability claim |
| **Wave / IDC tracker / etc. — all analyst material is secondary.** | Useful for category orientation; not for evidence-grade capability claims |
| **Vendor analyst-day briefings are vendor-controlled.** | What analysts hear at vendor analyst days is curated. Independent test reports and customer references carry more weight |
| **Analyst categories drift annually.** | The exact set of vendors named "SSE Leader" shifts year-to-year. Decisions made on Year-N positioning may not survive Year-N+1 |

For this repo's purposes: analyst material is cited as opinion with date + report ID, never as a capability claim. Primary-source vendor docs + independent test reports + customer references are the citation floor.

## Buyer's mental model for 2026

| Question | Answer in 2026 |
|---|---|
| Is CASB a product category? | No — it's a capability set inside SSE (with one exception, Skyhigh) |
| Should I buy a pure-play CASB? | Probably not — the market signal is consolidation; pure-plays have already been absorbed or repositioned |
| What's the most important non-CASB decision in a CASB procurement? | The IdP architecture. Inline CASB collapses without a clean IdP story |
| What's the second-most-important non-CASB decision? | The SIEM / retention model. CASB in-product retention is insufficient for BFSI audit |
| What's changing fastest? | GenAI controls. Procurement criteria from 2024 are stale; 2026 H1 criteria will be partly stale by 2026 H2 |
| What's the biggest hidden cost? | Adjacent-licence creep (see [`../07-implementation/cost-of-ownership.md`](../07-implementation/cost-of-ownership.md)) |

## Reading order from here

- [`../04-vendors/`](../04-vendors/) for per-vendor depth.
- [`../02-capabilities/capability-matrix.md`](../02-capabilities/capability-matrix.md) for cross-vendor capability comparison.
- [`../08-failure-modes/`](../08-failure-modes/) for the "what they all fail at" view.
