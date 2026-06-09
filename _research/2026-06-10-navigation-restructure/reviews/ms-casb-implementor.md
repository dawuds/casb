# Expert Microsoft CASB implementor - navigation review

Persona: senior MDA implementor, 20+ regulated-FS deployments. I have spent today opening the Defender portal at `security.microsoft.com` → Cloud apps → Policies → Policy management. When a doc says "policies" I expect a numbered list of things to switch on, in priority order, with what each reduces. I judge `03-policies/` against that bar.

---

## Diagnosis

The repo's centre of gravity is in the wrong folder for the user's own thesis.

The thesis (locked, `README.md` line 5 and `CLAUDE.md` line 9) is:

> *"For each of the 5 CASB products in scope, what policies can a practitioner actually configure — and what does the tool not cover?"*

A practitioner reading that thesis lands on the README and looks for "policies." They click `03-policies/_policy-archetypes.md` and get a 14-row cross-vendor *comparison matrix* — vendor support, caveat per cell, where it `Maps to 03-policies/`. The file is named "archetypes" and reads as a taxonomy. It is not a policy list. It is a *vendor capability table re-grouped by use case*. That is `02-capabilities/_capability-matrix.md` cut a different way.

Concretely:

- **`03-policies/_index.md` line 5** declares the discipline: *"A policy is what the firm decides to enforce."* Line 8: *"A real policy entry says: which apps, which users, which action, under which conditions, with which exception path, evidencing which control, at what false-positive cost."* That is the right schema. Then the file does not deliver it. The "Policy archetypes to draft" list (lines 27-40) is a slogan list — *"Unsanctioned-app block (with coach-and-redirect-to-sanctioned variant)"*, *"OAuth-app grant restriction (publisher / scope / risk-score gating)"*. These are policy *families*, not policies. The schema fields (Scope / Action / Required capabilities / Control mappings / False-positive risk / Operational cost / Privacy considerations) are declared and then not populated for any of the 14 archetypes in the sibling file.

- **`03-policies/_policy-archetypes.md`** then takes the slogan list and adds vendor support rows underneath each. Archetype #4 "Inline DLP on upload" has five vendor cells, each saying "Supported, see policy #X in vendor draft." That is the *capability matrix view* presented as policy content. There is no policy *entry* in this folder. There are zero files matching the schema declared in `_index.md`.

- **Compare to `04-vendors/microsoft-defender-for-cloud-apps.md`.** That file delivers what `03-policies/` promises. Policy 5 (Block download of Confidential-labelled files to unmanaged browsers) has: Risk reduced, What it does, ATT&CK technique, Critical prerequisite, Console path, Key configuration, Trap to avoid, Defeated by, Auditable evidence, SIEM artefact. That is a policy entry. The MDA file alone carries 13 of them, sequenced Day 1 / Day 30 / Day 90, with the bilateral CAAC trap, the SCIM cascade table, the Sentinel KQL appendix. **The real policy library is inside the vendor file, not in `03-policies/`.**

- **Capability vs policy split.** `CLAUDE.md` line 109-110 codifies the discipline: capability = "the tool can…", policy = "the firm has decided to…". That distinction holds in academia. It does not survive contact with the practitioner question. The practitioner does not ask "what can MDA do" separately from "what should I switch on." They ask one question: *"what is the configurable list of things, in what order, with what trade-offs."* `02-capabilities/_capability-matrix.md` and `03-policies/_policy-archetypes.md` are the same question cut two ways. The reader pays the cost twice.

- **Structural difference vs vendors / architecture / compliance / implementation.** Those four folders each answer a *clean* question: vendors = "what does this product do," architecture = "where does it sit," compliance = "what control does it map to," implementation = "how do I roll it out." Each is a single-axis cut. `02-capabilities/` and `03-policies/` are the *same axis cut twice* with a definitional fence between them. That is why the user is confused: the other four folders are domains; capabilities and policies are not domains, they are two views of the same domain.

- **README pathway disconnect.** README line 17 sends "cross-vendor policy archetype list" to `03-policies/_policy-archetypes.md`. Line 18 sends "one vendor at depth — 13 configurable policies" to `04-vendors/microsoft-defender-for-cloud-apps.md`. The word "policy" carries two different meanings on adjacent lines. Line 17 means *cross-vendor matrix view*; line 18 means *configurable policy with console path*. A practitioner who clicks line 17 hoping for line 18's content bounces.

