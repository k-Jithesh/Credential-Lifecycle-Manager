# Install CLM

## When to use this

Use this page if you want a clean deployment from the packaged solutions in this repository. Upgrades from older CLM environments can contain legacy owner fields and require environment-specific cleanup; this page describes a clean install.

## What you need

- A Power Platform environment with Dataverse.
- Permission to import solutions and edit custom connectors.
- Permission to create an Entra app registration and grant tenant-wide admin consent.
- Permission to assign Azure RBAC.
- A dedicated discovery service account with a suitable Power Automate/Dataverse licence.
- A DLP decision for Dataverse, Office 365 Outlook, and Microsoft Teams.
- The packages in the repository's `packages` folder.

## Current package versions

| Package | Required |
|---|---|
| `CLMTables_1_3_1_0.zip` | Yes |
| `clmPlatformOps_1_1_0_0.zip` | Yes |
| `CLMDiscoveryFlow_1_1_0_2.zip` | Yes |
| `CLMApp_1_1_0_0.zip` | Yes |
| `CLMNotifications_1_3_0_0.zip` | Yes |
| `CLMNotificationDispatchers_1_0_0_0.zip` | Optional; DLP-dependent |

## Steps

### 1. Import the tables

Import `CLMTables_1_3_1_0.zip`.

### 2. Import the connectors

Import `clmPlatformOps_1_1_0_0.zip`. Do not import the discovery flow yet.

### 3. Configure and test the connectors

1. Create the single-tenant connector app registration.
2. Add delegated Microsoft Graph `Application.Read.All` and `Directory.Read.All`.
3. Add delegated Azure Service Management `user_impersonation`.
4. Grant tenant admin consent.
5. Create a client secret and keep its value in an approved secret store.
6. Edit **CLM Graph Discovery** and **CLM Azure Discovery** with the app's client ID, secret, and tenant ID.
7. Save each connector and copy its unique redirect URL.
8. Register both redirect URLs as **Web** redirect URIs on the app registration.
9. As the dedicated discovery service account, create one connection for each connector.
10. Test Graph `GetOrganization`, `ListApplications`, and `ListServicePrincipals`.
11. Test Azure `ListSubscriptions`.

See [Identity and access](IDENTITY_AND_ACCESS.md) for the exact identity boundaries and [the connector reference](../CUSTOM_CONNECTORS.md) for every connector field.

### 4. Import discovery

Import `CLMDiscoveryFlow_1_1_0_2.zip` and map:

| Connection reference | Connection |
|---|---|
| `clm_dataverse` | Dataverse connection in the CLM environment |
| `clm_GraphDiscovery_adm` | Discovery service-account Graph connection |
| `clm_AzureDiscovery_adm` | Discovery service-account Azure connection |

### 5. Import the app

Import `CLMApp_1_1_0_0.zip`.

### 6. Import notification automation

Import `CLMNotifications_1_3_0_0.zip`. Map `clm_sharedcommondataserviceforapps_23bc7` to the Dataverse connection.

Enable:

- `Name-CLMNotificationGroupMembership` — event-driven.
- `Resolve-CLMNotificationGroups` — daily at 03:00 Australian Eastern time.
- `Queue-CLMCredentialNotifications` — daily at 07:00 Australian Eastern time.

### 7. Optionally import dispatchers

Only if DLP permits Dataverse with the required outbound connector, import `CLMNotificationDispatchers_1_0_0_0.zip`. Map:

- `clm_sharedcommondataserviceforapps_dispatchers` to Dataverse in this CLM environment.
- `clm_sharedoffice365_clmnotifications` to the approved sending mailbox connection.
- `clm_sharedteams_clmnotifications` to the approved Teams connection.

Enable email and Teams dispatchers independently. Each runs every five minutes and processes up to 100 oldest Pending or Retrying deliveries sequentially.

### 8. Complete initial configuration

1. Assign users `CLM Reader`, `CLM Owner`, or `CLM Platform Ops` as appropriate.
2. Create a monitored default triage Notification Group and Receiver.
3. Run discovery and inspect Credentials, Credential Owners, and Coverage Gaps.
4. Configure routing, run resolution, and inspect assignments.
5. Run queueing and inspect delivery records.
6. Test an approved dispatcher if installed.

## Example values

- App registration: `CLM Discovery Connectors`
- Tenant ID: `<tenant-id>`
- Application client ID: `<application-client-id>`
- Discovery account: `svc-clm-discovery@contoso.com`
- Default group: `CLM Triage`
- Receiver: `clm-operations@contoso.com`

## Expected result

All required solutions are present, connector tests succeed, the discovery flow creates credential or Coverage Gap records, and notification resolution assigns every eligible credential to a Notification Group or triage.

## Common problems

- **Discovery import asks for a connection:** create and test both custom-connector connections first.
- **No subscriptions appear:** grant the signed-in discovery service account Azure `Reader`.
- **Deliveries stay Pending:** no matching dispatcher is enabled, its connection is invalid, or DLP blocks it.
- **Imported memberships have no name briefly:** keep `Name-CLMNotificationGroupMembership` enabled.

## Technical reference

- [Detailed installation reference](../INSTALL.md)
- [Custom connector setup](../CUSTOM_CONNECTORS.md)
- [Solution architecture](../SOLUTION_ARCHITECTURE.md)
