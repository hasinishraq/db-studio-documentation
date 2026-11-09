# Development environment

## Toolchains

| Tool | Version |
|---|---|
| Rust | stable (2021 edition), with `rustfmt` and `clippy` |
| Node.js | 22 or later |
| Tauri CLI | from npm (`npx tauri`) |
| Windows | Visual Studio 2022 Build Tools (C++ workload) and a Windows SDK |
| Linux | WebKitGTK 4.1 and the usual Tauri system packages |
| macOS | Xcode command-line tools |

The `ring` crypto provider is used everywhere, so NASM is not needed on Windows.

## First run

```bash
npm install
npx tauri dev
```

## Test databases

The database tests need PostgreSQL, MySQL and MariaDB. With Docker:

```bash
docker compose -f docker/compose.yml up -d
```

| Service | Port | URL variable |
|---|---|---|
| PostgreSQL 17 | 54329 | `TEST_PG_URL=postgres://studio:studio@127.0.0.1:54329/studio` |
| PostgreSQL with TLS | 54330 | `TEST_PG_TLS_URL` |
| MySQL 8.4 | 33069 | `TEST_MYSQL_URL=mysql://studio:studio@127.0.0.1:33069/studio` |
| MariaDB 11.4 | 33079 | `TEST_MARIADB_URL=mysql://studio:studio@127.0.0.1:33079/studio` |

Without Docker, portable server builds bound to `127.0.0.1` on the same ports work too. Tests for
an engine whose variable is unset print *skipped* and pass; SQLite always runs.

## Demo data

`scripts/seed/` fills the servers with generated, deterministic demo data (large
enough to exercise paging and streaming): `postgres-shop.sql`, `postgres-analytics.sql`, `mysql-store.sql`,
`mariadb-crm.sql` and `sqlite-notes.py`.

## Running against a scratch profile

Set `STUDIO_DATA_DIR` to a scratch folder (debug builds only) so a development run leaves your
own connections alone, and `WEBVIEW2_USER_DATA_FOLDER` if an installed copy is open.
