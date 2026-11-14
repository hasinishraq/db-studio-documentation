# Reviews and safety

Every statement is classified in Rust before it runs. **Reads run at once. Anything else opens a
review.** The review shows the exact SQL that will run, what kind of statement it is, and which
connection and environment it targets.

## Workflow at a glance

```mermaid
flowchart TB
  S["Statement"] --> C{"Classified in Rust"}
  C -- "Read" --> RUN["Runs at once"]
  C -- "Change<br/>INSERT · UPDATE · DELETE" --> R1["Review this change"]
  C -- "Cannot be undone<br/>DROP · TRUNCATE · no WHERE · unparsable" --> R2["This cannot be undone<br/>+ tick to confirm"]
  R1 & R2 --> P{"Production?"}
  P -- "yes" --> T["Type the connection name"]
  P -- "no" --> A["Run on …"]
  T --> A
  A --> RO{"Read-only connection?"}
  RO -- "yes" --> REF["Refused in Rust"]
  RO -- "no" --> RUN2["Runs · logged · audited"]
  R1 & R2 -- "Don't run" --> D["Logged as declined"]
```

## Risk levels

=== "Change"

    ![Review this change: an UPDATE](../assets/screenshots/055-review-update.png){ .shot loading=lazy }

    Inserts, updates and deletes with a `WHERE`. Click **Run on *connection*** to approve, or
    **Don't run**.

    ![Review of a DELETE](../assets/screenshots/056-review-delete.png){ .shot loading=lazy }

=== "Cannot be undone"

    Statements that cannot be undone stay locked until you tick **I understand this cannot be
    undone**. That covers `DROP`, `TRUNCATE`, `ALTER … DROP`, `GRANT`/`REVOKE`, and an `UPDATE` or
    `DELETE` without `WHERE`. The review explains why.

    ![DELETE without WHERE removes every row](../assets/screenshots/057-review-delete-no-where.png){ .shot loading=lazy }

    | Locked | Confirmed |
    |---|---|
    | ![DROP TABLE, locked](../assets/screenshots/058-review-drop.png){ .shot loading=lazy } | ![DROP TABLE, box ticked](../assets/screenshots/059-review-drop-confirmed.png){ .shot loading=lazy } |

=== "Does not parse"

    ![Unparsable SQL treated as cannot be undone](../assets/screenshots/060-review-unparsed.png){ .shot loading=lazy }

    SQL that does not parse is **treated as a write** whose effect is unknown, so it needs the
    same confirmation. It never runs unseen. Even a typo in a read (`SELEC …`) is reviewed:

    ![A misspelt SELECT reviewed as an unknown statement](../assets/screenshots/098-error-syntax-reviewed.png){ .shot loading=lazy }

=== "Mixed scripts"

    ![A script with a SELECT and an UPDATE](../assets/screenshots/061-review-mixed.png){ .shot loading=lazy }

    In a script that mixes reads and writes, the review lists the statements that change data.
    Nothing in the script runs until you approve.

After you approve, the statement runs and the log records the rows affected (amber for changes):

![1 row affected in 6 ms](../assets/screenshots/062-change-applied.png){ .shot loading=lazy }

## Production

On a Production connection, **Run** stays locked until you type the connection's name.

| Waiting for the name | Name typed, Run unlocked |
|---|---|
| ![Type Shop to run it](../assets/screenshots/063-review-production.png){ .shot loading=lazy } | ![Shop typed](../assets/screenshots/064-review-production-typed.png){ .shot loading=lazy } |

The same rule applies to [grid edits](tables.md), [imports](import-export.md), new tables and
[assistant proposals](assistant.md#reads-writes-and-your-approval).

## Read-only connections

A read-only connection refuses every change, even after approval. The refusal comes from the
session in Rust, not from the interface. The title bar shows a *Read-only* label.

![this connection is read-only](../assets/screenshots/065-read-only-refused.png){ .shot loading=lazy }

## Transactions

**Auto commit** (the default) commits each statement. **Manual** keeps changes in one transaction
until you press **Commit** or **Roll back**. While work is pending, the toolbar shows
*Uncommitted changes*.

![Manual mode with uncommitted changes](../assets/screenshots/066-manual-transaction.png){ .shot loading=lazy }

While a manual transaction has uncommitted work, switching database or schema is refused. Typed
`COMMIT` / `ROLLBACK` statements are refused too, so the app's state cannot drift.

| Switching is refused | After Roll back |
|---|---|
| ![Commit or roll back first](../assets/screenshots/067-switch-refused-uncommitted.png){ .shot loading=lazy } | ![Rolled back](../assets/screenshots/068-rolled-back.png){ .shot loading=lazy } |

## Limits

**Settings → Results and queries** sets a statement time limit and a row limit for your own
queries. The assistant has its own, shorter limits.

![the statement timed out](../assets/screenshots/102-error-timeout.png){ .shot loading=lazy }

!!! info "How approvals work"
    Approving mints a random, single-use token that expires after 60 seconds and is bound to the
    SHA-256 of the exact SQL and to the session. Rust removes the token before comparing it. See
    the [safety engine](../architecture/safety-engine.md).
