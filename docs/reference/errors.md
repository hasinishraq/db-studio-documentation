# Error codes

Every command fails with an `AppError { code, message, detail? }`. `detail` carries the SQLSTATE,
the character position and a server hint for database errors, or the host-key facts for SSH.

| Code | Meaning | What to do |
|---|---|---|
| `NotConnected` | The session does not exist (closed or lost) | Connect again |
| `ApprovalRequired` | A write arrived without a token, or the token was already used | Run it again from the editor and approve it |
| `ApprovalExpired` | The token is older than 60 seconds | Approve again |
| `ApprovalMismatch` | The token does not fit this SQL, session or door | Approve the exact statement you run |
| `ReadOnlyProfile` | The connection refuses writes | Use a connection that is not read-only |
| `Timeout` | A statement ran past its time limit | Raise the limit in Settings, or make the query faster |
| `Cancelled` | Stopped by the user, or a dialog was cancelled | — |
| `Database` | The server reported an error | Read the message; the position is underlined in the editor |
| `Tls` | The TLS handshake or verification failed | Check the certificate, CA file and mode |
| `Ssh` | The SSH tunnel failed | Check the bastion, user and key |
| `HostKeyUnknown` | The bastion's host key is not yet trusted | Compare the fingerprint, then trust it |
| `HostKeyChanged` | The bastion's host key changed | Find out why before trusting it again |
| `DirtyTransaction` | A manual transaction has uncommitted work | Commit or roll back first |
| `Secret` | The OS keychain failed | Check that the keychain is unlocked and reachable |
| `Ai` | The assistant failed (provider, policy, memory) | Read the message; try *Test the model* |
| `InvalidInput` | Input failed validation | Correct the value named in the message |
| `Internal` | Anything else | Copy diagnostics from Settings and report it |
