# Known issues

## Open bugs

| Area | Issue |
|---|---|
| MySQL / MariaDB | A `SELECT SLEEP(n)` stopped by the statement time limit is reported as a success (`1`), because the server returns 1 instead of an error. PostgreSQL reports a timeout correctly. |
| Command palette | Closes on a click outside it, but not on ++esc++. |
| Side panel | *Commit or roll back first* stays under the connection after rolling back. |
| Settings | The section list can highlight the wrong section while scrolling. |
| Side panel | Deleting a connection does not ask for confirmation. |
| Assistant | The tab forgets its chosen connection when it is closed and reopened. |
| Diagram | Lines are drawn under tables and can look attached to the wrong one. |
| Charts | Bar labels are cut at 10 characters (horizontal bars show full names). |

## Limitations

| Area | Limitation |
|---|---|
| Security | Approvals are triggered by a click in the webview; a compromised webview could call `approve_query` itself. A native confirmation dialog is planned. |
| Assistant | Qualified names to other databases (`otherdb.table`) are not refused; the prompt tells the model to stay in the database in use. |
| Diagram | Table positions are not saved. Schema export omits types, triggers and functions; MySQL foreign-key cycles need `SET FOREIGN_KEY_CHECKS=0` by hand. |
| Export | Binary columns export their 64-byte preview, not the full value; Excel stops at the sheet row limit. |
| TLS | On MySQL, *Verify CA* behaves as strictly as *Verify full*. Server certificate details are shown for PostgreSQL only. |
| SSH | OpenSSH host certificates are refused; agent sign-in has not been exercised against a real agent. |
| Storage | The *encrypt the app database* setting exists but does nothing yet. |
| Platforms | macOS and Linux builds have not been tested; installers are unsigned; the updater is not configured. |
| IPC | Streaming back-pressure ends at the Rust side of IPC. |
