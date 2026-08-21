# CLM solution architecture

CLM uses modular Power Platform solutions and an Azure Action Group-style notification model.

## Target architecture

```text
CLMTables 1.1.0.6
   ├── CLMDiscoveryFlow
   ├── CLMNotifications 1.0.0.0
   └── CLMApp

clmPlatformOps
   └── CLMDiscoveryFlow
```

`CLMTables` is the Dataverse foundation. `clmPlatformOps` supplies discovery connectors. `CLMDiscoveryFlow` discovers credentials. `CLMNotifications` resolves Notification Groups and queues auditable delivery records. `CLMApp` provides the operator interface.

## Data foundation: CLMTables 1.1.0.6

The vanilla model includes the established discovery and lifecycle tables plus:

| Table | Purpose |
|---|---|
| Notification Group (`clm_notificationgroup`) | Stable operational notification destination |
| Notification Receiver (`clm_notificationreceiver`) | Email, shared mailbox, distribution or Microsoft 365 group, or Teams channel endpoint |
| Owner Mapping (`clm_ownermapping`) | Immutable source identifier mapped to a Notification Group |
| Notification Delivery (`clm_notificationdelivery`) | Per-receiver notification audit record |

Credential has a Notification Group lookup and retains Owner Tag as resolution input. It does not have custom Owner User, Owner Team, Owner Source, or Owner Locked columns in the desired vanilla model.

Owner Rule conceptually targets a Notification Group. Legacy assignment fields that remain in an upgraded environment are cleanup residue, not part of this architecture.

Notification Receiver may optionally reference the correct Dataverse User and Contact. Those associations provide identity and traceability; they do not replace the receiver endpoint.

## Responsibility model

CLM deliberately separates:

- **Dataverse ownership:** standard Owning User and Owning Team fields used by Dataverse security and record administration.
- **Notification responsibility:** the Notification Group assigned to a credential.

A Notification Group can fan out to multiple receivers, allowing a shared mailbox, distribution or Microsoft 365 group, Teams channel, or individual email endpoint to receive the same operational notification.

## Resolution and delivery

`Resolve-CLMNotificationGroups` uses this fixed precedence:

1. Immutable Owner Mapping
2. Owner Tag receiver match
3. Owner Rule
4. Default triage group

After resolution, `Queue-CLMCredentialNotifications` creates one deduplicated Notification Delivery record per receiver and channel. Delivery records preserve the credential, group, receiver, channel, queue state, time, and diagnostic details.

## Solution inventory and readiness

| Solution | Version or state | Responsibility |
|---|---|---|
| `CLMTables` | 1.1.0.6 packaged | Tables, choices, relationships, and roles |
| `clmPlatformOps` | 1.0.0.2 packaged | Graph and Azure custom connectors |
| `CLMDiscoveryFlow` | 1.0.0.26 packaged | Daily credential discovery |
| `CLMNotifications` | 1.0.0.0 packaged | Group resolution, deduplicated queueing, and delivery audit |
| `CLMApp` | 1.0.0.5 packaged | Model-driven operations interface with notification administration and audit pages |
## Installation

Normal deployment uses solution import. Clean installs start with `CLMTables_1_1_0_6.zip`, followed by the packaged connectors, discovery flow, `CLMNotifications_1_0_0_0.zip`, and `CLMApp_1_0_0_5.zip`.

See [`INSTALL.md`](INSTALL.md) for the precise readiness caveats and order.

## Current implementation gaps

| Gap | Impact |
|---|---|
| Environment DLP blocks Outlook and Teams with Dataverse | The packaged flow creates Pending/Retrying delivery records; an approved transport broker must dispatch them |
| Older environments may retain legacy owner fields | Upgrade cleanup must remove or retire fields outside the vanilla model |
| Enterprise-application owner discovery may be incomplete | Some records will depend on mappings, tags, rules, or triage |
| Discovery does not yet provide a complete run-level audit | Operational review still depends on flow history and existing event records |
