# CLAUDE.md — `casb`

Research repo: **Cloud Access Security Broker (CASB) — what the capability set actually is in 2026 (now mostly absorbed into SSE/SASE platforms), how it maps to vendor products, what policies it enforces, and how those policies satisfy regulator expectations.**

> **Status:** private until v0.1. Goal output is dual-purpose: (a) a public knowledge base that explains CASB-as-a-capability honestly (no MQ-position-as-truth, no vendor marketing language), (b) consulting-ready material — vendor selection grid, policy library, control-mapping evidence pack — droppable into a regulated-FS engagement. Assume eventual public release; write sanitised from day one.

## Thesis (LOCKED 2026-06-10)

> **"For each of the five CASB products in scope, what policies can a practitioner actually configure — and what does the tool *not* cover?"**

This is a **practitioner's per-vendor reference.** Not a buyer's guide (no scorecard), not a reference architecture (vendor-agnostic patterns are secondary), not a sellable policy pack (policies catalogued only where vendor X actually supports them). The user owns / may own one of these tools and needs to know what knobs exist and where the blind spots are.

**Primary unit of output:** `04-vendors/<vendor>.md` — each carries three named sections — **Capability set / Configurable policies / Limitations**. Synthesis pages (`02-capabilities/`, `03-policies/`, `08-failure-modes/`) aggregate across vendors, but the per-vendor file is canonical.

**User's immediate case:** Microsoft Defender for Cloud Apps ("if I already have it, what can I do with it"). That file gets the most depth and worked policy examples.

See [`00-meta/thesis.md`](00-meta/thesis.md) for the lock log and rationale.

## Component model

```
00-meta/             Thesis, scope, lock log, citation policy, glossary, sources methodology, three-lens review template
01-foundations/      What CASB is (and isn't), the four-pillar Gartner model, deployment modes (forward-proxy / reverse-proxy / API / hybrid), where CASB sits inside SSE/SASE, the pure-play → SSE-feature market motion
02-capabilities/     Cross-vendor capability matrix (one canonical file: `capability-matrix.md`). What each tool CAN do, cell-by-cell with caveats and `[unverified]` flags. Per-capability deep-dive files deferred to v0.2+.
03-policies/         Policy library — one file per policy, sub-foldered by control family (access-control / dlp / detect / oauth / genai / posture). Each policy carries a vendor-neutral spec and a 5-row vendor-implementation grid linking to `04-vendors/<vendor>.md`. The split is also surfaced as a public navigation principle in `02-capabilities/_index.md` and `03-policies/_index.md`.
04-vendors/          Vendor deep-dives — one file per vendor (Netskope, Zscaler, Microsoft Defender for Cloud Apps, Palo Alto Prisma Access / SaaS, Skyhigh Security — formerly McAfee / MVISION / Trellix CASB). Capability matrix, deployment mode, SSE integration, MY presence/partners, public pricing where available
05-architecture/     Reference architectures — proxy-only, API-only, hybrid; integration with IdP, SWG, ZTNA, DLP-at-rest; how CASB sits inside SSE; data flows and chokepoints
06-compliance/       Regulator mapping — BNM RMiT (cloud risk, data leakage), MAS TRM, HKMA, PDPA / GDPR (data residency, processor obligations), ISO/IEC 27001 / 27017 / 27018, NIST CSF 2.0, SOC 2. Control-to-capability-to-policy traceability.
07-implementation/   Programme view — readiness assessment, phased rollout (discovery → risk-scoring → policy enforcement), change-management, success metrics, SOC integration
08-failure-modes/    Where CASB demonstrably fails — over-blocking, OAuth blind spots, BYOD/unmanaged-device gaps, SSL/TLS inspection breakage, encrypted-upload bypass, vendor-lock-in, cost-blowout. Public-wedge candidate.
_sources/            Primary-source library — vendor architecture docs, regulator publications, analyst report IDs (cited as secondary), public incidents, NIST/ISO clause references. One file per source.
_research/           Working notes, reading list, scratch. Not citation-gated. Where the multi-agent workflow lands its raw output.
```

Folder-level `_index.md` files define folder-specific scope. Do not create speculative pages outside this structure.

## Locked decisions

