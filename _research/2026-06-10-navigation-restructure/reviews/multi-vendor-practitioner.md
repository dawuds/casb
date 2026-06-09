# Multi-vendor senior security architect (operations-focused) - navigation review

Reviewer persona: senior security architect, deployed MDA / Netskope / Zscaler in regulated FS. Lens is operational — what controls am I building, what evidence am I generating, what operational cost am I committing to. Reading priority: the moment I need to design or defend a CASB programme.

## Diagnosis (what is wrong with the current navigation)

**1. The README's "How to navigate" table puts orientation before action.** Row 1 is `01-foundations/` (what CASB is). Row 2 is the capability matrix. Rows 6 and 7 are `06-compliance/` and `07-implementation/`. A senior architect arriving at this repo with a real programme in flight does not need orientation — they need either (a) the capability picture cross-vendor, (b) one vendor's playbook, or (c) the rollout sequence. The README buries (c) in row 7 of a 9-row table. Fix by putting the *decision-grade* entry points at the top and demoting orientation.

**2. `03-policies/_policy-archetypes.md` is mis-named.** It is not a policy catalogue — it is a *capability comparison grouped under archetype headings*. Each of the 14 entries reads "vendor X claims feature Y, vendor Z has a caveat". That is a capability-matrix slice, not a policy entry. A policy says *which users, which apps, which action, which exception path, which evidence record, which FP cost*. None of the 14 entries in `_policy-archetypes.md` carries any of those fields. The `_index.md` schema is correct; the canonical file ignores its own schema.

**3. The split between `02-capabilities/_capability-matrix.md` and `03-policies/_policy-archetypes.md` is illusory.** Compare row by row. `02-capabilities/` Section 3 row "Inline DLP for sanctioned SaaS (pre-upload block)" and `03-policies/` Archetype #4 "Inline DLP on upload" are the same content with different prose. The user-named confusion is real: the repo has *two cross-vendor matrices that describe the same thing twice*, neither of them an actual policy library. The CLAUDE.md "Strict separation — capabilities in 02-, policies in 03-, never inline together" rule has been violated by promotion.

**4. The README claims `03-policies/` contains "the cross-vendor policy archetype list" — which is honest about the file but dishonest about what `03-policies/` should be.** The README is documenting the bug, not the design.

**5. There is no cross-vendor control-to-vendor index.** A senior architect wants: "I need a tenant-restriction control. Show me how each of the 5 vendors implements it." The `02-capabilities/_capability-matrix.md` Section 6 has the cell-level answer but it is buried in a wall of rows and not linked from a vendor file. The MDA file at `04-vendors/microsoft-defender-for-cloud-apps.md` Policy table does not back-link to the cross-vendor capability matrix row. Round-tripping is manual.

**6. The MDA file is excellent — and it has no peer.** `04-vendors/microsoft-defender-for-cloud-apps.md` carries Day 0 preconditions, gaps-and-compensating-controls, Day 1/30/90 policies with ATT&CK / KQL / SIEM artefact / SCIM cascade / DR matrix. The other four vendor files are explicitly "drafts — not lens-reviewed at MDA depth" per the promotion log. The asymmetry is honest in the promotion log but invisible in the README — row 4 says "vendor drafts for Netskope / Zscaler / Palo Alto / Skyhigh" with a single one-line caveat. The reader does not learn until they click that they are reading a 10× thinner file than MDA.

**7. `07-implementation/phased-rollout.md` is vendor-agnostic Day 0 → Phase 5, but `04-vendors/microsoft-defender-for-cloud-apps.md` is vendor-specific Day 1 / Day 30 / Day 90. The two timelines do not align.** Phase 3 in `phased-rollout.md` is "weeks 8-16" — that lines up with MDA Day 30, not Day 90. There is no cross-walk table. A reader doing real implementation has to mentally translate.

