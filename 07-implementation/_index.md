# 07-implementation

Programme view — how a CASB / SSE rollout actually proceeds. Not vendor-specific.

## To write

- `readiness-assessment.md` — what to inventory before vendor selection (SaaS estate, identity posture, network egress topology, BYOD population, regulator obligations).
- `phased-rollout.md` — discovery → risk-scoring → policy enforcement → tuning. Why this order, what breaks if reversed.
- `soc-integration.md` — alerts to SIEM, response playbooks, SOAR integration, true-positive rate management.
- `success-metrics.md` — defer metric design to sibling repo [`cyberkpis`](../../cyberkpis/) schema; capture only CASB-specific candidate metrics (Shadow IT app count trend, DLP true-positive rate, OAuth grant revocation count, etc.).
- `change-management.md` — end-user friction, exception process, communications, business-unit onboarding.
- `cost-of-ownership.md` — licensing model, hidden costs (SSL inspection cert mgmt, agent rollout, exception triage FTE), pricing-by-geography trap.
