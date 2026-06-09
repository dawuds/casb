# 04-vendors

Vendor deep-dives. One file per vendor. Schema-driven.

## Initial vendor scope (locked in `CLAUDE.md`)

1. **Netskope** — SSE-native, strong inline-CASB + API-CASB heritage.
2. **Zscaler** — SSE platform (ZIA / ZPA); CASB as feature inside ZIA.
3. **Microsoft Defender for Cloud Apps** — formerly MCAS; deeply integrated with Entra + Microsoft 365 + Defender XDR.
4. **Palo Alto Prisma Access** — SSE platform with SaaS Security / Inline CASB.
5. **Skyhigh Security** — formerly McAfee / MVISION / Trellix CASB; one of the few remaining stand-alone-credible CASB lineages.

Cisco Cloudlock, Forcepoint ONE, Lookout, iboss — added only if the locked wedge demands.

## Schema (to write at `_schema.md` when first entry lands)

Each vendor entry carries:

- **Name + lineage** — current name + history of renames / acquisitions.
- **CASB housing** — stand-alone product / feature inside SSE platform / feature inside identity platform.
- **Deployment modes supported** — forward-proxy (with traffic-steering options: agent / PAC / IdP-routing) / reverse-proxy / API connectors (which SaaS apps) / log-based discovery (which log sources).
- **Capability coverage** — link each row to the `02-capabilities/` entry; mark "supported / supported-with-caveats / not supported" with **source URL + access date** (vendor doc page).
- **SSE / SASE integration** — what else the platform does (SWG, ZTNA, FWaaS, RBI, DLP-data-at-rest) and where the boundaries are.
- **Identity dependency** — Entra / Okta / Ping / native IdP; SSO mechanism.
- **Malaysia presence** — local partner / reseller / SI; MY-region data centre? data-residency story?
- **Public pricing** — captured with date and URL where available; "negotiated" / "channel-only" noted where not.
- **Known limitations** — gathered from public deployment guides, customer-reference write-ups, independent test reports, public incidents.
- **Public incidents / vulnerabilities** — CVE refs, advisory IDs.
- **Three-lens sign-off** — Architect / Product / Compliance.

## Citation reminder

Vendor documentation is **vendor-controlled** — it states what the product *claims*. A capability claim needs corroboration from a second source (deployment guide, independent test, customer reference, incident response, CVE) before it leaves `_research/`.

## Renaming watch-list

CASB vendors rename frequently. Keep `00-meta/glossary.md` current:

- MCAS → Microsoft Cloud App Security → Microsoft Defender for Cloud Apps
- McAfee MVISION Cloud → Trellix CASB → Skyhigh Security
- Symantec CloudSOC → Broadcom Symantec Cloud SWG (CASB folded in)
- Bitglass → Forcepoint ONE
- Oracle CASB → discontinued