**8. The `04-vendors/microsoft-defender-for-cloud-apps.md` "Gaps and compensating controls" table is *the* canonical CASB-gap reference in this repo — and `08-failure-modes/` is a separate folder with its own files.** Same content, two locations, no cross-linking. The MDA gaps table is operationally tighter than the `08-failure-modes/` per-failure-mode files (those are vendor-agnostic by design). The architect does not know which one to cite.

**9. Section headings in `02-capabilities/_capability-matrix.md` are operationally arbitrary.** Section 1 = "Deployment modes". Section 4 = "Identity / OAuth / SSPM" (three different things crammed into one section because they share the IdP). Section 8 = "Data residency / control plane" (two unrelated rows). A senior architect parsing this for control-availability cannot scan to the section that owns the control. Re-section by *control family* (access / DLP / detect / respond / posture / data-protection), not by vendor product taxonomy.

**10. `_capability-matrix.md` cell content is heterogeneous.** Some cells are a single word ("S"), some are 4-line paragraphs with citations. Eye-scanning for cross-vendor differences is slow. Either standardise cells to "verdict + 1-line caveat with detail in a footnote" or accept the table is for reading down a column (one vendor at a time), not across a row.

## What `02-capabilities/` should be

**A capability is a tool primitive. One file per capability. Each file is the cross-vendor view for that single capability.**

`02-capabilities/_capability-matrix.md` should remain as the *summary* table — verdict-only S/C/N/U per cell, no caveats inline, link out to the per-capability file for the detail. Strip the prose; keep the cells.

Then split the current matrix into per-capability files, one each, named by capability:

```
02-capabilities/
  _index.md                          (existing; tighten schema)
  _capability-matrix.md              (verdict-only summary; link out)
  _schema.md                         (write per existing _index.md spec)
  shadow-it-discovery.md
  sanctioned-app-risk-scoring.md
  inline-dlp-sanctioned-saas.md
  api-mode-dlp.md
  gateway-encryption-tokenisation.md  (one-pager: dead capability; state explicitly)
  oauth-grant-discovery.md
  sspm-posture-drift.md
  ueba-anomalous-activity.md
  compromised-account-detection.md
  session-control-risk-based.md
  watermarking.md                    (one-pager: not native anywhere; routed via RBI or Purview)
  malware-saas-storage.md
  external-share-governance.md
  tenant-restriction.md
  data-residency-geo-policy.md
  genai-app-discovery.md
  genai-prompt-dlp.md
  step-up-mfa-from-casb-verdict.md   (one-pager: not native in 4 of 5; IdP is the response plane)
```

Each file follows the existing `_index.md` schema (Definition / Gartner pillar / Deployment-mode dependency / Vendor coverage matrix / Known limitations / Linked policies / Linked controls / Three-lens sign-off). Per-vendor cells link out to the corresponding section of the relevant `04-vendors/<vendor>.md` file. **The capability file is the place the architect comes to ask: "what does the field look like for this control".**

Critically: capability files MUST NOT contain "how to enforce this in the firm" decisions. Those are policies.

## What `03-policies/` should be (the user-named confusion — going deep)

The current `03-policies/_policy-archetypes.md` is a *cross-vendor capability slice grouped by an archetype label*. It is duplicative of `02-capabilities/`. Delete it as a canonical file (keep the source in `_research/` if useful for the cross-vendor capability summary, which is the only thing it really is).

**Replace with: a policy library, one file per policy, with the schema the existing `03-policies/_index.md` already specifies.** That schema is correct. The file just was never written. What goes in each file:

