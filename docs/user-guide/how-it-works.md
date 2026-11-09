# How everything works

A guided tour of DB Studio, one task at a time. Each step shows what you see and then, under
**Behind the scenes**, what the app does for you in Rust. For the full design, see the
[architecture overview](../architecture/index.md).

```mermaid
flowchart LR
  A["1 · Connect"] --> B["2 · Browse"]
  B --> C["3 · Query"]
  C --> D["4 · Review a change"]
  D --> E["5 · Edit tables"]
  E --> F["6 · Import / export"]
  F --> G["7 · Diagram"]
  G --> H["8 · Ask the assistant"]
  H --> I["9 · Audit"]
```

## The window

![The main window](../assets/screenshots/01-explorer-connections.png){ .shot }

| Area | What it holds |
|---|---|
| **Left bar** | Explorer, History, Saved queries, Assistant, Audit log, Settings |
| **Side panel** | Your connections, their databases, schemas and tables |
| **Work area** | Tabs: SQL queries, tables, diagrams, the assistant, the audit log, settings |
| **Title bar** | The connection and database of the active tab; a red band and *Production* label on production connections, amber on staging |

!!! abstract "Behind the scenes"
    The window is a Tauri webview showing a React app. The app cannot reach the network, the
    disk or a database by itself: its content security policy allows only IPC to Rust. Every
    button you press calls a typed command in `src/bindings.ts`, and Rust validates it again.

---

## 1. Connect to a database

Add a connection with **+** in the side panel. Pick the engine, host, port, user and password,
the **environment** (Development, Staging, Production) and whether it is **read-only**.

=== "General"

    ![Connection: General](../assets/screenshots/04-connection-general.png){ .shot }

=== "TLS"

    ![Connection: TLS](../assets/screenshots/05-connection-tls.png){ .shot }

=== "SSH"

    ![Connection: SSH](../assets/screenshots/06-connection-ssh.png){ .shot }

If the server's certificate cannot be trusted, the connection is refused and the reason is shown:

![A TLS verification failure](../assets/screenshots/47-error-tls-verify.png){ .shot }

