# Route Credentials to Owners

## When to use this

Use this page if you want each credential to resolve to one accountable Notification Group.

## What you need

- Discovered credentials.
- Active Notification Groups, Receivers, and Memberships.
- One monitored default triage group.
- Stable identifiers or carefully tested naming/tag patterns.

## How resolution decides

CLM assigns exactly one group in this order:

1. Active Owner Mapping.
2. All discovered App Registration owners produce exactly one distinct owner-resolution group.
3. First matching active Owner Rule by ascending Priority.
4. Default triage.

Existing assignments are sticky. The resolver only evaluates credentials whose Notification Group is blank.

## Steps

### Enable discovered-owner routing

1. Create each Notification Receiver once, using the owner's normalized UPN or email.
2. Add it to one or more groups through active Memberships.
3. Mark no more than one membership for that receiver **Use for Owner Resolution**.
4. Run discovery to refresh all App Registration owners.
5. Clear the credential's Notification Group if deliberately re-evaluating it.
6. Run `Resolve-CLMNotificationGroups`.

All active memberships receive notifications. Only the designated membership contributes a candidate during owner resolution. Zero or multiple distinct candidates fall through to rules and triage; CLM does not choose the first or majority owner.

### Create an immutable mapping

Create an active Owner Mapping, select its Notification Group, and populate one of:

- **App ID** or **Object ID** for an app-wide mapping.
- **Mapping Key** for one exact credential.

The resolver compares both App ID and Object ID with Credential **Object ID**. In the current discovery package, Entra Credential **Object ID contains the Application (client) ID**, not the Entra directory object ID. Mapping Key compares with Credential **External ID**. Matching is case-insensitive.

#### Copyable mapping recipes

Use the value shown in the credential record whenever possible.

**App Registration — all credentials for one application**

```text
App ID: <application-client-id>
```

You can alternatively put the same Application (client) ID in Owner Mapping **Object ID**.

**App Registration — one secret**

```text
Mapping Key: aad:app:<application-client-id>:secret:<credential-key-id>
```

**App Registration — one certificate**

```text
Mapping Key: aad:app:<application-client-id>:cert:<credential-key-id>
```

**Enterprise Application — all credentials for one application**

```text
App ID: <enterprise-application-app-id>
```

This is the service principal's `appId` (Application ID), not its directory object ID.

**Enterprise Application — one secret**

```text
Mapping Key: aad:sp:<enterprise-application-app-id>:secret:<credential-key-id>
```

**Enterprise Application — one certificate**

```text
Mapping Key: aad:sp:<enterprise-application-app-id>:cert:<credential-key-id>
```

**Key Vault — one exact secret**

```text
Mapping Key: kv:/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.KeyVault/vaults/<vault-name>:secret:<secret-name>
```

Copy Credential External ID rather than retyping it; the Azure resource ID is supplied by Azure and the resolver compares the whole value.

### Create fallback rules

Owner Rules use case-insensitive **contains**, not exact match or regular expressions. Use a unique Priority; lower numbers run first. Never leave Match Pattern blank.

**Key Vault tag routing**

```text
Match Scope: Tag
Match Pattern: payments-platform@contoso.com
Notification Group: Payments Platform
```

This examines the vault's first `Owner`, `owner`, or `OwnerEmail` value copied to Owner Tag.

**Vault-name routing**

```text
Match Scope: KeyVaultName
Match Pattern: kv-payments-prod
Notification Group: Payments Platform
```

This examines text before the first `/` in `<vault-name>/<secret-name>`.

**Resource-group routing**

```text
Match Scope: ResourceGroup
Match Pattern: rg-payments-prod
Notification Group: Payments Platform
```

This uses the resource-group segment parsed from Source Portal URL.

Other supported scopes are **DisplayName** and **Environment**.

### Configure default triage

1. Create a dedicated active group such as `CLM Triage`.
2. Add at least one monitored receiver.
3. Mark it as the single default triage group.
4. Review repeated triage assignments and replace them with corrected Entra ownership, mappings, or narrow rules.

### Re-resolve an assignment

1. Check whether an active mapping or discovered owner should still win.
2. Clear the credential's Notification Group.
3. Run `Resolve-CLMNotificationGroups`.
4. Verify the group and `NotificationGroupResolved` Renewal Event.

## Expected result

Each eligible credential has one Notification Group. Ambiguous or unresolved ownership safely reaches rules or default triage.

## Common problems

- **App-wide mapping does not match:** use the Application (client) ID; do not use the application/service-principal directory object ID.
- **Individual mapping does not match:** copy the complete Credential External ID into Mapping Key.
- **A changed rule does not move credentials:** clear the existing Notification Group before resolving.
- **A rule catches too much:** use a longer pattern; matching is substring-based.
- **Enterprise Application has no owners:** this is a current limitation; use mapping or rule routing.
- **Multiple owners produce no owner group:** their designated memberships resolve to zero or multiple distinct groups, so CLM falls through safely.

## Technical reference

- [Owner resolution internals](../OWNER_RESOLUTION.md)
- [Discovery](DISCOVERY.md)
- [Notifications](NOTIFICATIONS.md)
