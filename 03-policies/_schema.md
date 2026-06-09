# Policy entry schema

Every file in `03-policies/<family>/<policy>.md` follows the schema below. Sections appear in this order; do not reorder. Section headings are exact (`## Purpose`, etc.) so cross-file regex tooling works.

```markdown
# <Policy name>

> Status: <v0.0 — MDA-tier corroborated only / draft / etc.>
> Required capabilities: link to `../../02-capabilities/capability-matrix.md` row(s)
> Deployment-mode requirement: <forward-proxy / reverse-proxy / API / hybrid>
> MDA playbook reference: link to relevant policy section in `../../04-vendors/microsoft-defender-for-cloud-apps.md`

## Purpose

One sentence. Which named risk does this reduce. Cite ATT&CK / ATLAS technique where applicable.

## Action

<block / coach / monitor / quarantine / encrypt / step-up auth / require labelling / DLP-redact>

## Scope

- **Users:** which group / OU / role
- **Apps:** sanctioned / tolerated / unsanctioned / specific app
- **Device posture:** managed / unmanaged / BYOD
- **Network position:** on-prem / remote / mobile
- **Traffic:** upload / download / share / API call

## Vendor implementation grid

| Vendor | Console path | Key configuration values | Deployment-mode caveat | Known trap |
|---|---|---|---|---|
| MDA | <Defender portal → Cloud apps → Policies → ...> | <key fields> | <e.g. browser-only via CAAC; requires Entra ID P1> | <e.g. bilateral CAAC trap — needs CA policy + session policy> |
| Netskope | `[unverified]` until lens-reviewed | | | |
| Palo Alto Prisma Access | `[unverified]` | | | |
| Skyhigh | `[unverified]` | | | |
| Zscaler | `[unverified]` | | | |

Each cell links to the corresponding section of `04-vendors/<vendor>.md` for the full configuration walkthrough.

## Control mappings

- BNM RMiT clause(s): <link to `../../06-compliance/malaysia/bnm-rmit.md`>
- ISO 27017 control(s): <link to `../../06-compliance/iso-27017.md`>
- ISO 27018 control(s): <link to `../../06-compliance/iso-27018.md`>
- NIST CSF 2.0 subcategory(ies): <reference>

## False-positive risk

Named scenarios — not a rating. e.g. "Regex on PCI/SSN fires on test fixtures, employee IDs, order numbers". Cite source.

## Operational cost

- Exception-handling load: <named scenarios>
- Triage load: <alerts/day estimate where possible>
- End-user friction: <named UX consequences>

## Privacy / data-protection considerations

- SSL inspection on personal traffic? PDPA / GDPR implications.
- PII surfaced to admins? Access governance on the policy's evidence record.
- DPIA / TIA trigger?

## Coverage gaps

Cross-link to `../../08-failure-modes/<mode>.md` entries. What this policy demonstrably does not catch (mobile native, certificate-pinned apps, encrypted payloads, third-party-to-third-party API flows).

## Three-lens sign-off

- **Architect:** <findings + resolution>
- **Product:** <findings + resolution>
- **Compliance:** <findings + resolution>
```

## Per-section content guidance

- **Status line** — MDA column reliability + cross-vendor reliability flag. State Day-tier the MDA playbook places it in as metadata (not as the file's organising axis).
- **Purpose** — one sentence, no marketing language. Cite the ATT&CK / ATLAS technique countered (e.g. `T1530 Data from Cloud Storage Object`).
- **Vendor implementation grid** — MDA column is the one to populate fully at v0.0. Other columns carry `[unverified]` placeholders explicitly tied to the vendor draft's source-quality flags. Each cell ≤3 lines; the full walkthrough lives in `04-vendors/<vendor>.md`.
- **Control mappings** — at v0.0, the framework pages exist but the clause-by-clause mapping does not. Link to the framework page and add a `[VERIFY]` marker; the full clause map is deferred per [`../00-meta/promotion-log.md`](../00-meta/promotion-log.md) outstanding items.
- **False-positive risk** — pulled from the MDA playbook's "Trap to avoid" and "Defeated by" sections where applicable.
- **Three-lens sign-off** — empty block at v0.0; populated when the policy passes the three-lens review per [`../CLAUDE.md`](../CLAUDE.md).