| Field | Why a senior architect needs it |
|---|---|
| **Name** | Canonical, vendor-neutral |
| **Purpose** | One sentence: which named risk does this reduce |
| **Action** | block / coach / monitor / quarantine / encrypt / step-up / label / DLP-redact |
| **Scope** | User group / app set / device posture / network position / traffic direction. *This is the policy.* The MDA file's Policy 5 has it baked in; the cross-vendor file must surface it explicitly |
| **Required capabilities** | Link to `02-capabilities/` entries — the policy *uses* capabilities |
| **Deployment-mode requirement** | Inline only? API only? Hybrid? — collapses the candidate-vendor field |
| **Vendor implementation grid** | One row per vendor: console path, key configuration values, deployment-mode caveat, known trap. **This is the missing piece across the whole repo.** Architect uses this to write the policy spec for whichever vendor is in scope |
| **Control mappings** | Link to `06-compliance/` entries — what does this policy evidence |
| **False-positive risk** | Named scenarios — not a rating |
| **Operational cost** | Exception load / triage load / end-user friction / change-management touch |
| **Privacy / data-protection considerations** | SSL inspection on personal traffic? PII in admin view? DPIA trigger? |
| **Coverage gaps** | What this policy provably misses — fold in the relevant rows from `08-failure-modes/` |
| **Three-lens sign-off** | Architect / Product / Compliance |

The MDA file's Policy 5 (Block download of Confidential-labelled files to unmanaged browsers) is the existing reference shape for the *vendor-specific implementation block inside a vendor file*. The `03-policies/` file should be the **vendor-neutral policy spec with a 5-row implementation grid at the bottom**, each row linking to the corresponding `04-vendors/<vendor>.md` section.

Suggested policy file list (initial pass, derived from the MDA Day 1/30/90 list + the cross-vendor archetype list, de-duplicated):

```
03-policies/
  _index.md                                                       (existing; tighten)
  _schema.md                                                       (write)
  access-control/
    block-unsanctioned-app-with-coach.md
    tenant-restriction-corporate-only.md
    block-download-unmanaged-device.md
    block-download-byod-with-read-only.md
    geo-residency-block.md
  dlp/
    inline-upload-block-regulated-data.md
    api-data-at-rest-quarantine.md
    external-share-link-quarantine-two-stage.md
    auto-label-pci-data.md
    auto-label-pii-data.md
  oauth/
    post-consent-cleanup-and-ban.md
    application-permission-governance.md
    high-scope-grant-alert.md
  detect/
    mass-download-alert.md
    mass-delete-anomaly.md
    impossible-travel-alert.md
    terminated-user-activity-cross-saas.md
    b2b-partner-exfil-alert.md
  genai/
    discovery-and-auto-unsanction.md
    inline-prompt-dlp.md
    clipboard-paste-block.md
    sanctioned-tenant-pinning.md
  posture/
    sspm-tenant-misconfig-drift.md
    rbi-overlay-for-risky-sessions.md
```

The **`access-control/` and `dlp/` sub-folders** are deliberate. The current flat `03-policies/` namespace will not survive 25+ entries; sub-foldering by *control family* (not by vendor, not by archetype-number) is how an architect actually navigates.

## Per-vendor implementation — where does the mapping live?

Question was: in `03-policies/` per-policy entries, in `04-vendors/` per-vendor playbooks, or both with cross-links?

**Both, with cross-links. They are not redundant; they are reciprocal.**

- `03-policies/<policy>.md` carries a 5-row implementation grid. **Reader query: "I want to implement this policy. Which vendor in my estate can do it, with what trap?"** The grid links each cell to the corresponding section of `04-vendors/<vendor>.md`.
- `04-vendors/<vendor>.md` carries the vendor's full Day 1/30/90 playbook, ordered by rollout cadence. **Reader query: "I own vendor X. What can I switch on, in what order, with what evidence?"** Each policy entry in the vendor file links back to the `03-policies/<policy>.md` for the vendor-neutral spec.

Trade-off acknowledgement:
- **Pure (a) [03-policies/ owns the mapping]** — clean cross-vendor view, but the vendor reader has to read 25 policy files instead of one vendor file. Wrong default for the practitioner.
- **Pure (b) [04-vendors/ owns the mapping]** — fast vendor reading, but no cross-vendor view. The architect doing selection or doing multi-tenant work (different BUs on different CASBs) is blocked.
- **(c) both with cross-links** — duplication risk. Manageable if `03-policies/` carries the vendor-neutral spec + summary grid only (no full per-vendor configuration walkthrough — that stays in `04-vendors/`).

