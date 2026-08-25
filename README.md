# Agentic Process Evidence Standard

A proposed standard for **evidencing agentic processes**.
All agentic processes can be documented using this standard. We focus examples on agentic processes running as part of an SDLC (software development lifecycle) pipeline — such as code development, code review, version release, and other related processes affecting software release, but the same evidence model applies to any agentic process, and we welcome use beyond SDLC.

This repository defines how to bind agent session information to SDLC entities (commits, artifacts, application versions) so organizations can govern their AI-assisted work with the same rigor as traditional release process.

Entity definitions live under [spec/](./spec/).

## Conventions

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this repository are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals.

Field tables use a **Required** column (`yes` / `no` / `conditional`) for whether a field is present. RFC 2119 words appear in **Constraints** and in prose.

JSON field names defined by this standard use **camelCase**.

### Naming conventions

Two terms carry most of the model, and they are not interchangeable.

**Session** — a single agent run, from the agent's point of view. Whatever a harness calls a conversation, chat, thread, or run is a session here: one IDE conversation, one review-bot invocation, one support exchange. A session is captured as a [session log](spec/agentic-session-log.md) and identified by `sessionId`. Harness-native names for the same thing (for example Cursor `conversation_id`) MAY appear on timeline events and keep the producer's spelling, but `sessionId` is the identifier this standard searches on.

**Process** — the value unit, from the organization's point of view: the work whose outcome someone is accountable for. A process aggregates every session that contributed to one outcome — all coding sessions that yielded a commit, all review sessions on a pull request, a single customer-support case — and its outcome is the evidence **subject** (typically a git commit). One process MAY contain many sessions; one session belongs to one process. Process-level facts live on [process evidence](spec/agentic-process-evidence.md): `traceId`, `processSummary`, `result`, start and end timestamps, with `sessionsLogs` pointing at the sessions it covers.

Rule of thumb: if a fact is about what the agent did in one run, it belongs to a session; if it is about the outcome being governed, it belongs to the process.

---

## Why this exists

Agentic tools (IDE agents, review bots, release assistants) change code and influence releases, but their provenance is often invisible to governance, auditors, and policy engines. 

This introduces blind spots in SDLC where organizations have limited ability to identify and control these agentic processes or apply risk based decisions on how they are handled, and validated and also how they can be audited later on.   

We believe that organizations must be able to answer the following minimal questions  

- Which systems built/tested/approved our code?
- Was the system aware of our organizational policies and guidelines?
- Who is accountable?
- Was a human involved in the process?

We also believe, that more in-depth information must be available:

- Did the system in fact comply with the organization's intents?
- Thorough review of the systems logs must be supported

As agents take more roles, and as they become more independent, the ability for an organization to make sure its architecture and policies are preserved becomes a challenge. This project offers a way to bring that control and assurance back into software development. 

This standard enables:


| Capability                       | What it unlocks                                                                                                                                                                                                                                                                                                                         | usage examples                                                                                                                                 |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Troubleshooting & monitoring** | Trace agent sessions back to the commit or release they affected, and keep the logs in their correct context for as long as they could be needed                                                                                                                                                                                        | Supply chain traceability: MCP server is disclosed as malicious — search logs by `tools` and follow `commit` to every release that shipped it. |
| **Policy-as-code validation**    | Automatically check harnesses, models, tools, owners, and outcomes. Check that an agentic process: has evidence that exists, is signed, and is relevant (the SDLC entity is the subject); used approved policy documents as context; ran with approved agents and models; was reviewed by a human, and by whom; has a named human owner | A package is blocked from production because a merge commit ran on a non-allowlisted model and has no named `owner`.                           |
| **Human oversight**              | Allow optimization of human review to only when risk is identified, or when human oversight was missing from the process                                                                                                                                                                                                                | 235 of 240 commits are `ALIGNED` ; the 5 `MISALIGNED` on pricing require additional approval from the pricing features owner.                  |
| **Regulatory alignment**         | Persist process logs for retention windows (e.g. EU AI Act Art. 19: ≥ 6 months) and link them to the development process, so missing human oversight becomes identifiable                                                                                                                                                               | ---                                                                                                                                            |


