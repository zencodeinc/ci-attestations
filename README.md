# ci-attestations

Signed CI attestations for zencodeinc repositories.

Each file is an Ed25519-signed statement that the full CI manifest (`ci/manifest.yaml`) of one repository passed for one exact git tree. The repositories' own verifier checks these files. They contain no secrets: only step ids, exit codes, timings, log hashes, commit and tree ids, and a coarse host fingerprint.

- Signing key id: `cd5ce519f25dded1`
- Public key: [`ci-attest.pub`](ci-attest.pub) (PEM, Ed25519). The same PEM is the org Actions variable `CI_ATTEST_PUBKEY`.

## Layout

```
<repo-name>/<git-tree-sha>.json     one attestation per verified tree
revoked.json                        optional: {"jti": [...], "key_id": [...]}
ci-attest.pub                       trusted public key
```

An attestation is `{"payload": {...}, "sig": "<base64>", "key_id": "<16 hex>"}`. The payload carries the repo, the tree and commit, the manifest hash, per-step results, `issued_at`, `expires_at` (7 days) and a unique `jti`.

## Verify

From a checkout of the commit being checked, in a repo that ships `scripts/ci_attest_verify.py`:

```
python scripts/ci_attest_verify.py --repo zencodeinc/<repo-name> --pubkey ci-attest.pub \
  --attestation-url-base https://raw.githubusercontent.com/zencodeinc/ci-attestations/main
```

Exit 0 only if the signature matches the trusted key, the repo and tree match, the attestation is unexpired and not revoked, every step has exit code 0, and the manifest hash and step list match the tree's `ci/manifest.yaml`. Any problem exits 1 with one `FAIL:` line. Only Python and `openssl` are needed.

## Status

Shadow phase: verification reports a result but does not gate merges or deploys.
