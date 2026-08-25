# Credential Lifecycle Manager

Credential Lifecycle Manager (CLM) helps organizations find and manage credentials before they expire and disrupt services. Built on Microsoft Power Platform, it discovers credentials across Microsoft Entra ID and Azure Key Vault, centralizes them in Dataverse, and gives operations teams a model-driven app for monitoring renewal risk.

CLM can:

- Discover application secrets, certificates, enterprise application certificates, and Key Vault secrets.
- Track credential owners, expiry dates, source environments, and renewal activity.
- Highlight discovery failures and coverage gaps.
- Route renewal notifications to responsible operational groups.
- Queue auditable email and Microsoft Teams notifications.
- Operate through packaged Power Platform solutions with configurable connectors and security roles.

## Notification model

CLM separates Dataverse record ownership from notification responsibility.

- Dataverse **Owning User** and **Owning Team** control platform security and record ownership.
- A credential's **Notification Group** identifies the operational group responsible for renewal notifications.
- Reusable Notification Receivers can participate in multiple groups through Notification Group Memberships.
- Discovery stores every owner returned for an Entra application credential instead of retaining only the first owner.
- Optional Dataverse User and Contact lookups associate a receiver with the correct Dataverse records without making those records the notification-routing mechanism.

Resolution uses this precedence:

1. Immutable Owner Mapping
2. All discovered owners resolve to one distinct routing group
3. Owner Rule
4. Default triage group

## Install

Use Power Platform solution import for deployment. See:

**[Install CLM](docs/INSTALL.md)**

The data foundation is `CLMTables 1.3.0.0`. `CLMNotifications 1.2.0.0` resolves Notification Groups and creates deduplicated Notification Delivery queue records. Each Notification Group can select its own 90, 60, 30, 7, and expiry-day reminder schedule.

The core notification flows use only the Dataverse connector so they remain compliant with restrictive Power Platform DLP policies. `CLMNotificationDispatchers 1.0.0.0` is an optional solution with separate email and Teams dispatch flows for environments whose DLP policy permits those connectors with Dataverse.

## Solution responsibilities

| Solution | Purpose | Readiness |
|---|---|---|
| `CLMTables 1.3.0.0` | Dataverse tables, reusable receiver memberships, discovered owners, choices, relationships, roles, and curated public views | Packaged |
| `clmPlatformOps 1.1.0.0` | Graph and Azure custom connectors with paged application-owner discovery | Packaged |
| `CLMDiscoveryFlow 1.1.0.2` | Daily credential and all-owner discovery | Packaged |
| `CLMNotifications 1.2.0.0` | Multi-owner group resolution, per-group reminder scheduling, and delivery queue audit | Packaged |
| `CLMNotificationDispatchers 1.0.0.0` | Optional email and Teams delivery from queued records | Packaged |
| `CLMApp 1.1.0.0` | Model-driven operations app with owner, membership, notification, and audit pages | Packaged |

## More information

- [Solution architecture](docs/SOLUTION_ARCHITECTURE.md)
- [Installation](docs/INSTALL.md)
- [Notification responsibility resolution](docs/OWNER_RESOLUTION.md)
- [Custom connector setup](docs/CUSTOM_CONNECTORS.md)
- [Access and coverage](docs/RBAC_AND_COVERAGE.md)
- [Discovery flow](flows/Discovery-CLMCredentials/README.md)
