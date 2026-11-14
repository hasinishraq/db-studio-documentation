# Tables and inline editing

Click a table in the side panel to open it. The **Data** view pages through rows on the server,
100 at a time. **Structure** shows the table's definition.

## Workflow at a glance

```mermaid
flowchart LR
  O["Open a table"] --> P["Page · filter · sort"]
  P --> E["Edit cells · insert · delete"]
  E --> N["Not saved: n changed"]
  N --> R["Review and save<br/>(one review, all statements)"]
  R --> OK{"Every statement<br/>changed exactly one row?"}
  OK -- "yes" --> S["Committed together"]
  OK -- "no / DB error" --> U["Nothing saved<br/>edits stay pending"]
  N -- "Discard" --> P
```

## Browsing data

![The orders table, rows 1–100 of 10,000](../assets/screenshots/069-table-data.png){ .shot loading=lazy }

Each column header shows its name and type. The pager on the right shows the range and the total
(*1–100 of 10,000*). Use the arrows to page and the refresh button to reload.

![The next page: 101–200](../assets/screenshots/070-table-next-page.png){ .shot loading=lazy }

### Filter and sort

**Filter** opens a builder. Pick a column, an operator (*equals*, *contains*, *starts with*, *is
greater than*, *is empty*, …) and a value, and add as many conditions as you need. They are
combined with *and*.

![Filter rows: status equals delivered and total is greater than 500](../assets/screenshots/071-filter-dialog.png){ .shot loading=lazy }

Applied filters appear as chips above the grid, and the count updates (*1–100 of 1,725*). Remove
one with its **×**, or all with **Clear filters**. Click a **column header** to sort, and click
again to reverse. An arrow marks the sorted column.

![Two filter chips and total sorted descending](../assets/screenshots/072-filter-chips-sorted.png){ .shot loading=lazy }

The same view works on every engine. Here is a MySQL table:

![A MySQL products table](../assets/screenshots/073-mysql-table.png){ .shot loading=lazy }

## Editing in place

Tables with a primary key can be edited directly in the grid.

| Action | How |
|---|---|
| Edit a cell | Double-click, or press ++enter++ / ++f2++ on it |
| Keep and move on | ++enter++ (down) or ++tab++ (right) |
| Cancel | ++esc++ |
| Set NULL | The **NULL** button, or ++alt+n++ |
| Add a row | **Insert row** (new rows appear at the top, marked *New*, with *default* for unset columns) |
| Delete rows | Tick them, then **Delete rows** |
| Undo one change | Right-click → **Revert cell** / **Revert row**, or the undo arrow on a deleted row |

Nothing is saved yet. Changed cells are tinted, new rows are marked *New* and deleted rows are
struck through. A bar counts what is pending (*Not saved: 1 new · 1 deleted*) and offers
**Discard** or **Review and save**.

![One new row and one deleted row pending](../assets/screenshots/074-inline-edit-pending.png){ .shot loading=lazy }

**Review and save** shows every statement in one review. They run **all or nothing**, inside a
transaction (or a savepoint, in manual mode).

| One review for the whole save | Saved |
|---|---|
| ![The INSERT and DELETE in one review](../assets/screenshots/075-inline-edit-review.png){ .shot loading=lazy } | ![Saved, with a log line](../assets/screenshots/076-inline-edit-saved.png){ .shot loading=lazy } |

### When a save is refused

Every statement must change **exactly one row**. Each `UPDATE` also matches the values you loaded,
so if someone changed the row in the meantime, nothing is saved and your edits stay pending:

![Change 1 of 1 affected 0 rows instead of 1, so nothing was saved](../assets/screenshots/077-inline-edit-conflict.png){ .shot loading=lazy }

If the database refuses a statement (here a foreign key), the whole save is undone:

![Cannot delete or update a parent row: a foreign key constraint fails](../assets/screenshots/078-error-foreign-key.png){ .shot loading=lazy }

Paging, sorting, filtering or closing the tab with pending edits asks before discarding them:

![Discard unsaved changes?](../assets/screenshots/079-unsaved-changes-prompt.png){ .shot loading=lazy }

### What cannot be edited

=== "No primary key (views)"

    ![This table has no primary key, so cells cannot be edited in place](../assets/screenshots/082-view-read-only.png){ .shot loading=lazy }

    Tables and views **without a primary key** cannot be edited cell by cell. The grid explains
    why and offers a row form instead (double-click a row).

=== "Read-only connection"

    ![This connection is read-only, so rows cannot be changed](../assets/screenshots/083-read-only-connection-table.png){ .shot loading=lazy }

    On a **read-only** connection, *Insert row* and *Delete rows* are disabled.

=== "Binary"

    ![A BLOB column with a lock icon](../assets/screenshots/084-binary-locked.png){ .shot loading=lazy }

    **Binary** columns are locked (a padlock in the header). The grid holds only a 64-byte
    preview.

=== "Masked"

    ![The password_hash column locked](../assets/screenshots/085-sensitive-column-masked.png){ .shot loading=lazy }

    **Masked** columns are locked too. With *Hide masked columns in results too* switched on in
    [Settings](settings.md), their values show as `***`:

    ![password_hash shown as ***](../assets/screenshots/112-masked-column-hidden.png){ .shot loading=lazy }

## Structure

The **Structure** view shows the estimated row count, size on disk, owner, and every column,
index, foreign key and constraint, plus the `CREATE` statement with a Copy button.

| PostgreSQL | MySQL |
|---|---|
| ![Structure of a PostgreSQL table](../assets/screenshots/081-table-structure-postgres.png){ .shot loading=lazy } | ![Structure of a MySQL table](../assets/screenshots/080-table-structure-mysql.png){ .shot loading=lazy } |
