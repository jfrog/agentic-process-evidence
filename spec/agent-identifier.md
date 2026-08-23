# Agent identifier

This page defines the **Provider** type and its nested types **Harness**, **Agent**, and **LanguageModel**: identity of the agent runtime stack recorded on logs and evidence.

Provides data for full traceability and helps mitigate supply-chain style attacks: **harness → agent → language model**.

**Required** in the tables below: `yes` = must be present and non-empty; `no` = optional. **MUST**, **SHOULD**, **MAY**, and related words follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) as stated in [README Conventions](../README.md#conventions).

## Provider

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

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `harness` | Harness | yes | see [Harness](#harness) | Agentic provider harness |
| `agent` | Agent | no | see [Agent](#agent) | Agent running in the harness |
| `languageModels` | LanguageModel array | no | 0..*; see [Language model](#language-model) | Language models involved in the run |

Used as `predicate.providers` (Provider array) on [agentic process evidence](./agentic-process-evidence.md) — one entry per distinct stack in the process. Used as a single `provider` field on [alignment evidence](./alignment-evidence.md) and on [agentic session log](./agentic-session-log.md) payloads.

## Harness

The agentic product that hosts the session (IDE, CI agent runner, review bot, and similar).

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `name` | String | yes | non-empty | Harness product name |
| `version` | String | yes | non-empty | Harness version as emitted by the producer (not necessarily SemVer) |

## Agent

The agent running inside the harness.

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `id` | String | no | non-empty when present | Stable agent identifier |
| `name` | String | no | non-empty when present | Agent name |
| `version` | String | no | non-empty when present | Agent version as emitted by the producer (not necessarily SemVer) |

## Language model

A language model involved in the run: the one requested, the one actually used, or both when known.

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `inferenceProvider` | String | no | non-empty when present | LLM requested (name/version), if known |
| `resolved` | String | no | non-empty when present | LLM actually used (name/version), if known |
