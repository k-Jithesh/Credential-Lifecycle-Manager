# Credential Lifecycle Manager

Credential Lifecycle Manager (CLM) discovers expiring credentials, records coverage gaps, and provides a Dataverse model-driven app for operations.

## Install

Use **solution import** for normal deployment. PowerShell scripts are not required.

Follow the short guide:

**[Install CLM in a Power Platform environment](docs/INSTALL.md)**

The install uses four solutions:

1. `CLMTables`
2. `clmPlatformOps`
3. `CLMDiscoveryFlow`
4. `CLMApp`

The four deployment packages are in `packages/`.

## What each solution does

| Solution | Purpose |
|---|---|
| `CLMTables` | Dataverse tables and choices |
| `clmPlatformOps` | Graph and Azure custom connectors |
| `CLMDiscoveryFlow` | Daily credential discovery |
| `CLMApp` | Model-driven operations app |

## More information

- [Solution architecture](docs/SOLUTION_ARCHITECTURE.md)
- [Custom connector setup](docs/CUSTOM_CONNECTORS.md)
- [Owner resolution](docs/OWNER_RESOLUTION.md)
- [Access and coverage](docs/RBAC_AND_COVERAGE.md)
- [Discovery flow](flows/Discovery-CLMCredentials/README.md)
- [Custom connectors](connector/README.md)
