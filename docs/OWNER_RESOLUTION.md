# Notification responsibility resolution

CLM routes credential notifications through Azure Action Group-style Notification Groups. A group is the stable operational destination; its receivers define the actual delivery endpoints.

The former direct custom Owner User/Owner Team assignment model is superseded by `CLMNotifications 1.2.0.0`.

## Ownership versus notification responsibility

Dataverse **Owning User** and **Owning Team** are standard platform fields used for security, access, and record ownership. They do not identify who must receive credential-renewal notifications.

Notification responsibility is represented by the Credential **Notification Group** lookup. Discovery stores every Entra application owner as a related Credential Owner. Azure resource owner tags remain available in the Credential **Owner Tag** column for Tag-scope rules. The vanilla model has no custom Owner User, Owner Team, Owner Source, or Owner Locked columns.

## Data model

| Table | Purpose |
|---|---|
| Notification Group (`clm_notificationgroup`) | Named operational destination and configurable reminder schedule assigned to credentials |
| Notification Receiver (`clm_notificationreceiver`) | Reusable email or Teams delivery endpoint |
| Notification Group Membership (`clm_notificationgroupmembership`) | Associates one reusable receiver with one group and optionally designates that group for owner resolution |
| Credential Owner (`clm_credentialowner`) | One owner returned by Graph for one credential |
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
2. **Discovered owners** — all active Credential Owners are matched to active receiver email addresses. If their owner-resolution memberships produce exactly one distinct group, that group is selected.
3. **Owner Rule** — the first active, priority-ordered matching rule supplies a Notification Group.
4. **Default triage** — the configured triage group receives anything still unresolved.

Immutable mappings take precedence even if display names or personnel change. Rules should be used only for stable patterns, not as a substitute for mappings.

If discovered owners produce multiple candidate groups, CLM does not select the first owner or a majority. It records the ambiguity and continues to Owner Rules and default triage.

## Operator setup

### Create groups and receivers

1. Create a Notification Group for each accountable operational function, such as Platform Operations or Application Support.
2. Select one or more **Reminder Days**: 90, 60, 30, 7, and expiry day.
3. Create each Notification Receiver once, then select its type and enter the monitored email address or Teams channel destination.
4. Create an active Notification Group Membership for every group that should notify the receiver.
5. Set membership **Sort Order** to control delivery ordering.
6. Mark no more than one membership per receiver **Use for Owner Resolution**. Other memberships still receive notifications but do not make the receiver an ownership-routing candidate.
7. Optionally associate the correct Dataverse User or Contact.
8. Keep group memberships and reminder schedules current when operational responsibilities change.

Prefer shared, monitored destinations over personal addresses for durable coverage.

### Configure reminder days

Reminder Days is a multi-select setting on each Notification Group. The daily queue evaluates a credential against the schedule of its resolved group and creates deliveries only when the credential enters a selected threshold.

A blank Reminder Days value uses all five thresholds—90, 60, 30, 7, and expiry day—to preserve notification coverage for groups upgraded from an earlier release. To stop a group entirely, disable the group rather than clearing Reminder Days.

### Map credentials and applications

Create an Owner Mapping when responsibility is known and must survive renames. Use the most stable available identifiers, such as credential source identity, Entra App ID, or source Object ID, and select the responsible Notification Group.

For bulk onboarding:

1. Export application and credential identifiers, related Credential Owners, Azure owner tags, and expiry dates.
2. Obtain responsibility confirmation from the relevant operational teams.
3. Create mappings for confirmed assignments.
4. Improve Entra ownership where it is missing or stale.
5. Leave unresolved records for the default triage group rather than creating fragile name rules.

### Understand discovered-owner resolution

App registrations do not support Azure resource tags. For each discovered application credential, CLM stores every owner returned by Microsoft Graph as a Credential Owner. Each owner is matched to Notification Receiver email addresses using normalized UPN or email values.

Receiver membership and routing designation are separate:

- Every active membership makes the receiver a delivery endpoint for that group.
- Only memberships marked **Use for Owner Resolution** make their group a candidate when that receiver matches an owner.
- One distinct candidate group is assigned automatically.
- Zero candidates continue to Owner Rules and default triage.
- Multiple candidates are recorded as ambiguous, then continue to Owner Rules and default triage.

Use an immutable Owner Mapping when a credential or application needs a responsibility assignment that differs from an owner's normal routing group. Do not mark multiple memberships for the same receiver as owner-resolution memberships unless ambiguity is intentional.

For Azure Key Vault, the first available `Owner`, `owner`, or `OwnerEmail` resource tag remains in `clm_ownertag`. It is evaluated only by a Tag-scope Owner Rule; it does not use discovered-owner receiver matching.

Enterprise-application credentials do not currently populate Credential Owner rows. Route them with an immutable Owner Mapping or Owner Rule.

### Configure rules

Owner Rules are ordered, case-insensitive substring matches used only after immutable mappings and unambiguous discovered-owner resolution fail.

