# CLM discovery connectors

CLM uses two custom connectors because Power Platform custom connectors are single-host.

| Connector | Logical name | Host | Main operations |
|---|---|---|---|
| CLM Graph Discovery | `clm_5Fclmgraphdiscovery` | `graph.microsoft.com` | Applications, application owners, and service principals |
| CLM Azure Discovery | `clm_5Fclmarmdiscovery` | `management.azure.com` | Subscriptions, Key Vaults, and vault secret metadata |

## Solution ownership

The owner is **`clmPlatformOps 1.1.0.0`**. The Graph connector declares paged application-owner responses and exposes owner UPN and mail fields.

`CLMDiscoveryFlow 1.1.0.2` contains only the flow and depends on these connectors. Import and configure `clmPlatformOps` first.

## Authentication

Both connectors use the supported Azure AD OAuth identity provider with delegated connections. The unattended flow must use connections owned by an approved service account with the required Graph and Azure permissions.

The connector definitions contain neutral OAuth placeholders. A customer installation must replace them with the customer app registration values and supply a customer-owned client secret before creating connections.

The discovery flow references:

| Connection reference | Connector |
|---|---|
| `clm_GraphDiscovery_adm` | CLM Graph Discovery |
| `clm_AzureDiscovery_adm` | CLM Azure Discovery |
| `clm_dataverse` | Microsoft Dataverse |

Connector authentication values and service-account connections must be configured per environment; they must not be committed to source control.

## Install

Follow [`docs/CUSTOM_CONNECTORS.md`](../docs/CUSTOM_CONNECTORS.md), then continue with [`docs/INSTALL.md`](../docs/INSTALL.md).
