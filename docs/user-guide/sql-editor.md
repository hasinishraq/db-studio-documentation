# SQL editor

Each SQL tab has its own editor, connection, results and run log. The toolbar shows where the tab
runs (*in shop · schema public*). The title bar names the connection and database, and the status
bar shows the engine, server version and TLS state.

![An empty SQL tab](../assets/screenshots/041-sql-empty-tab.png){ .shot loading=lazy }

## Workflow at a glance

```mermaid
flowchart LR
  W["Write SQL<br/>(with completion)"] --> R["Ctrl+Enter"]
  R --> K{"Safety check"}
  K -- "read" --> G["Rows stream into the grid"]
  K -- "anything else" --> V["Review"]
  V --> G
  G --> F["Filter · select · copy · export"]
  G --> L["Run log line"]
  R -.-> X["Explain: plan as a tree"]
  R -.-> S["Stop: cancel on the server"]
```

## The toolbar

| Control | What it does |
|---|---|
| **Connection** | Which connection this tab runs on. |
| **in … · schema …** | The database and schema in use. Change them from the side panel. |
| **Run** / **Stop** | Runs the editor's SQL (or the selection). While a statement runs, the button turns into **Stop**. |
| **Explain** | Shows the plan without running the statement. |
| **Export as CSV** | Exports the result in the format chosen in Settings. **Other format** picks another. |
| **Auto commit / Manual** | The [transaction mode](safety.md#transactions). |

## Writing SQL

Completion suggests the tables and columns of the database in use, and you can turn it off in
Settings → Editor.

| Tables | Columns |
|---|---|
| ![Table names suggested after FROM](../assets/screenshots/042-editor-completion-tables.png){ .shot loading=lazy } | ![Column names suggested for an alias](../assets/screenshots/043-editor-completion-columns.png){ .shot loading=lazy } |

- **Run** with ++ctrl+enter++. With a selection, only the selected text runs.
- **Drafts** save automatically and appear under [*Saved queries*](#history-and-saved-queries).

## Results

![10,000 rows streamed in 74 ms](../assets/screenshots/044-results-10k-rows.png){ .shot loading=lazy }

- Rows stream into a **virtual grid**, so very large results stay responsive. A million rows has
  been measured at about 7 seconds with under 25 ms of UI stall.
- The bar above the grid counts the rows. **Close** hides the results until the next run.

### Select, filter and copy

| Select rows | Filter loaded rows |
|---|---|
| ![Selected rows highlighted](../assets/screenshots/045-results-row-selection.png){ .shot loading=lazy } | ![Filtering on "Asia"](../assets/screenshots/046-results-quick-filter.png){ .shot loading=lazy } |

- **Select rows** with click, ++ctrl++ + click and ++shift++ + click. ++ctrl+a++ selects all
  and ++esc++ clears. Copy and export then use only the selected rows.
- **Filter loaded rows** searches what is already on screen, in every column.
- **Copy** copies what is shown.

### Several statements

Several statements run in order and stop at the first error. Each one gets its own **Result** tab
and its own log line.

![Result 1 and Result 2 from two statements](../assets/screenshots/049-results-multiple-statements.png){ .shot loading=lazy }

### Export

**Export as CSV** (or the format set in Settings) writes the selected rows, else the filtered rows,
else the whole result. **Other format** offers CSV, JSON lines, Excel and SQL inserts. See
[Import and export](import-export.md).

![Exporting a result](../assets/screenshots/050-export-results.png){ .shot loading=lazy }

## The run log

Under the results, one line per statement: time, outcome, rows, duration and the SQL. Errors are
red, changes amber. Statements you declined in a review are logged as *Statement not run:
confirmation declined*. **Clear log** empties it.

![Results and the run log](../assets/screenshots/048-results-and-run-log.png){ .shot loading=lazy }

## Explain

**Explain** shows the plan as a tree, with the estimated cost and rows of each step. Steps that
cost more than their share are marked *Costly*.

![A plan with costly Sort, Aggregate and Hash Join steps](../assets/screenshots/047-explain-plan.png){ .shot loading=lazy }

## Stop a running statement

While a statement runs, the results show *Fetching* and both **Stop** buttons are live. Stopping
cancels on the server (a cancel request on PostgreSQL, `KILL QUERY` on MySQL), and the log says
*Stopped*.

| Running | Stopped |
|---|---|
| ![A long pg_sleep running, with Stop](../assets/screenshots/103-query-running-stop.png){ .shot loading=lazy } | ![The log reads Stopped](../assets/screenshots/104-query-cancelled.png){ .shot loading=lazy } |

## Other engines

The editor works the same way on every engine:

| MySQL | SQLite |
|---|---|
| ![A query on MySQL](../assets/screenshots/051-mysql-query.png){ .shot loading=lazy } | ![A query on SQLite](../assets/screenshots/052-sqlite-query.png){ .shot loading=lazy } |

## Errors

Database errors are shown exactly as the server reports them. When the server gives a position,
it is underlined in the editor. See [Troubleshooting](troubleshooting.md#query-errors) for more
examples.

![column "att" does not exist](../assets/screenshots/097-error-unknown-column.png){ .shot loading=lazy }

## History and saved queries

The left bar's **History** lists recent runs (the number kept is set in Settings → General).
**Saved queries** lists your drafts.

| History | Saved queries |
|---|---|
| ![History](../assets/screenshots/053-history.png){ .shot loading=lazy } | ![Saved queries](../assets/screenshots/054-saved-queries.png){ .shot loading=lazy } |
