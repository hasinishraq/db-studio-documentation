# Safety rules

These rules are non-negotiable. Each is enforced in Rust and covered by tests that must never be
weakened, skipped or deleted to make them pass.

| # | Rule | Enforced in |
|:-:|---|---|
| 1 | **The webview is untrusted.** Every command re-validates its input in Rust. | `src-tauri/src/commands/*` |
| 2 | **Secrets live only in the OS keychain.** Never returned to the frontend, never logged, never in a DTO. | `studio-store` (secrets), `secrecy` |
| 3 | **Only `studio-safety` decides whether SQL is read-only.** The frontend never decides. | `studio-safety`, re-checked in every session door |
| 4 | **SQL that does not parse is treated as a write** (fail closed). | `studio-safety` classifier |
| 5 | **Any non-read statement needs an approval token**: random, single use, 60 s, bound to `sha256(exact SQL)` and the session. The token is consumed (removed) before it is compared. | `studio-safety::ApprovalBook`, `studio-session` |
| 6 | **Tokens are minted only by commands triggered by a human click** (`approve_query`, `ai_review`). | A source-scanning test proves no other path mints one |
| 7 | **AI SQL runs without review only if `ai_auto` is true**, and then always inside a READ ONLY transaction with a timeout and a row cap. | `run_ai_read` (it has no token parameter at all) |
| 8 | **Read-only profiles refuse every non-read statement**, even with an approval. | `SessionPolicy`, every door |
| 9 | **Result rows never reach the model** unless the user enabled sharing, and are **never** stored in AI memory or logs. | `studio-ai` describe, masks and memory scrubbing |
| 10 | **TLS certificate verification is on by default.** Disabling it requires an explicit profile mode. | `TlsMode::VerifyFull` default, drivers |
| 11 | **Never weaken, skip or delete a safety test** to make it pass. | Code review |

## Supporting rules

- **Grid saves** go through their own door: grid tokens only, row changes only, each statement
  exactly one row, all or nothing.
- **Imports** are approved against a hash of the file and run in one transaction.
- **Typed `COMMIT`/`ROLLBACK`** are refused in manual mode, so the app's transaction state cannot
  drift.
- **Logs** never contain SQL parameters, row values or secrets.
- **Diagram and chart SVG** from the webview is size-capped and may not reference any image, file
  or URL before Rust renders it.
