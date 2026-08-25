# CLM Adoption Guide

**Guide version:** 1.0

**Applies to:** CLMTables 1.3.1.0, clmPlatformOps 1.1.0.0, CLMDiscoveryFlow 1.1.0.2, CLMNotifications 1.3.0.0, CLMNotificationDispatchers 1.0.0.0, and CLMApp 1.1.0.0

Use this guide if you want to deploy Credential Lifecycle Manager (CLM), route credentials to accountable teams, send reminders, or operate CLM day to day.

## If you want to...

| Goal | Start here |
|---|---|
| Deploy CLM for the first time | [Install CLM](INSTALL.md) |
| Understand which identity needs which permission | [Identity and access](IDENTITY_AND_ACCESS.md) |
| Configure and test the custom connectors | [Install CLM: configure connectors](INSTALL.md#3-configure-and-test-the-connectors) |
| Discover Entra and Key Vault credentials | [Discovery](DISCOVERY.md) |
| Route credentials to the right team | [Owner routing](OWNER_ROUTING.md) |
| Configure 30-day and 7-day reminders | [Notifications](NOTIFICATIONS.md#use-only-30-day-and-7-day-reminders) |
| Send email or Teams notifications | [Notifications](NOTIFICATIONS.md) |
| Run the daily operating process | [Operations](OPERATIONS.md) |
| Fix a user-visible problem | [Troubleshooting](TROUBLESHOOTING.md) |

## 15-minute quick start

Use this after a platform administrator has completed the [clean installation](INSTALL.md).

1. Open the CLM model-driven app.
2. Create one active **Notification Group**, for example `CLM Triage`.
3. Select its reminder days. For a simple schedule, select **30** and **7**.
4. Create a monitored **Notification Receiver**, such as `clm-operations@contoso.com`.
5. Add the receiver to the group with an active **Notification Group Membership**.
6. Make `CLM Triage` the single default triage group.
7. Run `Discovery-CLMCredentials`.
8. Review **Credentials**, **Credential Owners**, and **Coverage Gaps**.
9. Run `Resolve-CLMNotificationGroups`; confirm credentials have a Notification Group.
10. Run `Queue-CLMCredentialNotifications`; inspect Notification Deliveries.
11. If dispatchers are installed and permitted by DLP, enable one dispatcher and verify a controlled delivery changes from **Pending** to **Sent**.

This establishes safe fallback routing first. Add [owner mappings and rules](OWNER_ROUTING.md) after discovery shows the identifiers and naming patterns in your environment.

## What CLM currently does

- Discovers App Registration secrets and certificates.
- Discovers Enterprise Application secrets and certificates.
- Discovers Azure Key Vault secret metadata, never secret values.
- Stores all owners returned for App Registration credentials.
- Records discovery failures as Coverage Gaps.
- Resolves one Notification Group through mappings, discovered owners, rules, then default triage.
- Queues auditable, deduplicated email and Teams deliveries.
- Optionally dispatches queued messages where Power Platform DLP permits it.

CLM does not currently populate owners for Enterprise Application credentials or create complete Discovery Run audit records.

## Living document

This is a living document and reflects the latest packages on `main`. Any behavior, package-version, or permission change must update the relevant page in this guide in the same pull request.

`docs/guide` is also the canonical source for the GitHub Wiki. Maintainers must refresh affected Wiki pages after guide changes; see [Publish the CLM guide to GitHub Wiki](WIKI_PUBLISHING.md).

## Advanced references

- [Solution architecture](../SOLUTION_ARCHITECTURE.md)
- [Installation reference](../INSTALL.md)
- [Custom connector reference](../CUSTOM_CONNECTORS.md)
- [RBAC and coverage](../RBAC_AND_COVERAGE.md)
- [Owner resolution internals](../OWNER_RESOLUTION.md)
- [Packaged discovery flow](../../flows/Discovery-CLMCredentials/README.md)