- **Component model line 33 `04-vendors/`** says the vendor file is canonical for per-vendor depth. `CLAUDE.md` line 13 confirms: *"Primary unit of output: `04-vendors/<vendor>.md` — each carries three named sections — Capability set / Configurable policies / Limitations. Synthesis pages (`02-capabilities/`, `03-policies/`, `08-failure-modes/`) aggregate across vendors, but the per-vendor file is canonical."* This is correct. But the README does not signal it. README's "How to navigate" table treats `02-` and `03-` as first-class destinations equal to `04-`. They are not first-class. They are derivative. The README understates that hierarchy.

---

## What `02-capabilities/` should be

Drop the schema-driven "one file per capability" plan declared in `02-capabilities/_index.md` lines 22-37. It is a 14-entry catalogue the user does not need and will not maintain.

Keep one file: the cross-vendor matrix (already at `_capability-matrix.md`). Rename it `vendor-capability-matrix.md`. Promote the "What the matrix reveals" section (lines 78-96 of the matrix file) to the top — that is the read.

Rewrite the folder's role from "capability catalogue" to **"cross-vendor capability comparison."** One file. One purpose: when a practitioner is told "all five vendors do DLP," this file shows what that claim hides per cell. That is the matrix's actual job.

Delete the "Capabilities to draft" list in `_index.md`. The repo has chosen the matrix as the canonical synthesis (per the promotion log line 11). The individual capability files were aspirational; they should be retired from the plan.

---

## What `03-policies/` should be

This is the user-named confusion. Three options. I will name the right one.

### Option A — Delete `03-policies/` as a folder. Refold the content.

The cleanest answer. There is no policy catalogue in this repo today. There is a cross-vendor archetype-comparison matrix (currently at `_policy-archetypes.md`) which is structurally a *use-case-grouped capability matrix*. Move it to `02-capabilities/use-case-grouped-matrix.md` and let the two files in `02-` be: feature-grouped matrix + use-case-grouped matrix. Same domain, two cuts.

Then the *actual* policy library — the configurable things to switch on, in order — lives where it already lives: inside each `04-vendors/<vendor>.md`. The MDA file shows the shape. Add to the README a single sentence: *"Configurable policies live inside each vendor file, in Day 1 / Day 30 / Day 90 order."*

This option resolves the confusion by deletion. The capability/policy fence in CLAUDE.md becomes a *vendor-file internal section structure* (the three named sections "Capability set / Configurable policies / Limitations" from CLAUDE.md line 13), not a folder boundary. That is the correct level of granularity.

### Option B — Extract the MDA Day 1 / 30 / 90 pattern cross-vendor. Make `03-policies/` a vendor-neutral policy library.

Aspirational. Will not survive the v0.1 gate. The MDA file is v1 with five specialist reviews; the other four vendor drafts are *not* at that depth (per promotion log line 16-19: "Promoted as draft — not lens-reviewed at MDA depth"). Extracting cross-vendor Day 1 / 30 / 90 policies requires the other four vendor files to be promoted to MDA depth first. The outstanding-items list in the promotion log (line 75) explicitly defers this: *"Netskope / Zscaler / Palo Alto / Skyhigh practitioner playbooks at MDA depth — TBD — Apply the MDA template; run 5-specialist review per vendor."*

Until that work lands, every cross-vendor policy entry would have one anchored row (MDA) and four `[unverified]` rows. That is what `_policy-archetypes.md` already looks like. Re-doing it under a Day 1 / 30 / 90 frame would only restructure the unreliability, not remove it.

If this option is pursued *after* the other vendor files reach v1, each `03-policies/<policy>.md` file should look like:

