# CLM solution architecture

CLM uses modular Power Platform solutions and an Azure Action Group-style notification model.

## Target architecture

```text
CLMTables 1.3.1.0
   ├── CLMDiscoveryFlow
   ├── CLMNotifications 1.3.0.0
   ├── CLMNotificationDispatchers 1.0.0.0 (optional)
   └── CLMApp

clmPlatformOps
   └── CLMDiscoveryFlow
```

`CLMTables` is the Dataverse foundation. `clmPlatformOps` supplies discovery connectors. `CLMDiscoveryFlow` discovers credentials. `CLMNotifications` resolves Notification Groups and queues auditable delivery records. The optional `CLMNotificationDispatchers` solution sends queued email and Teams messages where DLP permits. `CLMApp` provides the operator interface.

## Data foundation: CLMTables 1.3.1.0

The vanilla model includes the established discovery and lifecycle tables plus:

| Table | Purpose |
|---|---|
| Notification Group (`clm_notificationgroup`) | Stable operational notification destination with selectable 90, 60, 30, 7, and expiry-day reminders |
| Notification Receiver (`clm_notificationreceiver`) | Email, shared mailbox, distribution or Microsoft 365 group, or Teams channel endpoint |
| Notification Group Membership (`clm_notificationgroupmembership`) | Reusable receiver membership, delivery order, and optional owner-resolution designation for one group |
| Credential Owner (`clm_credentialowner`) | Every current Entra owner discovered for a credential |
| Owner Mapping (`clm_ownermapping`) | Immutable source identifier mapped to a Notification Group |
| Notification Delivery (`clm_notificationdelivery`) | Per-receiver notification audit record |

Credential has a Notification Group lookup. Entra owners are stored as related Credential Owner rows; `clm_ownertag` remains an Azure resource-tag input for Tag-scope rules. Credential does not have custom Owner User, Owner Team, Owner Source, or Owner Locked columns in the desired vanilla model.

Owner Rule conceptually targets a Notification Group. Legacy assignment fields that remain in an upgraded environment are cleanup residue, not part of this architecture.

Notification Receiver may optionally reference the correct Dataverse User and Contact. Those associations provide identity and traceability; they do not replace the receiver endpoint.

## Responsibility model

CLM deliberately separates:

- **Dataverse ownership:** standard Owning User and Owning Team fields used by Dataverse security and record administration.
- **Notification responsibility:** the Notification Group assigned to a credential.

A Notification Group can fan out to multiple receivers, and one receiver can participate in multiple groups without duplicate receiver records.

Membership names use `<receiver> - <group>`. A form script generates the name before an interactive save. The event-driven `Name-CLMNotificationGroupMembership` flow applies the same rule to imported or API-created rows and when either lookup is reassigned.

## Resolution and delivery

`Resolve-CLMNotificationGroups` uses this fixed precedence:

1. Immutable Owner Mapping
2. All matched owners produce one distinct owner-resolution group
3. Owner Rule
4. Default triage group

Discovery stores every Graph owner for each Entra application credential. The resolver matches owner UPN/email values to active receivers and follows memberships marked **Use for Owner Resolution**. One distinct candidate group is selected; zero or multiple candidates continue to rules and triage, with ambiguity recorded in the Renewal Event.

After resolution, `Queue-CLMCredentialNotifications` evaluates the Reminder Days selected by the assigned Notification Group and creates one deduplicated Notification Delivery record per active membership receiver and channel at each enabled threshold. Delivery records preserve the credential, group, receiver, channel, queue state, time, and diagnostic details.

`Dispatch-CLMEmailNotifications` and `Dispatch-CLMTeamsNotifications` run independently every five minutes. Each processes up to 100 oldest Pending or Retrying records for its channel, sends through its dedicated connector, and updates the delivery record to Sent or Failed.

The reusable-receiver redesign does not require a dispatcher package change. Dispatchers consume Notification Delivery records that already identify the target Credential and Notification Receiver; group-membership expansion occurs earlier in `Queue-CLMCredentialNotifications`.

## Solution inventory and readiness

| Solution | Version or state | Responsibility |
|---|---|---|
| `CLMTables` | 1.3.1.0 packaged | Tables, reusable memberships, automatic form naming, all-owner records, choices, relationships, roles, schedules, and public views |
| `clmPlatformOps` | 1.1.0.0 packaged | Graph and Azure custom connectors, including paged application-owner discovery |
| `CLMDiscoveryFlow` | 1.1.0.2 packaged | Daily credential and all-owner discovery |
| `CLMNotifications` | 1.3.0.0 packaged | Multi-owner group resolution, imported membership naming, per-group reminder evaluation, deduplicated queueing, and delivery audit |
| `CLMNotificationDispatchers` | 1.0.0.0 packaged | Optional email and Teams delivery in DLP-compatible environments |
| `CLMApp` | 1.1.0.0 packaged | Model-driven operations interface with owner, membership, notification, and audit pages |
## Installation

Normal deployment uses solution import. Clean installs start with `CLMTables_1_3_1_0.zip`, followed by the packaged connectors, `CLMDiscoveryFlow_1_1_0_2.zip`, `CLMNotifications_1_3_0_0.zip`, and `CLMApp_1_1_0_0.zip`. Install `CLMNotificationDispatchers_1_0_0_0.zip` only where Dataverse, Outlook, and Teams are permitted together.

See [`INSTALL.md`](INSTALL.md) for the precise readiness caveats and order.

## Current implementation gaps

| Gap | Impact |
|---|---|
| Environment DLP blocks Outlook or Teams with Dataverse | Keep the affected dispatcher disabled and use an approved external broker |
| Older environments may retain legacy owner fields | Upgrade cleanup must remove or retire fields outside the vanilla model |
| Enterprise-application owner discovery may be incomplete | Some records will depend on mappings, tags, rules, or triage |
| Discovery does not yet provide a complete run-level audit | Operational review still depends on flow history and existing event records |
