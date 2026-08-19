# Owner resolution

`CLMOwnerResolver 1.0.0.6` assigns the `Owner User` or `Owner Team` lookup on discovered credentials.

It runs daily at **03:00 AUS Eastern Standard Time**, after the 02:00 discovery flow.

## Assignment order

For each credential that is not decommissioned, not named `SYSTEM`, and not owner-locked:

1. Read `Owner Tag`.
2. If it looks like an email, find an active Dataverse user by email or domain name.
3. Assign that user.
4. If the tag cannot resolve, evaluate active Owner Rules in ascending priority order.
5. Apply the first matching rule.
6. Record the assignment or failure in Renewal Events.

Manual assignments with **Owner Locked = Yes** are never changed.

## Owner source

| Source | When used |
|---|---|
| `AADOwner` | An Entra app-registration owner email resolves to a Dataverse user |
| `Tag` | A resource Owner Tag resolves to a Dataverse user |
| `Rule` | A fallback Owner Rule assigns a user or team |
| `Manual` | An operator sets and locks the owner |

If a previously tag-derived owner becomes invalid, the resolver clears the stale owner before evaluating fallback rules.

## Owner Rules

Create rules in the **Credential Lifecycle** app under **Owner Rules**.

| Field | Use |
|---|---|
| Rule Name | Human-readable purpose |
| Priority | Lower number wins |
| Match Scope | Field to inspect |
| Match Pattern | Case-insensitive substring; empty means catch-all |
| Assign To User | User assigned when the rule matches |
| Assign To Team | Team assigned when no user is configured |
| Active | Whether the resolver evaluates the rule |

Supported scopes:

- Display Name
- Owner Tag
- Environment
- Key Vault Name
- Resource Group

If both user and team are accidentally configured, the user takes precedence.

## Recommended starter rules

Use rules only for stable, meaningful patterns:

| Priority | Scope | Pattern | Assignment |
|---:|---|---|---|
| 100 | Environment | `production` | Production platform team |
| 200 | Resource Group | `integration` | Integration operations team |
| 300 | Key Vault Name | `shared` | Shared services team |
| 9999 | Display Name | empty | Ownership triage team |

The final catch-all prevents credentials from disappearing into an unassigned queue, but it should point to a triage team rather than an individual.

## Hundreds of randomly named app registrations

Name-based rules are not reliable when application names are inconsistent. Use this precedence:

1. **Entra application owners** - preferred source.
2. **Immutable application mapping** - map by App ID or Object ID.
3. **Resource ownership tags** - suitable for Azure resources.
4. **Pattern rules** - only for stable naming or environment conventions.
5. **Triage team** - catch anything unresolved.

### Improve existing Entra ownership

For each app registration:

- Assign at least one accountable Entra owner.
- Prefer a monitored operational owner rather than the original developer.
- Remove departed or disabled owners.
- Review applications with no owners as an exception queue.

The discovery flow reads the first Entra application owner and places the email in Owner Tag. The resolver then maps it to an active Dataverse user.

### Bulk mapping process

For an existing estate:

1. Export App ID, Object ID, display name, current owners, and credential expiry.
2. Send the list to application or platform teams for ownership confirmation.
3. Capture the approved primary owner email or team.
4. Update Entra owners wherever possible.
5. Use a temporary triage rule for records still awaiting confirmation.
6. Re-run discovery and owner resolution.
7. Review unresolved ownership events.

Do not create hundreds of display-name rules. They are difficult to maintain and break when applications are renamed.

### Recommended future enhancement

Add a `clm_ownermapping` table keyed by immutable identifiers:

| Column | Purpose |
|---|---|
| Source System | Entra application, enterprise application, Key Vault, and so on |
| Object ID | Stable source object identifier |
| App ID | Stable Entra application identifier |
| Owner User | Primary Dataverse user |
| Owner Team | Primary Dataverse team |
| Effective Until | Optional review/expiry date |
| Last Validated On | Ownership attestation date |

The future resolver order should be:

1. Locked manual owner
2. Immutable-ID mapping
3. Entra owner or resource tag
4. Owner Rule
5. Triage team

This removes dependence on random or changing names.

## Resolver safeguards

- Processes Dataverse pagination for large estates.
- Uses sequential credential processing to prevent rule-statistic update races.
- Ignores disabled Dataverse users.
- Clears the opposite lookup when assigning a user or team.
- Updates rule match count and last-matched time.
- Records assignments and failures in Renewal Events.

## Current limitations

- Entra enterprise-application owners are not yet collected by the discovery flow.
- Match Pattern is substring matching, not regular expressions.
- Direct immutable-ID mappings require the proposed `clm_ownermapping` enhancement.