| Decision | Choice | Rationale |
|---|---|---|
| **Anchor geography** | Malaysia | User's home market; aligned with sibling repos `CyberTTX`, `cyberkpis`, `cyber-business`; BNM RMiT is well-mapped |
| **Tier-1 comparator regulators** | MAS TRM (SG), HKMA, EU GDPR + DORA cloud-risk overlay, ISO/IEC 27017 + 27018 as cross-jurisdiction backbone | Cross-border SaaS data flow is the actual CASB problem; single-jurisdiction view is incomplete |
| **Vendor scope (initial)** | Netskope, Zscaler, Microsoft Defender for Cloud Apps, Palo Alto Prisma Access (with SaaS Security / Inline CASB), Skyhigh Security (formerly McAfee / MVISION / Trellix CASB). One file each. | Covers the SSE-leader CASB feature set + the ex-pure-play that still sells stand-alone. Cisco Cloudlock / Forcepoint ONE / Lookout / iboss only added if the wedge demands. |
| **CASB framing** | CASB as a *capability set* — primary lens; SSE-platform housing is secondary | Studying SSE end-to-end would balloon scope into ZTNA, SWG, RBI, FWaaS. Stay disciplined: this repo is the CASB lens. |
| **Capability vs Policy separation** | Strict — capabilities in `02-`, policies in `03-`, never inline together. The split is also surfaced as a public navigation principle in `02-capabilities/_index.md` and `03-policies/_index.md`. | The user's framing conflated them; the discipline forces clarity |
| **Three-lens review** | Every synthesis page passes through Architect / Product / Compliance lenses before promotion | Captures the user's role-play intent as a recurring review pattern, not a one-off workflow |
| **Out of scope (initial)** | SaaS-native security posture management (SSPM) as a distinct discipline — referenced only where it overlaps with API-mode CASB; CNAPP / CSPM (cloud *infrastructure* posture, not SaaS) — separate market; CWPP; secrets management. | Cloud-security category is huge; CASB is the SaaS-access slice. Adjacent disciplines named for boundary clarity, not covered. |
| **Citation backbone** | Vendor documentation = secondary (vendor-controlled); analyst MQ/Wave = secondary (label as opinion, not measurement); regulator publications + standards = primary | Vendor docs describe what the product *claims* to do; never cite as proof of behaviour without independent corroboration |
| **Visibility** | Private until v0.1 | Lets thesis lock, lets `_research/` mature, lets capability claims get verified before publication |

## Citation discipline (non-negotiable)

CASB is a vendor-marketing-dense category. Floor is high.

- **Capability claims** about a named vendor cite the vendor's own architecture / admin documentation page + access date. *And* flag as vendor-controlled. Cross-corroborate with a second source (a public deployment guide, an independent test report, a CVE/incident, a customer reference) before a capability claim is promoted to `02-capabilities/` or `04-vendors/`.
- **MQ / Wave / analyst-quadrant references** are *opinion*, not measurement. Cite as "Gartner [report title, ID, year]" or "Forrester Wave [title, year]" and label `[secondary — analyst opinion]`. Never write "industry-leading" or "Leader quadrant" as evidence of capability.
- **Regulator obligations** cite the specific clause + version + date. Same rule as `cyberkpis` / `cyber-business`. No "BNM requires…" without RMiT section reference + edition.
- **Standards clauses** (ISO 27017, NIST CSF) cite the control ID + the document version.
- **Public incidents** referenced for failure-mode entries cite the public disclosure (vendor advisory, CVE, breach notification, regulator action) — not vendor marketing's framing of the incident.
- **Policy claims** ("policy X reduces risk Y") need a stated control mapping and a stated false-positive risk. Vendor marketing routinely claims policies "prevent" things they detect after the fact; do not parrot.
- **Verification gate**: any new factual claim carries `[VERIFY]` until traced to a primary source and the URL / clause / report ID is captured in `_sources/`.
- **Disclaimer in any client-facing material**: this material supports CASB / SSE selection and policy design; it is not legal, regulatory, or audit advice.

## Sanitisation rules