!!! abstract "Behind the scenes"
    1. `save_profile` stores the profile in the app's SQLite file **without** the password.
       `set_secret` sends the password once to the OS keychain; it is never sent back.
    2. `connect` reads the password from the keychain, opens an **SSH tunnel** if configured
       (`studio-tunnel`, pinning the bastion's host key), then asks the engine's **driver** to
       connect with **rustls** (certificate and host name verified by default).
    3. `studio-session` registers an open session with a **policy** fixed at connect time
       (read-only or not, environment) and returns a session id. That id is all the webview holds.

    See [Networking, TLS and SSH](../architecture/networking.md) and
    [Storage and secrets](../architecture/storage.md).

## 2. Browse databases, schemas and tables

![MySQL and MariaDB databases](../assets/screenshots/03-explorer-mysql-mariadb.png){ .shot }

Click a database or schema to use it. Right-click it to make it the default, or to remember it for
the next connect:

![Right-click a schema](../assets/screenshots/02-right-click-default-schema.png){ .shot }

!!! abstract "Behind the scenes"
    `list_databases`, `list_schemas` and `list_objects` ask the driver's metadata queries.
    `use_database` on **MySQL** changes the default database for every pooled connection; on
    **PostgreSQL**, where a connection belongs to one database, it opens a **new session** and
    closes the old one only after the new one works. Switching is refused while a manual
    transaction has uncommitted work. The frontend caches these lists with TanStack Query, keyed
    by connection, so a change refreshes exactly what it touched.

## 3. Write and run a query

Open **New query**, type SQL and press ++ctrl+enter++. Completion knows the tables and columns
of the database in use.

![Completion in the editor](../assets/screenshots/07-editor-completion.png){ .shot }

Rows stream in as they arrive; the grid stays responsive even for very large results.

![Rows streaming into the grid](../assets/screenshots/08-results-10k-rows.png){ .shot }

Every statement gets a line in the run log, and **Explain** draws the plan as a tree:

| Run log | Explain |
|---|---|
| ![Run log](../assets/screenshots/11-results-and-run-log.png){ .shot } | ![Explain plan](../assets/screenshots/10-explain-plan.png){ .shot } |

Errors are reported as the server gives them, with the position underlined when the server
reports one:

| Unknown column | Duplicate key |
|---|---|
| ![Unknown column](../assets/screenshots/42-error-unknown-column.png){ .shot } | ![Duplicate key](../assets/screenshots/43-error-unique.png){ .shot } |

!!! abstract "Behind the scenes"
    ```mermaid
    sequenceDiagram
      participant UI as SQL tab
      participant R as Rust
      participant DB as Database
      UI->>R: prepare_query(sql)
      R-->>UI: one verdict per statement (all reads)
      UI->>R: run_query(sql, no token, channel)
      R->>R: classify again, check policy
      R->>DB: execute statement 1
      DB-->>R: columns, then rows
      R-->>UI: Columns · Rows(batch of 500) · … · StatementDone
      UI->>UI: ResultStore appends#59; the grid draws visible rows only
    ```

    `studio-safety` classifies the script **twice**: once for the interface and again inside the
    session, which never trusts what the interface was told. Reads run immediately. Rows travel in
    batches over a Tauri `Channel`; the `ResultStore` keeps them in the chunks they arrived in,
    and the virtual grid renders about 40 rows at a time. **Stop** cancels on the server
    (PostgreSQL cancel request, MySQL `KILL QUERY`). See
    [Life of a query](../architecture/query-lifecycle.md).

## 4. Review a change before it runs

Anything that is not a pure read opens a **review**: the exact SQL, its kind, and the connection
and environment it targets.

=== "A change"

    ![Review an UPDATE](../assets/screenshots/14-review-change.png){ .shot }

=== "Cannot be undone"

    ![Review a DROP](../assets/screenshots/15-review-danger.png){ .shot }

    Locked until you tick *I understand this cannot be undone*.

=== "Does not parse"

    ![Review unparsable SQL](../assets/screenshots/16-review-unparsed.png){ .shot }

    Unknown SQL is treated as a write.

=== "Production"

    ![Production review](../assets/screenshots/17-review-production.png){ .shot }

    You type the connection's name to unlock **Run**.

On a read-only connection the change is refused even after you approve it:

![Read-only refusal](../assets/screenshots/18-read-only-refused.png){ .shot }

!!! abstract "Behind the scenes"
    ```mermaid
    sequenceDiagram
      actor You
      participant UI as Review dialog
      participant Safety as studio-safety
      participant Session as studio-session
      UI->>Safety: prepare_query → verdict: Dml, risk reasons
      UI->>You: show exact SQL
      You->>UI: Run on "shop"
      UI->>Safety: approve_query(sql, Editor)
      Safety-->>UI: token (random, 60 s, single use)<br/>bound to sha256(sql) + session
      UI->>Session: run_query(sql, token)
      Session->>Session: policy: read-only? → refuse
      Session->>Safety: consume(token) — removed first, then compared
      Session->>Session: run, then write the audit entry
    ```

    Only a human click can mint a token: `approve_query` (editor, grid, import) and `ai_review`
    (assistant). Changing a single character of the SQL after approval makes the hash differ, and
    the run is refused. See the [safety engine](../architecture/safety-engine.md).

### Transactions

**Auto commit** commits each statement. **Manual** keeps everything in one transaction until you
press **Commit** or **Roll back**:

| Uncommitted changes | Switching is refused |
|---|---|
| ![Manual transaction](../assets/screenshots/19-manual-transaction.png){ .shot } | ![Commit first](../assets/screenshots/20-commit-first.png){ .shot } |

!!! abstract "Behind the scenes"
    `set_tx_mode(Manual)` begins a transaction on the session's current connection and keeps that
    connection for your tab. Each successful write marks it **dirty**. Typed `COMMIT` and
    `ROLLBACK` are refused so the app's idea of the transaction cannot drift from the server's.

## 5. Browse and edit a table

Click a table to page through it on the server, 100 rows at a time, with filters and sorting:

![Filter and sort](../assets/screenshots/21-table-filter-sort.png){ .shot }

Edit cells, insert rows and mark rows for deletion. Nothing is saved yet:

![Pending edits](../assets/screenshots/22-inline-edit-pending.png){ .shot }

**Review and save** turns everything into SQL and shows it in one review; it runs all or nothing:

| One review | Saved |
|---|---|
| ![Review](../assets/screenshots/23-inline-edit-review.png){ .shot } | ![Saved](../assets/screenshots/24-inline-edit-saved.png){ .shot } |

If someone changed a row since you loaded it, or the database refuses a statement, nothing is
saved and your edits stay pending:

| Row changed meanwhile | Refused by a foreign key |
|---|---|
| ![Row changed](../assets/screenshots/25-inline-edit-refused.png){ .shot } | ![Foreign key](../assets/screenshots/44-error-foreign-key.png){ .shot } |

!!! abstract "Behind the scenes"
    The webview sends **values only** (`ChangeSet`). Rust's `grid_edit_info` decides which columns
    are editable (a primary key is required; binary and masked columns are locked), and
    `plan_row_changes` writes the SQL: deletes, then updates, then inserts, one row per
    statement, each `WHERE` naming the primary key and, where the type compares exactly, the
    value you loaded. `run_row_changes` uses the grid's own door: it accepts only grid tokens,
    only INSERT/UPDATE/DELETE, at most 1,000 statements, each changing **exactly one row**, in
    one transaction (a savepoint in manual mode). See
    [Data flows](../architecture/data-flows.md#inline-edits).

## 6. Import and export

**Import CSV** maps file columns to table columns, then imports in batches:

| Map the columns | Import finished |
|---|---|
| ![Import mapping](../assets/screenshots/29-import-mapping.png){ .shot } | ![Import done](../assets/screenshots/30-import-done.png){ .shot } |

**Export** writes CSV, JSON lines, Excel or SQL inserts from a result, a selection or a whole table.

!!! abstract "Behind the scenes"
    The **file dialogs open in Rust**, so the webview never names a path: it gets an opaque file id
    and a preview. The import review text includes the file's SHA-256, so editing the file after
    approval makes the token fail. All batches run in **one transaction**. Exports stream from the
    database straight to disk (Excel in constant-memory mode), and only reads may be exported. See
    [Data flows](../architecture/data-flows.md#csv-import).

## 7. See the schema as a diagram

![Schema diagram](../assets/screenshots/31-diagram.png){ .shot }

Pan, zoom, drag tables and **Export image** as PNG, JPEG or PDF:

| Export | The saved PNG |
|---|---|
| ![Export dialog](../assets/screenshots/33-diagram-export.png){ .shot } | ![Exported PNG](../assets/screenshots/34-diagram-export-output.png){ .shot } |

!!! abstract "Behind the scenes"
    The diagram is laid out and drawn as SVG in the webview from `table_structure` metadata. To
    save, the webview serialises a standalone light-theme SVG; Rust **size-checks** it, **refuses
    any reference to a file or URL**, and renders it with `resvg` (PNG/JPEG at 2×) or `svg2pdf`
    using fonts bundled in the app, so it looks the same on every machine.

## 8. Ask the assistant

Ask in plain language. The assistant sees only the database in use:

![Scoped to the database in use](../assets/screenshots/38-assistant-scoped.png){ .shot }

Safe reads run on their own and can be drawn as charts from the real rows:

| Pie | Bar | Line |
|---|---|---|
| ![Pie](../assets/screenshots/35-assistant-pie.png){ .shot } | ![Bar](../assets/screenshots/36-assistant-bar.png){ .shot } | ![Line](../assets/screenshots/37-assistant-line.png){ .shot } |

Anything else waits for your approval:

![A proposed change](../assets/screenshots/39-assistant-write-review.png){ .shot }

!!! abstract "Behind the scenes"
    ```mermaid
    flowchart TB
      Q["Your question"] --> P["Policy check<br/>(off · local only · no row sharing)"]
      P --> S["System prompt: server, database, schema,<br/>table structure — no rows"]
      S --> M["Model"]
      M -->|"run_sql(sql, chart?)"| L{"Limits<br/>≤ 8 steps · ≤ 6 queries"}
      L --> C{"studio-safety:<br/>ai_auto?"}
      C -->|"yes, and auto-run on"| R["run_ai_read<br/>READ ONLY tx · 30 s · 5,000 rows"]
      C -->|"no"| V["Review card → your click<br/>→ ai_review mints token"]
      V --> A["run_ai_approved · 60 s"]
      R & A --> O["Rows → your screen and chart"]
      O --> B["Back to the model: column names and counts<br/>(a masked sample only if you allowed sharing)"]
      B --> M
    ```

    The model can only call four tools (`run_sql`, `describe_table`, `explain_query`,
    `list_tables`), and every statement goes through the same safety engine as yours. The model
    never receives a token and has no path to a raw connection. Charts name **columns only**; the
    numbers always come from the rows the database returned. See
    [AI assistant](../architecture/ai.md).

## 9. Check the audit log

Every non-read statement, from any source, is recorded and hash-chained:

![Audit log](../assets/screenshots/52-audit-log.png){ .shot }

!!! abstract "Behind the scenes"
    The session writes an `AuditEntry` (time, connection, environment, origin, kind, SQL, its
    SHA-256, rows, outcome) after each non-read. Each entry stores
    `sha256(prev_hash ‖ entry)`, so **Verify integrity** detects any changed or deleted entry.
    Row values and parameters are never logged.

## 10. Make it yours

| General | Results and queries |
|---|---|
| ![General settings](../assets/screenshots/48-settings-general.png){ .shot } | ![Query settings](../assets/screenshots/49-settings-queries.png){ .shot } |

| Assistant | Per connection |
|---|---|
| ![Assistant settings](../assets/screenshots/50-settings-assistant.png){ .shot } | ![Masking](../assets/screenshots/51-settings-masking.png){ .shot } |

Settings are stored as one JSON value per key in the app database; the API key goes to the
keychain. See the [settings reference](../reference/settings.md).
