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
- Notification groups contain one or more receivers, such as an email address, shared mailbox, distribution or Microsoft 365 group, or Teams channel.
- Optional Dataverse User and Contact lookups associate a receiver with the correct Dataverse records without making those records the notification-routing mechanism.

Resolution uses this precedence:

1. Immutable Owner Mapping
2. Owner Tag matched to an active receiver email
3. Owner Rule
4. Default triage group

## Install

Use Power Platform solution import for deployment. See:

**[Install CLM](docs/INSTALL.md)**

The data foundation is `CLMTables 1.1.0.7`. `CLMNotifications 1.0.0.0` resolves Notification Groups and creates deduplicated Notification Delivery queue records.

The core notification flows use only the Dataverse connector so they remain compliant with restrictive Power Platform DLP policies. `CLMNotificationDispatchers 1.0.0.0` is an optional solution with separate email and Teams dispatch flows for environments whose DLP policy permits those connectors with Dataverse.

## Solution responsibilities

| Solution | Purpose | Readiness |
|---|---|---|
| `CLMTables 1.1.0.7` | Dataverse tables, choices, relationships, and roles | Packaged |
| `clmPlatformOps 1.0.0.2` | Graph and Azure custom connectors | Packaged |
| `CLMDiscoveryFlow 1.0.0.26` | Daily credential discovery | Packaged |
| `CLMNotifications 1.0.0.0` | Notification-group resolution and delivery queue audit | Packaged |
| `CLMNotificationDispatchers 1.0.0.0` | Optional email and Teams delivery from queued records | Packaged |
| `CLMApp 1.0.0.5` | Model-driven operations app with notification administration and audit pages | Packaged |

## More information

- [Solution architecture](docs/SOLUTION_ARCHITECTURE.md)
- [Installation](docs/INSTALL.md)
- [Notification responsibility resolution](docs/OWNER_RESOLUTION.md)
- [Custom connector setup](docs/CUSTOM_CONNECTORS.md)
- [Access and coverage](docs/RBAC_AND_COVERAGE.md)
- [Discovery flow](flows/Discovery-CLMCredentials/README.md)
