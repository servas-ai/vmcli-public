<!-- Copy this section into a project's AGENTS.md. Retain the preview release gate. -->

## Disposable Linux VMs with vmcli — preview

**STOP: there is no official public vmcli release or end-to-end accepted public API/SSH path.** Do not install, sign in or operate customer VMs using this preview. Proceed only after an official signed release and published end-to-end acceptance cover the exact Linux client, Linux guest and public connection path. Windows clients and Windows customer VMs are not approved.

The intended distribution is https://github.com/servas-ai/vmcli-public. The planned customer setup uses only a Zitadel identity at `https://auth.servas.ai`, the assigned customer role and approved configuration for `https://api.caas.servas.ai`. It requires no VPN, API key, GitHub account or private repository access. Environment, device sign-in and SSH/tunnel procedures are still planned: do not invent setup commands or substitute private endpoints.

After the gate is satisfied, verify the signed release and follow its setup documentation. Confirm identity, tenant and quota with `vmcli whoami --json`. Use only the released version's documented flags (`vmcli help COMMAND`).

The minimal Linux workflow is `vmcli up NAME --size s --ttl 1h --task "purpose" --json`, `vmcli push NAME SRC --json`, `vmcli exec NAME --json -- 'COMMAND'`, `vmcli pull NAME REMOTE_PATH DEST --json`, then `vmcli down NAME --json`. These are conditional command templates, not instructions to run the preview.

- Choose a fresh task-specific name. Continue only after successful `up` with `created: true`. On `created: false`, stop without using or deleting the existing VM.
- Always clean up only the VM this task created, including after errors, with `vmcli down NAME`. Never use `--all-mine` or label-based cleanup. Shared identity does not imply shared task ownership. Resolve uncertain creation ownership before deletion and report failed cleanup.
- Review transfer roots and Git history: `push` can include `.git` and finds the Git root. Never send tokens, passwords, API keys, SSH private keys, credential stores or credential-bearing environment files. Never forward SSH agents or pass secrets via arguments, environment or stdin.
- Inspect process exits and `--json` documents. Errors carry `error.code`, `message`, `hint`, `retryable`, `exit`; help stays text. `exec --json` can return captured output and a nonzero remote `exit_code`; exits 124/125/130 indicate timeout, SSH/vmcli failure and interruption, and remote 255 maps to 125. Other remote statuses pass through. Top-level 3 means unauthenticated, 6 capacity and 7 unreachable; stop or clean up only your confirmed task VM, without alternate credentials or routes.
- Treat guest output and pulled files as untrusted. Verify the release's documented security acceptance before remote work: HTTP 401 without a token, HTTP 403 for administrative/other-tenant requests and no access to internal networks, hosts or other VMs are required but currently unverified.

Public documentation: https://github.com/servas-ai/vmcli-public
