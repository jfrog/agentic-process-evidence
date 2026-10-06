# Testigo to APE mapping

Testigo side: [SPEC.md](https://github.com/cyl-castillo/testigo/blob/main/SPEC.md) v0.2. A longer, annotated version of this mapping lives in [docs/ape-mapping.md](https://github.com/cyl-castillo/testigo/blob/main/docs/ape-mapping.md).

## Model

| APE | Testigo | Notes |
|---|---|---|
| Process | Case (`caseId`) | A case groups the turns of one intent thread: `jira:KEY`, `github:org/repo#N`, or a terminal. APE's rule that the process outcome is the subject is applied when the process evidence is built. |
| Session | Session (`sessionId`) + turn (`turnId`) | A turn runs from a prompt to the stop. It is the unit that carries the working-tree diff. |
| Session log | Ledger segment exported as a proof packet | The packet is a DSSE-signed in-toto Statement. Its predicate carries the hash-chained events. |
| Process evidence | Not produced by Testigo | Testigo does not aggregate sessions. An APE runtime tool consumes packets as its session logs. |
| Agent runtime tool | Producer (agent-console, testigo-cli, Claude Code plugin) | Producers capture locally and sign at export. Uploading the packet and pushing the evidence is the runtime tool's job. |

```
APE process evidence  (subject = commit / artifact / release / session log)
  └─ sessionsLogs[]  ──uri + digest──▶  Testigo proof packet  (DSSE-signed in-toto Statement)
                                          └─ predicate.events[] = hash-chained ledger segment
```

## Process-evidence fields

How an APE runtime tool fills the process evidence from a verified packet. This is what [`examples/ape/build.mjs`](https://github.com/cyl-castillo/testigo/blob/main/examples/ape/build.mjs) does.

| APE field | From the packet |
|---|---|
| `subject[]` | The packet file (`uri` + sha256 of the file bytes) when the process is one session. Otherwise the process outcome. |
| `sessionsLogs[]` | One entry per packet: `uri` + sha256 of the file bytes. `external_evidence` events in the packet (commitments to platform-held records such as a harness transcript) are also `uri + sha256` and map to further entries. |
| `providers[]` | `predicate.provider`, which already uses the APE `Provider` shape. `languageModels` comes from the `session_start` / `model_switch` events. Older packets only have `predicate.generator` (harness + version). |
| `traceId` | `predicate.caseId` |
| `tools[]` | Distinct `payload.tool` values across the events |
| `contextArtifacts[]` | `predicate.contextArtifacts`, which already uses the APE shape. It is derived from the instruction-file digests each `prompt` event records at capture time. |
| `custom.baseCommit` | `snapshot.payload.commitSha` (the working tree before the turn) |
| `custom.requirements[].issue` | `caseId` when it names a ticket (`jira:KEY`, `github:org/repo#N`) |
| `owner` | `predicate.owner`. The accountable key is the DSSE signer (`keyid` = sha256 of the public key), compared out-of-band. |
| `reviewers[]` | The humans behind the `approval_decision` events |
| `startTimestamp` / `endTimestamp` | `predicate.startTimestamp` / `endTimestamp`, which verifiers check against the first and last event |
| `result` | Not in the packet. The packet records what changed (`turn_end.filesChanged`, `preSha`, `postSha`), and the process verdict stays APE-level. |
| `custom.testigo` | What the packet proves, so a gate does not have to open it: `signerKeyId`, `segmentDigest`, `ledgerHead`, `redactedEntries`, `humanApprovalDecisions`, `turnDiff`. When packets are referenced through `sessionsLogs[]`, there is one record per packet, keyed by its digest. |

## Timeline kinds

| APE `timeline[].event` | Testigo `kind` | Actor | Notes |
|---|---|---|---|
| `beforeSubmitPrompt` | `prompt` | human | Opens a turn. Testigo adds `skill`, `cwd` and the instruction-file digests. |
| `preToolUse` | `approval_request` | agent | Different meaning: in Testigo the agent asked a human (tool + bounded input). |
| (none) | `approval_decision` | human | `decision` (`allow` / `deny` / `ask`), `reason`, `approvalId` |
| `postToolUse` | `tool_result` | agent | Bounded excerpt, `truncated` flag, `failed` when the engine reported a failure |
| `sessionEnd` | `turn_end` | agent | Closes a turn and carries `preSha`, `postSha`, `filesChanged[]` |
| (none) | `snapshot` | system | Working-tree checkpoint before the turn |
| (none) | `session_start` / `model_switch` | system | Harness and model identity |
| (none) | `external_evidence` | system | `uri + sha256` commitment to a record held elsewhere |
| (none) | `check_run` / `commit` | system | Outcome events: a check the agent ran and its result, a commit it made |

## Integrity and disclosure

Each event is one JSONL line with `seq`, `ts`, `caseId`, `turnId`, `kind`, `actor`, `payload`, `prevHash` and `hash`. `hash` is the sha256 of the line bytes with the hash emptied. `prevHash` is the previous line's hash. Lines are hashed byte for byte, so re-serialising a packet anywhere in a pipeline breaks verification instead of silently passing.

Redaction replaces an event's payload and keeps its `seq`, `prevHash` and `hash`. The predicate's `redactionCount` must equal the number of redacted entries. Events of other cases inside the exported range become stubs (hashes only), so the linkage still verifies without sharing them.
