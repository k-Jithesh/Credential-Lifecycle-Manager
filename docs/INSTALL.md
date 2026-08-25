# Install CLM

CLM is installed through Power Platform solution import. PowerShell is not required.

## Readiness

The notification data model is packaged as `CLMTables_1_2_2_0.zip`. Notification resolution and queueing are packaged as `CLMNotifications_1_1_0_0.zip`.

The deployment packages are:

- `CLMTables_1_2_2_0.zip`
- `clmPlatformOps_1_0_0_2.zip`
- `CLMDiscoveryFlow_1_0_0_26.zip`
- `CLMNotifications_1_1_0_0.zip`
- `CLMNotificationDispatchers_1_0_0_0.zip` (optional; requires compatible DLP)
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

Import `CLMTables_1_2_2_0.zip`.

When upgrading from `CLMTables 1.1.0.6`, review and re-save Notification Group **Digest Enabled** and **Enabled**, Notification Receiver **Active**, and Owner Mapping **Active**. Version 1.1.0.7 corrected their inverted Boolean labels to `0 = False` and `1 = True`, but importing metadata does not rewrite existing stored values.

Version 1.2.0.0 adds Notification Group **Reminder Days**. Existing groups with a blank value retain the default 90, 60, 30, 7, and expiry-day schedule until an operator saves a custom selection.

Version 1.2.1.0 adds **Notification Group** to all Credential public views so operators can see routing responsibility without opening each credential.

Version 1.2.2.0 updates **Orphans (No Notification Group)** to show active, non-system credentials whose Notification Group lookup is empty.

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

Import `CLMNotifications_1_1_0_0.zip` and map `clm_sharedcommondataserviceforapps_23bc7` to a Dataverse connection.

Enable these flows:

| Flow | Default schedule | Purpose |
|---|---|---|
| `Resolve-CLMNotificationGroups` | Daily at 03:00 Australian Eastern time | Assigns unassigned credentials using mappings, Owner Tag, rules, then default triage |
| `Queue-CLMCredentialNotifications` | Daily at 07:00 Australian Eastern time | Creates deduplicated Pending or Retrying Notification Delivery records |

The queue flow is intentionally Dataverse-only.

### 6. Import notification dispatchers where permitted

Import `CLMNotificationDispatchers_1_0_0_0.zip` only into a CLM environment whose DLP policy permits Dataverse with Office 365 Outlook and Microsoft Teams. The dispatchers use the current environment's CLM tables; installing them in a separate environment without the same CLM data will not process the source queue.

During import, map:

| Connection reference | Select |
|---|---|
| `clm_sharedcommondataserviceforapps_dispatchers` | A Dataverse connection in the CLM environment |
| `clm_sharedoffice365_clmnotifications` | The mailbox connection that sends CLM email |
| `clm_sharedteams_clmnotifications` | The Teams connection that posts CLM channel messages |

Enable the flows independently:

| Flow | Default schedule | Purpose |
|---|---|---|
| `Dispatch-CLMEmailNotifications` | Every 5 minutes | Sends Pending/Retrying Email deliveries and records Sent or Failed |
| `Dispatch-CLMTeamsNotifications` | Every 5 minutes | Posts Pending/Retrying Teams deliveries and records Sent or Failed |

Do not enable a dispatcher until its connection is owned by an approved service account. Each run processes up to 100 oldest matching records sequentially.

## Initial configuration

1. Create Notification Groups for accountable operational functions.
2. Select the Notification Group **Reminder Days** required by that team: 90, 60, 30, 7, and/or expiry day.
3. Add active Notification Receivers to each group.
4. Create immutable Owner Mappings for credentials or applications that need explicit routing.
5. Configure Owner Rules to target Notification Groups for stable fallback patterns.
6. Designate exactly one active Notification Group as the default triage destination.
7. Turn on and run `Discovery-CLMCredentials`.
8. Run `Resolve-CLMNotificationGroups`, then review assigned credentials.
9. Run `Queue-CLMCredentialNotifications`, then review Notification Delivery records.
10. In a DLP-compatible environment, enable the required dispatcher and confirm delivery rows move from Pending to Sent.

When a recipient starts renewal, set the Credential to **In Renewal**, add its **Renewal Ticket URL**, and set **Suppressed Until** to the next follow-up date. See [Notification responsibility resolution](OWNER_RESOLUTION.md) for reminder-bucket and suppression behavior.

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
| Notification flow import asks for a missing connection | Map the packaged connection references |
| Credential has no Notification Group | Check immutable mapping, Owner Tag receiver match, rules, then default triage |
| No delivery records appear | Queue flow status, credential expiry window, receiver activation, and flow run history |
| Delivery records remain Pending | Confirm the matching dispatcher is enabled, its connections are valid, and DLP permits the connector combination |
| Delivery record becomes Failed | Review Error Detail, correct the receiver or connection, then change the row to Retrying |
| No credentials appear | Discovery flow run history, then Coverage Gaps |
