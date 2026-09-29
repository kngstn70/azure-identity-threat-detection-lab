# Conditional Access Policies

All policies below were deployed in **Report-only** mode for lab testing purposes.

## Policy 1: Require MFA - All Users

| Setting | Value |
|---|---|
| Users | All users (or scoped to test accounts) |
| Target resources | All cloud apps |
| Grant control | Require multifactor authentication |
| State | Report-only |

## Policy 2: Block Legacy Authentication

| Setting | Value |
|---|---|
| Users | All users (or scoped to test accounts) |
| Target resources | All cloud apps |
| Conditions | Client apps: Exchange ActiveSync clients, Other clients |
| Grant control | Block access |
| State | Report-only |

## Policy 3: Require Compliant Device or MFA - Test App

| Setting | Value |
|---|---|
| Users | Test accounts |
| Target resources | Azure Resource Manager |
| Grant control | Require multifactor authentication OR Require device to be marked as compliant |
| Grant setting | Require one of the selected controls |
| State | Report-only |
