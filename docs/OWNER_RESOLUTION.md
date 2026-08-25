# Notification responsibility resolution

CLM routes credential notifications through Azure Action Group-style Notification Groups. A group is the stable operational destination; its receivers define the actual delivery endpoints.

The former direct custom Owner User/Owner Team assignment model is superseded by `CLMNotifications 1.1.0.0`.

## Ownership versus notification responsibility

Dataverse **Owning User** and **Owning Team** are standard platform fields used for security, access, and record ownership. They do not identify who must receive credential-renewal notifications.

Notification responsibility is represented by the Credential **Notification Group** lookup. Credential retains **Owner Tag** as discovery input, but the vanilla model has no custom Owner User, Owner Team, Owner Source, or Owner Locked columns.

## Data model

| Table | Purpose |
|---|---|
| Notification Group (`clm_notificationgroup`) | Named operational destination and configurable reminder schedule assigned to credentials |
| Notification Receiver (`clm_notificationreceiver`) | One delivery endpoint belonging to a Notification Group |
| Owner Mapping (`clm_ownermapping`) | Immutable credential or application identifier mapped to a Notification Group |
| Notification Delivery (`clm_notificationdelivery`) | Audit record for an attempted notification to a receiver |
| Owner Rule (`clm_ownerrule`) | Priority-ordered fallback that targets a Notification Group |

A Notification Receiver can represent:

- An individual email address
- A shared mailbox
- A distribution group
- A Microsoft 365 group
- A Teams channel

Where applicable, set the optional Dataverse User or Contact lookup to the correct record. These lookups improve traceability; the receiver address or channel remains the delivery endpoint.

## Resolution precedence

`Resolve-CLMNotificationGroups` assigns exactly one active Notification Group using this order:

1. **Immutable mapping** — an active Owner Mapping matches a stable credential, application, or source object identifier.
2. **Owner Tag** — the retained discovery tag matches an active receiver email address.
3. **Owner Rule** — the first active, priority-ordered matching rule supplies a Notification Group.
4. **Default triage** — the configured triage group receives anything still unresolved.

Immutable mappings take precedence even if display names or personnel change. Rules should be used only for stable patterns, not as a substitute for mappings.

## Operator setup

### Create groups and receivers

1. Create a Notification Group for each accountable operational function, such as Platform Operations or Application Support.
2. Select one or more **Reminder Days**: 90, 60, 30, 7, and expiry day.
3. Add at least one active Notification Receiver to each group.
4. Select the receiver type and enter the monitored email address or Teams channel destination.
5. Optionally associate the correct Dataverse User or Contact.
6. Keep group membership and reminder schedules current when operational responsibilities change.

Prefer shared, monitored destinations over personal addresses for durable coverage.

### Configure reminder days

Reminder Days is a multi-select setting on each Notification Group. The daily queue evaluates a credential against the schedule of its resolved group and creates deliveries only when the credential enters a selected threshold.

A blank Reminder Days value uses all five thresholds—90, 60, 30, 7, and expiry day—to preserve notification coverage for groups upgraded from an earlier release. To stop a group entirely, disable the group rather than clearing Reminder Days.

### Map credentials and applications

Create an Owner Mapping when responsibility is known and must survive renames. Use the most stable available identifiers, such as credential source identity, Entra App ID, or source Object ID, and select the responsible Notification Group.

For bulk onboarding:

1. Export application and credential identifiers, current Entra owners, Owner Tags, and expiry dates.
2. Obtain responsibility confirmation from the relevant operational teams.
3. Create mappings for confirmed assignments.
4. Improve Entra ownership where it is missing or stale.
5. Leave unresolved records for the default triage group rather than creating fragile name rules.

### Configure rules

Create active Owner Rules with explicit priorities and a target Notification Group. Lower priority numbers are evaluated first. Use scopes such as environment, resource group, vault name, display name, or Owner Tag only when the pattern is stable.

Do not target a Dataverse user or team. The desired rule output is a Notification Group.

### Configure default triage

Create a dedicated, active Notification Group with a monitored receiver for unresolved responsibility. Mark it as the single default triage group.

Review the triage queue regularly and replace repeated fallback assignments with immutable mappings or corrected Entra ownership.

## Interpret delivery records

`Queue-CLMCredentialNotifications` creates one deduplicated Notification Delivery queue record per credential, reminder bucket, receiver, and channel. Operators should use it to answer:

- Which credential and Notification Group generated the notification?
- Which receiver and channel were targeted?
- Was delivery attempted, successful, failed, or skipped?
- When was the attempt made?
- What provider response or error was recorded?
- Does a retry or receiver correction remain necessary?

One notification event can produce multiple delivery records when a group has multiple receivers. The optional `CLMNotificationDispatchers` solution provides separate email and Teams flows that process Pending or Retrying records, then record Sent or Failed with attempt details.

The daily queue creates one delivery per receiver and channel when a credential first enters a Reminder Day selected by its Notification Group. Supported thresholds are 90 days, 60 days, 30 days, 7 days, and expiry day. Deduplication prevents daily repeats inside a bucket. The queue does not create a new overdue bucket after the expiry-day threshold has passed.

Dispatch uses at-least-once delivery semantics. If a provider accepts a message but the subsequent Dataverse status update fails, an operator retry can deliver the message again. Review provider history before retrying ambiguous failures. A receiver deactivated after queueing is not contacted; its delivery is marked Skipped.

## Record renewal action and pause reminders

When remediation starts:

1. Set Credential **Status** to **In Renewal**.
2. Populate **Renewal Ticket URL** with the tracking item.
3. Set **Suppressed Until** to the next date on which CLM should remind the group if renewal is still incomplete.

The queue ignores the credential while `Suppressed Until` is in the future. When that time passes, CLM evaluates the credential again and queues the reminder for its current bucket; it does not backfill every bucket crossed during suppression.

Completing the work in the source system should result in discovery recording the new expiry date. Once **Days Until Expiry** is greater than 90, the credential naturally leaves the notification window. Set **Decommissioned** only when the credential is permanently retired.

The current queue permanently excludes only **Decommissioned**. **In Renewal** and **Renewed** are tracking states and do not stop notifications by themselves. `Suppressed Until` is therefore the temporary acknowledgement and pause control. CLM does not currently send a separate “action taken” confirmation message; operators review Status, Renewal Ticket URL, Suppressed Until, and Notification Delivery history in the app.

## Current limitations

- Environments that block Outlook or Teams with Dataverse can use resolution and queueing but must not enable the corresponding dispatcher.
- The dispatcher flows require the CLM queue data in the same DLP-compatible environment. Cross-environment delivery requires an approved external broker.
- Older environments may still contain legacy owner columns or rules and require environment-specific cleanup; those components are not part of the desired vanilla model.
