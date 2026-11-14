# Your first connection

This walk-through takes you from an empty window to your first query result in five steps.

```mermaid
flowchart LR
  A["1 · Open DB Studio"] --> B["2 · Add a connection"]
  B --> C["3 · Test it"]
  C --> D["4 · Save and connect"]
  D --> E["5 · Run a query"]
```

## 1. Open DB Studio

On first launch there are no connections. The welcome page lists the four shortcuts worth
remembering: the command palette (++ctrl+k++), a new query (++ctrl+t++), the side panel
(++ctrl+b++) and the assistant (++ctrl+j++).

![The welcome page with no connections](../assets/screenshots/001-welcome-empty.png){ .shot loading=lazy }

## 2. Add a connection

Click **Add a connection** (or **+** at the top of the side panel, or *New connection* in the
command palette). First choose the engine. Each card shows its default port.

=== "PostgreSQL"

    ![A blank PostgreSQL connection](../assets/screenshots/002-new-connection-blank.png){ .shot loading=lazy }

=== "MySQL or MariaDB"

    ![A blank MySQL connection](../assets/screenshots/003-new-connection-mysql.png){ .shot loading=lazy }

=== "SQLite"

    ![A SQLite connection only needs a file](../assets/screenshots/004-new-connection-sqlite.png){ .shot loading=lazy }

    SQLite needs only a **database file** (choose it with **Choose**). There are no TLS or SSH tabs.

Then fill in the details:

![A filled-in PostgreSQL connection marked Production](../assets/screenshots/005-connection-filled.png){ .shot loading=lazy }

| Field | Notes |
|---|---|
| **Name** | What the side panel, title bar and reviews show. |
| **Environment** | *Development*, *Staging* or *Production*. Staging adds an amber band to the window edge. Production adds a red band and a *Production* label, and you must type the connection name before any change runs. |
| **Host / Port** | The server address. |
| **User / Password** | The password goes straight to the operating system keychain. It is never shown again. |
| **Database** | Optional. You can pick one after connecting. |
| **Safety** | *Read-only: refuse every change, even if approved.* |
| **URL** | A read-only preview of the connection URL. It never includes the password. |

The **TLS** tab defaults to *Verify full*, and the **SSH tunnel** tab connects through a bastion
host. Both are covered in [Connections](../user-guide/connections.md#connection-settings).

## 3. Test the connection

**Test connection** connects once, without saving. A green box shows the server version and
whether the link is encrypted:

![A successful test showing the PostgreSQL version](../assets/screenshots/014-test-success-postgres.png){ .shot loading=lazy }

If it fails, the server's own reason is shown under the form (for example a wrong password). See
[Troubleshooting](../user-guide/troubleshooting.md#connection-errors) for each common message.

![A failed test: wrong password](../assets/screenshots/006-test-fail-wrong-password.png){ .shot loading=lazy }

## 4. Save and connect

**Save** adds the connection to the side panel. Connections can be reordered by dragging their
grip. Hover over one to see its **connect**, **edit** and **delete** buttons.

| After two connections | A full list |
|---|---|
| ![Two saved connections](../assets/screenshots/022-after-two-connections.png){ .shot loading=lazy } | ![Nine saved connections](../assets/screenshots/024-connections-saved.png){ .shot loading=lazy } |

Click the plug icon to connect. The side panel lists the databases and schemas. The ones in use
are highlighted and marked *in use*.

![A connected PostgreSQL server with the shop database in use](../assets/screenshots/028-tree-postgres-shop.png){ .shot loading=lazy }

## 5. Run a query

Click **New query** under the connection. The toolbar shows where the tab runs (*in shop · schema
public*). The status bar shows the engine, the server version and the TLS state.

![An empty SQL tab on a Production connection](../assets/screenshots/041-sql-empty-tab.png){ .shot loading=lazy }

Type SQL and press ++ctrl+enter++. Rows stream into the grid, and the run log records the
statement:

![10,000 rows returned in 74 ms](../assets/screenshots/044-results-10k-rows.png){ .shot loading=lazy }

Reads run immediately. Anything that could change data opens a review first; see
[Reviews and safety](../user-guide/safety.md).

## Optional: set up the assistant

Open **Settings → Assistant**, choose a provider (Ollama, or any OpenAI-compatible endpoint such as
OpenAI or OpenRouter), enter the model name, save an API key and press **Test the model**. Then
open the **Assistant** from the left bar. See [AI assistant](../user-guide/assistant.md).
