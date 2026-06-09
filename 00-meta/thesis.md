# Thesis (LOCKED 2026-06-10)

## Question

> **"For each of the five CASB products in scope, what policies can a practitioner actually configure — and what does the tool *not* cover?"**

## Lock rationale

User framing (verbatim, repo init 2026-06-09):

> "I'm looking at understanding the tech, if I do have it. Like Microsoft CASB, I want to be able to understand what policies I can apply — and what it can't do."

This rules out:
- **Buyer's guide / vendor selection** — user is not buying, may already own
- **Reference architecture** — vendor-agnostic patterns are useful but secondary to "what can I do with the tool in front of me"
- **Sellable policy pack** — policies catalogued only where the vendor in question supports them; not a normalised cross-vendor pack
- **Public failure-mode wedge** — limitations are recorded per vendor, not aggregated into a contrarian public piece (at least not for v0.1)

## What this commits to

| Aspect | Commitment |
|---|---|
| **Primary unit of output** | `04-vendors/<vendor>.md` per vendor, three named sections: Capability set / Configurable policies / Limitations |
| **User's immediate case** | Microsoft Defender for Cloud Apps gets deepest treatment + worked policy examples |
| **Vendor scope** | Netskope, Zscaler, Microsoft Defender for Cloud Apps, Palo Alto Prisma Access, Skyhigh Security (5 vendors, as locked in `CLAUDE.md`) |
| **Synthesis pages** | `02-capabilities/`, `03-policies/`, `08-failure-modes/` aggregate across the per-vendor files, but the per-vendor file is canonical |
| **Compliance overlay** (`06-compliance/`) | Lighter — each per-vendor file notes which BNM RMiT / ISO 27017 / PDPA controls its policies *evidence*, but full regulator-to-control traceability is deferred unless a sequel wedge demands it |
| **Architecture overlay** (`05-architecture/`) | Lighter — vendor sections note deployment-mode caveats (proxy vs API) inline; the full SSE-stack reference architecture is deferred |

## Out of scope for v0.1

- A normalised cross-vendor policy library that abstracts away vendor specifics
- Public-wedge anti-pattern essay
- ASEAN-wide vendor presence / channel intelligence (defer to sibling `cyber-business`)
- Vendor pricing benchmarks (defer to sibling `cyber-business`)

## Promotion gate (still applies)

Even with a practitioner wedge, the citation discipline + three-lens review + sanitisation rules in `CLAUDE.md` still gate promotion from `_research/`.

## Lock log

| Date | Event |
|---|---|
| 2026-06-09 | Repo initialised. Four candidate wedges proposed. |
| 2026-06-10 | User rejected the four pre-packaged options; reframed as "what can I do with the tool I have, and what can't it do." Wedge locked as practitioner per-vendor reference. |
