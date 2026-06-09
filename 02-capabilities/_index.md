# 02-capabilities

The **capability catalogue.** One file per capability. Schema-driven.

A *capability* is what the tool **can do** — independent of which firm decides to use it. Policies (what a firm decides to enforce) live in `03-policies/`. Keep the line clean.

## Schema (to write at `_schema.md` when first entry lands)

Each capability entry carries:

- **Name** — canonical name in this repo (avoid vendor branding).
- **Definition** — one paragraph, neutral, no marketing language.
- **Gartner pillar** — Visibility / Data Security / Threat Protection / Compliance.
- **Deployment-mode dependency** — forward-proxy / reverse-proxy / API / log-based discovery / hybrid. Some capabilities only work in some modes.
- **Vendor coverage matrix** — Netskope / Zscaler / Defender for Cloud Apps / Palo Alto Prisma / Skyhigh — supported? Caveats? Source URL + access date.
- **Known limitations** — what the capability does *not* do (BYOD gaps, OAuth blind spots, encrypted-content limits, mobile-app coverage, etc.).
- **Linked policies** — `03-policies/` entries that depend on this capability.
- **Linked controls** — `06-compliance/` control mappings the capability enables evidence for.
- **Three-lens sign-off** — Architect / Product / Compliance — when promoted from `_research/`.

## Capabilities to draft (initial pass — confirm at thesis-lock)

- Shadow-IT discovery (network-log-based)
- Sanctioned-app inventory + risk-scoring
- Inline DLP for sanctioned SaaS (proxy mode)
- API-mode DLP for sanctioned SaaS (post-upload scan)
- OAuth-app discovery and governance
- Anomalous-activity detection (UEBA-style)
- Compromised-account detection
- Data-residency / geo-policy enforcement
- Share-link governance (external sharing controls)
- Malware scanning in SaaS storage
- Encryption / tokenisation at the gateway
- Threat intelligence integration
- Risk-based session policy (step-up auth, read-only, watermark, block)
- Generative-AI-app discovery + governance (2024–2026 SSE feature wave)
