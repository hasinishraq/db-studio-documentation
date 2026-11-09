# Conventions

## Rust

- **Edition 2021, stable toolchain. No `unsafe`.**
- **No `unwrap()` / `expect()`** outside tests, except documented start-up invariants.
- **Errors**: library crates use `thiserror`; the app maps every error to
  `AppError { code, message }`.
- **Async everywhere** with Tokio. Blocking work (rusqlite, keychain, file I/O, image rendering)
  runs in `spawn_blocking` or a worker thread.
- **Never hold a lock across an `.await`** that does network I/O.
- **Logging** with `tracing`. Never log SQL parameters, row values or secrets.
- **Public items get a doc comment.** Keep functions small; prefer explicit types at crate
  boundaries.
- **Dependencies** are added with `cargo add`; shared ones go in the root
  `[workspace.dependencies]`.

## TypeScript

- **Strict mode; no `any` in new code.**
- **Small components**, state in contexts, multi-step interactions in `src/api` flows, pure logic
  in `src/lib`.
- **Call Rust only through `src/bindings.ts`**, which is generated and never edited.

## Adding a command

1. Add or extend DTOs in `studio-types` (derive `Serialize`, `Deserialize`, `specta::Type`).
2. Write the logic as a plain function in the right crate, with tests.
3. Add a thin `#[tauri::command] #[specta::specta]` wrapper in `src-tauri/src/commands/` that
   validates input and calls it.
4. Register it in `src-tauri/src/lib.rs`.
5. Regenerate bindings: `cargo test -p studio-app export_bindings`.
6. Wrap it in `src/api/index.ts` and use it from a flow or component.

If the command can change data, it must go through a session door and an approval; if it handles
a secret, the secret goes in and never comes back out.

## Working agreements

- Propose a plan (files, public API, tests) before writing code for a feature.
- Every change ships with its tests and passes the full gate.
- Update `docs/PROGRESS.md` with what was done, decisions, deviations and gaps.
- Changes are committed by a human after review.
