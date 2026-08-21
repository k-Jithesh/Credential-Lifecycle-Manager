# Credential Lifecycle Manager

Credential Lifecycle Manager (CLM) discovers expiring credentials, records coverage gaps, and provides a Dataverse model-driven app for operations.

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

The packaged notification flows use only the Dataverse connector so they remain compliant with restrictive Power Platform DLP policies. Email and Teams transport requires an approved delivery broker or a policy change that permits those connectors with Dataverse.

## Solution responsibilities

| Solution | Purpose | Readiness |
|---|---|---|
| `CLMTables 1.1.0.7` | Dataverse tables, choices, relationships, and roles | Packaged |
| `clmPlatformOps 1.0.0.2` | Graph and Azure custom connectors | Packaged |
| `CLMDiscoveryFlow 1.0.0.26` | Daily credential discovery | Packaged |
| `CLMNotifications 1.0.0.0` | Notification-group resolution and delivery queue audit | Packaged |
| `CLMApp 1.0.0.5` | Model-driven operations app with notification administration and audit pages | Packaged |

## More information

- [Solution architecture](docs/SOLUTION_ARCHITECTURE.md)
- [Installation](docs/INSTALL.md)
- [Notification responsibility resolution](docs/OWNER_RESOLUTION.md)
- [Custom connector setup](docs/CUSTOM_CONNECTORS.md)
- [Access and coverage](docs/RBAC_AND_COVERAGE.md)
- [Discovery flow](flows/Discovery-CLMCredentials/README.md)
