# Set up the CLM custom connectors

Complete this setup after importing `clmPlatformOps` and before importing `CLMDiscoveryFlow`.

The solution contains:

- **CLM Graph Discovery** for Microsoft Graph
- **CLM Azure Discovery** for Azure Resource Manager

Both connectors use one customer-owned, single-tenant Entra app registration. The Power Platform connections run as a dedicated service account.

## Accounts and permissions required

The person performing setup needs:

- Permission to create an Entra app registration
- Permission to grant tenant-wide admin consent
- Permission to edit Power Platform custom connectors
- Permission to assign Azure RBAC roles

The dedicated service account needs:

- A licence that allows it to run the Power Automate flow and access Dataverse
- Access to the target Power Platform environment
- Azure RBAC `Reader` on every scope CLM must discover

## 1. Create the Entra app registration

1. Open **Microsoft Entra admin center**.
2. Go to **Identity > Applications > App registrations**.
3. Select **New registration**.
4. Enter the name `CLM Discovery Connectors`.
5. Under **Supported account types**, select **Accounts in this organizational directory only**.
6. Leave **Redirect URI** empty for now.
7. Select **Register**.
8. On the Overview page, record:
   - **Application (client) ID**
   - **Directory (tenant) ID**

Do not use a multi-tenant registration. Each customer should own their registration and consent.

## 2. Create a client secret

1. Open **Certificates & secrets**.
2. Select **Client secrets > New client secret**.
3. Enter a description such as `CLM custom connectors`.
4. Select the expiry period approved by the customer.
5. Select **Add**.
6. Copy the secret **Value** immediately.

The value is displayed only once. Store it in the customer's approved secret store. Do not place it in documentation, source control, environment variables committed to Git, or support tickets.

The same client ID and client secret are entered into both connectors.

## 3. Add Microsoft Graph permissions

Open **API permissions > Add a permission > Microsoft Graph > Delegated permissions**.

Add:

| Permission | Used by |
|---|---|
| `Application.Read.All` | List applications, service principals, credentials, and application owners |
| `Directory.Read.All` | Read directory and organization information needed by the discovery operations |

These are delegated permissions because the custom-connector connection signs in as the service account.

Do not add application permissions for the current connector design.

## 4. Add Azure permission

Open **API permissions > Add a permission > APIs my organization uses**.

1. Search for **Azure Service Management**.
2. Select **Delegated permissions**.
3. Add `user_impersonation`.

This permission allows the connector application to request an Azure Resource Manager token. It does not grant access to subscriptions by itself; Azure RBAC for the signed-in service account controls what can be discovered.

## 5. Grant admin consent

On **API permissions**:

1. Confirm the list contains:
   - Microsoft Graph `Application.Read.All` - Delegated
   - Microsoft Graph `Directory.Read.All` - Delegated
   - Azure Service Management `user_impersonation` - Delegated
2. Select **Grant admin consent for `<tenant>`**.
3. Confirm the prompt.
4. Verify all three permissions show **Granted for `<tenant>`**.

Do not continue while any permission shows **Not granted**.

## 6. Configure CLM Graph Discovery

The imported connector contains neutral OAuth placeholders. Replace them with the customer app registration values.

In **make.powerapps.com**:

1. Select the customer environment.
2. Open **Solutions > clmPlatformOps**.
3. Open **CLM Graph Discovery**.
4. Select **Edit**, then open **Security**.
5. Enter:

| Setting | Value |
|---|---|
| Authentication type | OAuth 2.0 |
| Identity provider | Azure Active Directory |
| Client ID | Customer app registration client ID |
| Client secret | Customer app registration secret value |
| Login URL | `https://login.microsoftonline.com` |
| Tenant ID | Customer tenant ID |
| Resource URL | `https://graph.microsoft.com` |
| Scope | Leave blank |
| Enable on behalf of login | `false` |

6. Save or update the connector.
7. Copy the displayed **Redirect URL**.

## 7. Configure CLM Azure Discovery

Open **CLM Azure Discovery** and enter the same settings, except:

| Setting | Value |
|---|---|
| Resource URL | `https://management.azure.com` |

Save or update the connector, then copy its **Redirect URL**.

The Graph and Azure redirect URLs are different.

## 8. Register both redirect URLs

Return to the Entra app registration:

1. Open **Authentication**.
2. Select **Add a platform > Web**.
3. Add the Graph connector redirect URL.
4. Add the Azure connector redirect URL.
5. Select **Configure** or **Save**.
6. Leave **Implicit grant and hybrid flows** disabled.
7. Leave **Allow public client flows** disabled.

Both redirect URLs must be present exactly as displayed by the connectors.

## 9. Assign Azure RBAC

The Azure connector uses the signed-in service account's delegated identity, not the app registration's service principal.

Assign the service account:

| Role | Scope |
|---|---|
| `Reader` | Management group, subscription, or resource group that CLM must discover |

Use the highest approved scope to avoid silent coverage gaps. If access is limited to selected subscriptions, assign `Reader` on each one.

The current connector reads Key Vault secret **metadata** through Azure Resource Manager. It does not read secret values. Do not grant `Key Vault Secrets User`, `Contributor`, or `Owner` for this flow.

## 10. Create the Power Platform connections

Use a private browser session signed in as the dedicated service account:

1. Open **make.powerapps.com** and select the customer environment.
2. Open **Connections**.
3. Create a **CLM Graph Discovery** connection.
4. Sign in and accept the consent prompt.
5. Create a **CLM Azure Discovery** connection.
6. Sign in and accept the consent prompt.

The connection owner shown in Power Platform should be the dedicated service account.

## 11. Test both connectors

Open each custom connector from `clmPlatformOps` and use the **Test** tab.

| Connector | Operation | Expected result |
|---|---|---|
| CLM Graph Discovery | `GetOrganization` | Customer tenant details |
| CLM Graph Discovery | `ListApplications` | Applications in the customer tenant |
| CLM Graph Discovery | `ListServicePrincipals` | Enterprise applications/service principals |
| CLM Azure Discovery | `ListSubscriptions` | Subscriptions where the service account has Reader |

Do not import or enable the discovery flow until these tests succeed.

## Permission summary

| Layer | Identity | Required access |
|---|---|---|
| Entra app registration | Connector OAuth application | Graph delegated `Application.Read.All`, `Directory.Read.All`; Azure Service Management delegated `user_impersonation` |
| Microsoft Graph connection | Dedicated service account | Sign-in allowed and connection consent completed |
| Azure connection | Dedicated service account | Azure RBAC `Reader` on discovery scopes |
| Dataverse connection | Dedicated service account | CLM table read/write permissions in the target environment |

## Common errors

| Error | Likely cause |
|---|---|
| `AADSTS50011` redirect URI mismatch | One or both connector redirect URLs are missing or mistyped |
| Invalid client secret | The secret value is wrong, expired, or the secret ID was copied instead of the value |
| Tenant or issuer error | Connector tenant ID was not replaced with the customer tenant ID |
| Graph 401 | Client ID, secret, tenant, or connection sign-in is invalid |
| Graph 403 | Graph permission is missing or admin consent was not granted |
| Azure sign-in succeeds but no subscriptions appear | Service account lacks Azure RBAC Reader |
| Connector works in Test but flow fails | Flow connection reference is mapped to a different connection |

## Secret rotation

Before the client secret expires:

1. Create a new secret on the same app registration.
2. Update the secret in both custom connectors.
3. Save both connectors.
4. Re-authorize or recreate both Power Platform connections if prompted.
5. Test both connectors.
6. Delete the expired secret after the new connections work.
