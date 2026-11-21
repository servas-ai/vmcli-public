# Linux command examples — preview only

**Do not run these examples yet.** There is no official public vmcli release or end-to-end accepted public API/SSH path. These examples document minimal command syntax; they do not establish public availability. Use them only after a signed official release and published acceptance cover your Linux client, Linux guest and public connection path. Windows clients and Windows customer VMs are not approved.

Complete that release's verified installation and approved customer setup first. The intended sign-in authority is `https://auth.servas.ai` and target API is `https://api.caas.servas.ai`, with an assigned customer role. Environment selection, device sign-in and SSH/tunnel procedures are still planned and have no commands here. Do not use a VPN, API key, GitHub credentials or private infrastructure access to work around missing public setup.

These are separate commands to inspect step by step, not a script. Replace `task-a1b2c3` with a fresh name unique to your task. Check `vmcli help COMMAND` against the official release before using any command below.

## Confirm the approved client and identity

```bash
vmcli version --json
vmcli whoami --json
```

Stop unless the identity, tenant and quota match the intended authorized task.

## Create one short-lived Linux VM

```bash
vmcli up task-a1b2c3 --size s --ttl 1h --task "Linux command example" --json
```

Check the process exit and JSON. Proceed only with success and `created: true`; record the name for cleanup. On `created: false`, stop without using or deleting the existing VM and choose another name. On ambiguous failure, establish whether this task created that exact VM before cleanup.

## Run a command and retrieve its output

```bash
vmcli exec task-a1b2c3 --json -- 'mkdir -p /tmp/vmcli-example-output && uname -s > /tmp/vmcli-example-output/platform.txt'
vmcli pull task-a1b2c3 /tmp/vmcli-example-output/ ./artifacts --json
```

Inspect the `exec` process status and result, including `exit_code`, before continuing. The command is a single Linux shell argument interpreted by `bash -lc`. The trailing slash on the pull source copies the directory contents. Treat artifacts as untrusted.

## Optional project transfer

Use only a secret-free project that the user authorizes for remote execution. Replace `./project` with its reviewed source directory. `push` selects the Git root and can transfer `.git`; review history, remote URLs and the entire root first. Ignore files and dry-run output do not guarantee secrets are absent.

```bash
vmcli push task-a1b2c3 ./project --dry-run --json
vmcli push task-a1b2c3 ./project --json
```

Read `project_root` and `remote_dir` from the successful transfer result. When later invoked from that local project, `exec` uses its mapped guest directory. For artifact retrieval, pass the actual remote output path to `pull`; do not assume a project name.

## Always remove this task's VM

```bash
vmcli down task-a1b2c3 --json
```

Run this after success or failure only for the VM confirmed as created by this task. A missing named VM is a successful no-op. Never use `--all-mine` or label-based deletion. Report a cleanup failure rather than silently leaving a VM behind.

Output and exit details: [agent guide](../docs/agents.md). Release and isolation gate: [security](../docs/security.md).
