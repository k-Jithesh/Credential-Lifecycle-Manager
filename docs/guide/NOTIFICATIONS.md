# Configure Notifications

## When to use this

Use this page if you want reusable destinations, reminder schedules, auditable queueing, or optional email and Teams delivery.

## What you need

- `CLMNotifications 1.3.0.0`.
- Active Notification Groups and Receivers.
- A DLP decision before enabling `CLMNotificationDispatchers 1.0.0.0`.
- Approved service-account connections for any dispatcher.

## Steps

### Create groups, receivers, and memberships

1. Create one active **Notification Group** per accountable operational function.
2. Select its **Reminder Days**.
3. Create each **Notification Receiver** once. Supported destinations include individual email, shared mailbox, distribution group, Microsoft 365 group, and Teams channel.
4. Add the receiver to each applicable group with an active **Notification Group Membership**.
5. Set **Sort Order** if delivery order matters.
6. If the receiver can route a discovered Entra owner, mark no more than one membership **Use for Owner Resolution**.
7. Optionally associate the correct Dataverse User or Contact for traceability. The receiver endpoint still controls delivery.

Membership names are automatic: `<receiver> - <group>`. The form names interactive records; `Name-CLMNotificationGroupMembership` names imported/API-created records and renames reassigned memberships.

### Use only 30-day and 7-day reminders

On each applicable Notification Group:

1. Open **Reminder Days**.
2. Select **30**.
3. Select **7**.
4. Ensure **90**, **60**, and **expiry day** are not selected.
5. Save the group.

Do not leave Reminder Days blank. A blank value means **all five thresholds**: 90, 60, 30, 7, and expiry day. To stop a group entirely, disable it rather than clearing Reminder Days.

### Queue notifications

1. Ensure credentials have a Notification Group.
2. Run `Queue-CLMCredentialNotifications`, or use its daily 07:00 Australian Eastern schedule.
3. Review Notification Deliveries.

At each enabled threshold, CLM creates one deduplicated record per credential, reminder bucket, active membership receiver, and enabled channel. It does not repeat daily inside a bucket or create a new overdue bucket after expiry day.

### Configure email delivery

1. Confirm DLP permits Dataverse with Office 365 Outlook.
2. Map the dispatcher Dataverse connection to the CLM environment.
3. Map `clm_sharedoffice365_clmnotifications` to an approved mailbox connection.
4. Enable `Dispatch-CLMEmailNotifications`.
5. Test a controlled queued email delivery.

### Configure Teams delivery

1. Confirm DLP permits Dataverse with Microsoft Teams.
2. Map the dispatcher Dataverse connection to the CLM environment.
3. Map `clm_sharedteams_clmnotifications` to an approved Teams connection with access to the destination.
4. Enable `Dispatch-CLMTeamsNotifications`.
5. Test a controlled queued Teams delivery.

Dispatchers run independently every five minutes. Each takes up to 100 oldest Pending or Retrying records for its channel and records Sent or Failed.

### Test end to end

1. Use a controlled credential and monitored test destination.
2. Assign its test Notification Group.
3. Choose a reminder threshold that the credential currently enters.
4. Run queueing.
5. Confirm the expected receiver/channel delivery row is **Pending**.
6. Run or wait for the applicable dispatcher.
7. Confirm the destination received the message and the row is **Sent**.
8. Review provider history before retrying an ambiguous failure; dispatch is at-least-once and a retry can duplicate a message.

## Example values

- Group: `Payments Platform`
- Receiver: `payments-platform@contoso.com`
- Membership name: `payments-platform@contoso.com - Payments Platform`
- Reminder Days: `30`, `7`

## Expected result

The daily queue creates auditable, deduplicated records only at the group's selected thresholds. Approved dispatchers convert Pending records to Sent or Failed.

## Common problems

- **Unexpected 90-day or expiry delivery:** Reminder Days is blank, which enables all thresholds.
- **No delivery record:** check group assignment, threshold, active membership/receiver, suppression, decommissioning, and queue history.
- **Pending never changes:** enable the matching dispatcher and validate its connection and DLP.
- **A deactivated receiver was already queued:** the dispatcher marks its delivery Skipped.
- **One person receives from several groups:** receivers are intentionally reusable; review memberships.

## Technical reference

- [Owner resolution and delivery behavior](../OWNER_RESOLUTION.md)
- [Installation reference](../INSTALL.md)
- [Identity and access](IDENTITY_AND_ACCESS.md)
