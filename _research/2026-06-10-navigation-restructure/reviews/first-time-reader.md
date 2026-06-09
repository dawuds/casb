# First-time reader, new to CASB - navigation review

Reviewer persona: mid-career cybersecurity practitioner, general infosec, never deployed a CASB, opened the repo because someone said "this will help you understand CASB and plan a deployment." Read the README first, then the files instructed.

Verdict up front: **the repo is a practitioner's filing cabinet, not a learning path.** It assumes I already know what CASB is, already know the capability/policy distinction, and already know which of the five vendor files is mine. If I do not, the README hands me eight numbered folders and an internal-research thesis statement and walks away. The content underneath is high quality. The doorway is wrong.

---

## Diagnosis

What is wrong, concretely, with file paths and headings I actually read.

### 1. The README does not tell a first-time reader what CASB is

`README.md` line 3 is the one-line description of the repo. It contains the words "Cloud Access Security Broker (CASB) — what the capability set actually is in 2026, how it maps to vendor products…" That is a statement about *this repo*, not about CASB. After reading the README I know what *the repo is about*. I do not know what a CASB *is*. The README sends me to `01-foundations/` for that — but `01-foundations/` is not in the "How to navigate" table's first row by accident of layout; it is one row of nine, with no visual weight, sitting next to "the cross-vendor capability picture" and "one vendor at depth," which are useless to me at minute one. There is no "start here if you have never deployed CASB" row.

### 2. The README opens with the thesis — a research artefact, not a reader artefact

`README.md` line 5 — **"Thesis (locked 2026-06-10): 'For each of the 5 CASB products in scope, what policies can a practitioner actually configure — and what does the tool not cover?'"** This is the *author's research framing*. To a first-time reader it is noise — I am not here to validate a thesis, I am here to learn CASB. The thesis belongs in `00-meta/`, not in the second paragraph of the front door.

### 3. The "How to navigate" table is keyed off questions I do not yet know to ask

The left column lists: "Orientation," "The cross-vendor capability picture," "The cross-vendor policy archetype list," "One vendor at depth," "Vendor drafts," "Reference architectures," "Compliance / regulator mapping," "How to actually implement," "Where CASB demonstrably fails." Six of those nine rows are useful only if I already have a vendor decision or a deployment context. A reader who has never touched a CASB needs *one* path: foundations → architecture → implementation. The table buries it.

### 4. The folder names 02-capabilities / 03-policies are the named confusion — confirmed

I read both `_index.md` files. The conceptual distinction is correct and worth preserving (capability = "tool can do"; policy = "firm decides to do"). The execution is the problem:

- **`02-capabilities/_index.md`** opens with one sentence of definition, then jumps to a schema, then lists 14 unwritten capability entries. The reader has no example. The actual content I want — the capability matrix — is one folder deeper at `02-capabilities/_capability-matrix.md`, not surfaced in `_index.md`.
- **`03-policies/_index.md`** opens with one sentence of definition, then a schema, then a list of unwritten policy entries. Same shape. Same problem. Same "go look in `_policy-archetypes.md`" miss.
- Worse: the folders are named with the *abstract noun* ("capabilities," "policies") rather than what they actually contain ("**what tools can do**," "**what firms enforce**"). A first-timer reading the folder list cannot tell `02-capabilities/` from `03-policies/` from `05-architecture/` from `08-failure-modes/` without opening at least three of them.
- The "Strict separation" rule (capability in `02-`, policy in `03-`) is internal editorial discipline. The reader does not need that rule surfaced as a navigation principle. It reads as a warning notice on a fence: *don't cross the streams* — but the reader does not know what the streams are.

### 5. Capability matrix and policy archetypes file the same data twice, from two angles, with no signposting

`02-capabilities/_capability-matrix.md` columns are vendors, rows are capabilities. `03-policies/_policy-archetypes.md` rows are archetypes (policy patterns), inner rows are vendors. **Both files cite the same five `_research/2026-06-10-vendor-practitioner-reference/raw/<vendor>.md` source drafts.** That is fine; they are different lenses on the same evidence. But neither file says so at the top. A first-time reader, having dutifully read both, cannot articulate *why both exist*. The 03 file's "How to read this" section helps; the 02 file has no equivalent.

### 6. `04-vendors/microsoft-defender-for-cloud-apps.md` is brilliant but mislabelled in the README

