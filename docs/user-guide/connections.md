# Connections

A connection is a saved profile: an engine, an address, a user, an environment and a few safety
switches. The password and any SSH secrets live in the operating system keychain, never in the
profile.

## Workflow at a glance

```mermaid
flowchart LR
  N["New connection"] --> E["Choose the engine"]
  E --> F["Fill in Connection · TLS · SSH tunnel"]
  F --> T{"Test connection"}
  T -- "fails" --> F
  T -- "Connected" --> S["Save"]
  S --> C["Connect from the side panel"]
  C --> B["Browse databases, schemas, tables"]
  B --> D["Choose the database and schema in use"]
```

## Creating a connection

### Choose the engine

| PostgreSQL | MySQL or MariaDB | SQLite |
|---|---|---|
| ![PostgreSQL](../assets/screenshots/002-new-connection-blank.png){ .shot loading=lazy } | ![MySQL](../assets/screenshots/003-new-connection-mysql.png){ .shot loading=lazy } | ![SQLite](../assets/screenshots/004-new-connection-sqlite.png){ .shot loading=lazy } |
| Port 5432. A connection is tied to one database at a time. | Port 3306. Databases are listed as schemas. | A local file. One database (`main`). |

### Connection settings

=== "Connection"

    ![The Connection tab, filled in](../assets/screenshots/005-connection-filled.png){ .shot loading=lazy }

    Name, environment, host and port, user, password, an optional default database, and the
    **Read-only** switch. The **URL** line previews the connection string without the password.

    **Environment** changes how careful the app is:

    | Environment | What changes |
    |---|---|
    | Development | Nothing extra. A green band on the window edge. |
    | Staging | An amber band and a *Staging* label in the title bar. |
    | Production | A red band and a *Production* label. Every change asks you to type the connection name first. |

=== "TLS"

    ![TLS defaults to Verify full](../assets/screenshots/010-tls-verify-full-default.png){ .shot loading=lazy }

    | Mode | Meaning |
    |---|---|
    | **Verify full** *(default)* | Encrypted. The certificate must chain to a trusted CA **and** match the host name. |
    | **Verify CA** | Encrypted. The certificate must chain to a trusted CA. |
    | **Require** | Encrypted, but the server's identity is not checked. A warning says so. |
    | **Disable** | No encryption. A warning says so. Use only for a local server. |

    A **CA certificate**, **client certificate** and **client key** can each be chosen as a file.

    | Require (warning) | Disable (warning) |
    |---|---|
    | ![Require warns that identity is not checked](../assets/screenshots/012-tls-require-warning.png){ .shot loading=lazy } | ![Disable warns that traffic is not encrypted](../assets/screenshots/013-tls-disable-warning.png){ .shot loading=lazy } |

    A successful test also reports the TLS state of the link:

    ![A MySQL test over TLS](../assets/screenshots/016-test-success-mysql-tls.png){ .shot loading=lazy }

=== "SSH tunnel"

    Tick **Connect through an SSH bastion** to reach a database that is only visible from a
    bastion host. DB Studio opens a local port forward and connects through it. A dot on the tab
    shows the tunnel is on.

    | Off | Password |
    |---|---|
    | ![SSH tunnel off](../assets/screenshots/017-ssh-off.png){ .shot loading=lazy } | ![SSH with a password](../assets/screenshots/018-ssh-password.png){ .shot loading=lazy } |

    | Private key | SSH agent |
    |---|---|
    | ![SSH with a private key file and passphrase](../assets/screenshots/019-ssh-private-key.png){ .shot loading=lazy } | ![SSH through the agent](../assets/screenshots/020-ssh-agent.png){ .shot loading=lazy } |

    The bastion's host key is pinned on first use. A changed key is refused until you trust it
    again.

### Test, then save

**Test connection** connects once without saving and reports the server version and the TLS state:

| PostgreSQL | SQLite |
|---|---|
| ![PostgreSQL connected](../assets/screenshots/014-test-success-postgres.png){ .shot loading=lazy } | ![SQLite connected](../assets/screenshots/023-sqlite-test-success.png){ .shot loading=lazy } |

