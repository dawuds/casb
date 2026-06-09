# BYOD and unmanaged endpoint architecture

> The architectural pattern for partial coverage of personal and unmanaged devices. The honest answer is: CASB alone is insufficient; pair with MAM at the app layer and labels at the data layer.

## The problem statement

The firm has:
- Managed Windows / macOS endpoints — full coverage achievable (agent, IdP, posture)
- Managed mobile devices (Intune / Workspace ONE / Jamf) — partial coverage (MDM, MAM, app-protection policies)
- Personal mobile devices used for work email / Teams / OneDrive sync — partial coverage (MAM if the app is enrolled, none if not)
- Personal laptops used by contractors or for after-hours access — minimal CASB coverage; access decisions made at IdP layer

The CASB question for BYOD is **not "how do we cover BYOD"** — it's "how do we limit BYOD exposure with the controls we actually have".

## Coverage matrix by device class

| Device class | Browser session | Native mobile app | Native desktop app | Sync client |
|---|---|---|---|---|
| **Managed Windows + agent** | Full (forward-proxy) | Full (agent-routed) | Full (agent-routed) | Full |
| **Managed Windows, no agent** | Browser-only (CAAC) | Bypassed | Bypassed | Bypassed |
| **Managed iOS / Android (Intune MAM)** | Browser-only on managed-browser | Covered by MAM (app-protection policies, not CASB) | N/A | (Sync client coverage depends on MAM scope) |
| **Personal iOS / Android with MAM-enrolled work apps** | Bypassed | Covered by MAM | N/A | Covered if sync client enrolled |
| **Personal iOS / Android, no MAM** | Bypassed | Bypassed | N/A | Bypassed |
| **Personal Windows / macOS** | Browser-only (CAAC) — if the user signs in via the federated IdP | Bypassed (native apps' own auth) | Bypassed | Bypassed |
| **Contractor BYOD on personal network** | Browser-only (CAAC) | Bypassed | Bypassed | Bypassed |

The CASB control surface for BYOD is mostly **browser sessions via reverse-proxy CAAC**. Everything else needs adjacent layers.

## The four-control architecture for BYOD

For a partial-coverage answer to BYOD:

### Layer 1 — Identity (IdP Conditional Access)

| Decision | Configuration |
|---|---|
| Require managed device for sign-in to sensitive apps | Entra Conditional Access policy: device compliance grant required |
| Require approved client app | Entra CA: "Require approved client app" grant — restricts mobile to apps that support the IdP's policy framework (Intune MAM-enrolled apps) |
| Step-up MFA on risky sign-in | Entra ID Protection: sign-in risk High → require MFA |
| Block sign-in from non-compliant device entirely | Entra CA: block grant (heavy-handed; consider read-only alternative) |

The IdP layer is the upstream chokepoint. Decisions here propagate to every downstream component.

### Layer 2 — App protection (MAM)

| Platform | Capability |
|---|---|
| **Intune App Protection Policies (MAM)** | Inside-app controls: copy-paste restrictions, save-to-personal-cloud restrictions, screenshot prevention, selective wipe of work data on the personal device |
| **Workspace ONE / VMware Tunnel** | Equivalent on the VMware stack |
| **MobileIron / Ivanti** | Equivalent |

MAM is the actual answer for mobile BYOD coverage. CASB browser-session policy doesn't cover the dominant case (native mobile apps).

### Layer 3 — Data (sensitivity labels with encryption)

| Capability | What it does |
|---|---|
| **Microsoft Purview sensitivity labels (with encryption)** | The file carries the encryption; only authorised users can decrypt; survives the device — even if the file lands on a personal device, it can't be read without auth |
| **Equivalent labelling-with-encryption (other vendors)** | Same pattern; vendor-specific implementation |

This is the "if all else fails, the data is protected" layer. Doesn't depend on the device being managed.

### Layer 4 — CASB (the partial layer)

| What CASB can do for BYOD | Notes |
|---|---|
| Reverse-proxy CAAC on browser sessions to sanctioned SaaS | The most-cited BYOD coverage; in practice limited to browser |
| Block download of Confidential files from CAAC sessions | Effective for managed-vs-unmanaged distinction |
| Apply sensitivity labels on download | Layer 3 applies the protection; CASB triggers it |
| API-mode visibility into sanctioned SaaS activity from any device | Post-event; the user has already taken the action |

Treat CASB as the **layer that complements** layers 1-3, not as the primary BYOD control.

## The honest residual

Even with all four layers:

| Residual | Why |
|---|---|
| **Cert-pinned mobile apps** | TLS interception fails; MAM is the fallback |
| **Photos of screens** | No technical control catches this; only deterrent + IRM behavioural detection |
| **Personal-account sign-in to consumer SaaS** | Entra never sees it; CASB doesn't see it; only DLP at the data source catches it (and only if the data was labelled before access) |
| **Off-network, personal-device, native-app, personal-account** | The "all four bypasses" scenario; the firm has zero technical visibility |

For high-sensitivity data classes, the only complete answer is: don't put the data on systems that BYOD can access. That's a data-segregation decision, not a CASB decision.

## Read-only access alternative (the soft answer)

Instead of block-from-BYOD, configure a parallel session policy that:
- Allows sign-in from BYOD
- Allows in-session view + edit (no friction for the user)
- Blocks download of Confidential files
- Blocks copy / paste / print / share-externally
- Applies watermark via Purview label (visible-marking labels are Purview-side; CASB triggers the label-apply)

Users can do most legitimate work; data movement off the BYOD device is restricted.

This is the typical regulated-FI compromise. Heavy-handed block-from-BYOD pushes users to workarounds; read-only-from-BYOD lets them work while limiting exposure.

## Privacy posture for BYOD

| Concern | Treatment |
|---|---|
| **Personal device inspection** | The MAM-enrolled work app sees its own scope; the personal apps on the same device are out of scope. Document the workforce notice |
| **Selective wipe on offboarding** | MAM removes work data on offboarding; personal data on the device untouched. The user must understand this at enrolment |
| **No SSL inspection of personal traffic** | The forward-proxy doesn't see personal traffic on a personal device; the CAAC reverse-proxy only sees the IdP-mediated sessions |
| **PDPA / GDPR / HKPCPD compliance** | Workforce notice; lawful basis (employment context); proportionality. Documented privacy posture |

## Failure modes

| Failure | Cause | Mitigation |
|---|---|---|
| User pastes Confidential data from BYOD mobile app into a personal note app | MAM not enrolled OR app outside MAM scope | Restrict to MAM-enrolled apps; communicate to users |
| User downloads file to personal laptop via personal account on the SaaS | Personal account, no Entra signal, no CASB visibility | Block sanctioned SaaS access from personal-account sign-ins (where the SaaS supports tenant-restriction headers) |
| User screenshots a Confidential file on personal phone | No technical control; Purview visible-marking only deters | Visible watermark via Purview label; IRM behavioural detection for repeat patterns |
| Contractor BYOD bypasses the agent and reaches sanctioned SaaS | No managed-device check; only IdP layer sees them | Entra CA: require approved client app + step-up MFA from non-compliant device |
| Selective wipe doesn't fire on offboarding | MAM not configured properly | Pre-offboarding test of selective wipe on a known device |

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "Our CASB protects BYOD users" | Partial — browser sessions only |
| "MAM and CASB together cover all BYOD scenarios" | The all-bypasses scenario (off-network + personal device + native app + personal account) has zero coverage |
| "We can prevent screenshots of sensitive data" | No technical control does this reliably |
| "Selective wipe removes all work data" | Removes data inside the MAM-enrolled work apps; doesn't remove data the user manually copied to personal locations before offboarding |
