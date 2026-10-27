# Nimbus trust feed

This repository is the Nimbus trust feed's source. Every Nimbus install fetches the signed
`trust.json` it produces (from the `gh-pages` branch) and applies it: key revocations and
extension takedowns block extensions everywhere, and verified publishers get their badge.

`main` holds the SOURCE, never a signed document:

- `publishers/<publisherId>.json` — one per verified publisher: `{ "publisherId", "keys", "github" }`.
  This is the feed's current verification state. Removing the file un-verifies the publisher.
- `requests/<name>.json` — a block signed with the Nimbus trust root: `key.revoke` or
  `extension.takedown` (which must name `publisherId`). A block is permanent: deleting the
  file later removes nothing.
- `statements/<statement id>.json` — a verified publisher's own signed rotation or revocation.

Write them with the CLI, then open a pull request:

```bash
nimbus trust maintain verify-publisher . <publisherId> --key <public key file|base64> --github <handle>
nimbus trust maintain unverify-publisher . <publisherId>
nimbus trust maintain revoke-key . <fingerprint> --publisher <id> --reason <text>
nimbus trust maintain takedown . <id>[@<version>] --publisher <id> --reason <text>
nimbus trust maintain add-statement . <statement file>
nimbus trust maintain plan .
```

`verify-publisher` records that you checked the publisher's GitHub identity; it does not check it.

The **Trust feed check** workflow summarises what the merged result would sign. It is a reviewer aid,
not a guarantee: a pull request may only change `publishers/`, `requests/` and `statements/`,
and one that edits a workflow fails the check — but it could also edit the check itself. The
boundary is merging into `main` and the **Trust feed sign** workflow's required
approval: whoever can do both decides what every Nimbus install trusts.
