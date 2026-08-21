# CLM solution architecture

CLM uses modular Power Platform solutions and an Azure Action Group-style notification model.

## Target architecture

```text
CLMTables 1.1.0.7
   ├── CLMDiscoveryFlow
   ├── CLMNotifications 1.0.0.0
   ├── CLMNotificationDispatchers 1.0.0.0 (optional)
   └── CLMApp

clmPlatformOps
   └── CLMDiscoveryFlow
```

`CLMTables` is the Dataverse foundation. `clmPlatformOps` supplies discovery connectors. `CLMDiscoveryFlow` discovers credentials. `CLMNotifications` resolves Notification Groups and queues auditable delivery records. The optional `CLMNotificationDispatchers` solution sends queued email and Teams messages where DLP permits. `CLMApp` provides the operator interface.

## Data foundation: CLMTables 1.1.0.7

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

`Dispatch-CLMEmailNotifications` and `Dispatch-CLMTeamsNotifications` run independently every five minutes. Each processes up to 100 oldest Pending or Retrying records for its channel, sends through its dedicated connector, and updates the delivery record to Sent or Failed.

## Solution inventory and readiness

| Solution | Version or state | Responsibility |
|---|---|---|
| `CLMTables` | 1.1.0.7 packaged | Tables, choices, relationships, and roles |
| `clmPlatformOps` | 1.0.0.2 packaged | Graph and Azure custom connectors |
| `CLMDiscoveryFlow` | 1.0.0.26 packaged | Daily credential discovery |
| `CLMNotifications` | 1.0.0.0 packaged | Group resolution, deduplicated queueing, and delivery audit |
| `CLMNotificationDispatchers` | 1.0.0.0 packaged | Optional email and Teams delivery in DLP-compatible environments |
| `CLMApp` | 1.0.0.5 packaged | Model-driven operations interface with notification administration and audit pages |
## Installation

Normal deployment uses solution import. Clean installs start with `CLMTables_1_1_0_7.zip`, followed by the packaged connectors, discovery flow, `CLMNotifications_1_0_0_0.zip`, and `CLMApp_1_0_0_5.zip`. Install `CLMNotificationDispatchers_1_0_0_0.zip` only where Dataverse, Outlook, and Teams are permitted together.

See [`INSTALL.md`](INSTALL.md) for the precise readiness caveats and order.

## Current implementation gaps

| Gap | Impact |
|---|---|
| Environment DLP blocks Outlook or Teams with Dataverse | Keep the affected dispatcher disabled and use an approved external broker |
| Older environments may retain legacy owner fields | Upgrade cleanup must remove or retire fields outside the vanilla model |
| Enterprise-application owner discovery may be incomplete | Some records will depend on mappings, tags, rules, or triage |
| Discovery does not yet provide a complete run-level audit | Operational review still depends on flow history and existing event records |
