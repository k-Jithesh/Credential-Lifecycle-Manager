# Identity and Access

## When to use this

Use this page if you want to approve CLM access, troubleshoot authorization, or avoid assigning a role to the wrong identity.

## What you need

- An Entra administrator who can create an app registration and grant admin consent.
- An Azure administrator who can assign `Reader`.
- A Power Platform administrator who can import solutions, manage connections, and review DLP.
- Dedicated, monitored service accounts for unattended connections.

## Keep these identities separate

| Identity | What it is | Exact access |
|---|---|---|
| Connector app registration | OAuth client used by both custom connectors | Graph delegated `Application.Read.All`, `Directory.Read.All`; Azure Service Management delegated `user_impersonation`; tenant admin consent |
| Discovery connection owner | Signed-in service account and effective identity for Graph and Azure calls | Sign-in and connection consent; Power Platform environment access; Azure `Reader` on every approved discovery scope |
| Dataverse connection owner | Identity used by `clm_dataverse` and notification flows | Read/write access to the CLM tables used by the flows; normally use the included role appropriate to administration |
| Email dispatcher connection owner | Account behind the Office 365 Outlook connection | Approved sending mailbox access; Dataverse access through the separate dispatcher Dataverse connection |
| Teams dispatcher connection owner | Account behind the Microsoft Teams connection | Approved access to post to the configured channel; Dataverse access through the separate dispatcher Dataverse connection |
| CLM app user | Human reader, stakeholder, or operator | `CLM Reader`, `CLM Owner`, or `CLM Platform Ops` |

The connector app registration is **not** the effective Azure RBAC principal. The Azure connector uses delegated OAuth, so assign Azure `Reader` to the signed-in discovery service account, not to the app registration's service principal.

The current Graph connector is authorized through the admin-consented delegated permissions above. Do not add Global Reader merely as a substitute for missing connector consent; use a directory role only when your tenant's governance policy separately requires it.

## Steps

### Configure connector permissions

1. Create a customer-owned, single-tenant app registration.
2. Add Graph delegated `Application.Read.All`.
3. Add Graph delegated `Directory.Read.All`.
4. Add Azure Service Management delegated `user_impersonation`.
5. Grant tenant admin consent for all three.
6. Use the same app client ID and secret in both custom connectors.
7. Register each connector's exact redirect URL.
8. Sign in to both connections as the dedicated discovery service account.

`AuditLog.Read.All` is optional and only supports a future/audit-based creator fallback; it is not required for the current discovery paths.

### Grant Azure access

Grant the discovery service account Azure `Reader` at the highest approved scope:

- Management group for broad visibility.
- Subscription for subscription-scoped discovery.
- Resource group for a deliberately narrow scope.
- Vault scope where access is intentionally vault-specific.

The current connector reads vault and secret **metadata** through Azure Resource Manager. It does not read secret values. Do not grant `Key Vault Secrets User`, `Contributor`, or `Owner` for current discovery.

### Grant Dataverse and app access

Use the included security roles:

- `CLM Reader` for read-only users.
- `CLM Owner` for credential stakeholders.
- `CLM Platform Ops` for administrators and operators.

The Dataverse flow connection needs read/write access to the CLM records it manages, including credentials, source environments, coverage gaps, renewal events, rules, groups, mappings, owners, memberships, and deliveries.

### Review DLP before dispatch

Resolution and queueing use Dataverse only. Email dispatch additionally uses Office 365 Outlook; Teams dispatch additionally uses Microsoft Teams.

- If DLP allows Dataverse with Outlook, enable email dispatch.
- If DLP allows Dataverse with Teams, enable Teams dispatch only after confirming that the Teams Workflows (Power Automate) app and Flow bot are enabled and allowed for the destination.
- If either combination is blocked, keep that dispatcher disabled.
- Installing dispatchers in a separate environment does not process the source queue because dispatchers use the current environment's CLM tables.
- Use an approved external broker for cross-environment delivery.

## Example values

- Connector app: `CLM Discovery Connectors`
- Discovery account: `svc-clm-discovery@contoso.com`
- Dispatcher account: `svc-clm-notify@contoso.com`
- Azure scope: `/subscriptions/<subscription-id>`

## Expected result

Graph tests return tenant applications and service principals, Azure tests return only authorized subscriptions, Dataverse writes succeed, and enabled dispatchers can access their approved destinations.

## Common problems

- **Graph 403:** a delegated permission or tenant admin consent is missing.
- **Azure sign-in works but returns no subscriptions:** `Reader` is missing from the discovery account.
- **The app service principal has Reader but discovery still fails:** RBAC was granted to the wrong identity.
- **Dispatcher import or run is blocked:** review the environment's DLP connector grouping.
- **Personal account prompts interrupt unattended flows:** recreate connections under approved dedicated accounts.

## Technical reference

- [RBAC and coverage](../RBAC_AND_COVERAGE.md)
- [Custom connector setup](../CUSTOM_CONNECTORS.md)
- [Solution architecture](../SOLUTION_ARCHITECTURE.md)
