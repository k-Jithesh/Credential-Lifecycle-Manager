# Install CLM

CLM is installed through Power Platform solution import. PowerShell is not required.

## Before you start

You need:

- A Power Platform environment with Dataverse
- Permission to import solutions and edit custom connectors
- Permission to create an Entra app registration and grant admin consent
- A dedicated service account for the discovery connections

## 1. Import the tables

In **make.powerapps.com**, select the target environment and open **Solutions**.

Import:

`CLMTables_1_0_0_2.zip`

Wait for the import to finish.

## 2. Import and configure the connectors

Import:

`clmPlatformOps_1_0_0_2.zip`

The connectors contain neutral OAuth placeholders. They will not work until the customer app registration values are entered.

Follow [Set up the custom connectors](CUSTOM_CONNECTORS.md). Complete that guide and create both connections before continuing.

## 3. Import the discovery flow

Import:

`CLMDiscoveryFlow_1_0_0_26.zip`

During import, map:

| Connection reference | Select |
|---|---|
| `clm_dataverse` | A Dataverse connection in the target environment |
| `clm_GraphDiscovery_adm` | The service-account CLM Graph Discovery connection |
| `clm_AzureDiscovery_adm` | The service-account CLM Azure Discovery connection |

## 4. Import the app

Import:

`CLMApp_1_0_0_3.zip`

## 5. Run the first discovery

1. Open the `CLMDiscoveryFlow` solution.
2. Open `Discovery-CLMCredentials`.
3. Turn on the flow.
4. Run it manually.
5. Wait for the run to finish.
6. Open the **Credential Lifecycle** app.
7. Confirm records appear under **Credentials**.
8. Review **Coverage Gaps** for permission or connection failures.

The flow then runs daily at **02:00 AUS Eastern Standard Time**.

## 6. Assign access

Assign users one of the roles included in `CLMTables`:

| Role | Intended user |
|---|---|
| `CLM Reader` | Read-only users |
| `CLM Owner` | Credential owners |
| `CLM Platform Ops` | Administrators and operators |

## 7. Review ownership

Discovery captures an owner hint, but this release does not automatically convert that hint into a Dataverse user or team.

Follow [Owner resolution](OWNER_RESOLUTION.md) before relying on ownership fields or owner-based notifications.

## Quick troubleshooting

| Problem | First check |
|---|---|
| Connector sign-in fails | Client ID, client secret, tenant ID, and redirect URI |
| Graph actions return 403 | Graph delegated permissions and admin consent |
| Azure actions return 403 | Service-account Azure RBAC assignments |
| Flow import asks for missing connections | Create both custom-connector connections first |
| Credentials have an Owner Tag but no Owner User | This is expected; see `OWNER_RESOLUTION.md` |
| No records appear | Flow run history, then Coverage Gaps |
