# Changelog

## 0.1.4

- Renamed to **DB Studio**, with the new identifier `com.dbstudio.desktop`; the app starts with
  fresh data. The MSI upgrade code is pinned so installs upgrade in place.

## 0.1.3

- Visible name changed from *AI DB Studio* to *DB Studio*.

## 0.1.2

- **Default database and schema** from a right-click menu, optionally remembered per connection.
- The **assistant works on the database in use only**, names server, database and schema in the
  chat, and can switch database there. Conversations record their database.

## 0.1.1

- **Charts in the assistant**: bar, horizontal and stacked bar, line, area, pie, donut and scatter,
  requested by the model by column name and drawn from the real rows. A *Show as* chooser on every
  result; save charts as PNG, JPEG or PDF.

## 0.1.0

- **Inline table editing** with pending changes, one all-or-nothing review, and refusal when a row
  changed since loading.
- **Diagram export** as PNG, JPEG or PDF.
- MySQL connections report rows matched (`CLIENT_FOUND_ROWS`).
- Everything before: connections with TLS and SSH, the explorer, the SQL editor with streaming
  results, reviews and approvals, manual transactions, table data and structure, the schema
  diagram, CSV import and exports, the audit log, the AI assistant, settings, and the legacy
  importer.

## Unreleased

- Pie charts show at most five parts (one per series colour) and fold the rest into *Other*.
