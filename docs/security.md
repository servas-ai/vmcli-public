# Security — preview and release requirements

**Public vmcli use is blocked.** No official public vmcli release or end-to-end accepted public API/SSH path is available. This document states the intended design and mandatory acceptance criteria; it is not evidence that those protections have been implemented or verified. Do not use preview binaries or private connection paths for customer workloads.

## Identity and public access

The intended sign-in authority is Zitadel at `https://auth.servas.ai`. A customer identity needs the assigned customer role and approved configuration for `https://api.caas.servas.ai`. The target customer experience requires no VPN, API key, private repository access or GitHub credentials. Public release downloads from [servas-ai/vmcli-public](https://github.com/servas-ai/vmcli-public) should require no GitHub account.

Public environment selection, device sign-in and the SSH/tunnel connection procedure remain planned. Their syntax and security behavior must be published and accepted before use. A preview command or configuration field does not establish public support. The first release scope is Linux customer VMs; client platforms must be explicitly accepted in release notes. Windows clients and Windows customer VMs are not approved.

## Mandatory public release gate

An official signed release and documented end-to-end acceptance are both required. Acceptance must identify the tested release, supported Linux platforms, configuration and public API/SSH path, and demonstrate at least:

| Boundary | Required acceptance result |
|---|---|
| Missing authentication | Protected API requests without a token return HTTP 401. |
| Administrative access | Customer credentials receive HTTP 403 for administrative routes and operations. |
| Tenant separation | A customer's requests for another tenant's resources and operations return HTTP 403. |
| VM and network separation | Customer guests and connection paths cannot access internal networks, infrastructure hosts or other customers' VMs; unauthorized access to other VMs is denied. |
| Public lifecycle | A customer can sign in, create a Linux VM, connect over the approved public route, transfer permitted files, execute a command, retrieve output and delete only that VM. |
| Transport identity | The accepted connection procedure validates TLS certificates and SSH endpoint/host identities and fails on identity mismatch. |

These are **unverified requirements**, not claims about the present service. Passing a local test, seeing a CLI command in help or reaching the public API is insufficient. Until the published acceptance covers all required boundaries and the intended release, agents must stop.

## Release authenticity

The planned official distribution publishes binaries with `SHA256SUMS` and `SHA256SUMS.sig`. The signature format is a **Base64 encoding of the raw Ed25519 signature over the exact bytes of `SHA256SUMS`**. It is not an ASCII-armored signature, a signature over the filename, or a signature over a recomputed checksum-list digest. Do not normalize line endings or otherwise rewrite the checksum file before verification.

The public verification key is [`keys/vmcli-release-ed25519.pem`](../keys/vmcli-release-ed25519.pem). Its format and a future signature verification command are documented in [keys/README.md](../keys/README.md). The key is available now, but there are no public release assets to verify or install yet. Verify the checksum-list signature with an independently authenticated release key, then verify the selected binary against its entry in that signed list before execution. Missing files, an untrusted key or a failed verification must stop installation. A checksum without an authenticated signature proves no publisher identity.

This preview makes no claim that a public self-update path is available or that an existing binary enforces the future public release policy.

## Customer and agent responsibilities

- Keep authentication state and private keys on the authorized client. Do not share identities or copy token stores between machines.
- Never transfer secrets to a guest: tokens, passwords, API keys, SSH private keys, credential stores, `.env` files with credentials, Git credentials and secrets in repository history all count. Do not forward SSH agents or credential sockets.
- Review the entire transfer root before `push`; it can include `.git`. Ignore rules and a dry run are aids, not a security boundary. Use a clean staging project when required.
- Do not place secrets in command arguments, guest environment variables, stdin, diagnostics, screenshots or artifacts. Redact tokens and sensitive metadata before sharing logs.
- Use a fresh task-specific VM name. Verify creation and ownership, then remove only the VM this task created with `vmcli down NAME`. Do not use `--all-mine` or label-based cleanup. An identity or tenant shared by agents does not grant permission to delete their work.
- Treat guest code, command output and retrieved artifacts as untrusted. A disposable guest does not by itself prove isolation or make credential transfer safe.
- Do not bypass TLS or SSH verification, substitute internal endpoints or use infrastructure credentials to work around a public-path failure.

## Reporting a concern

Report suspected vulnerabilities privately using the contact options on [servas.ai](https://servas.ai). Include the release version, Linux platform, affected operation and a redacted reproduction. Do not publish tokens, keys, customer data or exploitable infrastructure details in public issues.
