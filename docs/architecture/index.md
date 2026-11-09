# Architecture overview

DB Studio is a [Tauri 2](https://tauri.app/) application. A Rust backend, split into a Cargo
workspace of eleven crates, owns every connection, secret and decision about safety. A React +
TypeScript interface runs in the operating system's webview and talks to Rust only through
generated, typed commands.

```mermaid
flowchart TB
  UI["<b>Webview</b> · untrusted<br/>React + TypeScript, calls only src/bindings.ts"]
  APP["<b>studio-app</b> · src-tauri<br/>commands validate every input"]
  AI["<b>studio-ai</b><br/>agent loop, tools, memory"]
  STORE["<b>studio-store</b><br/>app SQLite, keychain, audit log"]
  SESSION["<b>studio-session</b><br/>the only crate that runs SQL"]
  SAFETY["<b>studio-safety</b><br/>classifier + approvals, no I/O"]
  DRIVERS["<b>driver-postgres · driver-mysql · driver-sqlite</b><br/>studio-db traits; SSH via studio-tunnel"]
  DBS[("PostgreSQL · MySQL / MariaDB · SQLite")]
  LLM["AI provider<br/>OpenAI-compatible or Ollama"]
  KC["OS keychain"]

  UI -- "typed IPC (tauri-specta)" --> APP
  APP --> AI
  APP --> STORE
  APP --> SESSION
  AI --> SESSION
  SESSION -- "every statement" --> SAFETY
  SESSION --> DRIVERS
  DRIVERS -- "verified TLS" --> DBS
  AI -. "HTTPS" .-> LLM
  STORE --> KC
```

## Layers and the trust boundary

```mermaid
flowchart TB
  subgraph U["Untrusted"]
    L1["<b>Presentation</b> · src/components<br/>Shell, Explorer, SqlTab, TableTab, ErDiagram, AiTab, SettingsTab"]
    L2["<b>Client logic</b> · src/api, src/lib, src/context<br/>flows, ResultStore, pending edits, chart and diagram layout"]
  end
  subgraph T["Trusted (Rust)"]
    L3["<b>Commands</b> · src-tauri/src/commands<br/>validate input, map errors, open file dialogs"]
    L4["<b>Domain</b> · studio-session, studio-safety, studio-ai<br/>what may run, and how"]
    L5["<b>Infrastructure</b> · driver-*, studio-tunnel, studio-store<br/>wire protocols, TLS, SSH, SQLite, keychain"]
  end
  L1 --> L2
  L2 == "IPC: the trust boundary" ==> L3
  L3 --> L4 --> L5
```

Everything above the boundary may be wrong or hostile and still cannot change data: the commands
below it re-check every input, and only the domain layer decides what runs.

## From the screen to the code

| What you see | Frontend | Commands | Crates |
|---|---|---|---|
| ![Side panel](../assets/screenshots/01-explorer-connections.png){ .shot width="260" }<br/>Side panel | `shell/Explorer.tsx` | `list_profiles`, `connect`, `list_databases`, `list_objects` | `studio-store`, `studio-session`, `driver-*` |
| ![SQL tab](../assets/screenshots/08-results-10k-rows.png){ .shot width="260" }<br/>SQL tab | `workbench/SqlTab.tsx`, `ResultsGrid.tsx`, `api/run.ts` | `prepare_query`, `run_query`, `cancel_run`, `explain_query` | `studio-safety`, `studio-session` |
| ![Review](../assets/screenshots/14-review-change.png){ .shot width="260" }<br/>Review | `modals/ConfirmDialog.tsx` | `approve_query` | `studio-safety` (`ApprovalBook`) |
| ![Table grid](../assets/screenshots/22-inline-edit-pending.png){ .shot width="260" }<br/>Table grid | `workbench/TableTab.tsx`, `EditableGrid.tsx`, `lib/pendingEdits.ts` | `table_page`, `grid_edit_info`, `plan_row_changes`, `run_row_changes` | `studio-session` (`run_changes`) |
| ![Import](../assets/screenshots/29-import-mapping.png){ .shot width="260" }<br/>Import | `modals/ImportWizard.tsx` | `import_pick_file`, `import_plan`, `import_csv` | `studio-session` (`run_import`) |
| ![Diagram](../assets/screenshots/31-diagram.png){ .shot width="260" }<br/>Diagram | `workbench/ErDiagram.tsx`, `lib/diagram.ts` | `table_structure`, `export_diagram`, `export_schema` | `studio-app` (resvg, svg2pdf) |
| ![Assistant](../assets/screenshots/35-assistant-pie.png){ .shot width="260" }<br/>Assistant | `ai/AiTab.tsx`, `ReviewCard.tsx`, `ResultChart.tsx` | `ai_send`, `ai_review`, `ai_cancel` | `studio-ai` |
| ![Audit log](../assets/screenshots/52-audit-log.png){ .shot width="260" }<br/>Audit log | `workbench/AuditTab.tsx` | `audit_list`, `audit_verify`, `audit_export` | `studio-store` |
| ![Settings](../assets/screenshots/48-settings-general.png){ .shot width="260" }<br/>Settings | `workbench/SettingsTab.tsx` | `settings_get`, `settings_set`, `set_secret`, `masking_set` | `studio-store` |

## Principles

**The webview is untrusted.** It renders and asks; it never decides. Every command re-validates
its input in Rust. The webview never receives a secret, never decides whether SQL is safe, and
never supplies a file path for a write.

**One door to the database.** `studio-session` is the only crate that executes SQL, and every path
into it goes through `studio-safety` first. The assistant reaches databases only through the
session, so there is no route from the model to a raw connection.

**Fail closed.** The classifier is an allow-list: only statements it positively recognises as reads
are reads. Anything else, including SQL that does not parse, needs an approval.

**Approvals come from people.** A write needs a random, single-use, 60-second token bound to the
SHA-256 of the exact SQL and to the session. Only commands triggered by a human click
(`approve_query`, `ai_review`) can mint one.

**Small crates with one job.** Drivers know nothing of Tauri, AI or storage, so each can be tested
alone against a real server. The safety crate performs no I/O, so it can be tested exhaustively.

## Read on

| Page | Covers |
|---|---|
| [Technologies and languages](tech-stack.md) | Languages, frameworks, libraries and versions, and why each was chosen |
| [Workspace and crates](workspace.md) | Each crate's job, dependencies and public surface |
| [Startup, state and threads](runtime.md) | Process model, startup sequence, `AppState`, threads, errors, events, logging |
| [Life of a query](query-lifecycle.md) | From the Run button to the result grid |
| [Data flows](data-flows.md) | Inline edits, CSV import, export, diagrams, schema files, settings |
| [Safety engine](safety-engine.md) | Classification, risk, the AI gate, approval tokens |
| [Sessions and transactions](sessions.md) | Sessions, runs, the four doors, cancel, manual mode |
| [Database drivers](drivers.md) | The `Driver`/`Session`/`Conn` traits and each engine |
| [AI assistant](ai.md) | Agent loop, tools, charts, scope, privacy, memory |
| [Storage and secrets](storage.md) | App database, migrations, keychain, audit chain |
| [Networking, TLS and SSH](networking.md) | Transport security and tunnels |
| [Frontend](frontend.md) | React structure, state, the grid, charts |
| [IPC and bindings](ipc.md) | How commands are defined, typed and generated |
