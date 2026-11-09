# Development

| Page | Covers |
|---|---|
| [Environment](environment.md) | Toolchains, test servers, running the app |
| [Testing](testing.md) | Rust, frontend, real-server and end-to-end tests |
| [Releasing](releasing.md) | Versions, installers, signing, updates |
| [Conventions](conventions.md) | Code style, errors, logging, how to add a command |

## Command summary

| Task | Command |
|---|---|
| Run the app in development | `npx tauri dev` |
| Rust tests | `cargo test --workspace` |
| Lint | `cargo fmt --all --check && cargo clippy --workspace --all-targets -- -D warnings` |
| Frontend checks | `npm run typecheck && npm test` |
| Regenerate bindings | `cargo test -p studio-app export_bindings` |
| Licences and advisories | `cargo deny check` |
| Build installers | `npx tauri build` |
| This documentation | `cd documentation && mkdocs serve` |