**The vendor-neutral spec lives in `03-policies/`; the full configuration walkthrough lives in `04-vendors/`; the grid in `03-policies/` is the joining surface.**

## Day 1 / Day 30 / Day 90 — right axis for `03-policies/`?

**No. Day 1/30/90 is a vendor-specific rollout cadence; it belongs in `04-vendors/<vendor>.md` (where it already is) and in `07-implementation/phased-rollout.md` (where it already is, vendor-agnostic).**

`03-policies/` should be organised by *control family* (access control, DLP, detect, OAuth, GenAI, posture). Reasons:

1. Policies are reused across vendor migrations — a tenant-restriction policy spec survives a CASB rip-and-replace; the Day 30 label is a property of *this* deployment.
2. Different firms hit policies at different rollout points. A bank with mature DLP-at-endpoint may defer inline-DLP to Day 90; a greenfield firm may need impossible-travel on Day 1. Hard-coding Day 1/30/90 in the policy library forces a single firm's rollout opinion into the canonical reference.
3. Control-family is the audit-navigation axis. An IS auditor asks "show me your access-control policies", not "show me your Day 30 policies".

Cross-link from `04-vendors/<vendor>.md` Day 1/30/90 list to the relevant `03-policies/<family>/<policy>.md` files. Phased-rollout cadence is metadata on the vendor playbook; policy taxonomy is the spine.

## Proposed README structure

```
# casb

[Existing thesis para — keep verbatim, it is the load-bearing statement of intent.]

## Start here based on your question

| If you want | Read |
|---|---|
| **I own a CASB and need to know what to switch on** | 04-vendors/microsoft-defender-for-cloud-apps.md (worked at depth); other 4 vendor files are drafts at ~10% of MDA depth |
| **I have to design a policy and need the vendor-neutral spec** | 03-policies/ (one file per policy; vendor-implementation grid at the bottom of each) |
| **I am scoping a deployment and need the rollout sequence** | 07-implementation/phased-rollout.md → programme-readiness.md |
| **I am defending a control choice to an auditor** | 06-compliance/ (control-mapping framework) + the policy file's Control mappings section |
| **I am cross-comparing vendors for a single capability** | 02-capabilities/<capability>.md (one per capability) or _capability-matrix.md for the summary |
| **I need to know where CASB fails** | 08-failure-modes/ (vendor-agnostic) + the MDA Gaps table |
| **I am new to CASB and need orientation** | 01-foundations/ |

## What this repo IS

- A practitioner's per-vendor reference (5 vendors, MDA at depth, 4 as drafts)
- A cross-vendor capability index
- A vendor-neutral policy library with per-vendor implementation grids
- A vendor-agnostic implementation and failure-mode reference

## What this repo IS NOT

[Keep existing "What this repo does NOT cover" section verbatim — already strong.]

## Component model

[Keep existing tree; update folder descriptions to reflect the proposed 03-policies/ restructure once done.]

## How to read a policy entry

[2-3 lines pointing the reader at the schema fields they should expect — purpose / action / scope / vendor grid / control mapping / FP risk / privacy.]

## How to read a vendor entry

[2-3 lines — Day 0 preconditions / gaps table / Day 1/30/90 policies / SIEM artefacts / SCIM cascade.]

## Three-lens review / Caveats / Operating rules

[Keep existing.]
```

The "Start here based on your question" table replaces "How to navigate". Same idea, more decision-grade entries, ordered by *frequency of architect query* — vendor playbook first, policy library second, rollout third, audit fourth, capability comparison fifth, failure modes sixth, orientation last.

## Top changes (rank-ordered)

1. **Rewrite `03-policies/_policy-archetypes.md` out of existence.** Replace with one file per policy, schema-driven, sub-foldered by control family (access / DLP / detect / OAuth / GenAI / posture). Each file ends with a 5-row vendor implementation grid linking to `04-vendors/<vendor>.md`. *This is the single change that converts the repo from a comparison document into a programme-design reference.*