- **No client names.** Ever. Anonymised or not.
- **No engagement-level intelligence** — pursuit data, vendor pricing won at, internal selection scorecards, partner conversation content, vendor briefing under NDA.
- **No Big-4-internal data** — methodologies, partner-only material, internal vendor scoring.
- **Insights about competitive dynamics** abstract to archetype (e.g. "tier-1 ASEAN bank with hybrid M365 / Workspace estate"), never name specific deals.
- **No vendor-confidential roadmap content** from briefings, analyst days, or partner programs.
- **Public incidents / public disclosures** are fair game; cite the disclosure.

## Style

- Markdown; tables and short bullets over prose walls.
- British English (matches workspace convention).
- File names: `kebab-case.md`; numbered prefixes (`01-`, `02-`) only for ordered series.
- Each capability entry follows the schema in `02-capabilities/_schema.md`. Each policy entry follows `03-policies/_schema.md`. Each vendor file follows `04-vendors/_schema.md`. Schemas created when first entry is written. Do not improvise fields.
- Bold for emphasis; never light-coloured text in any code or terminal block.
- Reject vendor marketing words: "next-gen," "AI-powered," "best-of-breed," "industry-leading," "robust," "comprehensive," "single pane of glass," "consolidate your stack," "seamless," "enterprise-grade." Name the thing.

## File-creation discipline

- **No speculative files.** Each capability entry, vendor file, policy archetype, or framework page is created when there is something concrete to put in it. No "TODO: write this later" placeholders in v0.1-candidate folders.
- **One canonical file per topic.** Cross-link, do not duplicate.
- **One file per vendor / capability / policy archetype / standard.** Mixing is a smell.
- **Prefer extending** an existing file over creating a new one for the same topic.
- **`_research/`** is the only folder where messy drafts and clippings are acceptable. Multi-agent workflow output lands here first; promote only after the three-lens review + citation gate.

## Three-lens review (the user's role-play, as a recurring pattern)

Every synthesis page in `01-` … `08-` passes three lenses before promotion from `_research/`. The lenses are templated as agent personas — usable inline by a single Claude or fanned out as a multi-agent workflow.

| Lens | Role | Asks |
|---|---|---|
| **Architect** | Senior cybersecurity architect, regulated FS, hybrid M365 / Workspace estate | Does this deploy? Where does it sit in the SSE stack? What does it break? What's the perf / latency / cost cost? What identity dependency? What does *not* get covered (BYOD, unmanaged, mobile native apps, third-party API access)? |
| **Product** | Hands-on CASB product engineer who has actually run all major CASB consoles | Is this claim what the vendor says, or what the vendor *does*? What's the deployment-mode caveat (proxy can't see API-only traffic; API can't block in real time)? What's a known footgun in the console? What does the vendor *hide* in its docs? |
| **Compliance** | GRC lead / IS auditor familiar with BNM RMiT, MAS TRM, ISO 27017/27018, PDPA, GDPR | What control(s) does this satisfy and how is *evidence* generated? What auditor would accept this and what would they reject? What regulator timing/clock applies? Where does the policy create a privacy/data-protection conflict (e.g. SSL inspection on personal use)? |

Each promoted page carries a short `**Three-lens sign-off:**` block at the bottom listing what each lens flagged and how it was resolved. A page that fails any lens stays in `_research/`.

See [`00-meta/three-lens-review.md`](00-meta/three-lens-review.md) when written for the full prompt templates.

## Common pitfalls (anticipated; add as encountered)

