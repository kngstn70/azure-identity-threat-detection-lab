# Azure Identity Threat Detection & Response Lab

A hands-on Azure security lab covering identity access controls, custom threat detection engineering, and full incident response — built to demonstrate the security operations skill set requested for entry-level cloud security roles.

## Summary

I built an isolated Azure tenant with Microsoft Entra ID and Microsoft Sentinel, then:
1. Configured identity access controls (Conditional Access, Privileged Identity Management)
2. Wrote custom KQL detection rules to catch credential-based attacks
3. Simulated a brute-force attack against a test account
4. Investigated, classified, and contained the resulting incident

Full video walkthrough: **[Azure Threat Detection Lab](https://youtu.be/2LGjESUU7sY)**

## Architecture

```
Entra ID Sign-in Logs ─┐
Entra ID Audit Logs ───┼──> Log Analytics Workspace ──> Microsoft Sentinel
Azure Activity Logs ───┘                                      │
                                                    ┌───────────┴───────────┐
                                              Analytics Rules      Incident Created
                                          (Failed Sign-In Burst,          │
                                           PIM Role Activation,    Investigated & Classified
                                           Entra ID Protection            │
                                           correlation rule)       Containment:
                                                                   - Block sign-in
                                                                   - Revoke sessions
```

## Part 1: Identity Access Controls

**Conditional Access** (3 policies, deployed in Report-only mode — see `policies/conditional-access-policies.md`):
- Require MFA for all users
- Block legacy authentication protocols
- Require compliant device OR MFA for Azure Resource Manager access

**Privileged Identity Management (PIM):**
- Admin roles assigned as Eligible, not Active — no standing privileged access
- Just-in-time activation with a required justification and time-boxed duration

Screenshots: `screenshots/conditional-access-policies.png`, `screenshots/pim-activation.png`

## Part 2: Detection Engineering

Built in Microsoft Sentinel, on top of a Log Analytics workspace ingesting Entra ID sign-in logs, audit logs, and Azure Activity logs.

**Rule 1: Failed Sign-In Burst** (`kql-queries/failed-signin-burst.kql`)
Detects 5 or more failed sign-ins from the same account within a 10-minute window — a common brute-force / password-spray signature.

**Rule 2: PIM Role Activation** (`kql-queries/pim-role-activation.kql`)
Tracks privileged role activations, tying detection back to the PIM layer from Part 1.

**Rule 3: Entra ID Protection correlation** (built-in, via the Microsoft Entra ID Protection content hub solution)
Correlates Identity Protection's risk signals (e.g. atypical travel) into Sentinel incidents. Note: this rule requires a mature tenant with historical sign-in baselines to be meaningful — see "Lessons Learned" below.

Screenshot: `screenshots/sentinel-rule-config.png`

## Part 3: Incident Response

Simulated a brute-force attempt against a test account (`testvictim`) to trigger Rule 1 for real.

**Detection:** Sentinel correctly grouped 7 failed sign-in attempts, all within one 10-minute window, into a single incident and mapped it to the correct account entity.

**Investigation:** Reviewed the attack story, alert details, and underlying query results confirming the failed sign-in count and timing.

**Classification:** True Positive — Compromised Account.

**Containment:**
- Blocked sign-in on the account directly in Entra ID (prevents future authentication)
- Revoked all active sessions (invalidates any tokens already issued — a separate step from blocking sign-in)

**Resolution:** Incident closed as Resolved, with full documentation in the incident comments.

Screenshots: `screenshots/incident-overview.png`, `screenshots/account-disabled.png`, `screenshots/incident-resolved.png`

## Lessons Learned

- **Portal drift is real.** Several Microsoft-documented paths (NSG Flow Logs → VNet Flow Logs, "Azure Active Directory" connector → "Microsoft Entra ID") had been renamed or restructured since most tutorials were written. Working through this taught me to verify against current documentation rather than trust cached knowledge.
- **Policy-based connectors need a managed identity.** The Azure Activity connector's `DeployIfNotExists` policy requires an explicit System Assigned managed identity — otherwise the deployment fails silently until you dig into the error.
- **KQL time-bucketing matters for testing.** My first few test attempts were spread across separate 10-minute bins and never cleared the alert threshold, even though the total failed attempts across the morning exceeded it. Real attacks cluster in time; test data has to as well.
- **A brand-new tenant has no behavioral baseline.** The impossible-travel/atypical-sign-in correlation rule depends on Identity Protection having historical data to compare against — a fresh lab tenant can't fully exercise this rule. Next iteration: seed a tenant with a longer history of normal activity before testing anomaly-based rules.
- **Next time:** script user creation and policy assignment via Microsoft Graph API or Azure CLI instead of the portal UI, to make the lab fully reproducible.

## Tools Used

Microsoft Entra ID (Conditional Access, PIM, Identity Protection), Microsoft Sentinel, Log Analytics, KQL, Azure Policy, Azure CLI
