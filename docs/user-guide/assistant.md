# AI assistant

Open the **Assistant** from the left bar (++ctrl+j++). Ask questions in plain language. The
assistant writes SQL, runs safe reads, draws charts from the real rows, and proposes changes for
you to approve.

## Workflow at a glance

```mermaid
flowchart LR
  S["Settings → Assistant<br/>provider · model · key"] --> T["Test the model"]
  T --> O["Open the Assistant"]
  O --> W["Check where it works<br/>connection › database › schema"]
  W --> Q["Ask a question"]
  Q --> R{"What did it write?"}
  R -- "safe read" --> A["Runs by itself<br/>table or chart"]
  R -- "change" --> V["Review card<br/>Approve / Decline"]
```

## 1. Set up a provider

Open **Settings → Assistant**.

=== "Ollama (default)"

    ![Ollama on localhost:11434 with llama3.1](../assets/screenshots/123-assistant-settings-default.png){ .shot loading=lazy }

    A local model through [Ollama](https://ollama.com/). No key is needed, and nothing leaves
    your machine.

=== "OpenAI or compatible"

    ![OpenRouter endpoint and model](../assets/screenshots/124-assistant-settings-openrouter.png){ .shot loading=lazy }

    Any OpenAI-compatible endpoint: OpenAI, OpenRouter, or a self-hosted server. Set the
    **Endpoint** and **Model**, paste the **API key** and press **Save key**.

=== "Key saved"

    ![The key field is empty again after saving](../assets/screenshots/125-assistant-key-saved.png){ .shot loading=lazy }

    The key goes to the system keychain and is **never shown again**. The field goes back to
    *Paste a new key*.

**Test the model** sends one short message with the settings above:

![The model replied: ready](../assets/screenshots/126-assistant-model-test.png){ .shot loading=lazy }

If the test fails, the provider's own error is shown. See
[Troubleshooting](troubleshooting.md#assistant-errors).

| New-chat default | Meaning |
|---|---|
| **Run safe reads without asking** *(on)* | Only statements the safety check calls read-only, with a time and row limit. Each chat can change it. |
| **Let the assistant see result rows** *(off)* | Off by default. Masked columns stay hidden, and a connection set to never share rows still never shares them. |

## 2. Open the assistant

![Ask about your data, with four starter questions](../assets/screenshots/127-assistant-empty.png){ .shot loading=lazy }

- **Conversations** (left) lists past chats, each labelled with its connection and database.
  The **+** button starts a new chat.
- The **pickers** at the top choose the connection › database › schema the chat works in.
- **Run safe reads automatically** and **Let the model read query results** switch for this chat.
  The second says *(not allowed on this connection)* when a profile forbids it.
- The input says where it is working (*Working in Shop › shop › public*). ++enter++ sends and
  ++shift+enter++ adds a new line.

## Where it works

The assistant sees only the tables of the database in use, never the rest of the server. Ask about
something outside it and it says so, listing what it can see:

![Only the tables of shop are visible](../assets/screenshots/141-assistant-scoped-to-schema.png){ .shot loading=lazy }

- Change the database or schema with the pickers in the chat, or from the side panel. Both use the
  same switch, so SQL tabs and the assistant always agree.
- Switching mid-chat adds a note to the conversation.
- With no database in use (a MySQL connection opened without one), it lists the database names
  and asks which one to use.

| Switched to analytics | On MySQL |
|---|---|
| ![Working in analytics › public](../assets/screenshots/134-assistant-switch-database.png){ .shot loading=lazy } | ![The assistant on a MySQL connection](../assets/screenshots/135-assistant-mysql.png){ .shot loading=lazy } |

## Asking questions

Each answer shows the SQL it ran (tagged **Read**), the rows or a chart, the row count and time,
and the model's explanation. **Regenerate** asks again, and the copy button copies the answer.

![Describe the main tables in the sales schema](../assets/screenshots/131-assistant-describe.png){ .shot loading=lazy }

While it is answering, **Send** turns into **Stop**:

![Stop while the assistant answers](../assets/screenshots/140-assistant-answering-stop.png){ .shot loading=lazy }

Conversations are kept locally, without result rows:

![Past conversations](../assets/screenshots/142-assistant-conversations.png){ .shot loading=lazy }

## Charts

Ask for a chart ("…as a pie chart", "…per month as a line"), and the assistant asks the app to
draw one. It names **columns only**. The values always come from the rows the database returned,
so it cannot draw numbers it made up. With row sharing off, the model itself never sees the
values. It says so, and points you to the chart.

=== "Pie"

    ![Share of total revenue by customer region](../assets/screenshots/128-assistant-pie.png){ .shot loading=lazy }

    The smallest parts are combined into *Other*.

=== "Bar"

    ![Top products by units sold](../assets/screenshots/129-assistant-bar.png){ .shot loading=lazy }

=== "Line"

    ![Orders per month](../assets/screenshots/130-assistant-line.png){ .shot loading=lazy }

### Show as

Every result has a **Show as** chooser: table, bar, horizontal bar, stacked bar, line, area, pie,
donut or scatter. Forms that do not fit the data are greyed out with the reason. **Save chart**
writes PNG, JPEG or PDF.

| As a bar chart | As a table |
|---|---|
| ![The pie answer re-drawn as bars](../assets/screenshots/136-assistant-show-as.png){ .shot loading=lazy } | ![The same rows as a table](../assets/screenshots/137-assistant-show-as-table.png){ .shot loading=lazy } |

## Reads, writes and your approval

| The assistant… | What happens |
|---|---|
| Runs a **safe read** | Runs at once if *Run safe reads automatically* is on: in a READ ONLY transaction, with a 30-second limit and a 5,000-row cap. |
| Proposes **anything else** | A card, *The assistant wants to run this*, waits for **Approve** or **Decline**. On Production you type the connection name. |
| Is **declined** | It is told the statement did not run and does not submit it again. |
| Targets a **read-only** connection | Even after you approve, Rust refuses it. |

| Waiting for approval (Production) | Declined |
|---|---|
| ![The assistant wants to run this UPDATE](../assets/screenshots/132-assistant-write-review.png){ .shot loading=lazy } | ![Declined. It did not run.](../assets/screenshots/133-assistant-write-declined.png){ .shot loading=lazy } |

On a read-only connection, the card still appears, but the run is refused:

| Proposed | Refused |
|---|---|
| ![A DELETE proposed on a read-only connection](../assets/screenshots/139-assistant-read-only.png){ .shot loading=lazy } | ![this connection is read-only](../assets/screenshots/139b-assistant-read-only-refused.png){ .shot loading=lazy } |

## Privacy

![Let the model read query results is off, so the model never sees the values](../assets/screenshots/138-assistant-share-off.png){ .shot loading=lazy }

- **Result rows are not sent to the model** unless you tick *Let the model read query results*.
  Connections set to never share rows (and Production, by default) forbid it.
- **Masked columns** are replaced with `***` before anything reaches the model, and are never
  stored in its memory. Choose them in [Settings → Hidden columns](settings.md).
- **Per connection**, the assistant can be limited to local models, forbidden to see rows, or
  turned off.
- Conversations are kept locally, without result rows.

Models without tool calling fall back to fenced SQL in their replies, which still goes through the
same review.
