# Usage preview

**No public vmcli release is available yet, and public API/SSH access is not approved.** These examples describe the planned customer workflow after release. Their lifecycle command syntax has been checked against the current implementation; they do not establish public service availability.

Before running them, use an approved public release, complete its published configuration and Zitadel sign-in instructions, and confirm that the release supports your client platform. Device-flow sign-in and `vmcli env` are still planned, so no executable setup recipe is provided here. See [installation](installation.md) and [configuration](configuration.md).

The initial customer workflow creates Linux guest VMs. Windows clients and Windows guest workflows are not approved in this preview.

## Planned lifecycle after release

Check the signed-in identity and tenant quota:

```sh
vmcli whoami --json
```

Choose a name for this task, then request a short-lived Linux VM:

```sh
vmcli up task-example --os linux --size s --ttl 30m --task "example task" --json
```

The implemented `up` command waits for readiness by default. Repeating the same name with matching OS, template, revision and size reuses the existing VM; changing those properties produces a conflict. Use a name assigned to your task and inspect the result before working on an existing VM.

From a Git project directory, transfer the project:

```sh
vmcli push task-example --json
```

The project goes to `~/w/<project-directory-name>/` on the guest. The current command uses the Git root, honors `.gitignore`, includes `.git`, and excludes common dependency and cache directories. Review the files first: `.gitignore` is not a guarantee that credentials are absent. Do not transfer secrets or credential files to the VM. A preview of the transfer is available with `vmcli push task-example --dry-run --json`.

Run a command from the same local project directory:

```sh
vmcli exec task-example -- 'mkdir -p out && uname -s > out/os.txt'
```

`exec` uses the remembered project mapping from `push`. Without a matching mapping, it starts in the guest user's home directory; `--cwd` can select a working directory explicitly. On Linux, commands run with `bash -lc`. A single quoted argument is a shell script; multiple arguments are individually quoted. The separator `--` is required.

Retrieve the result. This example assumes the Git root directory is named `example-project`; replace that component with your actual directory name, or use the `remote_dir` returned by `push --json`:

```sh
vmcli pull task-example w/example-project/out/ ./out --json
```

`pull NAME REMOTE [DEST]` accepts a remote path relative to the guest home directory or an absolute path. The default destination is the current local directory. A trailing `/` on the remote path copies its contents. It never deletes local files, but existing destination files can be overwritten.

Delete your task VM when finished, including after an error:

```sh
vmcli down task-example --json
vmcli ls --json
```

`down` deletes the VM and its disks. Copy out anything you need first. A missing name is a successful no-op. Delete only the explicitly named VMs for your task; shared identities can own other people's or agents' work. Avoid broad cleanup selectors.

## Command syntax

The following syntax is implemented today and proposed for the public release:

| Command | Purpose |
|---|---|
| `vmcli whoami [--json]` | Show identity, tenant, quota, usage and authentication expiry |
| `vmcli up [NAME] --size SIZE --ttl DURATION --task TEXT [--json]` | Create or reuse a lease; Linux is the default OS |
| `vmcli exec NAME [--cwd DIR] [-e K=V] [--timeout DURATION] -- COMMAND` | Execute a command on the guest |
| `vmcli push NAME [SRC] [--dry-run] [--json]` | Transfer a project; source defaults to the current project |
| `vmcli pull NAME REMOTE [DEST] [--dry-run] [--json]` | Retrieve files without deleting local files |
| `vmcli down NAME… [--json]` | Stop and delete the named task VMs |
| `vmcli ls [--json]` | Inspect your VMs and tenant usage |

Use the released binary's `vmcli help COMMAND` for its exact flags. Do not assume commands from another servas.ai CLI also exist in vmcli.

The current lease defaults are four hours with a maximum of 72 hours; the release and tenant policy must confirm the available limits. Treat the `expires_at` value returned by the service as the lease end. Export results before expiry and explicitly delete a finished task VM.

## Output and automation

`--json` requests machine-readable command results on stdout; progress remains on stderr. Help and completion remain text. Ordinary vmcli failures use an `error` object with `code`, `message`, `hint`, `retryable` and `exit`. Check the process exit status and follow the returned hint.

For foreground `exec`, remote command exits normally pass through. Reserved statuses are `124` for an execution timeout, `125` for an SSH/vmcli failure (also remote exit `255`), and `130` for interruption. Usage errors use `2`. With `--json`, execution results contain the remote exit code and captured output; large output can be written to local files referenced by the result.

See [troubleshooting](troubleshooting.md) for the current error mapping and what to include in a support report.
