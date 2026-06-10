# 07-implementation entry schema

Each file in `07-implementation/<discipline>.md` follows this schema. Mirrors the 03-policies Tier-2 shape (use cases / implementation pattern / variants / integration / anti-patterns) so the two folders read consistently.

## Section order

```markdown
# <Discipline name>

> Programme-level discipline within a CASB / SSE deployment. Vendor-agnostic.
> Reader audience: <programme lead / SOC / risk / finance / etc.>
> Most-directly governs which `03-policies/` files: <list>

## Purpose

One paragraph. What this discipline owns, distinct from adjacent disciplines.

## What organisations use this discipline for

1-2 paragraphs of framing followed by 3-4 named use cases. Each use case:

### Use case <N> — <named pattern>

- **Org type:** industry vertical + size + regulatory context
- **Trigger:** what made them invest in this discipline (incident, audit, regulatory change, board ask)
- **Scope:** where they scoped it
- **Outcome:** what changed measurably (or claimed; flag if unverified)

## The discipline

The core content — readiness axes / rollout phases / SOC tiers / metrics catalogue / change-management tables / cost lines / etc. Preserve existing strong content here.

## Implementation pattern

How to roll out the DISCIPLINE itself (the meta-rollout). Typical 4-12 week table:

| Week | Activity | Output / gate |
|---|---|---|
| W1-W2 | ... | ... |

## Variants

### Industry-specific
- **BFSI:** ...
- **Healthcare:** ...
- **Tech:** ...
- **Retail:** ...
- **Public sector / Legal:** ...

### Maturity-based
- **Immature:** ...
- **Mature:** ...
- **Advanced:** ...

## Real-world experience

Named patterns from production deployments. Where applicable, typical-trajectory data (FP rate / cost trajectory / staffing ramp / etc.).

## Integration with broader programmes

How this discipline connects to:
- Specific `03-policies/` policy files it governs
- `06-compliance/` mappings it generates evidence for
- Adjacent `07-implementation/` disciplines
- Board / regulator / audit interfaces

## Anti-patterns specific to this discipline

What goes wrong with this discipline specifically. Distinct from generic CASB programme anti-patterns.

## Cross-references

- Adjacent 07-implementation/ files
- 03-policies/ governed by this discipline
- 06-compliance/ framework pages
- Sibling repos (cyberkpis / cyber-business / CyberTTX where applicable)

## Three-lens sign-off

- **Architect:** _pending_
- **Product:** _pending_
- **Compliance:** _pending_
```

## Per-section content guidance

- **Use cases** — concrete archetype framing only (industry + size + regulatory context); rotate archetypes across files to avoid the same "tier-1 ASEAN bank" repeating in every file. Use the archetype set: tier-1 ASEAN universal bank, mid-cap insurance carrier, digital-native challenger bank, multinational tech firm replacing legacy DLP, healthcare provider, manufacturing firm with M&A activity, professional services firm, higher-education institution, public sector body, tier-1 European bank under EU AI Act / DORA, merchant acquirer multi-jurisdiction CHD, pharma R&D.
- **Implementation pattern** — table week-by-week for THIS discipline's roll-out. Each discipline has a 4-12 week pattern of its own.
- **Variants** — industry differences (regulator pressure / data class / FP profile / staffing); maturity differences (Immature / Mature / Advanced trajectory).
- **Integration with broader programmes** — named cross-references to `03-policies/<family>/<policy>.md` files this discipline directly governs.
- **Anti-patterns** — discipline-specific. Not generic "skip change management" — be specific to what fails in THIS discipline.

## What an 07-implementation file is NOT

- Not a 03-policies entry (no Action / Scope / Vendor implementation grid / Worked configuration / FP risk / Coverage gaps — those are policy-level)
- Not a 06-compliance entry (no clause-by-clause regulator mapping)
- Not a 04-vendors entry (no vendor-specific console paths)
- Not a 01-foundations entry (not orientation; assume reader knows what CASB is)
- Not a 05-architecture entry (not a deployment pattern; assume architecture decided)
