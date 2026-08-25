# Discover Credentials

## When to use this

Use this page if you want to inventory Entra or Key Vault credentials, verify discovery access, understand tags, or investigate missing coverage.

## What you need

- Successfully tested Graph and Azure custom-connector connections.
- `CLMDiscoveryFlow 1.1.0.2`.
- The discovery service account and Azure `Reader` assignments described in [Identity and access](IDENTITY_AND_ACCESS.md).

## Steps

### Discover Entra credentials

1. Run `Discovery-CLMCredentials`, or allow its daily 02:00 AUS Eastern Standard Time schedule to run.
2. Review App Registration secrets and certificates in **Credentials**.
3. Review their related **Credential Owners**.
4. Review Enterprise Application secrets and certificates.
5. Route Enterprise Application credentials with mappings or rules because they currently have no discovered Credential Owner rows.

Discovery keeps credentials whose expiry is in the future or no more than 30 days in the past. It uses a 30-day renewal window for renewal-event handling.

For App Registrations, every owner returned by Graph is refreshed into Credential Owner records. For Enterprise Applications, owner discovery is not currently populated.

### Discover Azure subscriptions and Key Vault secrets

1. Confirm the Azure connector `ListSubscriptions` test returns each intended subscription.
2. Run discovery.
3. Review discovered vault secret metadata. CLM does not read secret values.
4. Confirm the displayed credential name follows `<vault-name>/<secret-name>`.
5. Review **Owner Tag**. CLM copies the first available vault resource tag named `Owner`, `owner`, or `OwnerEmail`.
6. Review **Coverage Gaps** for inaccessible subscriptions, vaults, or metadata.

The owner tag is read from the **vault**, not from the individual secret. It becomes routing input only when a Tag-scope Owner Rule uses it.

### Verify access after a change

1. Test `ListSubscriptions` in CLM Azure Discovery.
2. Run `Discovery-CLMCredentials` manually.
3. Confirm the target vault produces credentials.
4. Review open Coverage Gaps and their resolution hints.
5. Confirm a previously failing scope no longer produces a current failure.

## Example values

- App Registration: `Payments API`
- Application (client) ID stored in Credential **Object ID**: `<application-client-id>`
- Vault: `kv-payments-prod`
- Secret: `payment-api-client-secret`
- Vault tag: `Owner=payments-platform@contoso.com`
- Resource group: `rg-payments-prod`

## Expected result

CLM contains credential metadata, expiry dates, source links, and App Registration owners for accessible sources. Authorization, network, disabled-resource, throttling, and other discovery failures appear as Coverage Gaps instead of looking like an empty inventory.

## Common problems

- **No applications:** test Graph `ListApplications`; check `Application.Read.All` and admin consent.
- **No Enterprise Applications:** test `ListServicePrincipals`; check `Directory.Read.All`.
- **No subscriptions:** assign Azure `Reader` to the signed-in discovery service account.
- **A vault is missing:** verify its subscription is returned and the account has `Reader` at an applicable scope.
- **Owner Tag is blank:** add one of the supported tags to the vault resource; secret-level tags are not used.
- **No Discovery Run row:** current discovery initializes an ID but does not create or update complete Discovery Run records. Use flow history, Renewal Events, and Coverage Gaps.

## Current limitations

- Enterprise Application owner rows are not populated.
- Complete run-level audit records are not written.
- Firewall, private endpoint, Conditional Access, and tenant restrictions can still block delegated connector requests.
- The checked-in discovery source predates the package; `CLMDiscoveryFlow 1.1.0.2` is authoritative.

## Technical reference

- [Packaged discovery flow](../../flows/Discovery-CLMCredentials/README.md)
- [RBAC and coverage](../RBAC_AND_COVERAGE.md)
- [Discovery failure pattern](../DISCOVERY_FLOW_PATTERN.md) — includes target/future behavior; read its current-behavior warning.
