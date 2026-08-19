# CLM solution architecture

This document describes the four modular solutions included in `packages/`.

## Decision

CLM uses five modular solutions:

```text
CLMTables
   ├── CLMDiscoveryFlow
   ├── CLMOwnerResolver
   └── CLMApp

clmPlatformOps
   └── CLMDiscoveryFlow
```

`CLMTables` is the data foundation. `clmPlatformOps` supplies the discovery connectors. `CLMDiscoveryFlow` performs discovery. `CLMOwnerResolver` assigns users or teams. `CLMApp` provides the operator interface.

## Reviewed solution inventory

### CLMTables 1.0.0.3

**Root components:** 19

- Six tables
- Ten global choices
- Three security roles

The solution includes the standard table forms, views, relationships, charts, and the `clm_daysuntilexpiry` formula:

```powerfx
DateDiff(Now(), clm_expirydate, TimeUnit.Days)
```

The six tables are:

| Table | Purpose |
|---|---|
| `clm_credential` | Discovered credential inventory and lifecycle state |
| `clm_sourceenvironment` | Tenant, subscription, vault, and environment discovery scopes |
| `clm_coveragegap` | Discovery failures and inaccessible scopes |
| `clm_discoveryrun` | Intended run-level audit record |
| `clm_renewalevent` | Credential lifecycle and discovery event history |
| `clm_ownerrule` | Rules used to resolve or infer credential ownership |

The global choices are `clm_sourcesystem`, `clm_credentialtype`, `clm_credentialstatus`, `clm_reminderbucket`, `clm_runstatus`, `clm_eventaction`, `clm_matchscope`, `clm_scopetype`, `clm_gaptype`, and `clm_gapstatus`.

The `clm_ownersource` field is present on `clm_credential` as a local choice. It is no longer included as a global choice.

The included roles are `CLM Reader`, `CLM Owner`, and `CLM Platform Ops`.

### clmPlatformOps 1.0.0.2

**Root components:** 2

- `clm_5Fclmgraphdiscovery`
- `clm_5Fclmarmdiscovery`

This solution should be the long-term owner of the custom connectors.

### CLMDiscoveryFlow 1.0.0.26

**Root components:** 1

- `Discovery-CLMCredentials`

The flow runs daily at 02:00 AUS Eastern Standard Time and has four table dependencies: credential, source environment, coverage gap, and renewal event.

The custom connectors are owned only by `clmPlatformOps 1.0.0.2`.

### CLMOwnerResolver 1.0.0.6

**Root components:** 1

- `OwnerResolver-CLMCredentials`

The resolver runs daily at 03:00 AUS Eastern Standard Time and uses Dataverse only. It resolves Owner Tags to active users, applies priority-ordered Owner Rules, assigns either Owner User or Owner Team, and records audit events.

### CLMApp 1.0.0.3

**Root components:** 5

- Credential table component
- `CLM Operations` dashboard
- Connector icon web resource
- `clm_CredentialLifecycle` sitemap
- `clm_CredentialLifecycle` model-driven app

The app navigation references all six CLM tables. Its dashboard and charts also depend on table views and columns supplied by `CLMTables`.

## Installation

Normal deployment uses solution import only. Follow [`INSTALL.md`](INSTALL.md) for the single supported setup path.

## Architecture gaps

| Gap | Impact | Required change |
|---|---|---|
| Connector OAuth settings use neutral placeholders | Connections fail until customer values are entered | Set client ID, tenant ID, secret, and redirect URIs during installation |
| Enterprise-application Owner Tag is not populated | Enterprise-application credentials require rule or manual assignment | Add service-principal owner discovery |
| Random application names do not support reliable pattern rules | Large estates retain unresolved ownership | Add immutable App ID/Object ID mappings |
| Discovery flow does not write `clm_discoveryrun` | No run-level audit record despite the table and variable | Add create/update actions around each flow execution |
| Checked-in source predates the current packages | Scripts and JSON may recreate older architecture | Refresh source from the release packages before further development |
