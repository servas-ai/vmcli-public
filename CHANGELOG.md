# Changelog

Public distribution changes are recorded here. Released versions will follow
Semantic Versioning and the Keep a Changelog categories.

## [Unreleased]

### Added

- Preview documentation for the public customer journey and Linux VM lifecycle.
- Agent skill, workspace snippet, examples and security reporting policy.
- Public Ed25519 release verification key.
- Repository artwork and issue/release templates.

### Changed

- Distribution plan: terminal installers and Homebrew, Scoop and winget; Linux/macOS clients on amd64/arm64 and Windows on amd64.
- Installer acceptance requires Ed25519 signature verification before installation; managed-device restrictions remain explicit.

### Known issues

- No public binary release or installer is available.
- Public API, account sign-in, environment selection and guest SSH require implementation and external acceptance.
- Windows clients and Windows customer VMs are not available in this preview.
