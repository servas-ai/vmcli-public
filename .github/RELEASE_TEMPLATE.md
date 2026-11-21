# vmcli release notes checklist

Write a versioned public release note only after the listed checks pass. Replace
this checklist with concrete results; never publish empty fields or draft claims.

- Name the version and summarize the customer-visible change.
- List accepted client operating systems, architectures and exact asset names.
- Link installation, checksum/signature verification and the public key.
- Describe the released sign-in and public endpoint configuration commands.
- Describe the accepted guest SSH/transfer path without a VPN.
- Record successful external lifecycle and isolation acceptance.
- List limits, expiry behavior, known issues and migration/update instructions.
- Include signed build metadata without internal hosts, paths or credentials.
- Ship binaries and documentation only; no product source or signing secrets.

Do not claim automatic updates until the public update path has been accepted.
