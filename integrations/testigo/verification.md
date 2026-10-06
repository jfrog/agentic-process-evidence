# Verifying a referenced packet

A consumer that only matches digests treats a packet like any other session log: fetch the bytes at `uri` and compare their sha256 with `digest.sha256`. The steps below are what a gate or an auditor can do on top of that once it recognises the bytes as a Testigo packet. A plain timeline has nothing to verify beyond its digest.

## Steps

Steps 1 to 6 follow the verification algorithm in Testigo [SPEC §2.4](https://github.com/cyl-castillo/testigo/blob/main/SPEC.md). A packet is valid when all of them pass.

0. **Digest.** The sha256 of the fetched file equals the `digest.sha256` in `subject[]` or in the `sessionsLogs[]` entry.
1. **Format.** The packet's `format` is a known Testigo packet format (`testigo-proofpack/v0.1`).
2. **Signer.** The sha256 of the embedded `publicKey` equals `signatures[0].keyid`. For a gate, that key id must also be on the allow-list of operator keys. The allow-list is the trust anchor and is compared out-of-band.
3. **Signature.** The Ed25519 signature verifies over `PAE(payloadType, payload)` (DSSE).
4. **Subject.** The Statement's subject digest equals the sha256 over the compact serialization of `predicate.events`.
5. **Linkage.** Starting from `range.prevHashBefore`, every entry's `prevHash` (full, redacted or stub) equals the previous entry's `hash`.
6. **Content.** The content hash recomputes for every full, non-redacted entry, and `redactionCount` equals the number of redacted entries. Redacted and stub entries are reported visibly, never silently passed.
7. **Process context.** If `provider`, `contextArtifacts`, `owner` or the timestamps are present, they are well-formed, and `startTimestamp` / `endTimestamp` equal the `ts` of the first and last non-stub entries.

If the packet carries an RFC 3161 timestamp over its signature, the verifier reports it. It is informative unless the verifier also validates the token and the TSA certificate chain.

Then cross-check against the process evidence. The values under `custom.testigo` (`signerKeyId`, `segmentDigest`, `redactedEntries`, `humanApprovalDecisions`, `turnDiff`) must match what the verified packet shows. They are a convenience for gates, not a source of truth.

## Gates a verified packet can answer

In addition to the gates in the use cases:

- Every tool call the agent made was approved by a named human (one `approval_decision` per `approval_request`), and the decisions carry reasons.
- The turn changed nothing (`turn_end.filesChanged: []`), or changed exactly the files the ticket allowed.
- Which events were withheld from this receiver, stated explicitly.

## Tools

- [Hosted verifier](https://cyl-castillo.github.io/testigo/verifier/testigo-verifier.html): a single HTML file that runs offline. Drop the packet on it.
- [`conformance/verify.mjs`](https://github.com/cyl-castillo/testigo/tree/main/conformance): a zero-dependency Node verifier.
- [Conformance corpus](https://github.com/cyl-castillo/testigo/tree/main/conformance): valid and invalid vectors, including valid signatures wrapped around internal defects.

## Checking the example

```sh
curl -sO https://cyl-castillo.github.io/testigo/examples/fixy-deploy-verification.proofpack.json
sha256sum fixy-deploy-verification.proofpack.json
# 2c9d7f01ea070530b6bb25084d3e274a197c3e4d14508cb69281ab732145b23b
```

The hosted file is byte-identical to [`objects_examples/testigo-session-log.proofpack.json`](./objects_examples/testigo-session-log.proofpack.json). The verifiers report it as valid: signature valid for key `8caf0907…`, subject digest matches, linkage intact across 12 entries, content hashes recomputed for the 5 entries that are not redacted, and 7 redacted entries reported as such.
