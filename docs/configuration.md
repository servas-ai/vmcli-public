# Configuration preview

**Public customer configuration is not available yet.** There is no public vmcli release, and the public API and SSH routes have not passed acceptance. Do not treat the intended addresses below as working connection instructions.

## Intended customer connection model

| Component | Planned destination or behavior | Status |
|---|---|---|
| CaaS control API | `https://api.caas.servas.ai` | Target; public acceptance outstanding |
| Sign-in | Zitadel through `https://auth.servas.ai` | Required customer identity path; vmcli customer flow pending |
| Headless sign-in | OIDC device authorization flow | Planned; not implemented as the customer flow |
| Named environments | `vmcli env` | Planned; not an available command |
| Guest SSH and file transfer | Authenticated public route with host-key verification | Hostname, port and customer setup not yet approved |

The customer path is intended to require no VPN, Tailscale or API key. Downloading the CLI must require no private repository access. Tenant access must be granted to your own authorized identity through Zitadel.

There is no approved environment or login command sequence to copy yet. Commands, configuration keys and browser or device-code prompts for that sequence must be published with the accepted release. Do not substitute another CLI's environment settings or configure an unreleased binary against the target API address.

## Local settings in the current implementation

The following settings exist in the implementation. They are reference information for the proposed release, not a way to enable public connectivity.

| Setting or command | Current behavior |
|---|---|
| `~/.config/vmcli/config.yaml` | Default local configuration file |
| `--config FILE` | Select an explicit configuration file |
| `vmcli config path --json` | Report the selected file path and whether it exists |
| `vmcli config get ttl.default --json` | Read the configured default lease duration and its source |
| `vmcli config set ttl.default 2h` | Store a local default lease duration |
| `VMCLI_JSON=1` | Request JSON command results |
| `VMCLI_STATE_DIR` | Override the local state directory |
| `VMCLI_NO_AUTO_UPDATE=1` | Disable automatic update checks in the current implementation |

For configuration values, the implementation loads built-in defaults, then the selected file, then matching `VMCLI_*` environment variables. A supported command-line override takes precedence for its option. Changing a file value does not override an environment variable.

The default state directory is `~/.local/state/vmcli`. State can include authentication material, SSH keys and project mappings; keep it private. Existing implementation paths are not a final specification of the future device-flow token store. Never copy authentication state between users or machines or publish full configuration/state directories in a support report.

The public release must provide reviewed public connection defaults. This preview deliberately provides no gateway, SSH jump, account or authentication-provider configuration recipe.

## Tenant and quota

After public sign-in is released, `vmcli whoami --json` is the implemented command for discovering your identity, tenant, limits, usage and authentication expiry. Tenant resources and authorization are assigned by the service. Editing local configuration cannot grant more quota or access to another tenant.

In the current lifecycle implementation, running VMs consume CPU and memory quota; disk and guest counts also include stopped VMs and saved templates. A public release must verify that these rules match the accepted service policy before advertising them as available.

## Updates

The implementation includes signed update verification, but the public update delivery path has not been accepted. Do not assume that automatic updates are enabled for customers. Public release notes must state the supported update route, signing-key policy and rollback behavior. Manual replacement must follow the same signature and checksum checks as [initial installation](installation.md#signed-assets-and-verification).
