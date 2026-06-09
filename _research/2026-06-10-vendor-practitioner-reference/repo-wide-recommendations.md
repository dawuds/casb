# Repo-wide recommendations (NOT auto-applied)

These came out of the five-specialist review of the MDA playbook v0. They concern files **outside** the playbook itself — `CLAUDE.md`, the per-vendor template, glossary, synthesis files, and the compliance folder.

**Marked `[NOT YET APPLIED]`.** Review tomorrow and apply selectively.

---

## `CLAUDE.md` (repo root for `~/claude/casb/`) — [NOT YET APPLIED]

Add a vendor-playbook component-model section that requires every per-vendor practitioner doc to carry:

- **Citation discipline** — URL + accessed date on every Microsoft / vendor-named claim, OR a claim-classification tag (verifiable-and-correct / vendor-claim-only / practitioner-inference / verifiable-and-incorrect).
- **The locked policy-table schema** — Risk reduced | What it does | ATT&CK technique | Critical prerequisite | Console path | Key configuration | Trap to avoid | Defeated by | Auditable evidence (table + retention + field + correlator) | SIEM artefact + connector.
- **A Day 0 prerequisites section before Day 1** — the licence / dependency / region / data-source checks that determine whether the rest of the playbook applies at all.
- **A glossary section or first-use TLA expansion** — CAAC, DCS, SIT, SCIM, CNAPP, SSPM, DfE / DfI / DfO, DPIA, RBI, DSPM, CAE, OCAS, MIP, EDM, MAM, BYOK, CSF, RMiT, TRM.
- **ATT&CK + ATLAS coverage rows** per policy + an end-of-doc matrix.
- **DR / fail-mode and SCIM cascade sections** (per-app-relevant or cross-vendor link).
- **A refs companion file** keyed by claim-id (`<vendor>.refs.md`).

Sourced from: all five specialists' fixes converging on schema discipline.

---

## `_research/2026-06-10-vendor-practitioner-reference/_template.md` — [NOT YET APPLIED]

If a per-vendor template exists, update it to match the locked schema above. If not, **create** the template at this path so that the Netskope / Zscaler / Skyhigh / Palo Alto playbooks reuse the same shape and the cross-vendor capability matrix can pull consistent fields.

Sourced from: docs #1 (consistency); cyber #15 (cross-vendor coverage); qa ADD (refs companion).

---

## `_research/2026-06-10-vendor-practitioner-reference/_glossary.md` (or workspace-wide) — [NOT YET APPLIED]

A shared glossary keyed by TLA. House the expansions inline in the playbook (already in v1 Appendix G) AND in this shared file so other vendor playbooks can link rather than duplicate. Add vendor-equivalents columns where the same concept goes by different names across CASBs (e.g. "session control" in MDA ≈ "session policy" in Netskope ≈ "Browser Access" in some Zscaler docs).

Sourced from: docs (ADD); sysint (implicit via SCIM detail).

---

## `synthesis/capability-matrix.md` (cross-vendor) — [NOT YET APPLIED]

Add **two new columns** to the existing capability matrix:

- ATT&CK coverage per vendor (which techniques the vendor's capability set covers as Prevent / Detect / Respond).
- ATLAS coverage per vendor (AI-specific techniques).

The per-vendor playbooks then link to the matrix instead of restating coverage.

Sourced from: cyber #15.

---

## `synthesis/dr-failmode-matrix.md` (cross-vendor — NEW FILE) — [NOT YET APPLIED]

DR / fail-mode comparison across MDA / Netskope / Zscaler / Skyhigh / Palo Alto. Each vendor's behaviour when:
- The CASB service is in regional outage
- The IdP is in outage
- API connector throttling saturates
- Endpoint enforcement (MDE / agent / PAC / RBI) drops

Per-vendor docs link in. Eliminates per-vendor duplication.

Sourced from: sysint #9.

---

## `synthesis/scim-cascade-catalogue.md` (cross-vendor — NEW FILE) — [NOT YET APPLIED]

Top-10 BFSI SaaS apps with disable-signal latency and per-app footguns, **vendor-agnostic**. Per-vendor docs (whichever vendor is the IdP / CASB) link to it.

Currently MDA v1 has this as Appendix A — promote that out to the cross-vendor file and link.

Sourced from: sysint ADD.

---

## `06-compliance/microsoft-defender-for-cloud-apps-control-mapping.md` (separate compliance deliverable — NEW FILE) — [NOT YET APPLIED]

The full clause mapping deferred from the playbook body:
- BNM RMiT (current edition) — verify clauses against the gazetted document
- MAS TRM
- HKMA SA-2
- ISO/IEC 27017:2015
- ISO/IEC 27018:2019
- NIST CSF 2.0
- PCI DSS v4.0
- PDPA Malaysia (2010 + 2024 amendments)
- GDPR + DORA where relevant

Material on regulatory clauses is illustrative; not legal/regulatory advice. Verify against gazetted text and assurance lead.

Sourced from: qa #19; playbook itself.

---

## `_research/2026-06-10-vendor-practitioner-reference/microsoft-defender-for-cloud-apps.refs.md` (refs companion — NEW FILE) — [NOT YET APPLIED]

Keyed by claim-id (one per assertion in the playbook). Each entry: Microsoft Learn URL + accessed date + quoted phrase from the source. The playbook body then carries claim-id markers like `[ref:catalogue-size]` rather than inline URLs. This is the citation-discipline mechanism that v1 acknowledges is owed but doesn't itself implement.

Sourced from: qa ADD.

---

## `CHANGELOG.md` (playbook-adjacent or repo-level) — [NOT YET APPLIED]

Track v0 → v1 → v2 iterations for each per-vendor playbook. What was added / changed / removed in each pass. The per-playbook changelog file (`playbook-v0-to-v1-changelog.md`) is one specific instance; a rolled-up repo-level CHANGELOG would aggregate.

Sourced from: docs ADD.

---

## Cross-reference fixes — [NOT YET APPLIED]

The `02-capabilities/_index.md`, `03-policies/_index.md`, `04-vendors/_index.md`, `05-architecture/_index.md`, `06-compliance/_index.md`, `08-failure-modes/_index.md` index files were drafted before the MDA playbook revealed concrete schema requirements. After v1 has settled:

- Reconcile capability list in `02-capabilities/_index.md` against the locked policy-table schema in MDA v1.
- Reconcile policy archetype list in `03-policies/_index.md` against MDA v1's 13 policies (it has more than the initial 12 sketched).
- Update `04-vendors/_index.md` schema rows to match the v1 playbook structure (the per-vendor template should pull from this).
- Add a `gaps-and-compensating-controls` archetype entry in `08-failure-modes/` so the MDA v1 "Gaps and compensating controls" table has a sibling abstraction layer in the canonical folder.

Sourced from: docs #1, #16; cross-file consistency.
