# Installation preview

**There is no public vmcli release yet.** Public API access and the public SSH route have not passed acceptance. This page describes the release requirements; it is not a working installation procedure.

vmcli is the command-line client for CaaS virtual machines. The public repository, [servas-ai/vmcli-public](https://github.com/servas-ai/vmcli-public), is intended for customer documentation, skills, public release keys and, after release, downloadable binaries. It does not distribute product source code.

## Planned platforms

| Client platform | Planned asset | Status |
|---|---|---|
| Linux x86-64 | `vmcli-linux-amd64` | Planned; no public release |
| Linux ARM64 | `vmcli-linux-arm64` | Planned; no public release |
| macOS Intel | `vmcli-darwin-amd64` | Planned; no public release |
| macOS Apple silicon | `vmcli-darwin-arm64` | Planned; no public release |
| Windows x86-64 | `vmcli-windows-amd64.exe` | Planned; no public release |

The initial customer VM scope is Linux. A future client platform release does not imply support for Windows guest VMs.

The existing Linux guest workflow uses OpenSSH and rsync 3.2.3 or newer on the client. The default rsync supplied with macOS may be too old. The public release must confirm exact operating system requirements and verification tools for each supported platform.

## Planned download route

The supported distribution path will use the terminal: a shell installer on Linux/macOS and a PowerShell installer on Windows. Installers will download a selected version from [public releases](https://github.com/servas-ai/vmcli-public/releases) over HTTPS and verify its Ed25519 signature and SHA-256 hashes before installation. GitHub CLI (`gh`), access to private repositories and compilation are not part of the customer installation path.

Homebrew (macOS/Linux), Scoop and winget (Windows) are also planned. Package definitions and exact commands will be published only after their verification and platform acceptance. No working installer or package-manager command is advertised in this preview.

No Windows Authenticode certificate or Apple Developer ID is planned. macOS Apple silicon binaries require the Go toolchain’s ad-hoc signature. Terminal installation does not guarantee execution on managed devices: AppLocker, WDAC, Smart App Control and other organization policies can still prevent unsigned applications from running. Platform behavior must be checked during acceptance.

No release version or executable download command is provided here because no public version has been published. Wait for a release whose notes explicitly approve your platform and the public CaaS connection path.

## Signed assets and verification

The release must provide the selected platform binary, `SHA256SUMS`, `SHA256SUMS.sig`, and any separately downloadable skill. Every executable or skill to be installed must have an entry in the signed checksum manifest. The [public release key](../keys/README.md) is included in this preview. Its trust must be established before installation is enabled.

The signature format is:

| File | Format |
|---|---|
| Public release key | PEM SubjectPublicKeyInfo in `keys/vmcli-release-ed25519.pem`; the corresponding raw 32-byte key is also listed as Base64 |
| `SHA256SUMS` | SHA-256 manifest containing asset filenames and hexadecimal hashes |
| `SHA256SUMS.sig` | Standard Base64 encoding of a raw 64-byte Ed25519 signature over the exact bytes of `SHA256SUMS` |

`SHA256SUMS.sig` is not an OpenPGP or Minisign signature. Do not change the manifest's line endings or whitespace before checking it.

The approved installation procedure must perform these steps in order:

1. Select an explicitly published release and download its assets over HTTPS without running them.
2. Establish trust in the public release key through the published authenticated key and fingerprint. A key downloaded alongside an untrusted binary is not sufficient by itself.
3. Decode the public key and signature as specified above and verify the signature over the unchanged manifest using a trusted verifier that already exists on the client.
4. Require an exact manifest entry for each selected binary and skill; calculate and compare every selected asset's SHA-256 hash. A missing entry or any mismatch is a failure.
5. Only after both signature and hash verification succeed, install the binary in a user-owned directory on `PATH` and install any selected skill.
6. Run the verified binary's version command, complete the released sign-in procedure and confirm identity and connectivity before creating a VM.

Do not execute a downloaded binary to verify itself before its authenticity has been established. Stop if the key, signature, manifest entry or checksum cannot be verified. The release must supply exact platform-specific verification instructions before these steps can become an installation guide.

## Release prerequisites

Public installation remains blocked until all of the following are complete:

- A public version, complete asset inventory and release notes are published, with successful runtime acceptance for every advertised client platform.
- The public key, fingerprint and rotation policy are published; signatures, exact manifest coverage and platform verification instructions are checked against the actual published assets.
- The public API target `https://api.caas.servas.ai` has approved DNS, TLS, authentication, tenant authorization and lifecycle acceptance from an ordinary customer network.
- The public SSH and file-transfer route has approved routing, access controls and host-key verification. Its actual public hostname and port must be documented after acceptance.
- Zitadel sign-in through `https://auth.servas.ai` is implemented and accepted for the customer workflow, including headless use, expiry and logout. Device authorization flow and `vmcli env` are still planned.
- Customer access works without VPN, Tailscale, API keys, private repository permissions or undocumented local defaults.
- Linux guest provisioning, execution, transfer and deletion have passed end-to-end acceptance, including quota enforcement and cleanup after failed operations.
- Public configuration defaults and support instructions are reviewed; update delivery and rollback are either accepted and documented or explicitly unavailable in the release notes.

See [configuration](configuration.md) for the intended connection model and [usage](usage.md) for the proposed workflow after release.
