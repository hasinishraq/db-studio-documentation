# IPC commands

Every command the webview can call, grouped by module (`src-tauri/src/commands/`). Each one
validates its input in Rust. Types are in `src/bindings.ts`.

## Profiles and secrets

| Command | Purpose |
|---|---|
| `list_profiles` | List saved connections (with `has_password` flags, never secrets) |
| `save_profile` | Save a connection; an optional password goes to the keychain |
| `delete_profile` | Delete a connection, its secrets and its masking rules |
| `remember_default` | Remember or forget the database and schema a connection opens on |
| `set_secret` / `clear_secret` | Write or remove a secret (write-only) |

## Sessions and schema

| Command | Purpose |
|---|---|
| `test_connection` | Connect, report the server, disconnect |
| `connect` / `disconnect` | Open or close a session |
| `list_databases` / `use_database` | Databases on the server; choose the default |
| `use_schema` | Choose the PostgreSQL schema in use |
| `list_schemas`, `list_objects`, `columns` | Browse metadata |
| `table_structure` | Columns, indexes, keys, constraints, DDL |
| `table_page`, `table_count` | Server-side paging with sort and filters |

## Queries and transactions

| Command | Purpose |
|---|---|
| `prepare_query` | Classify a script; returns one verdict per statement |
| `approve_query` | **Human click only.** Mint an approval token for exact SQL |
| `run_query` | Run through the editor's door, streaming `RunEvent`s |
| `cancel_run` | Cancel a running script |
| `explain_query` | The plan of one statement |
| `set_tx_mode`, `commit_tx`, `rollback_tx`, `tx_state` | Manual transactions |
| `history_list`, `history_clear` | History |
| `drafts_list`, `drafts_save`, `drafts_delete` | Saved queries |

## Editing

| Command | Purpose |
|---|---|
| `grid_edit_info` | What the grid may edit (primary key, locked columns) |
| `plan_row_changes` | Plan a whole grid save as reviewed SQL |
| `run_row_changes` | Run a grid save through the grid's door (all or nothing) |
| `plan_row_change` | Plan one change from the row form |
| `plan_create_table`, `plan_create_database` | Plan DDL for the review |

## Files

| Command | Purpose |
|---|---|
| `export_query`, `export_rows`, `export_table` | Export to CSV, JSON lines, Excel or SQL |
| `import_pick_file`, `import_plan`, `import_csv` | The CSV import wizard |
| `export_schema`, `open_schema_file`, `outline_schema` | Schema files |
| `export_diagram` | Render an SVG (diagram or chart) to PNG, JPEG or PDF |

## Assistant

| Command | Purpose |
|---|---|
| `ai_send` | Start a turn, streaming `AiEvent`s |
| `ai_review` | **Human click only.** Approve or decline a proposed statement |
| `ai_cancel`, `ai_cancel_query` | Stop a turn or one query |
| `ai_conversations`, `ai_turns`, `ai_rename`, `ai_forget` | Conversations |
| `ai_test_provider` | Send one short request to check the provider |
| `masking_list`, `masking_set` | Hidden columns per profile |

## Other

| Command | Purpose |
|---|---|
| `settings_get`, `settings_set` | Settings |
| `audit_list`, `audit_verify`, `audit_export` | The audit log |
| `trust_host_key` | Trust a new or changed SSH host key |
| `diagnostics` | Version, OS and recent log lines for support |
| `import_legacy` | Import profiles from the earlier Electron app |
