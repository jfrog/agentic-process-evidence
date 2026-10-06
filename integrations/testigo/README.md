# Testigo integration

[Testigo](https://github.com/cyl-castillo/testigo) produces **signed session logs**: an agent session captured as a hash-chained ledger and exported as a proof packet. The packet is an in-toto Statement in a DSSE envelope, and its predicate carries the session's events. In APE terms a packet is a session log whose bytes carry their own integrity and signature.

This directory shows how a Testigo packet plugs into APE. Nothing in the standard changes: the packet is referenced the same way as any other session log, with `uri + digest`.

## What a signed session log adds

- **Verifies without access to the log store.** Anyone holding the bytes (a gate, an auditor, a customer) can recompute the chain and check the signature.
- **Selective disclosure.** Events can be redacted before the log leaves the organisation. The receiver can still verify the linkage across the redaction and is told which events were withheld.
- **Human approval decisions, per action.** Each decision comes with a stated reason and is hash-chained between the agent's request and the tool result.
- **Addressable per event.** Alignment evidence or a reviewer can cite one approval or one tool call by its content hash.

## Two ways to reference a packet

### 1. Through `sessionsLogs[]`

This is the general case. A process aggregates one or more sessions, and its subject is the outcome (a commit, an artifact, a release). A Testigo packet is one entry of `sessionsLogs[]`, next to plain timelines if there are any:

```json
"sessionsLogs": [
  {
    "uri": "https://myorg.jfrog.io/artifactory/agentic-session-logs/a1b2c3d4-e5f6-7890-abcd-ef1234567890.json",
    "digest": { "sha256": "7c3a1f8e…" }
  },
  {
    "uri": "https://cyl-castillo.github.io/testigo/examples/fixy-deploy-verification.proofpack.json",
    "digest": { "sha256": "2c9d7f01ea070530b6bb25084d3e274a197c3e4d14508cb69281ab732145b23b" }
  }
]
```

`digest.sha256` is the sha256 of the packet file bytes. That is the same rule as for a plain timeline, so a consumer that only matches digests needs no changes. A consumer that recognises the packet can also verify what is inside it (see [verification.md](./verification.md)).

Example: [`process-evidence.packet-in-sessionsLogs.json`](./objects_examples/process-evidence.packet-in-sessionsLogs.json). The subject is a release commit. Two sessions contributed to it: a coding session stored as a plain timeline and a post-deploy verification session stored as a Testigo packet. The subject and the plain-timeline entry are placeholders, in the style of the use-case examples. The packet entry is the real packet in this directory. Under `custom.testigo` the evidence carries, for each referenced packet keyed by digest, what that packet proves, so a gate can read it without opening the file.

### 2. As the subject

When the process is one session, the session log is the subject. This is the [Customer success](../../use_cases/customer_success/overview.md) pattern, with a signed packet as the log. `sessionsLogs[]` is omitted.

Example: [`process-evidence.packet-as-subject.json`](./objects_examples/process-evidence.packet-as-subject.json), generated from the packet by [`examples/ape/build.mjs`](https://github.com/cyl-castillo/testigo/blob/main/examples/ape/build.mjs). The script verifies the packet first and derives every field from its bytes.

## Contents

| File | What it is |
|---|---|
| [mapping.md](./mapping.md) | Testigo to APE mapping: model, process-evidence fields, timeline kinds |
| [verification.md](./verification.md) | Steps a gate or auditor follows to verify a referenced packet |
| [objects_examples/testigo-session-log.proofpack.json](./objects_examples/testigo-session-log.proofpack.json) | The signed session log (real, from production, redacted) |
| [objects_examples/process-evidence.packet-in-sessionsLogs.json](./objects_examples/process-evidence.packet-in-sessionsLogs.json) | Process evidence that references the packet through `sessionsLogs[]` |
| [objects_examples/process-evidence.packet-as-subject.json](./objects_examples/process-evidence.packet-as-subject.json) | Process evidence whose subject is the packet |

Both process-evidence examples are **unsigned on purpose**. They show the shape and make no provenance claim of their own.

## The example packet

The packet is real: a read-only verification of the Fixy production deploy (2026-07-15), captured by [agent-console](https://github.com/cyl-castillo/agent-console) (the Testigo reference implementation) and re-exported with version 0.79.0.

- One operator prompt and three agent commands, each approved by a human. No files changed.
- 12 events, 7 of them redacted (the prompt's home path automatically, three agent requests and three tool results by hand). The chain is intact.
- File sha256 `2c9d7f01ea070530b6bb25084d3e274a197c3e4d14508cb69281ab732145b23b`
- Segment digest `af21a0b7bfe48022764831415f1c82f45211f0b886ad59bbe4a19842308e0ceb`
- Signer key id `8caf09075df11abbbdea5cd1d120a5654d8d1ce2a32e2a6f018dde557c5014da`

Check it with the [hosted verifier](https://cyl-castillo.github.io/testigo/verifier/testigo-verifier.html), which runs as a single HTML file with no network access.
