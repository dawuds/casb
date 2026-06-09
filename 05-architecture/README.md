# 05-architecture

Reference architectures. Vendor-agnostic patterns for how CASB sits in a real deployment.

## Contents

| File | What it contains |
|---|---|
| [`proxy-only-pattern.md`](proxy-only-pattern.md) | Forward-proxy + reverse-proxy CASB topology, traffic-steering options (agent / PAC / IdP-mediated), SSL inspection trade-offs, when proxy-only is and isn't the right choice |
| [`api-only-pattern.md`](api-only-pattern.md) | API-mode CASB topology, connector-depth-by-vendor, the "API is not prevention" rule, Graph / API throttling, when API-only is enough |
| [`hybrid-pattern.md`](hybrid-pattern.md) | The typical end-state — proxy + API combined. App tiering (Tier 1 inline + API; Tier 2 API only; Tier 3 discovery; Tier 4 blocked). Multi-vendor hybrid considerations |
| [`sse-stack-integration.md`](sse-stack-integration.md) | How CASB composes with SWG / ZTNA / FWaaS / IdP / RBI / EDR / Email security / SIEM. Component ownership per policy class. Identity-is-upstream principle |
| [`byod-and-unmanaged.md`](byod-and-unmanaged.md) | The partial-coverage architecture for BYOD — the four-control architecture (Identity / App-protection MAM / Data labelling / CASB). Coverage matrix by device class |
| [`genai-app-overlay.md`](genai-app-overlay.md) | The 2024-2026 GenAI feature wave. Sanction taxonomy, DLP guardrails on sanctioned AI, block unsanctioned, approved-AI allowlist governance, DSPM-for-AI prompt-capture depth |

## Reading order

1. [`proxy-only-pattern.md`](proxy-only-pattern.md) and [`api-only-pattern.md`](api-only-pattern.md) — understand the two primary modes before the hybrid
2. [`hybrid-pattern.md`](hybrid-pattern.md) — the typical production deployment
3. [`sse-stack-integration.md`](sse-stack-integration.md) — composition with the broader stack
4. [`byod-and-unmanaged.md`](byod-and-unmanaged.md) — the residual-coverage architecture
5. [`genai-app-overlay.md`](genai-app-overlay.md) — the current feature wave

## What this folder does NOT cover

- **Specific vendor topology** — see [`../04-vendors/`](../04-vendors/) for per-vendor deployment guidance
- **The control-mapping framework** — see [`../06-compliance/control-mapping-framework.md`](../06-compliance/control-mapping-framework.md)
- **Programme rollout sequencing** — see [`../07-implementation/phased-rollout.md`](../07-implementation/phased-rollout.md)

## Cross-references

- [`../01-foundations/deployment-modes.md`](../01-foundations/deployment-modes.md) — the underlying mode-to-capability matrix
- [`../01-foundations/casb-inside-sse-sase.md`](../01-foundations/casb-inside-sse-sase.md) — the SSE positioning context
- [`../04-vendors/`](../04-vendors/) — per-vendor implementation
- [`../README.md`](../README.md) — repo-level navigation
