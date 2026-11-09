# Storage and secrets

`studio-store` owns everything DB Studio keeps between runs.

## The app database

A local SQLite file in the app data folder, opened with `sqlx` and migrated on start
(`crates/studio-store/migrations/`). Blocking work runs off the async runtime.

| Table | Holds |
|---|---|
| `profiles` | Connection profiles (no secrets), including the remembered default schema |
| `drafts` | Saved queries |
| `history` | Statements run, with duration, rows and status |
| `settings` | One JSON value per key |
| `conversations`, `turns` | Chat titles, summaries, words and statement outcomes (never rows) |
| `audit_log` | The hash-chained record of every write |
| `known_hosts` | Pinned SSH host keys |
| `masking_rules` | Hidden columns per profile |

See the [data model](../reference/data-model.md) for columns.

## Secrets

Passwords, SSH passphrases and AI keys live **only** in the operating system keychain
(Windows Credential Manager, macOS Keychain, Secret Service on Linux), under the service
`com.dbstudio.desktop`. Keychain calls run on a dedicated worker thread.

- Secrets are wrapped in `secrecy::SecretString` and never logged.
- Commands accept secrets (write-only) but never return them; profiles carry only `has_password`
  and `has_secret` flags.
- No DTO has a secret field.

## The audit chain

Each entry stores `prev_hash` and `entry_hash = sha256(prev_hash || canonical_json(entry))`,
where the struct's field order makes the JSON canonical. `audit_verify` walks the chain and reports the first entry that does not match, so both a changed
entry and a deleted one are detected. Tests cover both.

## Legacy import

`import_legacy` reads the earlier Electron app's `studio-storage.json` (connections, drafts,
history) from its old folder names, so users can carry their profiles over.
