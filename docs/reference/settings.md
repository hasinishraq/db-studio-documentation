# Settings

Settings are stored in the app database (`settings` table, one JSON value per key). Unknown or
missing fields fall back to their defaults.

## General and editor

| Setting | Default | Effect |
|---|---|---|
| Theme | System | System, Light or Dark |
| Text size | 100% | Scales the whole window (kept per machine) |
| Runs kept in history | 1000 | Older history entries are dropped |
| Hide masked columns in results too | Off | Masked columns show `***` in the grid, not only to the model |
| Font size | 13 | Editor font size in pixels |
| Spaces per tab | 2 | Editor indentation |
| Wrap long lines | Off | Editor word wrap |
| Show line numbers | On | Editor gutter |
| Show the overview map | Off | The editor's minimap |
| Suggest names while typing | On | Schema-aware completion |

## Results and behaviour

| Setting | Default | Effect |
|---|---|---|
| Stop a statement after (seconds) | 0 (never) | Time limit for **your** queries; the assistant keeps its own |
| Stop reading a result after (rows) | 0 (no limit) | Row cap for **your** queries; exports always write every row |
| Show empty values as | `NULL` | Text shown for NULL cells |
| Row height | Compact | Compact or Roomy |
| Open a query tab after connecting | Off | Opens a SQL tab on connect |
| Export button format | CSV | CSV, JSON lines, Excel or SQL inserts |

## Assistant

| Setting | Default | Effect |
|---|---|---|
| Provider | Ollama | Ollama, or OpenAI and compatible |
| Endpoint | `http://localhost:11434` | Base URL of the provider |
| Model | `llama3.1` | Model name, e.g. `gpt-4o-mini` or an OpenRouter id |
| API key | — | Stored in the OS keychain; never shown again |
| Run safe reads without asking, in new chats | On | Starting value of the chat's switch |
| Let the assistant see result rows, in new chats | Off | Starting value; profiles can forbid it |

## Per connection

| Setting | Values |
|---|---|
| What the assistant may do | Follow the global settings · Local models only · Never send rows · Off |
| Hidden columns | Schema, table and column; matched by column name (case-insensitive) |

Columns whose names contain `password`, `passwd`, `pwd`, `secret`, `api_key`, `apikey` or `token`
are always hidden from the model.
