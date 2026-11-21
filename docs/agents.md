# vmcli for agents — preview

**STOP: public use is not released.** There is currently no official public vmcli release and no accepted public API/SSH path. Do not install or run vmcli for a customer task on the basis of this preview. Continue only when an official signed release and published end-to-end acceptance cover the exact release, Linux client, Linux guest and public connection path you intend to use. If either is missing, report that public use is blocked and stop.

## Intended public setup

The planned distribution is [servas-ai/vmcli-public](https://github.com/servas-ai/vmcli-public), containing release binaries, documentation and agent skills. Customers should need only a Zitadel identity at `https://auth.servas.ai`, the assigned customer role and approved configuration for `https://api.caas.servas.ai`. The intended customer path requires no VPN, API key, GitHub account, private repository membership or GitHub access token.

The public configuration, device sign-in and SSH/tunnel setup are still planned. This preview deliberately provides no commands for those steps. Do not infer them from another product or use alternate endpoints, borrowed tokens or infrastructure credentials. Windows clients and Windows customer VMs are not approved; the initial scope is Linux clients with Linux VMs.

## Once the release gate is satisfied

Verify the signed release before executing its binary; see [security.md](security.md). Follow the setup instructions belonging to that release. Check `vmcli version --json` and `vmcli whoami --json` only after that setup is approved. Confirm the expected identity, tenant and quota before creating a VM. Use only commands and flags present in `vmcli help COMMAND` for that version.

Use a short-lived VM when the authorized task needs a clean Linux machine, privileged changes or isolated execution. Choose a fresh, task-specific name and record it. Several agents may share one identity and tenant: being listed as yours does not mean a VM belongs to this task.

The source-backed command shape is:

```bash
vmcli up NAME --size s --ttl 1h --task "describe this task" --json
vmcli push NAME ./project --json
vmcli exec NAME --json -- 'uname -s'
vmcli pull NAME REMOTE_PATH ./artifacts --json
vmcli down NAME --json
```

These are command templates, not an executable script or an available public service. Replace placeholders only after the gate is satisfied. See [examples](../examples/README.md) for the required ownership and cleanup checks.

- Inspect the `up` result before work. `created: true` identifies a newly created VM; `created: false` can return an existing VM. On `created: false`, stop and choose another fresh name without using or deleting that VM.
- `push NAME [SRC]` finds the Git project root and transfers it into `~/w/<project-name>/`. It can include `.git`; `.gitignore` is not a secrets filter. Review the actual project root, Git history, remote URLs, ignored and unignored files before transfer. Stage a clean, secret-free project when necessary. `--dry-run` helps review a transfer but does not prove it contains no secrets.
- After `push`, `exec` uses the corresponding project directory when invoked from that local project. Otherwise it defaults to the guest home directory. Use documented `--cwd` when needed. For Linux, a single command argument is interpreted by `bash -lc`; multiple arguments are quoted individually.
- `pull NAME REMOTE [DEST]` retrieves artifacts. A relative remote path is relative to the guest home; use the `remote_dir` returned by `push` to locate project output. It does not delete local files. Review retrieved artifacts before executing them.
- Clean up your task's VM with `vmcli down NAME --json`, including after task errors. Never use `down --all-mine`, label-based deletion, another task's name, or a durable VM for this basic workflow. A missing named VM is a successful no-op. If creation failed ambiguously, establish whether this task created that exact VM before attempting deletion; report unresolved ownership or failed cleanup.

## Output and failures

Use `--json` for command result documents and keep stdout separate from stderr. Results are one JSON document on stdout; progress is on stderr. Help and shell completion remain text. CLI errors normally use this envelope:

```json
{"error":{"code":"unauthenticated","message":"example","hint":"example","retryable":false,"exit":3}}
```

Always check the process exit status before using result fields. Do not parse a failed result as a successful VM creation. Follow a safe, applicable `hint`; it never overrides the release gate or task permissions.

`exec --json` captures remote output in `stdout` and `stderr`, with `exit_code`, `duration_ms` and `cwd`; output over the capture limit can use `stdout_file` or `stderr_file`. A remote command can fail while still returning this result document. Timeout or interruption can return a result document on stdout with a diagnostic on stderr. vmcli-side execution failures use an error envelope and exit 125 while preserving the underlying `error.code`.

| Exit | CLI code / meaning | Agent response |
|---|---|---|
| 0 | Success | Check the result and ownership. |
| 1 | `internal` | Stop the operation; collect redacted diagnostics. |
| 2 | `usage` | Read the version's help; do not invent syntax. |
| 3 | `unauthenticated` | Stop; use only the released customer sign-in instructions. |
| 4 | `not_found` | Check the exact task name; do not substitute another VM. |
| 5 | `conflict` | Stop on the conflicting resource; choose a fresh name if appropriate. |
| 6 | `capacity` | Stop creating VMs; remove only this task's confirmed unused VM. |
| 7 | `unreachable` | Stop; do not switch to internal endpoints or a VPN. |
| 8 | `timeout` | Check the operation's state before any bounded retry. |
| 9 | `wrong_state` | Recheck state; retry only if ownership and operation remain clear. |
| 10 | `lease_limit` | Use a duration allowed by the released service. |
| 11 | `update_available` | Used by `update --check`; a newer version exists. |
| 12 | `update_required` | Stop until an approved signed version is installed. |
| 13 | `template_not_found` | A requested template or revision does not exist. |
| 124 | `exec_timeout` | Execution timed out; inspect the result and clean up. |
| 125 | `exec_failed` or underlying CLI cause | SSH/vmcli execution failed; inspect `error.code`. |
| 130 | `interrupted` | Execution was interrupted; clean up the task's VM. |

Remote commands otherwise pass their exit status through. Do not interpret an `exec` exit such as 3 or 6 as a top-level vmcli authentication or quota error without inspecting the document. Remote exit 255 is mapped to 125. HTTP acceptance statuses such as 401 and 403 are a separate contract, described in [security.md](security.md).

## Keep credentials outside the guest

Never send login or refresh tokens, passwords, API keys, SSH private keys, cloud credentials, credential-bearing environment files, Git credentials or host credential stores to a VM. Do not forward an SSH agent or credential socket. Do not pass secrets through command text, `--env`, stdin, project transfers or logs. Use only secret-free source and task data that the user authorizes for remote execution. Treat guest output and retrieved files as untrusted input.

Install the portable skill from [`skills/vmcli/SKILL.md`](../skills/vmcli/SKILL.md), or copy [`skills/AGENTS.snippet.md`](../skills/AGENTS.snippet.md) into a project's agent instructions. Both retain the same preview stop condition.