When a test fails, the reason appears under the form. Every common message is explained in
[Troubleshooting](troubleshooting.md#connection-errors).

## The side panel

![Saved connections](../assets/screenshots/024-connections-saved.png){ .shot loading=lazy }

- **Connect / disconnect** with the plug icon that appears on hover, next to **edit** and
  **delete**.

    ![Row actions: connect, edit, delete](../assets/screenshots/025-connection-row-actions.png){ .shot loading=lazy }

- **Collapse and expand** each connection, or use the *Collapse all* / *Expand all* buttons at the
  top. The state is remembered. Collapsed, a connected entry shows the database in use next to
  its name.
- **A failed connection** shows its reason underneath, while the others carry on:

    | Expanded | Collapsed |
    |---|---|
    | ![Several connections, one failed](../assets/screenshots/026-connect-all-one-failed.png){ .shot loading=lazy } | ![Collapsed, with the failure still shown](../assets/screenshots/027-connections-collapsed.png){ .shot loading=lazy } |

- **Reorder** by dragging the grip, or with ++alt+up++ / ++alt+down++ on it.
- **New query** and **New database** buttons appear under a connected connection.

### Search tables

**Search tables** filters every connection at once, including collapsed ones. Schemas with no
match say *No match*.

![Searching for "product" across every connection](../assets/screenshots/036-search-tables.png){ .shot loading=lazy }

## Databases and schemas

=== "PostgreSQL"

    ![The shop database with its schemas and tables](../assets/screenshots/028-tree-postgres-shop.png){ .shot loading=lazy }

    The side panel lists every database on the server. Choosing another one opens a new session
    for it. Inside the database in use, each schema is listed. The schema in use is where
    unqualified names resolve first. Tables and views (eye icon) are listed under each schema,
    and the diagram icon on the right opens its [schema diagram](diagrams.md).

=== "MySQL / MariaDB"

    ![MySQL databases listed as schemas](../assets/screenshots/034-tree-mysql.png){ .shot loading=lazy }

    Databases are listed as schemas. Choosing one changes the default database for every pooled
    connection.

=== "SQLite"

    ![A SQLite file with its main schema](../assets/screenshots/035-tree-sqlite.png){ .shot loading=lazy }

    One file, one database (`main`).

### No default database

A connection saved without a database opens on the server's default (`postgres` here). Pick
another one from the list, and that one becomes the database in use.

| Opened without a database | After switching |
|---|---|
| ![Connected with postgres in use](../assets/screenshots/032-tree-no-default-database.png){ .shot loading=lazy } | ![Switched to another database](../assets/screenshots/033-tree-switched-database.png){ .shot loading=lazy } |

### Right-click menus

Left-click a database or schema to use it. **Right-click** (or press ++shift+f10++) for more:

=== "Connection"

    ![Refresh, Disconnect, New query, Edit](../assets/screenshots/029-menu-connection.png){ .shot loading=lazy }

=== "Database"

    ![Use as default, remember, new query here](../assets/screenshots/030-menu-database.png){ .shot loading=lazy }

=== "Schema"

    ![Schema menu with New table and Diagram](../assets/screenshots/031-menu-schema.png){ .shot loading=lazy }

| Item | What it does |
|---|---|
| **Refresh** | Reloads the list from the server. |
| **Disconnect** / **Edit…** | On a connection. |
| **Use as default** | Switches to it now. SQL tabs, table tabs and the assistant all work there. |
| **Use as default and remember** | Also saves it on the connection, so the next connect opens there. |
| **Forget remembered default** | Clears the saved choice. |
| **New query here** | Switches, then opens a SQL tab. |
| **New table…** / **Diagram** | On a schema. |

Switching is refused while a manual transaction has uncommitted work. Commit or roll back first
(see [Transactions](safety.md#transactions)).

## Creating databases and tables

=== "New database"

    ![Name, character set and Use it now](../assets/screenshots/037-dialog-new-database.png){ .shot loading=lazy }

    **New database** under a connection asks for a name and (on MySQL) a character set. Tick
    **Use it now** to switch to it once it exists.

=== "New table"

    ![Columns with type, Key and Can be empty](../assets/screenshots/038-dialog-new-table.png){ .shot loading=lazy }

    Right-click a schema → **New table…**. Add columns with a type, tick **Key** for the primary
    key and **Can be empty** for nullable columns, then **Review table**.

=== "Review"

    ![The CREATE TABLE statement in a review](../assets/screenshots/038b-new-table-review.png){ .shot loading=lazy }

    The exact `CREATE TABLE` goes through the normal [review](safety.md). On Production you type
    the connection name to run it.

## Editing and deleting a connection

Editing shows **Saved in the system keychain** instead of the password. **Replace** sets a new
one. The old password is never sent back to the window.

![Edit Shop: the password is Saved in the keychain](../assets/screenshots/039-edit-connection-saved-password.png){ .shot loading=lazy }

Deleting a connection removes it from the side panel at once:

![After deleting a connection](../assets/screenshots/040-connection-deleted.png){ .shot loading=lazy }
