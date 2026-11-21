# Troubleshooting preview

**There is no public vmcli release yet. Public API and SSH access are not approved.** An unavailable download, incomplete customer login or unreachable target endpoint is not evidence of a working public service. Check [release prerequisites](installation.md#release-prerequisites) before attempting the proposed workflow.

## Downloads and verification

| Symptom | Action |
|---|---|
| No public binary or version is listed | Wait for a published public release. Do not use private release links or build instructions as a customer installation path. |
| The expected platform is missing | Use only a platform explicitly approved by the release notes. Windows clients are not approved in this preview. |
| Signature verification fails, the key is untrusted, or the manifest is missing | Stop. Do not run or install the binary or skill. Recheck the release and authenticated public key; report a persistent failure. |
| An asset is absent from `SHA256SUMS`, or its hash differs | Stop. Download the assets from the same published release again and repeat signature and checksum verification. Do not bypass a mismatch. |
| macOS blocks a download | After a public release exists, follow its verified installation and platform acceptance instructions. This preview does not authorize bypassing platform security checks. |

Signature verification must use a trusted tool already on your machine; running a newly downloaded vmcli binary first defeats the initial authenticity check. See [the signature format](installation.md#signed-assets-and-verification).

## Sign-in and connectivity

Device authorization flow and `vmcli env` are planned. If an unreleased binary rejects environment commands, that does not mean your account needs a different permission or an API key. No public login repair command is approved yet.

The intended customer service uses Zitadel at `auth.servas.ai` and the API target `api.caas.servas.ai`, with no VPN, Tailscale or API key. The public guest SSH route remains pending. Do not invent a health URL, SSH hostname or port, disable TLS or host-key checks, or copy someone else's tokens to work around incomplete rollout.

After a public release has been approved, use its published sign-in instructions and its exact connectivity checks. Report a persistent service failure with the release version and a redacted diagnostic message.

## Diagnostics after release

The current implementation provides these commands. Use them only with a verified, approved binary and accepted public configuration:

```sh
vmcli version --json
vmcli whoami --json
vmcli doctor --json
vmcli ls --json
```

Review diagnostic output before sharing it. `whoami`, `doctor` and VM listings can contain account details, tenant metadata and connection information. Never include access tokens, private keys, credential files or URLs carrying credentials in an issue.

## Current lifecycle errors

This table describes implemented vmcli errors for the proposed post-release workflow. It does not promise that the public service is available. For foreground `exec`, vmcli-side errors can use exit `125` while preserving their specific error code in JSON; inspect both fields.

| Exit | Error code | Action after release |
|---|---|---|
| `1` | `internal` | Preserve the redacted message and run `doctor`; report a persistent failure. |
| `2` | `usage` | Read `vmcli help COMMAND`; do not guess flags. |
| `3` | `unauthenticated` | Follow the released Zitadel sign-in procedure for your own identity. |
| `4` | `not_found` | Check the exact name with `vmcli ls --json`. |
| `5` | `conflict` | Inspect the message for ownership or differing VM properties; use a new task name rather than deleting someone else's VM. |
| `6` | `capacity` | Inspect quota with `whoami`; delete only your own finished, explicitly named task VMs. |
| `7` | `unreachable` | Check the release's public connectivity instructions and your network; report persistent failure. |
| `8` | `timeout` | Inspect VM state and the returned hint before retrying; adjust a supported timeout only when appropriate. |
| `9` | `wrong_state` | Allow the in-progress operation to finish, then inspect and retry the same operation. |
| `10` | `lease_limit` | Request a duration within the released limit. |
| `11` | `update_available` | An update check found a newer version; follow the release's approved update procedure. |
| `12` | `update_required` | Follow the returned hint and the approved signed update procedure. |
| `13` | `template_not_found` | Inspect the templates offered by the released service. |
| `124` | `exec_timeout` | The executed command exceeded its time limit. |
| `125` | `exec_failed` or the underlying vmcli code | Inspect the JSON code and hint for SSH/vmcli failure; remote exit `255` also maps here. |
| `130` | `interrupted` | The operation was interrupted; inspect VM state and clean up your task VM when finished. |

Other foreground `exec` exit statuses normally come from the guest command. A nonzero test or build result alone does not mean provisioning failed.

## Transfers and cleanup after release

If a Linux transfer fails because rsync lacks `--mkpath`, check that the client has rsync 3.2.3 or newer; macOS may select an older system copy. Follow the released platform instructions and use `doctor` to confirm the selected tool.

If `exec` starts in the wrong directory, run it from the same local project directory used for `push`, or set a deliberate `--cwd`. Use the `remote_dir` from `push --json` when choosing a `pull` source.

Retrieve needed results before running `vmcli down NAME`: it deletes the VM and its disks. Repeat cleanup with the exact task name if necessary; an already missing VM succeeds. Never use broad deletion to troubleshoot quota or connectivity.

## Reporting a problem

After a public release, open an issue in [vmcli-public](https://github.com/servas-ai/vmcli-public/issues) with the release version, client OS/architecture, exact command with credentials removed, exit status, error code and a redacted message. Describe whether the failure happened during verification, sign-in, provisioning, execution, transfer or cleanup. Public issues must not contain secrets or full authentication/configuration directories.
