# 03-policies

The **policy library.** One file per policy, schema-driven, sub-foldered by control family.

## What this folder answers

> *"I have decided to enforce X. What does that policy look like? Which capabilities does it depend on? Which vendor in my estate can do it, and where is the trap?"*

If you are instead asking *"what can the tool do"*, see [`../02-capabilities/`](../02-capabilities/) — that is the capability matrix.

- **Capability** = the tool *can* do this (vendor property).
- **Policy** = the firm has *decided to enforce* this (configuration decision).

## Quick navigation by control family

| Folder | What's there | Policies |
|---|---|---|
| [`access-control/`](access-control/) | Gate who can access what under which conditions | 4 policies — block unsanctioned, tenant restriction, block download to BYOD, geo-residency |
| [`dlp/`](dlp/) | Detect and act on regulated data in motion or at rest | 4 policies — inline upload block, API at-rest quarantine, external-share quarantine, auto-label PCI |
| [`detect/`](detect/) | Anomaly + UEBA + post-event detection | 5 policies — mass-download, impossible-travel, mass-delete, terminated-user cross-SaaS, B2B partner exfil |
| [`oauth/`](oauth/) | OAuth-app discovery, ban, behavioural detection | 2 policies — post-consent cleanup, high-scope grant alert |
| [`genai/`](genai/) | Generative-AI app discovery and prompt-content DLP | 3 policies — discovery + auto-unsanction, inline prompt DLP, sanctioned-tenant pinning |
| [`posture/`](posture/) | SaaS configuration posture (lightweight; specialist SSPM goes deeper) | 1 policy — tenant misconfig drift |

## Folder-internal files

| File | What it contains |
|---|---|
| [`_schema.md`](_schema.md) | The canonical schema every policy entry follows |
| [`_archetype-index.md`](_archetype-index.md) | Legacy cross-vendor archetype index (formerly `_policy-archetypes.md`). Preserved as a navigation aid pointing to per-policy files; no longer the canonical synthesis |

## What a policy entry contains

Each per-policy file follows the schema in [`_schema.md`](_schema.md):

- **Status line** — corroboration level + MDA Day-tier reference
- **Purpose** — one sentence; risk reduced; ATT&CK technique countered
- **Action** — block / coach / monitor / quarantine / encrypt / step-up / label / DLP-redact
- **Scope** — users / apps / device posture / network position / traffic
- **Vendor implementation grid** — one row per vendor (MDA / NSK / PAN / SKY / ZS): console path, key configuration, deployment-mode caveat, known trap
- **Control mappings** — BNM RMiT / ISO 27017 / ISO 27018 / NIST CSF references
- **False-positive risk** — named scenarios
- **Operational cost** — exception load, triage load, end-user friction
- **Privacy / data-protection considerations**
- **Coverage gaps** — cross-link to `08-failure-modes/`
- **Three-lens sign-off** — Architect / Product / Compliance

## Status

At v0.0, only the **MDA-tier policies** have full corroboration. Other vendor columns in each policy file's grid carry `[unverified]` until those vendor playbooks reach MDA depth (per outstanding items in [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md)).

## Cross-references

- [`../02-capabilities/`](../02-capabilities/) — capability matrix; the policies depend on these capabilities
- [`../04-vendors/`](../04-vendors/) — full per-vendor configuration walkthrough (Day 1 / 30 / 90)
- [`../06-compliance/`](../06-compliance/) — clause-to-policy mapping framework
- [`../07-implementation/phased-rollout.md`](../07-implementation/phased-rollout.md) — when to enable which policies (Day 1 / 30 / 90 cadence, vendor-agnostic)
- [`../08-failure-modes/`](../08-failure-modes/) — where each policy class demonstrably fails
- [`../README.md`](../README.md) — repo-level navigation
