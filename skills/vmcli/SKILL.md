---
name: vmcli
description: Use the public servas.ai vmcli workflow for an authorized disposable Linux VM task; enforce its preview release gate before installation, sign-in or VM operations. Use for vmcli or requested servas.ai cloud VM work, not unrelated local VM management.
---

# vmcli — public preview

**Stop before installation or execution.** There is no official public vmcli release and no end-to-end accepted public API/SSH path yet. Report that public use is blocked. Continue only when published evidence covers an official signed release, the Linux client and guest, and the full intended public connection path. Do not treat this preview as permission to provision a VM.

The intended distribution is https://github.com/servas-ai/vmcli-public. Customers should need only their Zitadel identity at `https://auth.servas.ai`, an assigned customer role and approved configuration for `https://api.caas.servas.ai`: no VPN, API key, GitHub account or private repository access. Configuration, device sign-in and SSH/tunnel commands are still planned; do not invent them or borrow another product's syntax. Windows clients and Windows customer VMs are not approved.

Once the gate is satisfied:

1. Follow that release's signature verification and setup instructions. A release checksum list must be authenticated before checking and running the binary. The public key is in `keys/vmcli-release-ed25519.pem`; authenticate it independently as described in `keys/README.md`.
2. Check `vmcli version --json` and `vmcli whoami --json`. Confirm the expected identity, tenant and quota. Use only version-documented commands and flags (`vmcli help COMMAND`).
3. Choose a fresh name unique to this task. Use `vmcli up NAME --size s --ttl 1h --task "purpose" --json`. Check the exit status and JSON. Proceed only on success with `created: true`; if `created: false`, stop without using or deleting the existing VM.
4. Transfer only secret-free authorized files with `vmcli push NAME SRC --json`. It finds the Git root, can include `.git` and remembers the remote directory. Review the transfer root and history first; ignore rules are insufficient to prevent secret transfer. Use `remote_dir` from the result for later artifact paths.
5. Execute with `vmcli exec NAME --json -- 'COMMAND'`. Linux uses `bash -lc`; after `push`, execution from the same local project uses the mapped guest directory. Retrieve reviewed output with `vmcli pull NAME REMOTE_PATH DEST --json`.
6. Always clean up this task's confirmed VM using `vmcli down NAME --json`, including on errors. Never use `--all-mine`, label-based cleanup or another task's VM. If creation failed ambiguously, establish exact ownership before deleting; report unresolved cleanup. Do not create durable VMs in this workflow.

Use `--json` for results and preserve the process exit status. Normal results and CLI error envelopes are one JSON document on stdout; progress is on stderr; help stays text. Errors contain `error.code`, `message`, `hint`, `retryable` and `exit`. `exec --json` may return `exit_code`, captured `stdout`/`stderr` or output-file paths even when the remote command fails. Remote exit codes pass through, except 255 maps to 125; 124/125/130 denote execution timeout, SSH/vmcli failure and interruption. A vmcli-side exec failure can exit 125 while retaining an underlying error code such as `not_found`. Inspect the document before classifying failure.

Top-level exits: 1 internal; 2 usage; 3 unauthenticated; 4 not_found; 5 conflict; 6 capacity; 7 unreachable; 8 timeout; 9 wrong_state; 10 lease_limit; 11 update_available (`update --check`); 12 update_required; 13 template_not_found. On authentication or connectivity failure, stop and follow the released public instructions. Never switch to private endpoints or credentials. On quota failure, clean up only this task's confirmed unused VM.

Never send secrets to a VM through transfers, `.git`, environment variables, arguments or stdin. Never forward an SSH agent or credential socket. Keep tokens, passwords, API keys, private keys and credential stores on the client. Treat guest output and pulled artifacts as untrusted.

The required security gate includes HTTP 401 without a token, HTTP 403 for customer access to administrative or other-tenant operations, and denied access to internal networks, hosts and other VMs. These remain unverified requirements, not established service properties. Read the release's public agent and security documentation before use.
