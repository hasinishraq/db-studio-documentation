# Import and export

## Exporting

| From | What is written |
|---|---|
| A SQL tab | The selected rows, else the filtered rows, else the whole result |
| A table tab | The selected rows, else the filtered rows, else the whole table (streamed) |

Formats: **CSV**, **JSON lines**, **Excel** (`.xlsx`) and **SQL inserts**. The **Export as …**
button uses the format chosen in **Settings → Behaviour**. **Other format** picks another.

![Exporting a query result](../assets/screenshots/050-export-results.png){ .shot loading=lazy }

The save dialog is opened by Rust, so the interface never supplies a file path. Masked values stay
masked in exports. Exports always write every row, whatever the row limit in Settings.

## Importing CSV

```mermaid
flowchart LR
  A["Import CSV"] --> B["Choose a CSV file"]
  B --> C["Map columns<br/>(or skip)"]
  C --> D["Rows per batch"]
  D --> E["Review import"]
  E --> F["Approve<br/>(type the name on Production)"]
  F --> G["All batches in one transaction"]
  G --> H["Imported n rows"]
```

Open a table and choose **Import CSV**.

=== "1 · Choose a file"

    ![Import into products: Choose a CSV file](../assets/screenshots/086-import-start.png){ .shot loading=lazy }

    The file stays in Rust. The interface sees only a preview.

=== "2 · Map the columns"

    ![Each table column mapped to a file column](../assets/screenshots/087-import-mapping.png){ .shot loading=lazy }

    Map each table column to a file column, or choose **Skip this column** (for example an
    identity `id`). Set the **rows per batch**. The preview below shows the first rows as they
    will be imported.

=== "3 · Review"

    ![This imports 10,000 rows. One bad row undoes the whole import.](../assets/screenshots/088-import-review.png){ .shot loading=lazy }

    **Review import** shows the `INSERT` and how many rows it covers, and warns that one bad row
    undoes everything. The review covers a hash of the file, so editing the file after the review
    makes the approval fail.

=== "4 · Done"

    ![Imported 10,000 rows](../assets/screenshots/089-import-done.png){ .shot loading=lazy }

    The table's total updates (*1–100 of 10,500*), and the import appears in the
    [audit log](audit-log.md) with source *Import*.

Every batch runs in **one transaction**: a single bad row undoes the whole import.

## Schema files

The [schema diagram](diagrams.md) tab can **export** a schema's structure (tables in dependency
order, foreign keys and views) as a `.sql` file, and **import** one.

| Export schema | Import schema (preview) |
|---|---|
| ![Exported the structure of sales: 5 tables and views](../assets/screenshots/121-schema-export.png){ .shot loading=lazy } | ![Preview of sales-schema.sql](../assets/screenshots/122-schema-import.png){ .shot loading=lazy } |

An imported file is **drawn as a preview first**. The banner says how many tables it creates and
which statements could not be read, and that *Nothing has been applied*. **Apply to …** sends it
through the normal review. **Close preview** discards it.
