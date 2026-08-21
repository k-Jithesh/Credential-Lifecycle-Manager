# Install CLM

CLM is installed through Power Platform solution import. PowerShell is not required.

## Readiness

The notification data model is packaged as `CLMTables_1_1_0_7.zip`. Notification resolution and queueing are packaged as `CLMNotifications_1_0_0_0.zip`.

The deployment packages are:

- `CLMTables_1_1_0_7.zip`
- `clmPlatformOps_1_0_0_2.zip`
- `CLMDiscoveryFlow_1_0_0_26.zip`
- `CLMNotifications_1_0_0_0.zip`
- `CLMApp_1_0_0_5.zip`

## Before you start

You need:

- A Power Platform environment with Dataverse
- Permission to import solutions and edit custom connectors
- Permission to create an Entra app registration and grant admin consent
- A dedicated service account for discovery connections
- The packaged solutions listed above
- An approved notification delivery broker, or a DLP policy that permits the selected email and Teams connectors with Dataverse, if outbound delivery is required

## Clean-install order

### 1. Import the tables

Import `CLMTables_1_1_0_7.zip`.

When upgrading from `CLMTables 1.1.0.6`, review and re-save Notification Group **Digest Enabled** and **Enabled**, Notification Receiver **Active**, and Owner Mapping **Active**. Version 1.1.0.7 corrects their inverted Boolean labels to `0 = False` and `1 = True`, but importing metadata does not rewrite existing stored values.

The vanilla model includes:

- Notification Group (`clm_notificationgroup`)
- Notification Receiver (`clm_notificationreceiver`)
- Owner Mapping (`clm_ownermapping`)
- Notification Delivery (`clm_notificationdelivery`)
- A Notification Group lookup and Owner Tag on Credential
- Owner Rules that target a Notification Group

Credential does not use custom Owner User, Owner Team, Owner Source, or Owner Locked columns in the vanilla model. Older environments may require separate cleanup of those legacy components.

### 2. Import and configure the connectors

Import `clmPlatformOps_1_0_0_2.zip`.

The connectors contain neutral OAuth placeholders. Follow [Custom connector setup](CUSTOM_CONNECTORS.md), enter deployment-specific values, and create both connections before continuing.

### 3. Import the discovery flow

Import `CLMDiscoveryFlow_1_0_0_26.zip`.

During import, map:

| Connection reference | Select |
|---|---|
| `clm_dataverse` | A Dataverse connection in the target environment |
| `clm_GraphDiscovery_adm` | The service-account Graph discovery connection |
| `clm_AzureDiscovery_adm` | The service-account Azure discovery connection |

### 4. Import the app

Import `CLMApp_1_0_0_5.zip`.

The app exposes Credentials, discovery and audit records, Owner Rules, Notification Groups, Notification Receivers, Owner Mappings, and Notification Deliveries under its Operations navigation group.

### 5. Import notifications

Import `CLMNotifications_1_0_0_0.zip` and map `clm_sharedcommondataserviceforapps_23bc7` to a Dataverse connection.

Enable these flows:

| Flow | Default schedule | Purpose |
|---|---|---|
| `Resolve-CLMNotificationGroups` | Daily at 03:00 Australian Eastern time | Assigns unassigned credentials using mappings, Owner Tag, rules, then default triage |
| `Queue-CLMCredentialNotifications` | Daily at 07:00 Australian Eastern time | Creates deduplicated Pending or Retrying Notification Delivery records |

The queue flow is intentionally Dataverse-only. It does not send email or Teams messages until an approved transport broker is connected.

## Initial configuration

1. Create Notification Groups for accountable operational functions.
2. Add active Notification Receivers to each group.
3. Create immutable Owner Mappings for credentials or applications that need explicit routing.
4. Configure Owner Rules to target Notification Groups for stable fallback patterns.
5. Designate exactly one active Notification Group as the default triage destination.
6. Turn on and run `Discovery-CLMCredentials`.
7. Run `Resolve-CLMNotificationGroups`, then review assigned credentials.
8. Run `Queue-CLMCredentialNotifications`, then review Notification Delivery records.

See [Notification responsibility resolution](OWNER_RESOLUTION.md) for operating guidance.

## Access

Assign users an included role appropriate to their duties:

| Role | Intended user |
|---|---|
| `CLM Reader` | Read-only users |
| `CLM Owner` | Credential stakeholders |
| `CLM Platform Ops` | Administrators and operators |

## Troubleshooting

| Problem | First check |
|---|---|
| Connector sign-in fails | Client ID, client secret, tenant ID, and redirect URI |
| Graph actions return 403 | Graph permissions and admin consent |
| Azure actions return 403 | Service-account Azure RBAC assignments |
| Notification flow import asks for a missing connection | Map the packaged Dataverse connection reference |
| Credential has no Notification Group | Check immutable mapping, Owner Tag receiver match, rules, then default triage |
| No delivery records appear | Queue flow status, credential expiry window, receiver activation, and flow run history |
| Delivery records remain Pending | Configure an approved delivery broker; the packaged queue does not bypass DLP |
| No credentials appear | Discovery flow run history, then Coverage Gaps |
