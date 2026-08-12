# Agent identifier

Identity of the agent runtime stack, included in agentic process evidence.

Provides data for full traceability and helps mitigate supply-chain style attacks: **harness → agent → language model**.

## Shape

```json
{
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
}
```

## Fields

| Field | Description |
|---|---|
| `harness.name` / `harness.version` | Agentic provider harness |
| `agent.id` | Stable agent identifier |
| `agent.name` / `agent.version` | Agent name and version |
| `languageModels[].inferenceProvider` | LLM requested (name/version), if known |
| `languageModels[].resolved` | LLM actually used (name/version), if known |

Embedded under `predicate.provider` in any [agentic session evidence](./agentic-session-evidence.md). Also recorded on [Agentic session log](./agentic-session-log.md) payloads.
