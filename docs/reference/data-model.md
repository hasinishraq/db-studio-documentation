# Data model

The app database is SQLite, migrated by `sqlx` from `crates/studio-store/migrations/`.

```mermaid
erDiagram
  profiles ||--o{ masking_rules : "hides"
  profiles ||--o{ conversations : "chats on"
  conversations ||--o{ turns : "has"
  profiles ||--o{ history : "ran"
  profiles ||--o{ drafts : "saved"
  profiles ||--o{ audit_log : "changed"

  profiles {
    text id PK
    text name
    text engine
    text host
    int port
    text database
    text username
    text tls_mode
    text ssh_json "no secrets"
    text environment
    int read_only
    text ai_policy
    text default_schema
  }
  conversations {
    text id PK
    text profile_id
    text title
    text summary
    text database
  }
  turns {
    text id PK
    text conversation_id FK
    int seq
    text user_text
    text assistant_text
    text queries_json "no rows"
  }
  audit_log {
    int id PK
    int at
    text origin
    text kind
    text sql_hash
    text sql_text
    int rows_affected
    text outcome
    text prev_hash
    text entry_hash
  }
  history {
    text id PK
    text sql
    int duration_ms
    int row_count
    text status
  }
  drafts {
    text id PK
    text title
    text sql
  }
  masking_rules {
    text profile_id
    text schema_name
    text table_name
    text column_name
  }
```

Also: `settings (key, value_json)` and `known_hosts (host, port, key_type, fingerprint)`.

| Migration | Adds |
|---|---|
| `0001_init.sql` | All tables above |
| `0002_defaults.sql` | `profiles.default_schema`, `conversations.database` |

Secrets are **not** in this database; they live in the OS keychain.
