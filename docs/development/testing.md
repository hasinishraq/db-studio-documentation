# Testing

Every change ships with its tests, and the full gate runs before a change is called done.

## The gate

```bash
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace          # with the test servers up for full coverage
cargo deny check
npm run typecheck && npm test
```

## Layers

| Layer | Where | What |
|---|---|---|
| **Unit** | each crate's `src/` | Pure logic: classifier tables, planners, chart specs, rendering |
| **Snapshot** | `insta` | SQL planned per dialect (row edits, CREATE TABLE, grid saves) |
| **Fake driver** | `studio-session` `testing` feature | Policy and doors without a database: tokens, origins, read-only, manual mode |
| **Conformance** | `studio-db` kit, run by each driver | Streaming, cancel, timeouts, paging, metadata against real servers |
| **Real server** | `studio-session/tests/*`, driver tests | The safety suite and grid saves on PostgreSQL, MySQL, MariaDB and SQLite |
| **Agent** | `studio-ai/tests/agent.rs` | A scripted model proves the gates: no skipping review, no rows leaked, scope |
| **Frontend** | `src/test/*` (Vitest) | Flows, pending edits, charts, menus, the assistant tab |
| **End to end** | `tests/e2e/*.mjs` | The real app through WebView2's debug port, with real file dialogs |

## Highlights

- **Safety suite**: every statement the assistant may run unattended is executed inside a READ
  ONLY transaction against each real engine, and a sentinel table proves nothing changed.
- **Tamper tests**: changing or deleting an audit entry is detected.
- **Rule 6 scan**: a test reads the source to prove only `approve_query` and `ai_review` mint
  tokens.
- **Hostile input**: long chains, executable comments, quote tricks and NUL bytes all fail closed.

## End-to-end tests

Build with embedded assets, then start the app with a debug port:

```bash
npm run build
cargo build -p studio-app --features tauri/custom-protocol
# start target/debug/studio-app.exe with
#   WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS=--remote-debugging-port=9222
#   STUDIO_DATA_DIR=<scratch>
node tests/e2e/core-flows.mjs ./shots
```

`tests/e2e/screenshots/` drives the same way to retake the documentation's screenshots.
