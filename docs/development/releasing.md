# Releasing

## Version

The version lives in three places and must match:

- `Cargo.toml` → `[workspace.package] version` (every crate inherits it)
- `package.json` (and `package-lock.json`, via `npm version <x.y.z> --no-git-tag-version`)
- `src-tauri/tauri.conf.json` → `version`

Bump it for every installer you hand out, so Windows treats the new MSI as an upgrade.

## Building installers

```bash
npx tauri build                 # MSI and NSIS setup on Windows
npx tauri build --bundles msi   # MSI only
```

Output goes to `target/release/bundle/`.

!!! important "Identity"
    - The **identifier** `com.dbstudio.desktop` names the app data folder and the keychain
      service. Changing it makes the app start fresh.
    - The **MSI upgrade code** is pinned in `tauri.conf.json`
      (`bundle.windows.wix.upgradeCode`). Never change it, or Windows installs side by side.

## CI

`.github/workflows/release.yml` builds Windows, macOS and Linux installers on a tag and drafts a
release named *DB Studio &lt;tag&gt;*.

## Signing and updates

Not yet configured; both need the owner's accounts:

1. **Updater keys**: `npx tauri signer generate -w ~/.tauri/dbstudio.key`. Store the private key
   and its password as CI secrets, put the public key in `plugins.updater.pubkey` and the
   endpoint in `plugins.updater.endpoints`. Back the private key up offline: without it,
   installed apps can never update again.
2. **Code signing**: a Windows code-signing certificate, and an Apple Developer ID with
   notarisation.

See `docs/RELEASING.md` in the repository for the step-by-step list.