2. **Add the per-policy → per-vendor cross-link grid as a first-class navigation surface.** Without it, the architect cannot answer "I have vendor X and need policy Y" without manual round-tripping across two cross-vendor matrices and a vendor file.

3. **Reorder the README's navigation table by query frequency, not folder number.** Vendor playbook (#1), policy library (#2), rollout sequence (#3) — not orientation (#1), foundations (#2), capability matrix (#3).

4. **Split `02-capabilities/_capability-matrix.md` into per-capability files.** Keep the matrix as a verdict-only summary; move the caveat-rich content into one file per capability so it can be cross-linked from policy files and vendor files without duplicating the prose.

5. **Re-section the capability matrix by control family, not by current ad-hoc grouping** ("Identity / OAuth / SSPM" is three different things; "Data residency / control plane" mixes residency with geo policy).

6. **Surface the MDA depth asymmetry on the README explicitly.** "Microsoft Defender for Cloud Apps: ~620-line worked playbook with Day 0 preconditions, ATT&CK matrix, KQL pack, SCIM cascade, DR matrix. Netskope / Zscaler / Palo Alto / Skyhigh: draft-quality, pre-three-lens-review, do not cite as audit evidence." The current "vendor drafts for…" phrasing badly under-describes the gap.

7. **Cross-link `04-vendors/<vendor>.md` Gaps table ↔ `08-failure-modes/<failure-mode>.md` files.** Same content, two folders, no joining. Pick one as canonical (recommend `08-failure-modes/` as the vendor-agnostic spine; vendor files cite into it). Where the MDA gap is genuinely vendor-specific (e.g. CAAC bilateral failure, OCAS subset SKU), leave inline in the vendor file with a note "vendor-specific; not in `08-failure-modes/`".

8. **Add a Day 1/30/90 ↔ Phase 1-5 cross-walk table** to `07-implementation/phased-rollout.md`. Currently the two cadence vocabularies do not reconcile.

## What you would NOT change

1. **`04-vendors/microsoft-defender-for-cloud-apps.md` structure.** Day 0 precondition checklist → one-page summary card → Gaps and compensating controls (read before any policy) → Day 1/30/90 → Appendices (SCIM cascade / Sentinel KQL / CAAC deprovisioning / DR matrix / ATT&CK+ATLAS / threat-actor profile / glossary / three-lens sign-off). This is the right shape for a vendor playbook. Apply it as the template for the other four vendors. **Do not redesign it.**

2. **The CLAUDE.md "strict separation — capabilities in 02-, policies in 03-" rule.** The rule is correct; the canonical files have drifted from the rule. Fix the files, not the rule.

3. **The thesis and the locked decisions.** Thesis-lock date is documented; the "practitioner per-vendor reference" framing is the right one. Do not let a navigation re-structure pull the thesis sideways.

4. **`07-implementation/programme-readiness.md` and `07-implementation/phased-rollout.md` content.** Both are vendor-agnostic, operationally tight, FS-shaped. The four readiness axes (SaaS estate / identity / endpoint / regulatory) and the Phase 1-5 sequence with named failure modes for reversing pairs are usable as-is in a real programme. Cross-walk to Day 1/30/90 (#8 above), don't rewrite.

5. **The promotion log discipline.** `00-meta/promotion-log.md` is the kind of artefact most research repos do not maintain. Keep the same format for the navigation restructure event when it lands.

6. **`08-failure-modes/` as a separate folder.** Architect-side, "where CASB fails" is its own reading task — preserve as a first-class navigation surface (also surfaced on the public-wedge candidate per CLAUDE.md). Cross-link to vendor-specific gaps; do not fold in.

7. **The five-specialist review pattern that produced the MDA v1 playbook.** Documented in `_research/2026-06-10-vendor-practitioner-reference/` per the README. Replicate for the other four vendors before promoting them out of draft. The structural fix above does not displace this — it sits on top of it.
