# 06-compliance

Regulator and standards mapping. **Capability → policy → control → evidence** traceability.

## Mapping targets (initial — confirm at thesis-lock)

- **BNM RMiT** (Malaysia) — cloud risk management, data leakage, third-party risk, outsourcing
- **PDPA 2010 + 2024 amendments** (Malaysia) — processor obligations, cross-border transfer, breach notification
- **Cybersecurity Act 854 (2024) + NACSA** (Malaysia) — sectoral CNII expectations relevant to BFSI cloud
- **MAS TRM** (Singapore) — cloud computing, third-party risk
- **HKMA SA-2 / cloud guidance** (Hong Kong)
- **EU GDPR + DORA** — processor obligations, ICT third-party risk
- **ISO/IEC 27001:2022** — relevant Annex A control families
- **ISO/IEC 27017:2015** — cloud-services controls
- **ISO/IEC 27018:2019** — cloud-PII processor controls
- **NIST CSF 2.0** — subcategory mapping
- **NIST SP 800-53 Rev 5** — relevant control families (AC, AU, SC, SI)
- **SOC 2** — common criteria mapping (where relevant for vendor-attestation alignment)

## File-per-instrument rule

One file per regulator / standard, named `<jurisdiction>/<instrument-slug>.md` (mirrors the `cyberkpis` pattern). Inside each file: clauses relevant to CASB, with explicit mapping to `02-capabilities/` and `03-policies/`.

## Citation rule (reminder)

No regulator obligation cited without clause + version + date. No "BNM requires…" without the RMiT section reference and edition.
