# Public release notes

There are no public vmcli releases yet. The repository is a documentation preview.

The first release must include platform-specific client binaries, `VERSION`,
`BUILDINFO`, `SKILL.md`, `SHA256SUMS` and `SHA256SUMS.sig`. Checksums must cover the
binaries and every other shipped payload, including build metadata and the skill.
The signature covers the exact checksum file. Only customer distribution assets
belong in a public release; no operator credentials or server components.

A release note must name the actually accepted client platforms, verification
steps, public login and endpoint configuration, guest connection method, known
limitations and upgrade instructions. Public access acceptance must cover the
complete install → sign-in → create → exec → push/pull → delete flow from a clean
machine without a VPN, plus tenant and network isolation checks.

Use the [release template](../.github/RELEASE_TEMPLATE.md) when publishing the first
release. A documentation preview is not a release candidate or a service launch.
