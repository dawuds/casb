# Cost blowout

> CASB / SSE total cost of ownership routinely exceeds procurement-time estimates by 2-3×. The blowout is structural, not a planning failure.

## The failure

Procurement evaluates Vendor A based on per-user/month licensing. Implementation services are estimated based on vendor proposal. Year-1 budget is approved. By year 2, the actual spend is 2-3× the projection. The CISO has to defend the over-run to the CFO without a satisfactory explanation.

## What inflates the cost beyond the procurement model

| Cost line | Why it's missed at procurement |
|---|---|
| **SIEM ingest growth** | Vendor proposal cites baseline; verbose-policy uplift (clipboard inspection, GenAI prompt capture) inflates 2-5× in production |
| **Adjacent-platform licence creep** | "But you also need Entra ID P1 / Purview IRM / Defender XDR / MDE Network Protection / ChatGPT Enterprise SSO" — each is a separate licence; vendor doesn't itemise |
| **Implementation hours over-run** | Per-app onboarding (especially multi-IdP, custom apps, B2B scenarios) takes 2-4× the vendor estimate |
| **Operational headcount under-counted** | Programme team is sized for steady-state; missed the 6-12 month tuning ramp |
| **SOC alert-handling capacity** | Alert volume grows with rollout; SOC headcount lags |
| **Per-user pricing scales with extended user populations** | Contractors, B2B guests, service accounts often counted toward licence |
| **Exception-triage tuning effort** | Not budgeted as a line item; absorbs platform-team and SOC time |
| **Schema-change re-validation** | Vendor schema changes break SIEM queries; quarterly re-test costs ongoing time |
| **Migration / re-tier work** | Apps move between tiers; policies migrate; cost not in the original model |

## Typical actual vs projected spend

| Year | Projected | Actual (typical) |
|---|---|---|
| **Year 1** | Licence + impl. services + initial integration | + 20-40% over projection due to integration complexity |
| **Year 2** | Same licence + ongoing ops | + 40-80% over projection — adjacent-platform licences surface; SIEM ingest peaks |
| **Year 3** | Steady-state | Often + 30-60% over Year-1 projection — adjacent-licence creep, ops headcount expansion |
| **Year 4** | Renewal | Contracted price increases bite; renewal pricing 10-25% up |
| **Year 5** | Switching or staying | Switching = migration cost; staying = renewal pricing |

The "we'll save money by consolidating into one SSE vendor" pitch often delivers 5-15% net savings vs the legacy stack, not the 30-50% the proposal claimed.

## Which programmes exhibit it

Programmes that:
- Buy on per-user-licence headline pricing only
- Don't estimate SIEM ingest from the verbose-policy use cases they intend to deploy
- Don't itemise adjacent platforms in the procurement model
- Don't size ops headcount realistically
- Skip the exit-cost estimate (locks them in at renewal)

This is a procurement-discipline failure more than a vendor failure. Better procurement modelling catches it; vendor pricing transparency doesn't change.

## Why the failure exists

Vendor proposals are competitively priced on the visible line (per-user licence). Hidden lines (adjacent platforms, SIEM ingest, services over-run) aren't competitively priced because the customer doesn't know to compare. The information asymmetry is structural.

CASB and SSE category complexity makes apples-to-apples comparison hard. Vendors structure proposals to look like the cheapest option on the headline metric.

## What compensates

| Procurement discipline | Effect |
|---|---|
| **5-year TCO model, not Year-1** | Forces adjacent-platform and renewal-pricing into the model |
| **Itemised line-item proposal** | Every adjacent capability (Entra ID P1, Purview IRM, MDE NP, App Governance, ChatGPT Enterprise SSO) priced separately |
| **Independent SIEM ingest estimate** | Don't use the CASB vendor's estimate; pull from your own tenant's existing event volume + verbose-policy uplift |
| **Implementation cost ceiling** | Fixed-price or capped time-and-materials |
| **Ops headcount sized to programme phase** | Year-1 headcount (impl + tuning) is higher than Year-3 (steady-state) — both modelled |
| **Annual TCO review** | Each year, review against original projection; surface deltas; track adjacent-licence creep |
| **Vendor scorecard including price-stability metric** | Vendors that change pricing structures mid-contract penalised in scorecard |
| **Multi-year price cap at renewal** | Negotiated at Year-1 procurement; cap year-over-year increase |

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "The platform is cost-neutral vs the legacy stack" | Rarely true at year 2 |
| "SSE consolidation will save us 30%" | Sometimes; verify with actual TCO modelling, not vendor projections |
| "Per-user pricing scales linearly" | It does on the licence line; total cost doesn't (adjacent platforms + ingest + ops) |
| "The CASB vendor handles all our compliance needs" | The CASB licence does; the adjacent licences (Purview, IRM, Sentinel for retention, attestation pulls) often dominate |

## What an auditor / CFO will ask

| Question | What you need |
|---|---|
| Show me the year-by-year actual spend against the original procurement projection | Tracked; deltas explained |
| What's the unit cost per active user per month, all-in? | Computed; trended; benchmarked |
| What changed between Year-1 and Year-2 projections? | Documented; surfaces the adjacent-platform creep |
| What's the cost-to-leave the vendor? | Documented; updated annually |
| What's the year-5 cost trajectory? | Modelled; includes contracted price increases + adjacent-licence creep |

## The five most common cost surprises

| Surprise | Typical magnitude |
|---|---|
| SIEM ingest growth with verbose policies (clipboard inspection, prompt capture) | 50-200% of original SIEM-line estimate |
| Adjacent platform licences (Entra ID P1, Purview IRM, MDE NP, etc.) | 30-80% of original licence-line estimate |
| Multi-IdP federation work (Okta-fronted apps onboarded to inline mode) | 100-200% of original implementation estimate per affected app |
| Ops headcount ramp during tuning phase | 50-100% of steady-state headcount estimate |
| Exception-triage and tuning effort | Often unbudgeted; absorbed by existing teams under capacity pressure |

## Promotion candidate

Public-wedge candidate; currently a residual-risk reference.
