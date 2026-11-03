# Agentic session log

Persistence and search model for agentic session logs stored as [Session log (BOM)](./agentic-session-log.md) artifacts.

Harness-native hook fields copied into `timeline[]` MAY keep the producer’s original names.

## Purpose

Provide a persistence store and search capabilities for agentic process session logs so operators can download and review them as part of human oversight.

Logs should be searchable by the subject that was produced (PR, commit, version). Searchable properties (tools, agents, suspected policy issues) support additional locate-and-review scenarios.

## Regulatory note

For some software products, persistence is required for at least **6 months**, or longer depending on context.

> EU AI Act, Article 19: the logs shall be kept for a period appropriate to the intended purpose of the high-risk AI system, of at least six months.

## Example use cases

- Locate all sessions that used a tool later reported to contain malware, and track those processes to relevant application versions
- Alert and review sessions flagged with suspected policy issues

## Artifact reference

Session artifacts are referenced through **uri + digest** from evidence statements.

## Payload example

```json
{
  "sessionId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "parentSessionId": "4o7g54cd-8957-7503-45ga-495867457986",
  "affectedEntities": [{"gitCommit": "bf02510c8ce0b804a099797510af"}],
  "tokens": 100,
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
  "timeline": [
    {
      "eventId": "evt_01HZX9K2M3N4P5Q6R7S8T9",
      "event": "beforeSubmitPrompt",
      "ts": "2026-04-08T08:51:02.114Z",
      "hook_event_name": "beforeSubmitPrompt",
      "prompt": "Implement rate limiting on /api/orders",
      "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "cwd": "/Users/dev/gh-org/gh-repo",
      "workspace_roots": ["/Users/dev/gh-org/gh-repo"]
    },
    {
      "eventId": "evt_01HZX9K2M3N4P5Q6R7S8T0",
      "event": "preToolUse",
      "ts": "2026-04-08T08:51:18.441Z",
      "hook_event_name": "preToolUse",
      "tool_name": "Write",
      "tool_use_id": "toolu_01ABC...",
      "tool_input": {
        "path": "/Users/dev/gh-org/gh-repo/src/orders/rate_limit.py",
        "contents": "..."
      },
      "path_hashes": {
        "/Users/dev/gh-org/gh-repo/src/orders/rate_limit.py": null
      },
      "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    },
    {
      "eventId": "evt_01HZX9K2M3N4P5Q6R7S8T1",
      "event": "postToolUse",
      "ts": "2026-04-08T08:51:19.102Z",
      "hook_event_name": "postToolUse",
      "tool_name": "Write",
      "tool_use_id": "toolu_01ABC...",
      "path_hashes": {
        "/Users/dev/gh-org/gh-repo/src/orders/rate_limit.py": "e3b0c44298fc1c149afbf4c8996fb924..."
      },
      "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    },
    {
      "eventId": "evt_01HZX9K2M3N4P5Q6R7S8T2",
      "event": "afterAgentResponse",
      "ts": "2026-04-08T08:52:01.880Z",
      "hook_event_name": "afterAgentResponse",
      "text": "Added token-bucket rate limiting on /api/orders.",
      "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    },
    {
      "eventId": "evt_01HZX9K2M3N4P5Q6R7S8T3",
      "event": "sessionEnd",
      "ts": "2026-04-08T08:52:05.010Z"
    }
  ]
}
```

Notes:

- Timeline entries capture harness events. Standard overlay fields are `eventId`, `event`, `ts`. All other properties SHOULD keep the producer’s original names (e.g. Cursor `hook_event_name`, `tool_name`, `path_hashes`, `conversation_id`).
- `path_hashes` on tool events is optional (pre/post tool-use fingerprints).
- `sessionEnd` (or a harness `stop` equivalent) triggers immediate flush to durable storage.

**Required** in the tables below: `yes` = must be present and non-empty; `no` = optional. **MUST**, **SHOULD**, **MAY**, and related words follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) as stated in [README Conventions](../README.md#conventions).

## Log object

| Field | Type | Required | Constraints | Description | Usages |
|---|---|---|---|---|---|
| `sessionId` | String | yes | non-empty | Session identifier | Search; correlate to a session log referenced from process evidence `sessionsLogs` |
| `parentSessionId` | String | no | non-empty when present | Parent session / run id | Locate related sessions |
| `affectedEntities` | Object Array | no | subject keys (e.g. `gitCommit`) | entities produced or handled | Locate logs by commit / version |
| `tokens` | Integer | no | no | Number of tokens used as reported by the agent | Cost and efficiency calculations |
| `provider` | Provider | yes | see [Provider](./agent-identifier.md) | Harness, agent, and language models for this log | Search; allowlist checks |
| `timeline` | TimelineEvent array | yes | ≥1; see [Timeline event](#timeline-event) | Ordered hook events | Human review of the session |

## Timeline event

Captured harness events, plus a small overlay this standard owns. Additional producer properties MAY be present and MUST NOT be renamed. Do not add log `sessionId` here; correlate via the envelope.

| Field | Type | Required | Constraints | Description | Usages |
|---|---|---|---|---|---|
| `eventId` | String | yes | unique within the log | Event identifier | Dedup; citation |
| `event` | String | yes | non-empty; SHOULD be a [Timeline event kind](#timeline-event-kind) when one applies | Kind of event | Filter tool uses vs prompts |
| `ts` | Timestamp | yes | ISO 8601 | Event time | Ordering; duration |

## Timeline event kind

Known values for the standard `event` field. Producers SHOULD use one of these when the event matches. Other values MAY be used for harness-specific events that have no equivalent here. Extra harness fields (for example `hook_event_name`) MAY still repeat the native name.

| Value | Meaning |
|---|---|
| `beforeSubmitPrompt` | User or system prompt about to be submitted |
| `preToolUse` | Tool invocation about to run |
| `postToolUse` | Tool invocation finished |
| `afterAgentResponse` | Agent produced a response |
| `sessionEnd` | Session finished; flush to durable storage |

## Searchable attributes

Used to identify breached components and locate related processes:

| Property | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `tools` | Tool array | no | 0..*; see [Tool](./agentic-process-evidence.md#tool) | Tools used |
| `agent` | Agent | no | from `provider.agent`; see [Agent](./agent-identifier.md#agent) | Agent metadata |
| `subject` | Object | no | subject keys | Locating change commit(s) / other subjects |
| `sessionId` | String | yes | same as log `sessionId` | Run / session id |
| `parentSessionId` | String | no | | Parent session / run id |

## Out of scope

- Agent authentication methods and data
- Intents analysis
