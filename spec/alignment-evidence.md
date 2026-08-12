# Alignment evidence

Evidence based on the agentic process that improves control over **intent alignment** to desired intent and corporate rules.

`predicateType`: `https://jfrog.com/evidence/agentic-alignment/v1`

## Generation process

On commit, the client-side agentic process:

1. Collects all process logs (same path that creates development process evidence)
2. Checks the customer intents-policy resource against the process logs
3. Emits evidence with verdict `ALIGNED` | `MISALIGNED` and a summary of problematic intents

## Example

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    {
      "uri": "https://github.jfrog.info/JFROG/evidence/commit/bf02510c8ce0b804a099797510afc19325acfb979538c5b521304c83cde63892",
      "digest": {
        "gitCommit|sha256": "bf02510c8ce0b804a099797510afc19325acfb979538c5b521304c83cde63892"
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
      "language_models": [
        {
          "inference_provider": "anthropic/claude-opus-4-8",
          "resolved": "anthropic/claude-opus-4-8-20260701"
        }
      ]
    },
    "sessionsLogs": [
      {
        "uri": "policy doc url",
        "digest": "sha..."
      }
    ],
    "contextArtifacts": [
      {
        "type": "policy",
        "uri": "link-to-intents-policy",
        "data": "The complete artifact document inline",
        "digest": "policy-sha256"
      }
    ],
    "custom": {
      "baseCommit": {
        "uri": "https://github.jfrog.info/JFROG/evidence/commit/bf02510c8ce0b804a099797510afc19325acfb979538c5b521304c83cde63892",
        "digest": {
          "gitCommit": "bf02510c8ce0b804a099797510afc19325acfb979538c5b521304c83cde63892"
        }
      },
      "violations": ["short violation description"]
    },
    "result": "ALIGNED|MISALIGNED",
    "owner": "login|email",
    "startTimestamp": "ISO 8601 timestamp",
    "endTimestamp": "ISO 8601 timestamp"
  },
  "createdAt": "2026-04-08T08:55:17.242Z",
  "createdBy": "dev-auto-reviewer"
}
```

## Notes

- Subject is typically the same git commit as [AI Process evidence](./ai-process-evidence.md)
- `contextArtifacts` should include the intents policy (uri and/or inline `data` + digest)
- Intents analysis algorithms are out of scope; this entity defines the evidence envelope only
- See [`agent-identifier.md`](./agent-identifier.md) for provider identity
