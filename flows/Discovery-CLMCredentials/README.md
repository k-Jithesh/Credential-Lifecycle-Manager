# Discovery-CLMCredentials

`Discovery-CLMCredentials` is the cloud flow exported in `CLMDiscoveryFlow 1.0.0.26`.

## Schedule and connections

- Runs daily at **02:00 AUS Eastern Standard Time**
- Uses `clm_dataverse`
- Uses `clm_GraphDiscovery_adm`
- Uses `clm_AzureDiscovery_adm`
- Uses a 30-day renewal window

## Current discovery paths

### Entra applications

The Graph leg pages through applications, reads the first owner, and processes password and certificate credentials. The owner's UPN or email is stored as `clm_ownertag`.

### Enterprise applications

The enterprise-application leg pages through service principals and processes their certificate credentials. It does not currently populate an Owner Tag.

### Azure and Key Vault

The ARM leg lists subscriptions, Key Vaults, and Key Vault secret metadata. It uses the first available `Owner`, `owner`, or `OwnerEmail` vault tag as `clm_ownertag`.

### Failure handling

Separate handlers record Graph, enterprise-application, and ARM failures. Coverage failures are written to `clm_coveragegap` with error, scope, status, and resolution information.

## Dataverse dependencies

The exported flow declares dependencies on:

- `clm_credential`
- `clm_sourceenvironment`
- `clm_coveragegap`
- `clm_renewalevent`

The flow reads or writes fields including:

- Credential identity, source, type, status, expiry, portal URL, owner tag, and last-discovered time
- Source-environment scope, tenant, parent scope, and enabled status
- Coverage-gap detection, HTTP/error detail, resolution hint, and failure count
- Renewal-event action, occurrence time, payload, and flow-actor flag

`CLMTables 1.2.0.0` must be imported before this solution.

## Current limitation

The flow initializes `DiscoveryRunId`, but the reviewed export does not create or update a `clm_discoveryrun` row. Run-level audit records therefore require a future flow revision.

The flow captures Owner Tag hints but does not resolve them to Dataverse Owner User or Owner Team lookups. See [`docs/OWNER_RESOLUTION.md`](../../docs/OWNER_RESOLUTION.md).

## Install

Follow [`docs/INSTALL.md`](../../docs/INSTALL.md). Normal deployment uses solution import and does not require PowerShell.

The checked-in `definition.json` and `manifest.json` predate the current release package. Treat `CLMDiscoveryFlow 1.0.0.26` as authoritative until the source tree is refreshed from that package.
