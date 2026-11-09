# Reviews and safety

Every statement is classified in Rust before it runs. **Reads run at once; anything else opens a
review.** The review shows the exact SQL that will run, what kind of statement it is, and which
connection and environment it targets.

## Risk levels

=== "Change"

    ![Review of an UPDATE](../assets/screenshots/14-review-change.png){ .shot }

    Inserts, updates and deletes. Click **Run on *connection*** to approve, or **Don't run**.

=== "Danger"

    ![Review of a DROP](../assets/screenshots/15-review-danger.png){ .shot }

    Statements that cannot be undone stay locked until you tick **I understand this cannot be
    undone**: `DROP`, `TRUNCATE`, `ALTER … DROP`, `GRANT`/`REVOKE`, and an `UPDATE` or `DELETE`
    without `WHERE`.

=== "Unparsed"

    ![Review of unparsable SQL](../assets/screenshots/16-review-unparsed.png){ .shot }

    SQL that does not parse is **treated as a write** and reviewed. It never runs unseen.

## Production

On a Production connection you type the connection's name before **Run** unlocks.

![Production review](../assets/screenshots/17-review-production.png){ .shot }

## Read-only connections

A read-only connection refuses every change, even after approval. The refusal comes from the
session in Rust, not from the interface.

![Read-only refusal](../assets/screenshots/18-read-only-refused.png){ .shot }

## Transactions

**Auto commit** (the default) commits each statement. **Manual** keeps changes in one transaction
until you press **Commit** or **Roll back**; the toolbar shows *Uncommitted changes*.

![Manual transaction](../assets/screenshots/19-manual-transaction.png){ .shot }

While a manual transaction has uncommitted work, switching database or schema is refused, and
typed `COMMIT` / `ROLLBACK` statements are refused so the app's state cannot drift.

![Commit first](../assets/screenshots/20-commit-first.png){ .shot }

## Limits

**Settings → Results and queries** sets a statement time limit and a row limit for your own
queries. The assistant has its own, shorter limits.

![A statement stopped by the time limit](../assets/screenshots/45-error-timeout.png){ .shot }

!!! info "How approvals work"
    Approving mints a random, single-use token that expires after 60 seconds and is bound to the
    SHA-256 of the exact SQL and to the session. Rust removes the token before comparing it. See
    the [safety engine](../architecture/safety-engine.md).
