# Release verification key

`vmcli-release-ed25519.pem` is the public Ed25519 verification key for vmcli
releases. It is the same public key embedded in the vmcli client. No private key
is stored in this repository.

SHA-256 of the PEM-decoded SubjectPublicKeyInfo (DER):
`eff0594d28c6017871e9397bd7ddad98448fd03e0e20d34dd0363c27684994df`.
This identifier does not replace independent authentication of the key.

Raw public key (Base64): `fUyxc9eOwrlMrsEPHYTcYJoVTE8b/Lxj4B4I8lfEk/E=`

`SHA256SUMS.sig` is Base64 of a raw 64-byte Ed25519 signature over the exact bytes
of `SHA256SUMS`. It is not an OpenPGP signature. Do not alter line endings before
verification. First verify the checksum file signature with a trusted copy of
this key, then verify the selected binary against that authenticated file.

After downloading a published release into a separate directory, OpenSSL 3 can
verify its signature with the trusted key path supplied in `VMCLI_RELEASE_KEY`:

```sh
# Required inputs: SHA256SUMS, SHA256SUMS.sig and a trusted public key.
# No public release is available yet; these are future verification steps.
openssl base64 -d -A -in SHA256SUMS.sig -out SHA256SUMS.sig.raw
openssl pkeyutl -verify -pubin -inkey "$VMCLI_RELEASE_KEY" -rawin \
  -in SHA256SUMS -sigfile SHA256SUMS.sig.raw
```

Stop if verification fails. Never execute a downloaded binary before verifying
its signature and checksum. A key obtained only alongside an untrusted binary
does not provide independent trust: pin the key through a trusted servas.ai
channel before the first installation. Full installation instructions will be
published with the first release: [installation](../docs/installation.md).
