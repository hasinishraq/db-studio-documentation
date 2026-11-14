# Settings

Open with the gear in the left bar or ++ctrl+comma++. Changes save as you make them (*Changes save
as you make them* at the bottom of the list, then *Saved*). The API key has its own **Save key**
button. Every field is listed in the [settings reference](../reference/settings.md).

## Find a setting

Type in **Search settings** to show only the matching settings, from every section at once:

![Searching for "row"](../assets/screenshots/111-settings-search.png){ .shot loading=lazy }

## Sections

=== "General"

    ![Theme, text size, history, masking](../assets/screenshots/105-settings-general.png){ .shot loading=lazy }

    - **Theme**: *Match the system*, *Dark* or *Light* (see [Light and dark](themes.md)).
    - **Text size**: − / + / Reset (also ++ctrl+plus++ / ++ctrl+minus++ anywhere).
    - **Runs kept in history**: 1000 by default.
    - **Hide masked columns in results too**: otherwise they are hidden only from the assistant.

=== "Editor"

    ![Font size, tab size, wrapping, line numbers, overview map, suggestions](../assets/screenshots/106-settings-editor.png){ .shot loading=lazy }

    Font size, spaces per tab, wrapping long lines, line numbers, the overview map (a small
    picture of the whole query at the right edge), and table and column suggestions while typing.

=== "Results and queries"

    ![Statement time limit, row limit, NULL text, row height](../assets/screenshots/107-settings-results.png){ .shot loading=lazy }

    - **Stop a statement after** *n* seconds (0 means never). The assistant keeps its own
      shorter limit.
    - **Stop reading a result after** *n* rows (0 means no limit). Exports always write every
      row.
    - **Show empty values as**: the text shown for NULL.
    - **Row height**: *Compact* or *Roomy*.

=== "Behaviour"

    ![Open a query tab after connecting; Export button format](../assets/screenshots/108-settings-behaviour.png){ .shot loading=lazy }

    Whether connecting opens a query tab, and which format the **Export** button writes (others
    stay in its menu).

=== "Assistant"

    ![Provider, endpoint, model, key, new-chat defaults](../assets/screenshots/124-assistant-settings-openrouter.png){ .shot loading=lazy }

    Provider, endpoint, model and API key, **Test the model**, and the defaults for new chats.
    See [AI assistant](assistant.md#1-set-up-a-provider).

=== "Hidden columns"

    ![sales.customers.email hidden on Shop](../assets/screenshots/109-settings-hidden-columns.png){ .shot loading=lazy }

    Per-connection rules for what the assistant may read:

    - **What the assistant may do** on this connection: *Follow the global settings*, *Local
      models only*, *Never send rows*, or *Off*.
    - **Columns hidden from the assistant**, by schema, table and column. **Hide a column** adds
      a row, and **Save hidden columns** stores them. Hidden columns show as `***` to the model.
      Password-like columns are always hidden.

=== "Troubleshooting"

    ![Copy diagnostics](../assets/screenshots/110-settings-troubleshooting.png){ .shot loading=lazy }

    **Copy diagnostics** copies the app version, your system and recent log lines, ready to paste
    into a bug report.

## Masked columns in the grid

With *Hide masked columns in results too* on, masked columns are hidden in grids and results as
well. A padlock marks them, and the values show as `***`:

![password_hash shown as ***](../assets/screenshots/112-masked-column-hidden.png){ .shot loading=lazy }

## Text size

Use **− / + / Reset** in General, ++ctrl+plus++ / ++ctrl+minus++ / ++ctrl+0++, or the command
palette. It scales the whole window.

![The window with bigger text](../assets/screenshots/117-text-size-bigger.png){ .shot loading=lazy }
