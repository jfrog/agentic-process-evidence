# Agentic session log

Persistence and search model for agentic session logs stored as [Session log (BOM)](./agentic-session-log.md) artifacts.

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
  "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "parent_session_id": "4o7g54cd-8957-7503-45ga-495867457986",
  "subject": {"gitCommit":"bf02510c8ce0b804a099797510af"},
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
  "timeline": [
    {
      "event_id": "evt_01HZX9K2M3N4P5Q6R7S8T9",
      "event": "beforeSubmitPrompt",
      "ts": "2026-04-08T08:51:02.114Z",
      "hook_event_name": "beforeSubmitPrompt",
      "prompt": "Implement rate limiting on /api/orders",
      "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "cwd": "/Users/dev/gh-org/gh-repo",
      "workspace_roots": ["/Users/dev/gh-org/gh-repo"]
    },
    {
      "event_id": "evt_01HZX9K2M3N4P5Q6R7S8T0",
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
      "event_id": "evt_01HZX9K2M3N4P5Q6R7S8T1",
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
      "event_id": "evt_01HZX9K2M3N4P5Q6R7S8T2",
      "event": "afterAgentResponse",
      "ts": "2026-04-08T08:52:01.880Z",
      "hook_event_name": "afterAgentResponse",
      "text": "Added token-bucket rate limiting on /api/orders.",
      "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    },
    {
      "event_id": "evt_01HZX9K2M3N4P5Q6R7S8T3",
      "event": "processEnd",
      "ts": "2026-04-08T08:52:05.010Z",
      "conversation_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    }
  ]
}
```

Notes:

- Timeline events should preserve harness hook fields verbatim (e.g. Cursor hooks).
- `path_hashes` on tool events is optional (pre/post tool-use fingerprints).
- `processEnd` (or `stop`) triggers immediate flush to durable storage.

## Searchable attributes

Used to identify breached components and locate related processes:

| Property | Description |
|---|---|
| `tools` | Tools used (lists and versions) |
| `agent` | Agent metadata |
| `subject` | Subject keys for locating change commit(s)/other subjects |
| `session_id` | Run id |
| `parent_session_id` | Parent session / run id |

## Out of scope

- Agent authentication methods and data
- Intents analysis
