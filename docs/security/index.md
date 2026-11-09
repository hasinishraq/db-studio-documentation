# Security

DB Studio connects to databases that matter, often in production, and lets an AI model take
part. Its security rests on a few properties that hold no matter what the interface or the model
does:

1. **Nothing changes data without a person approving the exact SQL.**
2. **Secrets never leave the operating system keychain**, except to the server they belong to.
3. **The model sees no rows unless you choose so, and never sees masked columns.**
4. **Transport is verified by default.**
5. **Every change is recorded in a tamper-evident log.**

| Page | Covers |
|---|---|
| [Threat model](threat-model.md) | What we defend against, and what we do not |
| [Safety rules](rules.md) | The eleven rules the code enforces, and where |

## Reporting a vulnerability

Please report security issues privately to the maintainers rather than in a public issue, with
steps to reproduce. Do not include real credentials or data.

## Dependency checks

`cargo deny check` runs licence and advisory checks. Each accepted advisory is documented in
`deny.toml` with its reason; current exceptions are unmaintained crates with no known flaw (under
Tauri's Linux stack, specta and the image renderer's font parser) and RSA timing
(RUSTSEC-2023-0071) for SSH RSA keys, which sign only on the user's own machine.
