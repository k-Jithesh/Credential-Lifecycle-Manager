# Troubleshoot CLM

## When to use this

Use this page when CLM's visible result is missing, unexpected, or stuck. Start with the symptom, then follow the linked advanced reference if the first checks do not resolve it.

## What you need

- Power Automate flow run history.
- Access to connector Test tabs and CLM Coverage Gaps.
- The identity inventory from [Identity and access](IDENTITY_AND_ACCESS.md).

## Symptoms and steps

### Connector sign-in fails

1. Verify tenant ID, application client ID, and client secret **value**.
2. Confirm the secret is current.
3. Confirm both connector redirect URLs are registered exactly as **Web** redirect URIs.
4. Recreate or reauthorize the connection as the dedicated discovery service account.

`AADSTS50011` means a redirect URI is missing or mistyped.

### Graph returns 401 or 403

1. For 401, verify connector OAuth values and connection sign-in.
2. For 403, verify delegated `Application.Read.All` and `Directory.Read.All`.
3. Verify tenant admin consent shows Granted.
4. Test `GetOrganization`, `ListApplications`, and `ListServicePrincipals`.

See [custom connector setup](../CUSTOM_CONNECTORS.md).

### Azure sign-in succeeds but no subscriptions appear

1. Confirm the connection owner is the intended service account.
2. Assign that account Azure `Reader` at the approved scope.
3. Do not assign RBAC only to the app registration service principal.
4. Retest `ListSubscriptions`.

See [RBAC and coverage](../RBAC_AND_COVERAGE.md).

### A Key Vault or its secrets are missing

1. Confirm its subscription appears in `ListSubscriptions`.
2. Confirm the discovery account has `Reader` at an applicable scope.
3. Review Coverage Gaps for permission or network failures.
4. Check firewall, private endpoint, Conditional Access, and tenant restrictions.
5. Run discovery again.

CLM reads metadata only; secret-value roles are not required.

### Enterprise Application credentials have no owner

This is current behavior. Enterprise Application credentials do not populate Credential Owner rows. Use an app-wide/individual [Owner Mapping or Owner Rule](OWNER_ROUTING.md).

### An App Registration mapping does not match

1. For app-wide routing, use the Application (client) ID in Owner Mapping App ID.
2. Remember Credential Object ID currently contains that Application (client) ID.
3. For one credential, copy Credential External ID to Mapping Key.
4. Clear the existing Notification Group and rerun resolution.

### A credential has no Notification Group

1. Check active Owner Mappings.
2. Check App Registration owners, matching Receiver email/UPN, and the designated owner-resolution Membership.
3. Check active rules by ascending Priority.
4. Confirm one active group is designated default triage.
5. Clear stale assignment only when deliberate, then run resolution.

See [owner routing](OWNER_ROUTING.md).

### A rule routes too many or too few credentials

1. Confirm the correct Match Scope.
2. Use a specific case-insensitive substring.
3. Never leave Match Pattern blank.
4. Give active rules unique priorities.
5. Test a positive and a near-match negative case before broad use.

### No Notification Deliveries appear

1. Confirm the credential is not Decommissioned.
2. Confirm Suppressed Until is not in the future.
3. Confirm its group has the intended Reminder Days.
4. Confirm the current days-to-expiry enters a selected threshold.
5. Confirm active memberships and receivers.
6. Review queue flow history.

Blank Reminder Days enables all five thresholds; it does not disable reminders.

### Deliveries remain Pending

1. Confirm the channel's dispatcher is installed and enabled.
2. Validate its Dataverse and provider connections.
3. Confirm DLP permits Dataverse with Outlook or Teams.
4. Confirm the dispatcher is in the same CLM data environment.

### A delivery is Failed

1. Read Error Detail and provider history.
2. Correct the receiver or connection.
3. Change the delivery to Retrying only after checking whether the provider already accepted it.

At-least-once dispatch means a retry can duplicate an accepted message.

### A renewed credential still sends notices

1. Run discovery and confirm the new expiry or replacement record.
2. Mark the replaced record Decommissioned if rotation created a new record.
3. Change its existing Pending or Retrying deliveries to Skipped.
4. Clear obsolete suppression/tracking values only on the active replacement.

### Discovery Run records are absent

This is a current implementation limitation. The package initializes a DiscoveryRunId but does not create or complete Discovery Run rows. Use flow history, Coverage Gaps, and Renewal Events.

## Expected result

The visible symptom is either corrected or tied to a documented current limitation without granting unnecessary privilege.

## Common problems

- Testing with a personal connection instead of the deployed service-account connection.
- Granting Azure RBAC to the connector app rather than the delegated connection owner.
- Assuming blank Reminder Days means disabled.
- Editing a rule without clearing sticky assignments.
- Enabling a dispatcher before DLP and connection ownership are approved.

## Technical reference

- [Installation reference](../INSTALL.md)
- [Custom connectors](../CUSTOM_CONNECTORS.md)
- [RBAC and coverage](../RBAC_AND_COVERAGE.md)
- [Owner resolution](../OWNER_RESOLUTION.md)
- [Discovery flow](../../flows/Discovery-CLMCredentials/README.md)
