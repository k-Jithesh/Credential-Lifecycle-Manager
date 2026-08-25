# Publish the CLM Guide to GitHub Wiki

## Canonical source

`docs/guide` is the canonical, version-controlled CLM adoption guide. The GitHub Wiki is a published copy for convenient browsing; do not make Wiki-only content changes.

When a pull request changes guide behavior, package versions, permissions, procedures, or links:

1. Merge the repository change first.
2. Refresh the corresponding Wiki page from `docs/guide`.
3. Adapt links using the rules below.
4. Refresh `Home.md` and `_Sidebar.md` when navigation or page names change.
5. Check every Wiki link after publishing.

## Recommended page mapping

| Canonical source | Wiki filename | Wiki page title |
|---|---|---|
| `docs/guide/README.md` | `Home.md` | CLM Adoption Guide |
| `docs/guide/INSTALL.md` | `Install-CLM.md` | Install CLM |
| `docs/guide/IDENTITY_AND_ACCESS.md` | `Identity-and-Access.md` | Identity and Access |
| `docs/guide/DISCOVERY.md` | `Discovery.md` | Discover Credentials |
| `docs/guide/OWNER_ROUTING.md` | `Owner-Routing.md` | Route Credentials to Owners |
| `docs/guide/NOTIFICATIONS.md` | `Notifications.md` | Configure Notifications |
| `docs/guide/OPERATIONS.md` | `Operations.md` | Operate CLM |
| `docs/guide/TROUBLESHOOTING.md` | `Troubleshooting.md` | Troubleshoot CLM |
| Navigation generated from the template below | `_Sidebar.md` | Sidebar |

Keep `WIKI_PUBLISHING.md` in the repository as the maintainer procedure; it does not need to be an end-user Wiki page.

## Adapt links

Internal guide links need only a predictable filename substitution:

| Source target | Wiki target |
|---|---|
| `README.md` | `Home` |
| `INSTALL.md` | `Install-CLM` |
| `IDENTITY_AND_ACCESS.md` | `Identity-and-Access` |
| `DISCOVERY.md` | `Discovery` |
| `OWNER_ROUTING.md` | `Owner-Routing` |
| `NOTIFICATIONS.md` | `Notifications` |
| `OPERATIONS.md` | `Operations` |
| `TROUBLESHOOTING.md` | `Troubleshooting` |

Preserve heading fragments when present. For example:

```text
INSTALL.md#3-configure-and-test-the-connectors
```

becomes:

```text
Install-CLM#3-configure-and-test-the-connectors
```

Links from the guide to files outside `docs/guide` must become repository links. Prefix the repository-relative path with:

```text
https://github.com/k-Jithesh/Credential-Lifecycle-Manager/blob/main/
```

For example, `../OWNER_RESOLUTION.md` becomes:

```text
https://github.com/k-Jithesh/Credential-Lifecycle-Manager/blob/main/docs/OWNER_RESOLUTION.md
```

Do not copy advanced reference files into the Wiki unless they are added to this mapping and maintained from a canonical repository source.

## `_Sidebar.md` template

```markdown
## CLM Adoption Guide

- [Home](Home)
- [Install CLM](Install-CLM)
- [Identity and Access](Identity-and-Access)
- [Discover Credentials](Discovery)
- [Route Credentials to Owners](Owner-Routing)
- [Configure Notifications](Notifications)
- [Operate CLM](Operations)
- [Troubleshoot CLM](Troubleshooting)

---

[Repository](https://github.com/k-Jithesh/Credential-Lifecycle-Manager)
```

## Publication check

- The guide version and package versions on `Home.md` match `docs/guide/README.md`.
- The changed source pages and their Wiki copies have the same substantive content.
- Internal links use Wiki page names rather than `.md` source filenames.
- Advanced references open the corresponding file on the repository's `main` branch.
- `_Sidebar.md` lists every end-user page once.
- No credentials, tenant-specific identifiers, or internal-only data were introduced during publication.
