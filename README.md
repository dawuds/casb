# casb

Research repo on **Cloud Access Security Broker (CASB)** as a capability set, the SSE/SASE platforms that house it in 2026, the policies firms enforce with it, and how those policies map to regulator expectations — anchored Malaysia, with MAS / HKMA / EU / ISO as comparators.

**Status:** private until `v0.1`. Thesis lock pending — see [`CLAUDE.md`](CLAUDE.md) for candidate wedges.

## What's here

```
00-meta/             Thesis, scope, citation policy, three-lens review template
01-foundations/      What CASB is (and isn't); deployment modes; SSE/SASE positioning
02-capabilities/     Capability catalogue (one entry per capability)
03-policies/         Policy catalogue (one entry per policy archetype)
04-vendors/          Vendor deep-dives (Netskope, Zscaler, MS Defender for Cloud Apps, Palo Alto, Skyhigh)
05-architecture/     Reference architectures (proxy / API / hybrid; SSE integration)
06-compliance/       Regulator mapping (BNM RMiT, MAS TRM, PDPA, ISO 27017/27018, NIST CSF, etc.)
07-implementation/   Programme rollout — discovery → risk-scoring → enforcement
08-failure-modes/    Where CASB demonstrably fails
_sources/            Primary-source library
_research/           Working notes; multi-agent workflow outputs land here first
```

## Three-lens review

Every synthesis page passes through three lenses before promotion from `_research/`:

- **Cybersecurity architect** — deployment, integration, what does *not* get covered
- **CASB product expert** — vendor claims vs vendor behaviour, deployment-mode caveats, console footguns
- **Cyber-compliance expert** — control mapping, audit evidence, regulator clock, privacy conflict

See [`CLAUDE.md`](CLAUDE.md) for the full operating rules.