| Field | Required behavior |
|---|---|
| Rule Name | Human-readable purpose, such as `Payments applications` |
| Active | Only active rules are loaded |
| Priority | Lower numbers are evaluated first; use unique values to avoid nondeterministic ties |
| Match Scope | Credential value to inspect |
| Match Pattern | Case-insensitive substring; do not leave blank because an empty substring matches every value |
| Notification Group | Group assigned when the rule wins |
| Match Count / Last Matched On | Present for reporting, but the current resolver does not update them |

The resolver loads up to 500 active rules that have a Notification Group and orders them by Priority ascending. For each unassigned, non-decommissioned, non-system credential, the first matching rule wins.

#### Match scopes

| Match Scope | Value evaluated | Example pattern | Example use |
|---|---|---|---|
| DisplayName | Credential Display Name, falling back to Name | `payments-` | Route applications with a stable naming prefix |
| Tag | Azure owner tag stored in `clm_ownertag` | `platform@contoso.com` | Route a stable Key Vault owner-tag value |
| Environment | Credential Environment | `production` | Route credentials discovered in a named environment |
| KeyVaultName | Text before the first `/` in the Key Vault credential display name | `kv-payments-prod` | Route secrets from a specific vault |
| ResourceGroup | Resource-group segment parsed from Source Portal URL | `rg-payments-prod` | Route Azure credentials in a resource group |

Patterns use `contains`, not exact match or regular expressions. Prefer a sufficiently specific value to prevent unintended matches; for example, `rg-payments-prod` is safer than `prod`.

#### Precedence and persistence

Rules cannot override an immutable mapping or unambiguous discovered-owner result. The resolver also evaluates only credentials whose Notification Group is empty. An existing assignment is therefore sticky: changing owners, memberships, rule priority, or patterns does not move already assigned credentials.

To deliberately re-evaluate a credential:

1. Confirm no immutable Owner Mapping or unambiguous discovered-owner routing should take precedence.
2. Clear the Credential's Notification Group.
3. Run `Resolve-CLMNotificationGroups`.
4. Verify the resulting group and the `NotificationGroupResolved` Renewal Event.

#### Test a rule safely

1. Keep the production rule inactive while drafting it.
2. Select a unique priority and confirm no active rule uses the same value.
3. Inspect representative Credential values for the selected scope.
4. Test with a narrow pattern in a non-production environment or against a controlled unassigned credential.
5. Confirm the expected Notification Group and `OwnerRule` resolution source in the Renewal Event.
6. Test at least one near-match that must not be assigned.
7. Activate the production rule only after both the positive and negative cases pass.
8. Review **Orphans (No Notification Group)** after the next resolver run.

Do not target a Dataverse user or team. The rule output is always a Notification Group.

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

## Pause reminders during renewal

When remediation starts:

1. Set Credential **Status** to **In Renewal**.
2. Populate **Renewal Ticket URL** with the tracking item.
3. Set **Suppressed Until** to the next date on which CLM should remind the group if renewal is still incomplete.

The queue ignores the credential while `Suppressed Until` is in the future. When that time passes, CLM evaluates the credential again and queues the reminder for its current bucket; it does not backfill every bucket crossed during suppression.

## Stop alerts after renewal

After rotating or renewing a certificate, secret, or key:

1. Complete the change in Entra ID, Key Vault, or the originating system.
2. Run `Discovery-CLMCredentials`, or wait for its next scheduled run.
3. Confirm that CLM shows the replacement credential or the updated expiry date.
4. If discovery updated the same Credential and **Days Until Expiry** is greater than 90, no further queue records are created for it.
5. If rotation created a new Credential record, set the replaced Credential's **Status** to **Decommissioned**. This permanently excludes the old record from resolution and future queueing.
6. Open Notification Deliveries for the replaced Credential. Change any existing **Pending** or **Retrying** deliveries to **Skipped** so a dispatcher cannot send alerts that were queued before renewal completed.
7. Set the replacement Credential to the appropriate active status and clear any obsolete **Suppressed Until** or renewal-ticket values after confirming the new expiry.

Do not deactivate a shared Notification Receiver to stop alerts for one credential. The receiver may participate in multiple Notification Groups.

The current queue permanently excludes only **Decommissioned**. **In Renewal** and **Renewed** are tracking states and do not stop notifications by themselves. `Suppressed Until` is therefore the temporary acknowledgement and pause control. CLM does not currently send a separate “action taken” confirmation message; operators review Status, Renewal Ticket URL, Suppressed Until, and Notification Delivery history in the app.

## Current limitations

- Environments that block Outlook or Teams with Dataverse can use resolution and queueing but must not enable the corresponding dispatcher.
- The dispatcher flows require the CLM queue data in the same DLP-compatible environment. Cross-environment delivery requires an approved external broker.
- Older environments may still contain legacy owner columns or rules and require environment-specific cleanup; those components are not part of the desired vanilla model.
- The resolver does not currently update Owner Rule Match Count or Last Matched On.
- Equal rule priorities do not have a guaranteed secondary ordering.
- The model relies on operators to keep at most one **Use for Owner Resolution** membership per receiver; multiple designated groups safely become an ambiguous result.
