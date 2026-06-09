# OAuth blind spot

> CASB OAuth-grant governance covers delegated permissions reasonably well. It covers application (client-credentials) permissions poorly. The 70% of OAuth abuse that practitioners think is solved isn't.

## The failure

A user clicks "Allow" on a third-party app's OAuth consent screen, granting the app delegated permissions to read mail, files, contacts, or other M365 / Google Workspace / Salesforce resources. CASB OAuth-app governance discovers this and lets administrators ban / revoke.

Separately, an admin (or a compromised admin / a malicious service principal) creates a service-principal-with-credentials grant — an application-only permission that runs unattended with broader scope. **This second class is largely invisible** to the OAuth-apps view of most CASB platforms.

## What enables it

| Mechanism | Why CASB misses it |
|---|---|
| **Delegated permissions surface in OAuth-apps view** | CASB connects to Entra / Google admin / Salesforce admin via API; reads the delegated-consent records |
| **Application permissions live in a different surface** | Service-principal credentials are an admin-grant, not a user-consent. The OAuth-apps view typically doesn't display them; vendor coverage varies |
| **Service-principal abuse is the dominant cloud-persistence pattern** | Storm-0558 (2023), MIDNIGHT BLIZZARD (2024) — both used service-principal compromise paths. Service principals run unattended with broad scopes — the attacker's ideal foothold |
| **No human in the loop** | Delegated grants require a user click; application grants don't. Detection patterns built for "user clicked consent" don't fire |

## The estimate that should be on every register

| Class | CASB coverage |
|---|---|
| Delegated permissions (user-clicked) | Covered by Policy 2a (per MDA playbook) — discovery, ban, anomaly detection |
| Application permissions (admin-granted) | **Largely uncovered** by classic CASB OAuth governance. Need Defender XDR / app governance add-on / Entra Workload Identities |

A practitioner-grade estimate (no published source): of OAuth-grant abuse incidents in regulated FIs, delegated grants account for roughly 30%; application / service-principal grants account for the remaining 70%. The CASB OAuth-apps page therefore covers ~30% of the risk surface.

## Which vendors exhibit it

| Vendor | Application-permission coverage |
|---|---|
| **MDA classic OAuth apps page** | Mostly delegated only. App Governance add-on covers some application-permission anomalies for M365 graph-scope app-only tokens — but coverage is partial |
| **Netskope / Zscaler / Palo Alto** | Vendor-specific. Generally less deep than Microsoft on Microsoft-native service-principal surface |
| **Skyhigh** | Per-app, varies |

The structural issue is that application-permission abuse is detected best where the application identities live (Entra Workload Identities). CASBs that don't have first-party visibility into the workload-identity plane will under-detect.

## Why the failure exists (technical)

The OAuth model has two grant classes:
- **Delegated** — user-consent; tied to a user session; expires with the user
- **Application** — admin-granted; service-principal identity; long-lived credentials; broad scope

These are different objects in Entra (User → ServicePrincipal vs Application → ServicePrincipal credentials). CASB platforms built around user-traffic surveillance see the user-consent path well and the admin-grant path poorly.

The harder layer is that an attacker who compromises one app's service-principal can:
- Create new service-principal credentials (T1098.001 Additional Cloud Credentials)
- Grant the new credentials broad scopes
- Persist indefinitely
- Exfiltrate via the application identity, never re-authenticating as a user

None of this is in the OAuth-apps view.

## What compensates

| Control | Notes |
|---|---|
| **Defender XDR / app governance add-on** | Microsoft's answer — Predictive Risk policies (2025 H2) cover some application-permission anomalies |
| **Microsoft Entra Workload Identities + Conditional Access for Workload Identities** | First-party Entra control plane for service-principal hygiene; the right answer for Microsoft-native estates |
| **Microsoft Entra ID Governance** | Periodic application access reviews; named owners per service principal |
| **Power Platform Admin Center DLP** | For Power Platform service-principal consents — a sub-class CASB tends to miss |
| **Splunk / Sentinel / SOAR detection on Entra `AuditLogs`** | KQL queries on `Add service principal credentials`, `Update application`, `Consent to application` (with applicationGrant scope) |
| **Periodic service-principal review cadence** | Quarterly minimum; named owner per high-permission service principal |

The CASB Policy 2 (per [`../04-vendors/microsoft-defender-for-cloud-apps.md`](../04-vendors/microsoft-defender-for-cloud-apps.md)) is necessary but not sufficient. The companion controls live outside CASB.

## What practitioners should NOT promise

| Don't say | Reason |
|---|---|
| "CASB covers OAuth abuse" | It covers ~30% of it (the delegated class) |
| "We've reviewed all OAuth grants" | If "all" means delegated, sure. If "all" means application too, no |
| "The CASB will catch a service-principal compromise" | It won't, mostly |

## Detection KQL (for the application-permission class)

```kql
// High-privilege application permission grants (last 30 days)
AuditLogs
  | where TimeGenerated > ago(30d)
  | where OperationName == "Add app role assignment to service principal"
  | extend AppId = tostring(TargetResources[0].id)
  | extend Permission = tostring(parse_json(tostring(TargetResources[0].modifiedProperties))[0].newValue)
  | where Permission has_any ("Mail.ReadWrite.All", "Files.ReadWrite.All", "Sites.FullControl.All", "User.ReadWrite.All", "Directory.ReadWrite.All")
  | project TimeGenerated, AppId, Permission, InitiatedBy_user
```

This catches the admin-grant class that the CASB OAuth-apps page misses. Pair with similar queries for service-principal credential additions:

```kql
// Service principal credential additions (last 90 days)
AuditLogs
  | where TimeGenerated > ago(90d)
  | where OperationName has_any ("Add service principal credentials", "Update application certificates and secrets management")
  | project TimeGenerated, OperationName, TargetResources, InitiatedBy_user
```

## Public-incident notes

- **Storm-0558 (2023)** — Microsoft consumer signing key compromise enabled forged tokens; downstream the technique class lives in the service-principal / OAuth domain
- **MIDNIGHT BLIZZARD (2024)** — Microsoft tenant compromise involved OAuth app abuse and service-principal credentials

Public reporting on both indicates that user-side CASB visibility had limited contribution to detection; the detection signals lived in identity-plane logging.

## Promotion candidate

Like the BYOD gap, this is a promotion candidate for the public-wedge piece if the repo wedge expands. Currently a residual-risk reference.