| Pitfall | Counter |
|---|---|
| Treating "CASB" as a product category in 2026 | It is a *capability set* mostly delivered inside SSE/SASE platforms. State the housing (Defender for Cloud Apps, Netskope SSE, etc.) when discussing any specific product. |
| Conflating capability with policy | Capability = "the tool can…"; Policy = "the firm has decided to…". Strict folder separation: `02-` vs `03-`. |
| Conflating CASB with SSPM / CSPM / CNAPP | CASB = SaaS *access* security. SSPM = SaaS *configuration* posture (Salesforce admin settings, M365 hardening). CSPM = IaaS configuration. CNAPP = workload posture. Different markets, different buyers. |
| Citing Gartner MQ position as capability evidence | MQ is analyst opinion on execution + vision. It is not a feature test. Cite vendor docs for what it *claims*, independent reports for what it *does*. |
| "Block Shadow IT" as a policy | "Block" is a *response*; "Shadow IT" is a *discovery output*. The actual policy needs: which app set (sanctioned/tolerated/unsanctioned), which user group, which action (block / coach / monitor / quarantine), how the exception is granted. Vague policies fail audit. |
| Forward-proxy assumed = sees everything | Forward-proxy CASB needs traffic steering (agent / PAC / SSL inspection / IdP routing). Mobile native apps, unmanaged devices, third-party-to-third-party API traffic routinely bypass. Name the gap. |
| API-mode assumed = real-time blocking | API-mode is *near-real-time monitoring + retroactive remediation*. Not inline enforcement. Many "DLP for SaaS" claims rely on API mode = post-upload detection, not prevention. |
| SSL/TLS inspection treated as a free lunch | Breaks certificate pinning, breaks some banking apps, breaks personal-use HTTPS in BYOD contexts, breaks privacy expectations under PDPA / GDPR when over-applied. Trade-off, not capability. |
| "OAuth governance" as a tick-box | OAuth grants from M365 / Workspace bypass network controls entirely. CASB API-mode is the only chokepoint for some of these. Coverage varies sharply by vendor; never assume parity. |
| Treating BNM RMiT cloud-risk requirements as the only Malaysian gate | PDPA (esp. processor obligations + cross-border), the new MyDigital ID overlays, NACSA Cybersecurity Act 854 (2024) sectoral expectations all bite. Cite the specific instrument, not "Malaysian regulation." |
| Quoting US pricing for MY deployment | SSE pricing varies sharply by region and channel. State the geography. |
| Multi-agent workflow output treated as researched material | Workflow output lands in `_research/`. It is *draft* — agents pull from vendor marketing as easily as primary sources. Three-lens review + citation gate before promotion. |

## Workflow

1. **Question first** — every session opens with a specific question that advances the locked thesis (or a candidate wedge if thesis is unlocked). Vague "let me read about CASB" sessions belong in `_research/`.
2. **Primary-source ingest** → `_sources/<source-slug>.md` — verbatim citations, version, date, URL, access date. Vendor docs flagged as vendor-controlled.
3. **Synthesis draft** → `_research/` first. Multi-agent workflow output also lands here.
4. **Three-lens review** → Architect / Product / Compliance, inline or via workflow. Findings recorded in the page's sign-off block.
5. **Promotion** → `01-` … `08-` folder when the page passes all three lenses and the citation gate.
6. **Pre-public gate** before `v0.1`: thesis locked, every `[VERIFY]` cleared, citation policy passed, sanitisation pass, vendor claims independently corroborated, README.md rewritten for public audience.

## Sibling repos in this workspace

- [`cyber-business`](../cyber-business/) — cybersecurity market mechanics. SSE / CASB vendor unit economics, MY market presence, and BFSI-buyer behaviour overlap with this repo's `04-vendors/` and `07-implementation/`.
- [`cyberkpis`](../cyberkpis/) — cyber KPI/KRI/KCI methodology. Metrics for CASB programme success (Shadow IT reduction, DLP true-positive rate, policy false-positive rate) should follow that repo's schema, not be reinvented here.
- [`CyberTTX`](../CyberTTX/) — MY tabletop exercise consulting. Cloud-account-compromise and SaaS-data-exfiltration scenarios will reference CASB controls; cross-link rather than duplicate.
- [`references/`](../references/) — local regulatory and standards library (BNM RMiT, MAS TRM, NIST, ISO, NACSA). Cite from here for regulator publications instead of re-downloading.
- [`KWAP-TRMF`](../KWAP-TRMF/) — likely related TRMF / cloud-control mapping work; check before duplicating any control mapping.
- [`AI-Governance`](../AI-Governance/), [`AI-IAM`](../AI-IAM/) — adjacent for the AI-app-governance overlap as CASB extends into Generative-AI-app discovery/blocking (a 2024-2026 SSE feature wave).

## Hard-marker mode

Default tone: terse, exacting, source-driven. Reject vendor marketing language. Name the thing, name the source, name the trade-off. If a claim cannot be sourced beyond the vendor's own marketing, label it as such or do not make it.
