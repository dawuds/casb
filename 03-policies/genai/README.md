# 03-policies/genai

Policies governing **Generative-AI** app discovery, sanction taxonomy, prompt-content DLP, and personal-account tenant pinning. The 2024-2026 feature wave.

## Contents

| File | What it does |
|---|---|
| [`discovery-and-auto-unsanction.md`](discovery-and-auto-unsanction.md) | Continuously discover GenAI; auto-unsanction above risk threshold; propagate block to endpoint via MDE Network Protection |
| [`inline-prompt-dlp.md`](inline-prompt-dlp.md) | Block clipboard paste of regulated content into sanctioned GenAI browser sessions (CAAC + SAML-federated AI vendors only) |
| [`sanctioned-tenant-pinning.md`](sanctioned-tenant-pinning.md) | Block sign-in to consumer-tenant or personal-account variants of sanctioned GenAI from corporate network / managed device |

## Important framing

GenAI controls in CASB are **partial**. The bypass surface is broad:

- Personal account / personal device / personal network = uncovered (Entra never sees it)
- Image / audio prompts = DCS does not OCR / transcribe at session layer
- **Typing instead of pasting** = no paste event fires
- Desktop ChatGPT apps (Mac / Windows clients) = bypass CAAC
- Supply-chain compromise of an approved AI vendor (ATLAS AML.T0010) = allowlist becomes the attack surface

## 2026 platform alternatives

For Edge-on-Windows estates licensed for GSA, prefer these over the CASB → MDE Network Protection chain:

- **Microsoft Edge for Business `gen-ai.app-control`** (H2 2026 GA — verify)
- **Microsoft Global Secure Access (GSA) native AI category enforcement**
- **Microsoft Purview Adaptive Protection for GenAI** (2026)

## DSPM-for-AI prompt-capture depth

Deep capture (prompt + response body) is **ChatGPT Enterprise / Copilot only**. Gemini / Claude / Perplexity = shallow capture (metadata only). Consumer ChatGPT / personal-account = no capture. The BFSI buying argument hinges on this — do not assume parity.

## Typical deployment cadence

- **Day 90 (all three):** GenAI controls require operational maturity in SOC + exception process + MDE NP block-mode posture

## Cross-references

- [`../README.md`](../README.md) — policy-library overview
- [`../../05-architecture/genai-app-overlay.md`](../../05-architecture/genai-app-overlay.md) — the architecture pattern for GenAI controls
- [`../../04-vendors/microsoft-defender-for-cloud-apps.md`](../../04-vendors/microsoft-defender-for-cloud-apps.md) — Policies 7, 8, 9
- [`../../08-failure-modes/byod-and-unmanaged-coverage-gap.md`](../../08-failure-modes/byod-and-unmanaged-coverage-gap.md) — the personal-account bypass
