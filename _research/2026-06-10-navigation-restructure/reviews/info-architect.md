# Information architect / technical documentation expert - navigation review

Scope: `README.md`, `02-capabilities/`, `03-policies/`, the reader pathway from arrival to useful content. Adjacent files (`04-vendors/microsoft-defender-for-cloud-apps.md`, `07-implementation/programme-readiness.md`, `07-implementation/phased-rollout.md`, `00-meta/promotion-log.md`) read for context.

---

## Diagnosis

The "How to navigate" table at `README.md` lines 13-24 fails on four counts. Naming them in order of damage.

**1. Mixed axis of organisation.** The first column "If you want" is not a single axis — it interleaves at least four:

- A **reader goal / question** ("orientation", "how to implement")
- A **content type** ("vendor drafts", "reference architectures")
- A **cross-cutting view** ("cross-vendor capability picture", "cross-vendor policy archetype list")
- A **failure framing** ("where CASB demonstrably fails")
- A **workflow surface** ("research scratch + workflow output")

A reader's eye cannot find its row because rows do not share a class. This is the jumbled feel — eleven things, no taxonomy.

**2. Wrong granularity in the right-hand column.** The "Read" column mixes folder pointers (`01-foundations/`), single-file pointers (`02-capabilities/_capability-matrix.md`), and folder-with-six-files pointers (`07-implementation/`). One row lands the reader at a 600+ line single page (MDA), another at a folder of six files (`07-implementation/`), another at a stub folder. The cost of a click varies by an order of magnitude with no signal which is which.

**3. The MDA row is a paragraph, not a row.** Line 18 reads "13 configurable policies (Day 1 / 30 / 90), the bilateral CAAC trap, the gaps-and-compensating-controls table, ATT&CK + ATLAS coverage matrix, KQL starter pack" — that is the table of contents for the MDA file embedded in the navigation table. Every other row gets one phrase. The asymmetry telegraphs "this is what the repo is really about" but undermines the table as scanning surface.

**4. Two parallel reading paths declared, not chosen.** "How to navigate" and "Component model" are both at top level and both describe the same folder set. Lines 13-24 say "go here for this question." Lines 28-40 say "here is what each folder is for." A first-time reader does not know which to read first. They overlap ~80% and disagree on framing — the table sells folders by reader goal; the component model sells folders by content type. Pick one as the entry point or merge them.

Other diagnostic notes:

- The thesis sentence at line 5 is the single most useful piece of orientation in the file. It is buried below the title and above the navigation table — and never referenced again. The reader who reads it and then hits the table does not see the table organised around it.
- The line at line 11 "Start where the question lives" is the right framing — but the table that follows is not organised by question. It is organised by folder, with questions retrofitted onto rows.
- `02-capabilities/_index.md` and `03-policies/_index.md` are well-written individually but neither names the other or explains the relationship. A reader landing in `02-capabilities/` cannot see why `03-policies/` exists as a sibling without reading both `_index.md` files and triangulating.
- `CLAUDE.md` lines 109-110 ("Capability = 'the tool can…'; Policy = 'the firm has decided to…'") is the cleanest framing of the distinction in the repo. It is in the rules file, not the navigation. A public reader will never see it.

**Can a reader identify their reading path within 30 seconds?** No. The reader reads the title, reads the thesis, hits an 11-row table where rows do not share a class, defaults to the first row ("Orientation"), and lands in `01-foundations/` regardless of why they came. The repo's actual centre of gravity — the MDA vendor playbook — is row 4 of 11 and gets read by readers who scroll, not by readers who scan.

---

## What 02-capabilities/ should be

`02-capabilities/` answers one question: **what can the tooling category do, abstracted across vendors?**

That is a reference question. The user lands here when they want the cross-vendor cell-by-cell view — "does every vendor support inline DLP, and if so with what caveats?" The matrix at `02-capabilities/_capability-matrix.md` already serves that question well.

What is wrong now:

- `_index.md` advertises a per-capability file pattern (Shadow-IT discovery, OAuth governance, etc.) at lines 22-37. None of those files exist. The matrix is the only artefact. The folder shape promises a granularity it does not deliver.
- The matrix is named `_capability-matrix.md` with an underscore. By the repo's own convention (numbered prefixes for ordered series; `_` prefix elsewhere for indices/schemas), an underscore-prefixed file reads as scaffolding, not as the canonical artefact. It is the canonical artefact. Rename.
- The folder's `_index.md` does not name `_capability-matrix.md` as the primary read. A reader entering the folder via `cd` or directory listing sees `_index.md` first and reads a schema, not the content.

What it should be:

- **One canonical matrix file** as the primary artefact (the current `_capability-matrix.md`, renamed `capability-matrix.md` or just `capabilities.md`).
- **An `_index.md` that points to the matrix in its first paragraph,** explains the cross-vendor question it answers, and lists the per-capability files only if/when they exist. Do not advertise unwritten files.
- Per-capability deep-dive files (`shadow-it-discovery.md`, `oauth-governance.md`) only when there is something to write that does not belong in the matrix. The matrix should be the default. Per-capability files are exceptions.

