# Troubleshooting

DB Studio shows errors as the server, the network or the AI provider reports them, without
rewording them. This page lists the ones you are most likely to see, what each one means and what
to do about it.

!!! tip "Reporting a problem"
    **Settings → Troubleshooting → Copy diagnostics** copies the app version, your system and
    recent log lines, ready to paste into a bug report.

## Connection errors

These appear under the form when you press **Test connection**, or under a connection in the side
panel when connecting fails.

=== "Wrong password"

    ![authentication failed: password authentication failed for user "studio"](../assets/screenshots/006-test-fail-wrong-password.png){ .shot loading=lazy }

    The server rejected the user name or password. Re-enter the password (in an existing
    connection, **Replace** next to *Saved*).

=== "Unknown database"

    ![database "shopx" does not exist](../assets/screenshots/007-test-fail-unknown-database.png){ .shot loading=lazy }

    The **Database** field names a database the server does not have. Fix the name, or leave it
    empty and choose one after connecting.

=== "Connection refused"

    ![The target machine actively refused it](../assets/screenshots/008-test-fail-refused.png){ .shot loading=lazy }

    Nothing is listening on that host and port. Check that the server is running, the port is
    right, and no firewall is in the way.

=== "Unknown host"

    ![No such host is known](../assets/screenshots/009-test-fail-unknown-host.png){ .shot loading=lazy }

    The host name does not resolve. Check for typos, VPN or DNS.

=== "SSH bastion unreachable"

    ![SSH connection failed: actively refused](../assets/screenshots/021-test-fail-ssh-unreachable.png){ .shot loading=lazy }

    The tunnel could not reach the bastion. Check the **Bastion host**, port and sign-in method
    in the *SSH tunnel* tab.

In the side panel, a failed connection keeps its reason underneath while the others stay
connected:

![One connection failed among several](../assets/screenshots/026-connect-all-one-failed.png){ .shot loading=lazy }

### TLS errors

TLS errors mean verification is doing its job. Do not lower the mode on a remote server just to
make them go away.

=== "Server does not support TLS"

    | Verify full | Require |
    |---|---|
    | ![TLS error: server does not support TLS](../assets/screenshots/011-test-fail-tls-required.png){ .shot loading=lazy } | ![The same error in Require mode](../assets/screenshots/012-tls-require-warning.png){ .shot loading=lazy } |

    The TLS mode asks for encryption, but the server offers none. Enable TLS on the server. For a
    local development server only, choose *Disable*.

=== "Unknown issuer"

    ![invalid peer certificate: UnknownIssuer](../assets/screenshots/015-test-fail-mysql-self-signed.png){ .shot loading=lazy }

    The certificate is self-signed or from a private CA. Choose the server's CA file under **CA
    certificate** in the TLS tab, or install that CA on your system.

## Query errors

Query errors appear above the results and in the run log. When the server reports a position, it
is underlined in the editor.

| Unknown column | Division by zero |
|---|---|
| ![column "att" does not exist](../assets/screenshots/097-error-unknown-column.png){ .shot loading=lazy } | ![division by zero](../assets/screenshots/099-error-division-by-zero.png){ .shot loading=lazy } |

| Duplicate key | Missing table |
|---|---|
| ![A unique constraint violated](../assets/screenshots/100-error-unique.png){ .shot loading=lazy } | ![The table does not exist](../assets/screenshots/101-error-missing-table.png){ .shot loading=lazy } |

| Time limit | A typo |
|---|---|
| ![the statement timed out](../assets/screenshots/102-error-timeout.png){ .shot loading=lazy } | ![SELEC reviewed as an unknown statement](../assets/screenshots/098-error-syntax-reviewed.png){ .shot loading=lazy } |

- **The statement timed out**: it ran past *Stop a statement after* in **Settings → Results and
  queries**. Raise the limit (0 means never) or make the query cheaper; **Explain** helps.
- **A typo** in a read is not a read any more. SQL that does not parse is reviewed as an unknown
  statement, so a misspelt `SELECT` asks for confirmation instead of running. Choose
  **Don't run** and fix it.
- **Constraint errors** name the constraint. The failed statement stays in the run log.

## Saving table edits

| Row changed meanwhile | Foreign key |
|---|---|
| ![affected 0 rows instead of 1, so nothing was saved](../assets/screenshots/077-inline-edit-conflict.png){ .shot loading=lazy } | ![a foreign key constraint fails](../assets/screenshots/078-error-foreign-key.png){ .shot loading=lazy } |

Nothing is saved in either case, and your edits stay pending. Refresh to see the current row and
edit again, or fix the related rows first. See
[Tables → When a save is refused](tables.md#when-a-save-is-refused).

## Changes that are refused

| Read-only connection | Uncommitted transaction |
|---|---|
| ![this connection is read-only](../assets/screenshots/065-read-only-refused.png){ .shot loading=lazy } | ![Commit or roll back first. Then change database.](../assets/screenshots/067-switch-refused-uncommitted.png){ .shot loading=lazy } |

- **this connection is read-only**: by design. Use a writable connection.
- **Commit or roll back first**: a manual transaction has pending work. Press **Commit** or
  **Roll back** in the toolbar.

## Assistant errors

=== "Unknown model"

    ![status 400 Bad Request: not a valid model ID](../assets/screenshots/143-assistant-test-unknown-model.png){ .shot loading=lazy }

    The provider does not know the **Model** name. Copy the exact id from the provider's model
    list.

=== "Same error in a chat"

    ![the model failed: ProviderResponseError](../assets/screenshots/144-assistant-chat-provider-error.png){ .shot loading=lazy }

    The chat shows the provider's error in place of an answer. Fix the setting, then press
    **Regenerate**.

=== "Provider unreachable"

    ![could not reach the model: error sending request](../assets/screenshots/145-assistant-test-unreachable.png){ .shot loading=lazy }

    Nothing answered at the **Endpoint**. Check the URL. For Ollama, check that it is running.

=== "Fixed"

    ![The test passes again](../assets/screenshots/146-assistant-test-restored.png){ .shot loading=lazy }

    Once the settings are right, **Test the model** passes again.
