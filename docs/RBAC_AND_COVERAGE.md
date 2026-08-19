# CLM discovery access and coverage

The current implementation uses Power Platform custom-connector connections owned by an approved service account. The Graph and Azure connectors use delegated OAuth; Dataverse uses a separate connection reference for CLM table writes.

This is different from the earlier proposed certificate-based, app-only discovery design. Permissions must be granted to the identity that owns the deployed connections.

## Current connection model

| Connection reference | Purpose | Effective identity |
|---|---|---|
| `clm_GraphDiscovery_adm` | Microsoft Graph discovery | Signed-in connection owner |
| `clm_AzureDiscovery_adm` | Azure Resource Manager and Key Vault discovery | Signed-in connection owner |
| `clm_dataverse` | CLM table reads and writes | Dataverse connection owner |

Use a dedicated service account rather than a personal account. Apply Conditional Access and credential-rotation controls appropriate for an unattended workload.

## Microsoft Graph

The connector application requires delegated permissions with tenant admin consent.

| Permission | Purpose | Failure if missing |
|---|---|---|
| `Application.Read.All` | Read applications and their password/certificate metadata | Graph application leg fails with 403 |
| `Directory.Read.All` | Read service principals and resolve owners | Enterprise-app discovery or owner resolution fails |
| `AuditLog.Read.All` | Optional creator/audit fallback | Audit-based owner fallback is unavailable |

The service account must be able to consent to and use the configured connector connection.

## Azure Resource Manager

Assign Azure RBAC to the Azure connector's service-account identity.

| Scope | Minimum role | Purpose |
|---|---|---|
| Management group | `Reader` | Broad subscription and resource visibility |
| Subscription | `Reader` | Subscription-scoped discovery |
| Resource group | `Reader` | Narrow fallback scope |

Insufficient access should create `clm_coveragegap` records with a list/read permission failure.

## Azure Key Vault

The current Azure connector reads Key Vault and secret metadata through the Azure management API. It does not request secret values.

Start with Azure **Reader** at the required subscription, resource-group, or vault scope. Do not grant secret-value access to the discovery service account.

Firewall, Conditional Access, and tenant restrictions can still block connector requests. These failures should be recorded as coverage gaps rather than silently skipped.

## Dataverse

The Dataverse connection owner requires read/write access to:

- `clm_credential`
- `clm_sourceenvironment`
- `clm_coveragegap`
- `clm_renewalevent`
- `clm_ownerrule`

It also requires read access to Dataverse users and teams so the owner resolver can populate lookup fields.

The current flow does not write `clm_discoveryrun`, although that table is part of `CLMTables`.

Use the least-privileged role included in `CLMTables`: `CLM Reader`, `CLM Owner`, or `CLM Platform Ops`.

## Coverage-gap mapping

| Signal | Gap classification |
|---|---|
| 401 or token acquisition failure | `AuthFailed` |
| 403 on an outer enumeration call | `NoListPermission` |
| 403 on an object or metadata read | `NoReadPermission` |
| Firewall, private endpoint, or Conditional Access denial | `NetworkBlocked` |
| 404 or deleted scope | `NotFound` |
| Disabled resource or environment | `Disabled` |
| Exhausted 429 retry budget | `ThrottledMaxRetries` |
| Unclassified failure | `UnknownError` |

## Operational controls

- Connections must be owned by a dedicated, monitored service account.
- Connector client secrets and connection credentials must not be stored in source control.
- Graph and Azure access should be read-only.
- Dataverse write access should be limited to the CLM tables used by the flow.
- Test discovery manually after changing connections, permissions, or solution versions.
- Review open coverage gaps after every failed or partial flow run.

## Future architecture

An app-only workload identity can be reconsidered if the connector platform and hosting model support it. Until then, documentation and access reviews must reflect the delegated connection model used by this release.
