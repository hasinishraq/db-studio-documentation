# Sessions and transactions

`studio-session` holds every open connection and is the only place SQL meets a database.

## Sessions

`SessionManager::connect(profile, password, ssh_secret)` opens a driver session (through an SSH
tunnel when configured) and registers an `OpenSession`:

| Field | Meaning |
|---|---|
| `profile` | The connection profile it was opened from |
| `session` | The driver's `Session` (a pool, or a single SQLite handle) |
| `policy` | Read-only or not, fixed at connect time |
| `tx` | `TxState { mode: Auto \| Manual, dirty }` |
| `database`, `schema` | Where unqualified names resolve |

**Choosing a database.** MySQL changes the default database in place, pointing each pooled
connection at it when taken. PostgreSQL ties a connection to one database, so `use_database` opens
a new session first and closes the old one only once the new one works. A remembered default
schema is applied on connect if it still exists, and is never carried to another database.

## Connections and scopes

`Session::acquire(scope)` hands out a `Conn`, and the `Conn` owns its canceller:

- `ConnScope::Current`: the connection a manual transaction lives on, or any pooled connection in
  auto mode.
- `ConnScope::Isolated`: a separate connection, so the assistant's reads never land inside your
  manual transaction.

## The doors

| Door | Token | Statements | Wrapping |
|---|---|---|---|
| `run_user` | Editor (for non-reads) | Any; stops at the first error | none; your commit mode applies |
| `run_changes` | Grid | INSERT / UPDATE / DELETE only, ≤ 1,000, each exactly 1 row | `BEGIN … COMMIT`, or `SAVEPOINT` in manual mode |
| `run_import` | Import | One INSERT per batch | one transaction for the whole file |
| `run_ai_read` | none accepted | `ai_auto` only | READ ONLY transaction, 30 s, 5,000 rows |
| `run_ai_approved` | Ai | exactly one | 60 s, 5,000 rows |

Every door checks the profile's policy before anything runs: a read-only profile refuses every
non-read statement, even with a valid token.

## Manual transactions

`set_tx_mode(Manual)` begins a transaction on the session's current connection. Statements then
run inside it and mark it *dirty*; **Commit** and **Roll back** finish it and open the next one.
While a manual transaction is open:

- typed `COMMIT` / `ROLLBACK` are refused (the app tracks the state),
- changing database or schema is refused while it is dirty,
- grid saves use a savepoint, so a failed save rolls back only itself.

!!! note "MySQL read-only transactions"
    On MySQL and MariaDB, DDL commits implicitly and would escape a plain
    `START TRANSACTION READ ONLY`. The driver therefore also sets
    `SET SESSION TRANSACTION READ ONLY` for read-only work; a test proves DDL cannot escape.

## Cancel

Each run is registered with a cancellation token. `cancel(run_id)` cancels the token and calls
the connection's canceller, which cancels on the server. A cancelled run is never reported as a
partial success.

## Audit

Every non-read statement is recorded through an `AuditSink` with its origin, kind, SQL, SHA-256,
rows and outcome. The session depends only on the trait; the app wires it to `studio-store`.
