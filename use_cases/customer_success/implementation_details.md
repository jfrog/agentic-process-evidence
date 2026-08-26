# Customer success - Implementation

Two elements. Protocol needs both.

1. **Agent runtime tool** — telemetry capture, bind on session end, signed **agentic-support-process-evidence** push
2. **Server** — persist (audit + troubleshooting), alignment checks, policy gates (policy-as-code)

Runtime tool produces. Server keeps and enforces.

What the **agentic-support-process-evidence** is *for*:

- **Agent identity** — harness, agent, model (requested vs resolved)
- **Provenance** — support process bound to that session log
- **Execution traceability** — points to the session timeline (policy reads, order lookup, MCP)
- **Integrity & non-repudiation** — signed in-toto / DSSE, not editable after the fact
- **Policy-as-code** — gates evaluate the evidence before a consequential action (refund, close, escalate)

---

## 1. Agent runtime tool

Sits next to the support agent. Active on every session you intend to govern.

### Telemetry capture

As the agent handles the ticket, the runtime tool logs operational context into **agentic-session-log**s buffer. Harness hooks are one way to capture that timeline.

The telemetry must also provide information that will allow to bind the session log to the entity space. Here there is no git commit — bind to the ticket / `sessionId`.

Each log carries `sessionId`, `provider` (harness / agent / models), and `parentSessionId` when it is a subagent.

Example: `[objects_examples/agentic-session-log.json](./objects_examples/agentic-session-log.json)`.

### Aggregating on session end

For this use case we decided that **session end** is the process end. Process = that one session.

On that event the runtime tool:

- Finalizes this session log (incl. subagents of this run)
- Stamps `affectedEntities` with this `sessionId`
- `traceId` = this session (or the ticket id)
- Unions `providers` and `tools`
- `startTimestamp` = session start
- `endTimestamp` = session end

Don't emit a second process evidence later. There is no commit to wait for. Subject is the session log itself.

### Evidence push

Upload the session log to persistent storage (`uri` + `sha256`).

Then craft **agentic support process evidence**, sign it (in-toto + DSSE), and push to the platform. Subject = that session log. Omit `sessionsLogs[]` — the subject already identifies the log.

Example: `[objects_examples/agentic-support-process-evidence.json](./objects_examples/agentic-support-process-evidence.json)`.

That is the handoff. After this the log and the signed evidence live on the server.

---

## 2. Server

Receives the push. Keep it, check it, evaluate it at later choke points.

### Persistence — audit and troubleshooting

Session logs and evidence stay as long as the case / product requires. For some products that is at least **6 months** (EU AI Act Art. 19).

Store them searchable. Minimum keys:

- `sessionId` (and `parentSessionId`)
- ticket / order
- `agent` / harness / models
- `tools`

If a tool or MCP is later identified as malicious: search the session logs that used it, walk those logs to the tickets they handled.

Process evidence on the session log is the index. The session log is the drill-down.

### Alignment check

The server MAY emit **alignment evidence** per session log (`ALIGNED` | `MISALIGNED`). Subject = the session log.

One way: run another agent over the persisted session log — prompts, thinking, tool use — against support policies (refund window, privacy, escalation).

If a session never saw a required policy, that is this check. The log + `contextArtifacts` are the proof.

Example: `[objects_examples/agentic-session-alignment-evidence.json](./objects_examples/agentic-session-alignment-evidence.json)`.

### Policy gates (policy-as-code)

Session end is not the last control point. Before a consequential action — **issue a refund**, close the case, escalate — policy gates evaluate the attached agentic support process evidence.

Typical gate: allow the agent's outcome only if the session was aligned with refund / privacy policy.

Look up the process evidence on that session log, then the alignment evidence, and verify:

- There is a human accountable (`owner`)
- Mandatory context was present (`contextArtifacts` + logs)
- The agent stayed aligned with support policies (alignment evidence, `intents`)
- The agent did not attempt unapproved lateral operations
- The case was handled by an approved LLM
- The agent used approved tools / harness
- Evidence exists and is signed

Fail → block the action (or route to a human). Pass → the case outcome may stand.

Same evidence, later choke point. The runtime tool does not gate. The server does.

---

## Artifacts

### agentic-session-log

`[objects_examples/agentic-session-log.json](./objects_examples/agentic-session-log.json)`

Captured by the runtime tool, persisted by the server. One per support session.

- `sessionId` — this session
- `affectedEntities.sessionId` — the session is the subject
- `provider` — harness, agent, models (requested vs resolved)
- `parentSessionId` — when it is a subagent
- `timeline[]` — timeline events such as prompts, tool use, thinking tokens, can be captured via hooks.

### agentic-support-process-evidence

`[objects_examples/agentic-support-process-evidence.json](./objects_examples/agentic-support-process-evidence.json)`

Signed, attached to the session log, queried by policy gates.

- `predicateType`: `https://jfrog.com/evidence/agentic-support-process/v1`
- `subject` = this session log (uri + digest)
- `sessionsLogs[]` omitted — the subject already identifies the log
- `owner` = the support platform (or the human the case was routed to)
- `result` = `COMPLETED` when the case is resolved
- `contextArtifacts` = policies that were injected (`policy`, …)
- `custom.ticket` / `custom.orderId` = the case identifiers
- `intents` / `processSummary` extracted from the session
- `createdBy` = the runtime tool

### agentic-session-alignment-evidence

`[objects_examples/agentic-session-alignment-evidence.json](./objects_examples/agentic-session-alignment-evidence.json)`

Optional. Produced on the server from the persisted session log. Used by the gates. Does not replace process evidence — process evidence says what happened, this says whether it was ok.

- `predicateType`: `https://jfrog.com/evidence/agentic-session-alignment/v1`
- `subject` = the evaluated session log (uri + digest)
- `sessionsLogs[]` omitted — the subject already identifies the log
- `provider` = the alignment checker (harness / agent / models), not the support agent
- `result` = `ALIGNED` | `MISALIGNED`
- `contextArtifacts` = policies the session was checked against
- `custom.violations` = short violation list when `MISALIGNED`
- `owner` = who ran / owns the check
- `createdBy` = the alignment checker

## See also

- `[overview.md](./overview.md)`
- `[objects_examples/](./objects_examples/)`
- `[spec/agent-runtime-tool.md](../../spec/agent-runtime-tool.md)`
- `[spec/agentic-session-log.md](../../spec/agentic-session-log.md)`
- `[spec/agentic-process-evidence.md](../../spec/agentic-process-evidence.md)`
- `[spec/alignment-evidence.md](../../spec/alignment-evidence.md)`
