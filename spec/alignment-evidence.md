# Alignment evidence

Evidence based on the agentic process that improves control over **intent alignment** to desired intent and corporate rules.

`predicateType`: `https://jfrog.com/evidence/agentic-alignment/v1`

JSON field names defined by this standard use **camelCase**. Shared predicate fields follow [agentic process evidence](./agentic-process-evidence.md) unless this page says otherwise.

## Generation process

The generation process can be implemented in different stages, few examples: commit, PR checks, artifact builds, or release promotion.
The subject of the evidence depends on the planned usage and can be set to the session log, commit, or other relevant entities which the session log(s) are relevant to. 
We do recommend that the alignment evidence subject be set to the session log artifact to simplify locating where a violation resides. In that case, each session log included in a commit will be evaluated against policy documents and generate an alignment evidence with the session log artifact as its subject.

On commit, the client-side agentic process:

1. Collects all session logs (same path that creates development process evidence)
2. Checks the customer intents-policy resource against the session logs
3. Emits evidence with verdict `ALIGNED` | `MISALIGNED` and a summary of problematic intents

## Example (session log as subject)

The evaluated session log is the subject, so one alignment evidence is produced per log and `sessionsLogs` is omitted — the subject already identifies what was evaluated. The intents policy the log was checked against is a `contextArtifacts` entry.

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    {
      "uri": "https://myorg.jfrog.io/artifactory/agentic-session-logs/a1b2c3d4-e5f6-7890-abcd-ef1234567890.json",
      "digest": {
        "sha256": "9f2b8c1d7a4530e6...b8d0c7f1a2934ee41a"
      }
    }
  ],
  "predicateType": "https://jfrog.com/evidence/agentic-alignment/v1",
  "predicate": {
    "provider": {
      "harness": {
        "name": "cursor",
        "version": "3.5.33"
      },
      "agent": {
        "id": "stable-agent-id",
        "name": "cursor-agent",
        "version": "1.0"
      },
      "languageModels": [
        {
          "inferenceProvider": "anthropic/claude-opus-4-8",
          "resolved": "anthropic/claude-opus-4-8-20260701"
        }
      ]
    },
    "contextArtifacts": [
      {
        "tags": ["policy"],
        "uri": "https://myorg.jfrog.io/artifactory/policies/intents-policy-v4.md",
        "digest": {
          "sha256": "41c0e93b6d28...7fa5029b8c1d64"
        }
      }
    ],
    "custom": {
      "baseCommit": {
        "uri": "https://myorg.jfrog.io/artifactory/gitCommit-entity/1b94ebd1ef32a4be4480dc70ed8ec2c6d55a2a0c.json",
        "digest": {
          "gitCommit": "1b94ebd1ef32a4be4480dc70ed8ec2c6d55a2a0c"
        }
      },
      "violations": ["short violation description"]
    },
    "result": "MISALIGNED",
    "owner": "login|email",
    "startTimestamp": "2026-04-08T08:54:03.771Z",
    "endTimestamp": "2026-04-08T08:55:11.004Z"
  },
  "createdAt": "2026-04-08T08:55:17.242Z",
  "createdBy": "dev-auto-reviewer"
}
```

Provider identity uses the field names in [`agent-identifier.md`](./agent-identifier.md), and `contextArtifacts[].data` replaces `uri` when the policy is inlined rather than linked.

**Required** in the tables below: `yes` = must be present and non-empty; `no` = optional; `conditional` = see Constraints. **MUST**, **SHOULD**, **MAY**, and related words follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) as stated in [README Conventions](../README.md#conventions).

## Alignment result

Closed set for `predicate.result` on alignment evidence. This is not the same enum as process evidence `COMPLETED` / `FAILED` / `CANCELLED`.

| Value | Meaning |
|---|---|
| `ALIGNED` | Evaluated logs comply with the intents policy |
| `MISALIGNED` | One or more violations against the intents policy |

## Alignment predicate differences

| Field | Type | Required | Constraints | Description | Usages |
|---|---|---|---|---|---|
| `provider` | Provider | yes | see [Provider](./agent-identifier.md) | Stack that performed the alignment check | Allowlist the checker |
| `result` | String | yes | [Alignment result](#alignment-result) | Verdict | Gate promotion / require extra review |
| `sessionsLogs` | ResourceDescriptor array | conditional | required (≥1) when the subject is an SDLC entity; omit when the subject is the evaluated session log; see [Resource descriptor](./agentic-process-evidence.md#resource-descriptor) | Logs covered by this verdict | Drill-down |
| `contextArtifacts` | ContextArtifact array | yes | ≥1; see [Context artifact](./agentic-process-evidence.md#context-artifact); SHOULD include the intents policy (`tags` contains `policy`) | Policy documents used | Reproduce the check |
| `custom.violations` | String array | no | 0..*; SHOULD be present when `result` is `MISALIGNED` | Short violation descriptions | Human review |

## Alternative subject (SDLC entity)

When the gate acts on the SDLC entity rather than on an individual log — for example a release promotion check that evaluates every log in the release — set the subject to that entity. Here `sessionsLogs` is required, because it is the only record of which logs the verdict covers. The rest of the predicate is unchanged. The trade-off is that a `MISALIGNED` verdict names a set of logs rather than pinpointing one, which is why the session log subject is recommended.

```json
{
  "subject": [
    {
      "uri": "https://github.com/MYORG/evidence/commit/1b94ebd1ef32a4be4480dc70ed8ec2c6d55a2a0c",
      "digest": {
        "gitCommit": "1b94ebd1ef32a4be4480dc70ed8ec2c6d55a2a0c"
      }
    }
  ],
  "predicate": {
    "sessionsLogs": [
      {
        "uri": "https://myorg.jfrog.io/artifactory/agentic-session-logs/a1b2c3d4-e5f6-7890-abcd-ef1234567890.json",
        "digest": {
          "sha256": "9f2b8c1d7a4530e6...b8d0c7f1a2934ee41a"
        }
      },
      {
        "uri": "https://myorg.jfrog.io/artifactory/agentic-session-logs/7c3e91af-2b58-4d10-9e6f-1a2b3c4d5e6f.json",
        "digest": {
          "sha256": "3ad70f5c9e18...42b6d8091fae57"
        }
      }
    ]
  }
}
```

## Notes

- Subject in each alignment evidence is typically each session log included in the git commits as [agentic process evidence](./agentic-process-evidence.md)
- `contextArtifacts` should include the intents policy (uri and/or inline `data` + digest)
- Intents analysis algorithms are out of scope; this entity defines the evidence envelope only
- See [`agent-identifier.md`](./agent-identifier.md) for provider identity
