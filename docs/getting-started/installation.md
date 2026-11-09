# Installation

## System requirements

| Platform | Requirement |
|---|---|
| Windows | Windows 10 or 11, 64-bit. Microsoft Edge WebView2 (preinstalled on Windows 11). |
| macOS | macOS 11 or later *(builds are produced by CI but have not yet been tested)*. |
| Linux | A desktop with WebKitGTK 4.1 *(builds are produced by CI but have not yet been tested)*. |

## Windows

Two installers are produced for each release:

| File | Use it when |
|---|---|
| `DB Studio_<version>_x64_en-US.msi` | You deploy with Group Policy or prefer Windows Installer. |
| `DB Studio_<version>_x64-setup.exe` | You want a simple per-user install. |

Run either one and follow the prompts. A newer MSI upgrades an existing installation in place,
including installs of the earlier "AI DB Studio" builds.

!!! warning "Unsigned builds"
    Current builds are not code-signed, so Windows SmartScreen shows a warning the first time.
    Choose **More info → Run anyway** only for installers you built yourself or received from a
    trusted source. Signing is described in [Releasing](../development/releasing.md).

## Where DB Studio keeps its data

| What | Where |
|---|---|
| Profiles, history, drafts, settings, audit log, chat history | `%APPDATA%\com.dbstudio.desktop\` (Windows) |
| Passwords, SSH secrets, AI keys | The OS keychain (Windows Credential Manager), service `com.dbstudio.desktop` |
| Crash reports | A crash file in the app data folder; nothing is sent anywhere |

To start completely fresh, quit DB Studio, delete the app data folder and remove the
`com.dbstudio.desktop` entries from the keychain.

## Building from source

See [Development environment](../development/environment.md). In short:

```bash
npm install
npx tauri build          # installers in target/release/bundle/
```
