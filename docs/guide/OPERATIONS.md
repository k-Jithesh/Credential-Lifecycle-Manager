# Operate CLM

## When to use this

Use this page if you run CLM day to day, acknowledge renewal work, correct routing, or close out replaced credentials.

## What you need

- `CLM Platform Ops` for administrative operations, or the role appropriate to your duties.
- Access to Credentials, Coverage Gaps, Renewal Events, Notification Deliveries, and flow run history.
- A monitored default triage process.

## Daily steps

### Review discovery and coverage

1. Check `Discovery-CLMCredentials` flow history.
2. Review open **Coverage Gaps**, especially after failed or partial runs.
3. Follow the recorded resolution hint and correct the connection, consent, RBAC, network, or scope.
4. Re-run discovery after access changes.
5. Confirm the source now produces credential metadata.

Do not treat “no credentials found” as proof of complete coverage while a relevant Coverage Gap is open.

### Review routing gaps and orphans

1. Review **Orphans (No Notification Group)** and the triage group.
2. Inspect App Registration Credential Owners.
3. Correct reusable receiver email/UPN and owner-resolution membership where appropriate.
4. Add an immutable mapping for known responsibility.
5. Use narrow Owner Rules only for stable fallback patterns.
6. Re-resolve deliberately by clearing Notification Group and running `Resolve-CLMNotificationGroups`.

### Review delivery

1. Review Pending, Retrying, Failed, and Skipped Notification Deliveries.
2. Correct invalid receivers or dispatcher connections.
3. Check provider history before moving Failed to Retrying.
4. Keep dispatchers disabled when DLP does not permit them.

## Renewal steps

### Acknowledge and suppress

1. Set Credential **Status** to **In Renewal**.
2. Add the tracking link to **Renewal Ticket URL**.
3. Set **Suppressed Until** to the next follow-up date.

`In Renewal` does not stop queueing by itself. A future Suppressed Until date pauses queueing. When it passes, CLM evaluates only the current bucket; it does not backfill every threshold crossed.

### Complete a renewal

1. Rotate or renew in Entra, Key Vault, or the source system.
2. Run discovery or wait for the next daily run.
3. Confirm the replacement or updated expiry appears in CLM.
4. If the same credential record now has more than 90 days remaining, no further current reminder is queued.
5. If rotation created a new Credential record, mark the replaced record **Decommissioned**.
6. On the replaced record, change existing **Pending** or **Retrying** deliveries to **Skipped**.
7. On the active replacement, clear obsolete Suppressed Until and renewal-ticket values after verification.

Only **Decommissioned** is permanently excluded. `In Renewal` and `Renewed` are tracking states and do not stop reminders by themselves.

### Decommission safely

1. Confirm the source credential is retired.
2. Set its CLM Status to **Decommissioned**.
3. Mark any queued Pending or Retrying deliveries **Skipped**.
4. Retain delivery and renewal history for audit.

Do not deactivate a shared Receiver to stop one credential; it may serve several groups.

## Expected result

Coverage failures are visible and remediated, unresolved responsibility reaches triage, renewal work is temporarily suppressed with a follow-up date, and replaced credentials cannot send stale queued notices.

## Common problems

- **In Renewal still generated a notice:** set Suppressed Until; status alone does not pause queueing.
- **A stale notice sent after renewal:** existing queue records were not changed to Skipped.
- **A corrected rule did not reassign:** clear the current Notification Group before resolving.
- **A retry may duplicate delivery:** check the provider history first; dispatch uses at-least-once semantics.
- **No run-level record exists:** use flow history, Coverage Gaps, and Renewal Events; complete Discovery Run writes are not implemented.

## Technical reference

- [Owner resolution operations](../OWNER_RESOLUTION.md)
- [Discovery](DISCOVERY.md)
- [Troubleshooting](TROUBLESHOOTING.md)