---

## What 03-policies/ should be

This is the user-named confusion. Goes deep.

**Diagnosis of the current state.** `03-policies/` and `02-capabilities/` look like siblings in the folder tree, in the README "How to navigate" table (rows 3 and 4), and in the component model. They are *not* siblings as reader artefacts. They are different abstractions over the same underlying material:

- A **capability** is a vendor property — "this tool can do X."
- A **policy archetype** is a deployment decision pattern — "if you want to enforce Y, here is the configuration shape across vendors."

The CLAUDE.md framing ("Capability = the tool *can*…; Policy = the firm has *decided to*…") is right as a discipline, but it does not survive contact with the artefacts. Look at the actual content:

- `02-capabilities/_capability-matrix.md` row "Inline DLP for sanctioned SaaS (pre-upload block)" — describes which vendors can do it, with caveats.
- `03-policies/_policy-archetypes.md` archetype #4 "Inline DLP on upload (pre-upload block)" — describes which vendors can do it, with caveats.

These are the same content viewed twice. The matrix is the capability cell. The archetype entry is the policy-flavoured retelling. Both quote the same vendor drafts. Both list the same caveats. The deduplication problem is real.

What is *actually* different between the two:

- Archetypes carry an explicit **purpose** sentence ("Inspect HTTP/S upload body in transit; block uploads of regulated content…") that the matrix does not.
- Archetypes carry **common false-positive risk** that the matrix does not.
- Archetypes carry a **deployment-mode requirement** statement that the matrix carries only in column form.
- Archetypes carry a **"maps to `03-policies/`"** forward pointer that points to files that do not exist.

The archetype list is *the matrix annotated with deployment guidance.* That is a useful artefact. But calling it a "policy catalogue" sibling to the capability catalogue overclaims and creates the reader confusion.

The `_index.md` files compound the problem. `02-capabilities/_index.md` line 5 says "Capabilities (what the tool can do) live in `02-capabilities/`" — fine. `03-policies/_index.md` line 5 says "A *policy* is what the firm **decides to enforce** — built on top of capabilities." Also fine. But the artefact that sits in `03-policies/` — `_policy-archetypes.md` — is not "what the firm decides to enforce." It is "what enforcement looks like across vendors when a firm decides to enforce." It is *one abstraction layer above* a real firm-specific policy. The naming overpromises. A reader expects to find policy templates ready to drop into a firm — they find a cross-vendor essay on policy shape.

**Reader-intent test.** Three readers walk in. Which folder do they want?

| Reader | Wants | Currently goes to |
|---|---|---|
| "Can vendor X do feature Y?" — vendor selection | The capability matrix | `02-capabilities/_capability-matrix.md` — correct |
| "What does a real DLP-upload-block policy look like at config level? Show me the knobs." | A vendor's actual policy config | `04-vendors/microsoft-defender-for-cloud-apps.md` Policies 5/6 — correct, but they have to know |
| "I have decided to enforce 'no PCI to unsanctioned SaaS.' What does that policy look like across vendors? What are the false-positive risks? What does it not cover?" | Exactly archetype #4 in `_policy-archetypes.md` | Correct content, wrong framing — they expect to find this in a "policy library" and find an "archetype list" instead |

The third reader is the one the folder structure currently serves least well, and they are the user's stated audience.

**Recommendation: do not merge `02-` and `03-`. Reframe both, and rename `03-`.**

- `02-capabilities/` becomes the **what-the-tools-can-do** reference. One matrix file, optional per-capability deep dives. Schema in `_index.md`.
- `03-policies/` becomes `03-policy-patterns/` (or `03-policy-archetypes/` — match the artefact name). Reframe as: **what enforcement patterns look like across vendors.** Make explicit in the `_index.md` that these are not drop-in policies — they are deployment-decision shapes. The "Maps to `03-policies/`" forward pointers in the archetype file go away (they point to a future state that may never exist; they confuse the present).
- Cross-link explicitly. Each archetype names the capability matrix row it depends on. Each capability matrix row names the archetype(s) it enables. Today neither does.

If the user later wants a **third** artefact — "drop-in policy templates ready for a regulated FI" — that is a separate folder (`09-policy-library/` or similar) with its own framing, not a third reinterpretation of the same matrix.

The boundary that matters is not capability-vs-policy. It is **cross-vendor reference (`02-` + `03-`) vs per-vendor deep dive (`04-`).** Frame that boundary, and the capability/policy distinction recedes to where it belongs — as the two angles of the cross-vendor view.

---

## Proposed README structure

Concrete. Section headings, one-line description per section.

