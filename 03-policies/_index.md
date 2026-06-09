# 03-policies

The **policy library.** One file per policy, schema-driven, sub-foldered by control family.

## What this folder answers

> *"I have decided to enforce X. What does that policy look like? Which capabilities does it depend on? Which vendor in my estate can do it, and where is the trap?"*

If you are instead asking *"what can the tool do"*, see [`../02-capabilities/`](../02-capabilities/) — that is the capability matrix. The two folders are deliberately distinct:

- **Capability** = the tool *can* do this (vendor property).
- **Policy** = the firm has *decided to enforce* this (configuration decision).

The capability matrix tells you what to expect in an RFP demo. The policy library tells you what decisions you have to make in a design workshop, and what each decision costs you per vendor.

## What a policy entry looks like

Each file in this folder is structured identically — see [`_schema.md`](_schema.md) for the canonical schema. Summary of sections:

- **Name** — canonical, vendor-neutral
- **Purpose** — one sentence; which named risk does this reduce
- **Action** — block / coach / monitor / quarantine / encrypt / step-up / label / DLP-redact
- **Scope** — users, apps, device posture, network position, traffic class
- **Required capabilities** — links to `02-capabilities/capability-matrix.md` rows the policy depends on
- **Deployment-mode requirement** — forward-proxy / reverse-proxy / API / hybrid
- **Vendor implementation grid** — one row per vendor (MDA / Netskope / Palo Alto Prisma / Skyhigh / Zscaler): console path, key configuration values, deployment-mode caveat, known trap
- **Control mappings** — `06-compliance/` references (BNM RMiT, ISO 27017, ISO 27018, NIST CSF)
- **False-positive risk** — named scenarios, not a rating
- **Operational cost** — exception load, triage load, end-user friction
- **Privacy / data-protection considerations** — SSL inspection on personal traffic, PII surfaced to admins, DPIA triggers
- **Coverage gaps** — cross-link to `08-failure-modes/`
- **Three-lens sign-off** — Architect / Product / Compliance

## Organisation

Files are sub-foldered by control family, **not** by Day 1/30/90 (cadence is a per-firm and per-vendor rollout property, not a policy property — see [`../07-implementation/phased-rollout.md`](../07-implementation/phased-rollout.md) for the rollout view):

```
03-policies/
  _index.md
  _schema.md                                              <- this folder's schema
  _archetype-index.md                                     <- cross-vendor archetype index (legacy, now navigation aid)
  access-control/
    block-unsanctioned-app-with-coach.md
    tenant-restriction-corporate-only.md
    block-download-unmanaged-device.md
    geo-residency-block.md
  dlp/
    inline-upload-block-regulated-data.md
    api-data-at-rest-quarantine.md
    external-share-link-quarantine.md
    auto-label-pci-data.md
  detect/
    mass-download-alert.md
    impossible-travel-alert.md
    mass-delete-anomaly.md
    terminated-user-cross-saas.md
    b2b-partner-exfil-alert.md
  oauth/
    post-consent-cleanup-and-ban.md
    high-scope-grant-alert.md
  genai/
    discovery-and-auto-unsanction.md
    inline-prompt-dlp.md
    sanctioned-tenant-pinning.md
  posture/
    sspm-tenant-misconfig-drift.md
```

## Status

At v0.0, only the **MDA-tier policies** (those backed by the lens-reviewed MDA v1 playbook) have enough corroboration to populate a full vendor-implementation grid. Other vendor cells in the grid will carry `[unverified]` until those vendor playbooks reach MDA depth (per outstanding items in [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md)).

Until then, treat per-policy files as **partial** — the MDA column is reliable; the other four columns inherit the source-quality flags from the underlying vendor drafts.

The legacy cross-vendor archetype index ([`_archetype-index.md`](_archetype-index.md), formerly `_policy-archetypes.md`) is preserved as a navigation aid — it maps each archetype to the per-policy file(s) under it — but is no longer the canonical synthesis. The canonical synthesis is the per-policy file.
