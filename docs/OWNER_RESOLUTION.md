# Owner resolution

## Current behaviour

`CLMDiscoveryFlow 1.0.0.26` captures an owner hint in `clm_credential.clm_ownertag`.

| Credential source | Owner Tag value |
|---|---|
| Entra application secret or certificate | First application owner's UPN or email |
| Key Vault secret | First available vault tag: `Owner`, `owner`, or `OwnerEmail` |
| Enterprise application certificate | Not currently populated |

The flow does **not** populate:

- `Owner User` (`clm_owneruser`)
- `Owner Team` (`clm_ownerteam`)
- `Owner Source` (`clm_ownersource`)

The four release solutions do not include an owner-resolver flow. The `Owner Rules` table is present for future automation, but the current discovery flow does not evaluate it.

## What a new installer should do

After the first discovery run:

1. Open the **Credential Lifecycle** app.
2. Open **Credentials**.
3. Review the **Owner Tag** value.
4. Set **Owner User** or **Owner Team**.
5. Set **Owner Source** to `Manual`.
6. Set **Owner Locked** to `Yes`.
7. Save.

`Owner Locked` records the intent that future owner automation must not replace the manual assignment. The current discovery flow only refreshes the Owner Tag and does not overwrite the Owner User or Owner Team lookups.

## Owner Source values

| Value | Meaning |
|---|---|
| `Tag` | Derived from an Azure resource tag |
| `AADOwner` | Derived from the first Entra application owner |
| `Rule` | Derived from an Owner Rule |
| `Manual` | Set by an operator |

## Recommended owner data

For better discovery results:

- Assign at least one owner to every Entra app registration.
- Add an `Owner` or `OwnerEmail` tag to each Key Vault.
- Use a monitored team mailbox or distribution address when ownership belongs to a team.

## Automation gap

Automated owner resolution requires a future solution-aware flow that:

1. Finds credentials with an Owner Tag and no locked owner.
2. Matches the tag to an enabled Dataverse user.
3. Falls back to active `clm_ownerrule` records.
4. Sets Owner User or Owner Team.
5. Sets Owner Source to `AADOwner`, `Tag`, or `Rule`.
6. Marks unresolved credentials as orphaned.
7. Never replaces a record where Owner Locked is `Yes`.

Until that flow is added and tested, documentation and demos must describe owner resolution as manual.