By attaching **in-toto-style evidence** to git commits, artifacts and application releases, agentic activity becomes first-class release provenance—collectable SLSA-style evidence.

---

## High-level model

Agentic session evidence model: session log artifacts, the generic agentic session evidence, and its agentic-code-development, agentic-pr-review and agentic-alignment-check implementations

*Editable source: [docs/diagrams/high-level-model.excalidraw](docs/diagrams/high-level-model.excalidraw) — open it at [excalidraw.com](https://excalidraw.com).*

The model has two building blocks and a set of process-specific implementations:

- **Session log artifacts (BOM)** — the raw agent timeline of each session, persisted in searchable durable storage.
- **Agentic session evidence** — one generic, in-toto based provenance model that carries the provider stack, session identifiers, session log references, tools, context artifacts, result, owner and reviewers.
- **Implementations** — each agentic process specializes the generic evidence with its own `predicateType`, subject and process-specific fields: `agentic-code-development`, `agentic-pr-review` and `agentic-alignment-check`.

Every implementation references the session logs it produced (`sessionsLogs[].uri` + `digest`) and is attached to an SDLC subject — a git commit, an artifact digest, an application version, or the session log artifact itself in the case of an alignment check.

### 1. Agentic Session log

The **agent session log** relevant to the agentic session — prompts, tool uses, responses, timestamps.


| Attribute                 | Guidance                                                                                           |
| ------------------------- | -------------------------------------------------------------------------------------------------- |
| **Schema**                | [Agentic session log](spec/agentic-session-log.md)                                                 |
| **Cardinality**           | Many session logs per commit; one commit per session log (the commit that session created/updated) |
| **Type**                  | Generic artifact                                                                                   |
| **Location**              | Persistent storage with search capabilities                                                        |
| **Searchable Attributes** | `agent`, `tools`, `commit`, `sessionId` (and optionally `parentSessionId`)                         |
| **Used for**              | Deep troubleshooting                                                                               |
| **Retention**             | Aligned to release retention (minimum of 6 months according to EU regulation for certain software) |


#### Relationships

- Produced and uploaded by the [Agent runtime tool](spec/agent-runtime-tool.md) on commit
- Referenced from AI Process evidence via `sessionsLogs[].uri` + `sessionsLogs[].digest`

### 2. Agentic process evidence

**Provenance evidence** whose subject is the SDLC entity the process produced—typically a **git commit**; for release approval, the **application release,** and for an artifact, the **artifact digest**. One process evidence covers every session that contributed to that subject.

The agentic process evidence should only be created once the agentic process completes, e.g. code is committed, code review is completed, release was promoted, alignment check done. 


| Attribute         | Guidance                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Schema**        | [Agentic process evidence](spec/agentic-process-evidence.md)                                                                                                                               |
| **predicateType** | e.g. `https://jfrog.com/evidence/agentic-code-review`, `https://jfrog.com/evidence/agentic-dev-process`                                                                                    |
| **Contains**      | Provider (harness / agent / LLMs), process id (`traceId`), session logs, tools, context artifacts, result, intents, summary, owner, reviewers, timestamps and process-specific custom data |
| **Used for**      | Provenance on the agentic process allowing for Policy checks and auditing                                                                                                                  |
| **Retention**     | Aligned to release retention                                                                                                                                                               |


### 3. Agent runtime tool

Monitors an agentic process and its sessions and on completion:

1. Collects all relevant [Session logs](./agentic-session-log.md) and enriches them with process data
2. Extracts provenance information
3. Uploads the session logs into remote persistence storage
4. Uploads [agentic process evidence](./agentic-process-evidence.md) referencing the uploaded agentic session logs

#### Requirements

- Should be active on every agentic process intended for governance
- Emits or attaches [agentic process evidence](./agentic-process-evidence.md) with subject = the target entity of the process

#### Related entities


| Entity                                                    | Relationship                                                                                                       |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [Agentic session log](./agentic-session-log.md)           | Payload / timeline schema of the agentic session                                                                   |
| [Agentic process evidence](./agentic-process-evidence.md) | Provenance statement uploaded on the SDLC entity the process produced (commit, artifact, or release)               |
| [Alignment evidence](./alignment-evidence.md)             | An optional second statement on a session log(s) from the same commit-time collection indicating policy violations |
| [Agent identifier](./agent-identifier.md)                 | Provider stack recorded on logs and evidence                                                                       |


## Use Cases

The below are potential implementation examples.

Business goals an agentic **process** achieves. Normative field definitions stay in `[spec/](../spec/)`.


| Use case                                                           | Process shape                                   | Subject               |
| ------------------------------------------------------------------ | ----------------------------------------------- | --------------------- |
| [Human-in-the-loop code development](./use_cases/human_in_the_loop_code_development/overview.md)       | Human + IDE agent; many sessions → one commit   | git commit            | 
| [Autonomous code development](./use_cases/autonomous_code_development/overview.md)  | agent; many sessions → one commit   | git commit   |
| [Code review agent](./use_cases/code_review_agent/overview.md)     | Review agent; automerge negligible; one session | git commit (reviewed) |
| [Customer success agent](./use_cases/customer_success/overview.md) | Autonomous agent; process = one session         | session log           |


---

## How to use this standard

### Adopt in an organization

1. **Pick subjects** — Start with `git commit` for development and review; extend to application release promotion or approval.
2. **Instrument the runtime** — Ensure the agent harness emits a session timeline (hooks or equivalent) and the Agent runtime tool flushes logs + evidence appropriately (e.g. on commit for development process).
3. **Store session logs** — Persist the session logs and enable minimal searchable attributes (`tools`, `agent`, `commit`, `sessionId`) so you can find all sessions that used a compromised tool or flagged policy issue.
4. **Publish AI Process evidence** — Sign and attach the in-toto statement to the commit.
5. **Create policy-as-code** — Whitelist harnesses, agents, LLMs; require owners/reviewers; gate on `result` and alignment verdicts.
6. **Route exceptions to humans** — Use `reviewers`, and when available `intents` and `processSummary`, and session log URIs for rapid approval when policy cannot decide.
7. **Retain** — Keep session logs and evidence at least as long as release is relevant.

### Integrate in a pipeline (typical flow example)

```text
Agent session (IDE / CI)
        │
        │  timeline events (prompt, tool use, response, stop)
        ▼
Agent runtime tool  ──►  upload Session log artifact(s)
        │
        │  on commit
        ▼
Build Agentic process evidence (subject = git commit)
        │
        ├─► optional: Alignment evidence vs intents policy (subject = session log artifact)
        │
        ▼
Collect evidence of a release version
        │
        ▼
Policy engine + human review / audit drill-down
```

### Implement or validate against the spec

- Treat `[spec/](./spec/)` as the normative field list and examples.  
- Prefer digest-linked references over mutable URLs alone (`sessionsLogs`, `contextArtifacts`).  
- Keep agent identity searchable: harness + agent + language model (requested and resolved).  
- Out of scope for this version: agent authentication methods; deep intents-analysis algorithms (only the alignment evidence envelope is specified).

### Example consumers


| Role                       | How they use evidence                                                                    |
| -------------------------- | ---------------------------------------------------------------------------------------- |
| **Security / AppSec**      | Find commits whose sessions used a non-approved or vulnerable tool or non-approved model |
| **Compliance / auditors**  | Sample releases and drill into commit-level agentic provenance + logs                    |
| **Platform / DevEx**       | Require evidence presence before merge or promote                                        |
| **Developers / reviewers** | Oversight via summaries, intents, and linked session logs                                |


---

## Repository contents


| Path                       | Description                                                                               |
| -------------------------- | ----------------------------------------------------------------------------------------- |
| `[spec/](./spec/)`         | session log, agentic process evidence, agent identifier, alignment evidence, runtime tool |
| `[README.md](./README.md)` | Orientation and adoption guide (this file)                                                |


---

## Out of scope

- Agent authentication methods and credentials  
- Algorithms for intents analysis (only the evidence shape for alignment verdicts)

---

## Status

Working draft toward a shared **agentic process evidence** practice for SDLC governance. Feedback and implementations should align field names and predicate types with `[spec/](./spec/)` so evidence remains interoperable across harnesses and aggregators.