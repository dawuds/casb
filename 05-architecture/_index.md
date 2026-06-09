# 05-architecture

Reference architectures. Vendor-agnostic patterns. How CASB sits in a real deployment.

## To write

- `proxy-only-pattern.md` — forward / reverse proxy; traffic-steering (agent, PAC, IdP-routing, SSL inspection); what gets covered, what bypasses.
- `api-only-pattern.md` — API connectors to sanctioned SaaS; near-real-time monitoring; retroactive remediation; what cannot be enforced inline.
- `hybrid-pattern.md` — proxy + API combined; common chokepoints; failure modes when one half degrades.
- `sse-stack-integration.md` — CASB alongside SWG / ZTNA / FWaaS / DLP-at-rest / identity; data flows; policy ownership boundaries.
- `byod-and-unmanaged.md` — the gap layer; reverse-proxy for unmanaged; agentless flows; what residual risk remains.
- `genai-app-overlay.md` — emerging pattern (2024–2026) — discovering and gating GenAI apps via CASB; prompt-content DLP; vendor-by-vendor capability variance.