- **Risk reduced** (one sentence)
- **Vendor support per archetype** (5-row mini-matrix: action verb, scope, console path, deployment-mode dependency, deployment effort)
- **Trade-off** (what it breaks, false-positive cost, privacy load)
- **Day-of-rollout placement** (Day 1 / 30 / 90 per vendor, since the cadence varies — MDA's Day 90 may be Day 30 for Netskope on the same archetype)
- **Control mappings** (BNM RMiT clause, ISO 27017 control, NIST CSF subcategory)
- **Compensating controls when vendor X cannot do it**

But this is a v0.2 deliverable, not v0.1.

### Option C — Keep `03-policies/` but rewrite it as the *vendor-neutral decision frame*

Make `03-policies/` answer: *given a regulator-driven control objective, what does the firm need to decide, and what does that decision constrain at the vendor level?* This is the "policy-as-firm-decision" reading of CLAUDE.md.

Content would be:
- Policy decisions a CASB programme must take (BYOD position, personal-use position, action-class default, exception process — already drafted in `07-implementation/programme-readiness.md` lines 52-59)
- For each decision, the downstream constraint on vendor selection and configurable enforcement
- Each decision frame linked to: relevant archetypes in the capability matrix, the control mappings it generates evidence for, the privacy/regulatory considerations

This rescues the capability-vs-policy fence — it puts genuinely *firm-decision* content in `03-policies/` instead of more vendor-feature comparison. But it duplicates `07-implementation/programme-readiness.md`. The "Decisions that must be locked before policy design" table there (lines 50-59) is exactly this content already.

### Verdict

**Option A.** Delete `03-policies/` as a folder. The capability/policy fence is academic; the practitioner does not need both folders. The matrix in `_policy-archetypes.md` is a second cut of the same data and belongs alongside the feature-grouped matrix. The actual policy library is in the vendor files. The decision-frame content is in `07-implementation/programme-readiness.md`. There is no residual content that needs `03-policies/` as a home.

Option B is the right answer *after* all five vendor files reach v1. Until then it is theatre.

---

## Proposed README structure

```
# casb

[Thesis, locked date]
[Status, v0.0]

## Start here — what to read first

| You are | Read |
|---|---|
| New to CASB / SSE | 01-foundations/ — what CASB is, deployment modes, where it sits in SSE |
| Already own Microsoft Defender for Cloud Apps and need to know what to switch on | 04-vendors/microsoft-defender-for-cloud-apps.md — 13 numbered policies, Day 1 / 30 / 90, with the bilateral CAAC trap and KQL appendix |
| Choosing between vendors | 02-capabilities/ — two matrices: by feature (vendor-capability-matrix.md), by use case (use-case-grouped-matrix.md) |
| Running an implementation programme | 07-implementation/ — readiness, phased rollout, SOC integration, metrics, change, cost |
| Asked by a regulator / auditor what control this satisfies | 06-compliance/ — control mapping framework + BNM RMiT / ISO 27017 / ISO 27018 |
| Asked "why didn't CASB catch this" after an incident | 08-failure-modes/ — BYOD, OAuth, SSL/TLS, API-mode, encrypted upload, over-blocking, lock-in, cost |

## How the content is organised

The canonical content is the per-vendor file. Configurable policies — the things to switch on in the vendor console — live inside `04-vendors/<vendor>.md` in the "Day 1 / Day 30 / Day 90" sections, with risk-reduced, console path, traps, defeats, and audit evidence per policy.

Everything else aggregates across vendors:
- `02-capabilities/` — what the products can do (feature comparison + use-case comparison)
- `06-compliance/` — what regulators expect; how vendor capabilities generate control evidence
- `07-implementation/` — how to roll out, vendor-agnostic
- `08-failure-modes/` — where CASB demonstrably fails

## Component model

[Existing block, with `03-policies/` removed]

## Three-lens review pattern
[Existing block]

## What this repo does NOT cover
[Existing block]

## Caveats
[Existing block]

## Operating rules
[Existing block]
```

Key shifts vs current README:

- **"How to navigate" renamed to "Start here — what to read first."** Reader-state-led, not file-led.
- **The MDA vendor file is the second row, not the fourth.** It is the deepest worked example and the user's own immediate case. Promote it.
- **`02-capabilities/` is described as "two matrices."** No mention of capability catalogue / one-file-per-capability. The plan is dropped.
- **`03-policies/` is removed from the table and the component model.** The folder is deleted.
- **One sentence on hierarchy:** "configurable policies live inside each vendor file." This is the single missing sentence in the current README that costs the practitioner the most.

---

## Top changes (rank-ordered)

### 1. Delete `03-policies/` as a folder. Move `_policy-archetypes.md` to `02-capabilities/use-case-grouped-matrix.md`.

This is the change. Everything else is detail.

Mechanics:
- `git mv 03-policies/_policy-archetypes.md 02-capabilities/use-case-grouped-matrix.md`
- Delete `03-policies/_index.md` (its declared schema was aspirational and never populated)
- Delete the `03-policies/` directory
- Update the promotion log to record the move (do not pretend it didn't happen)
- Update CLAUDE.md line 13 to drop the `03-policies/` synthesis-page reference
- Update CLAUDE.md line 25 "Component model" block to remove the `03-policies/` row
- Update CLAUDE.md line 45 "Capability vs Policy separation" — change to "Capability/policy split lives inside each `04-vendors/<vendor>.md` as Capability set / Configurable policies / Limitations sections, not at the folder level"
- Update CLAUDE.md line 109-110 pitfall row — same reframing

The capability/policy discipline is *preserved* — it just moves down one level, from folder boundary to per-file section structure. The MDA file already implements it that way. CLAUDE.md just needs to catch up.

### 2. Rewrite README "How to navigate" → "Start here — what to read first." Lead with reader state.

Current pattern: "If you want X, read folder Y." File-led. Reader has to translate their question to one of the seven offered phrasings.

Replacement pattern: "You are [persona / scenario], read [destination]." State-led. The MDA-tenant practitioner sees themselves in row 2 and clicks. The auditor sees themselves in row 5. The first-vendor-shortlist practitioner sees themselves in row 3.

Add the single load-bearing sentence: *"Configurable policies live inside each vendor file in Day 1 / Day 30 / Day 90 order. The per-vendor file is the canonical unit. Cross-vendor matrices in `02-capabilities/` are the comparison view."*

### 3. Make the MDA file's "Day 1 / Day 30 / Day 90" structure visible at README level.

Promote it into the README itself as a single one-line summary card at the top of the navigation section. Format:

```
Worked example — Microsoft Defender for Cloud Apps:
  Day 1  (no CAAC) — Shadow IT discovery / OAuth post-consent / mass-download alert / impossible-travel alert
  Day 30 (CAAC) — sensitive-label download block / auto-label PCI / quarantine stale shares / mass-delete / B2B exfil / App Governance Predictive Risk
  Day 90 (advanced) — auto-unsanction GenAI / clipboard-to-ChatGPT block / terminated-user / IRM signal boost
  See 04-vendors/microsoft-defender-for-cloud-apps.md for the configuration recipe per policy.
```

This makes the *shape* of the deliverable visible from the README. A practitioner can see: "yes, this repo will answer my question, in the format I need." That signal does not exist in the current README.

---

## What you would NOT change

The following structural choices are correct and should be preserved.

- **`04-vendors/microsoft-defender-for-cloud-apps.md` as canonical unit and primary depth.** The Day 1 / 30 / 90 cadence, the Risk-reduced → Console-path → Trap → Defeated-by → Auditable-evidence → SIEM-artefact policy schema, the bilateral CAAC trap, the SCIM cascade catalogue, the Sentinel KQL appendix, the Gaps-and-compensating-controls table at the top, the pre-CAAC compatibility test list, the concentration-risk section. This is the right shape for a practitioner reference. Replicate it across the other four vendor files. Do not rewrite it.

- **`02-capabilities/_capability-matrix.md` as it stands (after renaming the file).** The matrix structure (capability × vendor with S/C/N/U coding and lens-review unverified flags) is correct. The "What the matrix reveals" summary section is the strongest synthesis content in the repo after the MDA file. Keep it.

- **`07-implementation/`.** Programme readiness, phased rollout, SOC integration, success metrics, change management, cost of ownership. Six files. Clean axis. Vendor-agnostic. This is the cleanest folder in the repo and the user said so. Do not touch it.

- **`08-failure-modes/`.** Eight named failure modes (BYOD, OAuth, SSL/TLS, API-mode, encrypted-upload, over-blocking, lock-in, cost). Each addresses a specific common pitfall. The folder is the public wedge candidate per CLAUDE.md line 30. Do not touch it.

- **`06-compliance/`.** Control mapping framework + framework pages (BNM RMiT, ISO 27017, ISO 27018). The structure of "framework page per regulator/standard, control-mapping methodology page above" is correct. The outstanding work is content depth (full clause-by-clause mapping, listed in promotion log line 67-68), not structure.

- **The three-lens review pattern.** Architect / Product / Compliance. Documented in CLAUDE.md and `00-meta/three-lens-review.md`. Applied to the MDA v1 file. This is methodology, not navigation, and it works.

- **`00-meta/promotion-log.md`.** The discipline of recording what was promoted, what is outstanding, and what criteria were relaxed at promotion time is correct and should be preserved. The proposed `03-policies/` deletion should be recorded there as a structural change at promotion.

- **The thesis lock.** Locked 2026-06-10. Per-vendor practitioner reference. Do not destabilise the thesis to fix the navigation; fix the navigation to serve the thesis.