```
# casb

[one-paragraph repo description — unchanged]

**Thesis:** [unchanged]

**Status:** [unchanged]

## What this is for

Three reading goals, three entry points. Pick yours.

| If you are… | Read this first |
|---|---|
| Choosing or evaluating CASB tooling across vendors | `02-capabilities/capability-matrix.md` — cross-vendor cell-by-cell with caveats |
| Already deploying Microsoft Defender for Cloud Apps | `04-vendors/microsoft-defender-for-cloud-apps.md` — Day 1 / 30 / 90 playbook, traps, KQL |
| Planning a CASB programme (any vendor) | `07-implementation/programme-readiness.md` → `07-implementation/phased-rollout.md` |

Everything else is supporting reference. See "Folder map" below.

## Folder map

Three classes of folder:

- **Orientation (read once)** — `01-foundations/`
- **Cross-vendor reference (look up as needed)** — `02-capabilities/`, `03-policy-patterns/`, `08-failure-modes/`
- **Per-vendor + per-programme deep dives (work from these)** — `04-vendors/`, `05-architecture/`, `06-compliance/`, `07-implementation/`

[then the existing component model block, trimmed]

## How content gets here

[the three-lens review section, unchanged but shortened — link to `00-meta/three-lens-review.md` for detail]

## What this repo does NOT cover

[unchanged]

## Caveats

[unchanged]

## Operating rules

See `CLAUDE.md`.
```

Notes on the structure:

- **Three rows, not eleven.** Each row is a reader, not a folder. Folders are the mechanism, not the entry point.
- **Folder map is a typology of folder roles** ("orientation / cross-vendor reference / deep dive"), not a re-listing of folder contents. The component model block stays for the reader who wants the full list.
- The thesis sentence stays at the top; the "What this is for" table is the operationalisation of the thesis.
- The "Read this first" column points to **specific files**, not folders, with one exception (`07-implementation/programme-readiness.md` is named, not the folder). Granularity uniform.
- "Status" and "Thesis" stay above the table. They are read-once context, not navigation.

---

## Top changes (rank-ordered)

**1. Replace the 11-row "How to navigate" table with the 3-row "What this is for" table.**
Concrete: delete README lines 9-24, replace with the three-row table above. Eleven competing entry points becomes three. The reader picks an entry point in 10 seconds, not 30.

**2. Rename `02-capabilities/_capability-matrix.md` → `02-capabilities/capability-matrix.md`. Rewrite `02-capabilities/_index.md` to point to it in line 1.**
Concrete: the underscore prefix signals scaffolding; the file is the canonical artefact. Stop hiding the destination behind an index that schematises files that don't exist.

**3. Rename `03-policies/` → `03-policy-patterns/`. Rewrite `_index.md` to state explicitly: "these are cross-vendor enforcement-pattern shapes, not drop-in policies for a specific firm." Delete the "Maps to `03-policies/`" forward pointers in `_policy-archetypes.md` until those files exist.**
Concrete: this is the user-named confusion. The naming overpromises (sounds like a policy library) and the structure underdelivers (no per-policy files). Rename to match the artefact; stop pointing at a future state.

**4. Cross-link capability matrix rows ↔ policy-pattern archetypes explicitly, bidirectionally.**
Concrete: every archetype names the capability matrix row(s) it builds on. Every matrix row names the archetype(s) that depend on it. Today neither does. This kills the "are these the same content twice?" confusion by showing the dependency.

**5. Move the CLAUDE.md "Capability vs Policy" framing (currently lines 109-110 of the private rules file) into the public `_index.md` of both `02-` and `03-`.**
Concrete: this is the cleanest framing of the distinction in the repo, and it is in a file the public reader will never see. Surface it where it does work.

Honourable mention (not in top 5 but real): the README "Component model" block at lines 28-40 and the "How to navigate" table at lines 13-24 are two takes on the same folder list. Pick one as primary. The proposed structure folds the component model into "Folder map" and keeps it brief.

---

## What you would NOT change

- **The 00-08 + `_research` + `_sources` folder skeleton.** Numbered for ordered series, underscore-prefixed for the scratch/source surfaces. Convention is clean. Workspace-wide grep for `_research/` works. Don't touch.
- **`00-meta/promotion-log.md`.** That file is the right artefact at the right path. Traceability done well.
- **The capability matrix's content.** The matrix itself is dense, well-cited, lens-flagged, and is the strongest cross-vendor artefact in the repo. The only change is the filename and the index pointing to it.
- **`04-vendors/microsoft-defender-for-cloud-apps.md` structure.** The Day 1 / Day 30 / Day 90 cadence with per-policy uniform schema (Risk / Action / ATT&CK / Prereq / Path / Config / Trap / Defeated by / Evidence / SIEM artefact) is the right shape for a vendor playbook. Other vendor playbooks should match this template, not vice versa.
- **The thesis sentence at README line 5.** Sharp, locked, named. Keep verbatim.
- **`07-implementation/programme-readiness.md` and `phased-rollout.md` separation.** Readiness gates the rollout; rollout phases come after readiness. Two files is correct. The reader-pathway proposed above names readiness first, rollout second — the files already match that order.
- **The CLAUDE.md / README split itself.** Private rules vs public navigation is the right discipline. The fix is moving one specific framing (Capability vs Policy) across the seam, not collapsing the seam.
