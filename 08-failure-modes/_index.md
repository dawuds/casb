# 08-failure-modes

Where CASB **demonstrably fails.** Public-wedge candidate — contrarian, shareable, demolishes vendor decks.

## To write

- `byod-and-unmanaged-coverage-gap.md` — agent-dependent steering; reverse-proxy partial coverage; what residual exposure remains.
- `oauth-blind-spot.md` — grants made by users in M365 / Workspace bypass network controls entirely; API-mode-only chokepoint; vendor coverage variance.
- `ssl-tls-inspection-breakage.md` — certificate pinning, banking apps, personal-use HTTPS, PDPA / GDPR conflict on personal traffic inspection.
- `encrypted-upload-bypass.md` — DLP can't read what it can't decrypt; user-side encryption, zero-knowledge SaaS, self-hosted-key SaaS.
- `mobile-native-app-bypass.md` — iOS / Android native apps with non-standard certificates; VPN bypass; MAM dependency.
- `api-mode-is-not-prevention.md` — near-real-time != inline; "DLP" claims that are actually post-upload detection.
- `over-blocking-and-user-circumvention.md` — block-everything policies cause workarounds; personal-device upload of work data; the productivity-vs-control failure.
- `policy-drift-and-shelfware.md` — initial enforcement; tuning never finished; alerts ignored; spend-without-value.
- `vendor-lock-in.md` — policy languages don't port; capability migration cost across SSE platforms.
- `cost-blowout.md` — per-user pricing that scales unfavourably with consultants / contractors / BYOD; SSL inspection compute; log-storage costs.

Each entry includes: **the failure mode**, **what enables it**, **what compensates** (other controls, not other CASB features), **public incidents or independent test references** where they exist.