The MDA file is the deepest and most useful artefact in the repo. The README calls it "One vendor at depth." That under-sells it. It is a Day-1 / Day-30 / Day-90 implementation playbook with a precondition checklist, a gaps table, ATT&CK mappings, and KQL — it is closer to "the example deployment plan in this repo" than to "a vendor reference." For a first-timer with MDA in their tenant (the user's actual case) this should be flagged from the README as **the worked example to read after the foundations**, not buried at row 4 of a 9-row table.

### 7. `07-implementation/programme-readiness.md` is the *real* starting point for anyone planning a deployment — and it is not flagged as such

The readiness file is a textbook Day-0 inventory: four axes, 13-line inventory checklist, four named owner roles, seven locked-decisions table, five failure modes. This is what someone planning a deployment actually needs to read first. It is hidden behind the generic README row "How to actually implement (programme readiness, phased rollout, SOC integration, metrics, change management, TCO)." That row reads like a TOC dump; nothing in it says "read this before you do anything else."

### 8. The component-model code-block under "How to navigate" duplicates the table above it

`README.md` lines 28-40 give a code-fenced folder tree with one-line descriptions. Lines 14-25 give a table with one-line descriptions. Same information, two formats, no value-add from the duplication. Pick one.

### 9. The promotion log is in the navigation path for someone who shouldn't see it

`00-meta/promotion-log.md` is a workflow artefact — useful to the author and to a returning collaborator. A first-time reader has no use for it. The README points to it at line 7 as the status indicator. Fine — but it should be the *only* meta-level pointer in the README front matter, not the second line.

---

## What 02-capabilities/ should be

Rename or re-frame. The folder is the *answer to the question* "**what can a CASB do?**" — not "the capability catalogue."

- **`_index.md` should lead with a 2-3 sentence definition of what a capability is**, plus **one worked example** (e.g. "Shadow-IT discovery is a *capability*: the tool reads SWG/firewall logs and lists every cloud app users touched. Whether your firm chooses to *block* the unsanctioned ones is a *policy*, in `03-policies/`."). The current `_index.md` defines by negation against `03-policies/`, which is circular if the reader has not yet read `03-`.
- The schema block should move below the worked example, or to a separate `_schema.md` file. A first-time reader does not need the schema on minute one.
- The list of 14 capabilities-to-draft is *promissory* — it says "here is what we plan to write." That is fine in a project tracker; it is wrong-shaped for a public-facing index. **Either write the entries or remove the list from `_index.md`.** The single canonical artefact in this folder today is `_capability-matrix.md`. Surface it.
- The `_capability-matrix.md` file itself is sound. Add a 3-line "How to read this" at the top (vendor codes, cell legend, source provenance) — the file does have one buried at lines 5-8, but it reads as fine print rather than the entry point.

**Proposed folder framing:** keep the name `02-capabilities/`. Front-load `_index.md` with the worked example + a direct link to `_capability-matrix.md`. Move the schema and the to-draft list to `_meta.md` or to the bottom of `_index.md` under "internal scope tracking."

---

## What 03-policies/ should be (the user-named confusion — go deep)

This is the harder one. The confusion is not solved by a better intro paragraph. The structural decisions need re-stating.

### What is currently wrong

**Problem 1 — the word "policy" is overloaded.** In CASB-vendor-speak, a "policy" is a configured rule in the console (e.g. "MDA file policy #4: PCI label auto-apply"). In risk-management-speak, a "policy" is a governance document (information security policy, acceptable use policy). In this repo, `03-policies/` means **policy archetypes** — the *pattern* of decision a firm can make, abstracted across vendors. None of those three meanings is signposted. A first-time reader sees "policies" and assumes the second meaning (governance docs), then opens the matrix and finds vendor configuration patterns, then has to reconcile.

**Problem 2 — `_policy-archetypes.md` and `_capability-matrix.md` look like the same file at first glance.** Both are wide tables, vendors as columns / archetypes-or-capabilities as rows, same five vendor codes, same `[unverified]` flagging convention, same source drafts. The reader cannot tell *why both exist* without reading both end-to-end. The 03 file does a better job of explaining itself ("How to read this" section, lines 5-11). The 02 file does not.

**Problem 3 — the strict 02/03 separation is presented as editorial discipline, not as a navigation principle the reader needs.** From `CLAUDE.md` line 45: "*Capability vs Policy separation: Strict — capabilities in 02-, policies in 03-, never inline together. The user's framing conflated them; the discipline forces clarity.*" This is *author-facing* — it tells the author not to mix them. The reader needs the *inverse* signpost: "If you are asking 'what can the tool do?', go to 02. If you are asking 'what should my firm decide to enforce?', go to 03."

**Problem 4 — the actual policy artefact lives in the MDA vendor file, not in `03-policies/`.** The MDA file's Day 1 / 30 / 90 policies (13 of them) are the real worked-example "policies" in the repo. `03-policies/_policy-archetypes.md` is a cross-vendor abstraction over those + the four other vendor drafts. So the reader hitting `03-policies/` first sees the abstraction without having read any of the concrete instances. **Abstract before concrete is the wrong order for a learner.**

### What it should be

Three concrete changes:

1. **Rename the folder for what it answers**, or at minimum re-frame the `_index.md` with that question:
   - **Currently named:** `03-policies/`.
   - **What it answers:** "What enforcement decisions does a firm have to make when deploying a CASB, and which vendor supports which decision?"
   - **Option A — rename:** `03-policy-decisions/` or `03-enforcement-patterns/` or `03-policy-archetypes/`. "Archetypes" is the right word; surface it.
   - **Option B — keep the folder name and re-frame `_index.md`** with the question explicitly. This is the lower-effort, lower-blast-radius option and probably the right one given existing links.

2. **`_index.md` should open with the 02/03 navigation principle, expressed for the reader, not the author.** Two sentences:
   > **If you are asking what a CASB *can* do, see [`02-capabilities/`](../02-capabilities/).** This folder catalogues the *policy decisions* a firm has to make when deploying CASB — what to block / coach / monitor / quarantine, for which users, against which apps, with which exception path. Each archetype names the capability it depends on, the control it generates evidence for, and the false-positive risk it carries.

3. **`_policy-archetypes.md` should lead with one worked example, not the index table.** Show what an archetype looks like — pick #4 (Inline DLP on upload) because it is the most intuitive, walk the reader through purpose → scope → deployment-mode requirement → vendor variation in one screen — *then* hand them the 14-row index. The current order (index table → 14 walkthroughs) is right for a reference document, wrong for a first read.

### What I would NOT do to 03-policies/

- **Do not merge it into `02-capabilities/`.** The conceptual distinction is real and load-bearing. Vendor RFPs and audit reports both routinely confuse "vendor supports X" with "firm enforces X," and the separation here is exactly what makes the matrices useful. Keep two folders. Fix the framing.
- **Do not split policies into per-archetype files yet.** The folder index lists 14 archetypes; only the cross-vendor matrix exists. Per-archetype files would be the right v0.2 move once each is fleshed out. Today, the matrix is the asset. Do not fragment it prematurely.

---

## Proposed README structure

Replace the current README with this shape. Section headings + one-line descriptions.

```
# CASB practitioner reference

## What this is
One paragraph. What CASB is in 2026 (capability set mostly absorbed into SSE).
What this repo is (practitioner reference, 5 vendors, anchor MY).
What this repo is not (not a buyer's guide, not legal advice).

## Start here
Three-row table for three reader archetypes:
| If you are | Read in this order |
|---|---|
| New to CASB | 01-foundations/what-casb-is-and-isnt.md  →  01-foundations/deployment-modes.md  →  07-implementation/programme-readiness.md |
| Planning a deployment | 07-implementation/programme-readiness.md  →  07-implementation/phased-rollout.md  →  04-vendors/<your-vendor>.md  →  08-failure-modes/ |
| Already running MDA | 04-vendors/microsoft-defender-for-cloud-apps.md  (Day 0 checklist → Gaps table → Day 1 policies) |

## What's in each folder
The current "Component model" code block, but trimmed — one line per folder, no duplication with a separate table.

## What this repo does NOT cover
Keep current list.

## Caveats
Keep current list (regulatory illustrative-only, vendor-controlled, fast-moving).

## For collaborators
Pointer to CLAUDE.md, three-lens review pattern, promotion log, _research/.
This is the ONLY meta-pointer block in the README.
```

The thesis statement goes to `00-meta/thesis.md` (where the README already links it) and out of the README front matter. A reader who needs the thesis will go find it.

---

## Top changes (rank-ordered, concrete)

1. **Rewrite the README opening to lead with a reader-archetype "Start here" table, not the thesis.** Three rows: new-to-CASB / planning-deployment / already-running-MDA. Each row gives the *file path sequence* to read, in order, with hyperlinks. The current 9-row "How to navigate" table goes below this, retitled as "Browse by topic." Impact: highest. Effort: 30 minutes. Risk: zero (additive).

2. **Add a `01-foundations/_index.md` (if not present) and surface `01-foundations/what-casb-is-and-isnt.md` from the README front and centre.** The repo has the orientation content (per the promotion log line 24); the README does not route a first-timer to it. A new reader should be one click from "what is a CASB" — not from "what is this repo about a CASB." Effort: 15 minutes.

3. **Rewrite `02-capabilities/_index.md` and `03-policies/_index.md` with one worked example each, before the schema.** Current `_index.md` files lead with abstraction. The reader needs concrete first. Pick one capability (Shadow-IT discovery) and one policy archetype (Inline DLP on upload). Walk through. Then the schema, then the to-draft list. Effort: 1 hour each. Impact: this is what resolves the user-named confusion.

4. **Add a 3-sentence "How `02-capabilities/` and `03-policies/` differ" callout** — either in the README "Start here" section or in both `_index.md` files. Phrased for the reader, not for the author. "Capabilities = what the tool *can* do. Policies = what your firm *decides* to enforce on top. The 02 matrix tells you what to expect in an RFP demo. The 03 matrix tells you what decisions you have to make in design workshops." Effort: 10 minutes. Impact: high — directly addresses the named confusion.

5. **Re-flag `04-vendors/microsoft-defender-for-cloud-apps.md` in the README as "the worked deployment example."** Not "one vendor at depth." The MDA file is structured as a *playbook* (Day 0 / 1 / 30 / 90), not a vendor brochure. For anyone planning an MDA deployment specifically, *or* anyone wanting to see what a good CASB deployment plan looks like, that file is the answer. The README should say so. Effort: 5 minutes. Impact: high for the user's primary case.

Lower-priority but worth queueing:

6. **Add `_index.md` "How to read this" stanzas to `_capability-matrix.md` and any folder where one is missing.** The 03 file has it (lines 5-11). The 02 file does not. Pattern-match.

7. **Move the thesis line out of the README front matter to a clearly labelled "Author's research framing" footnote at the bottom**, or remove it entirely and rely on the `00-meta/thesis.md` link. A public-audience README does not lead with a research thesis.

8. **Move the `00-meta/promotion-log.md` link out of the README front matter into a "For collaborators" section at the bottom.** A first-time reader has no use for a promotion log. A returning author does.

---

## What you would NOT change (what is currently working)

Credit where due — the content is strong. The navigation problem is structural, not editorial. Specifically:

- **The folder numbering scheme (`01-foundations`, `02-capabilities`, … `08-failure-modes`).** Numbered prefixes do impose an order, which is what a learner needs. Do not abandon the numbering for "flat" folders — that would make it worse.
- **The 02/03 conceptual separation.** It is right. It is just badly signposted. Fix the signposting, do not collapse the folders.
- **`07-implementation/programme-readiness.md`.** This file is the model for what every spine page should look like — a clear four-axis frame, an actionable inventory, named owner roles, named failure modes. It is the right *shape*; the README just doesn't point at it loudly enough.
- **`07-implementation/phased-rollout.md`.** Same praise. Discovery → risk-scoring → enforcement is the right phased model, the done-criteria per phase are concrete, the "failure modes from reversing the sequence" table is the kind of hard-marker content that distinguishes this from vendor blogware.
- **`04-vendors/microsoft-defender-for-cloud-apps.md`.** The Day 0 precondition checklist + Gaps and compensating controls table at the top is the single best on-ramp pattern in the repo. Reuse this template for the other four vendor files when they get promoted to MDA depth (the promotion log already names this as outstanding work).
- **The `[unverified]` flagging convention in the matrices.** That kind of honest source-quality marking is the opposite of vendor-marketing-driven content and is exactly what a regulated-FS practitioner needs. Keep it. Make sure it is explained at the top of any file that uses it.
- **`08-failure-modes/` exists as a discrete folder.** Most CASB write-ups bury limitations in footnotes. Promoting failure modes to a top-level folder is a strong editorial decision. Keep that visible from the README.
- **The "What this repo does NOT cover" and "Caveats" sections of the README.** Both are honest, scoped, useful. The "vendor capability claims are vendor-controlled" caveat in particular is the kind of disclosure that builds trust with a security audience. Do not lose these in the rewrite.
- **`CLAUDE.md`.** The author-facing instruction set is excellent — citation discipline, sanitisation rules, three-lens review pattern, common pitfalls. Leave it untouched. It is doing its job.

---

End of review.
