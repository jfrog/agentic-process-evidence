# Agentic Process Evidence Standard

A proposed standard for **evidencing agentic processes**.
With this standard we foicus on any agentic process running as part of an SDLC (software development lifecycle) pipeline, such as code development, code review, version release, and other related processes effecting software release, but we do aknowledge the same evidence might fit other scenarios and welcome any use.

This repository defines how to bind agent session information to SDLC entities (commits, artifacts, application versions) so organizations can govern their AI-assisted work with the same rigor as traditional release process.

Entity definitions live under `[spec/](./spec/)`. 

---

## Why this exists

Agentic tools (IDE agents, review bots, release assistants) change code and influence releases, but their provenance is often invisible to governance, auditors, and policy engines.

This introduces blind spots in SDLC where organizations have limited ability to identify and control these agentic processes or apply risk based decisions on how they are handled, and validated and also how they can be audited later on.   

As agents take more roles, and as they become more independent, the ability for an organization to make sure its architecture and policies are preserved become a challenge. This project offers a way to bring that control and assurance back into software development. 

This standard enables:


| Capability                       | What it unlocks                                                                                                                                                             |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Troubleshooting & monitoring** | Trace agent sessions back to the commit or release they affected and allow tracking logs in their correct context and for as long as they could be needed                   |
| **Policy-as-code validation**    | Automatically check harnesses, models, tools, owners, and outcomes                                                                                                          |
| **Human oversight**              | Allow optimization of human review to only when risk is identified or when a human oversight was missing from the process                                                   |
| **Regulatory alignment**         | Persist process logs for retention windows (e.g. EU AI Act Art. 19: ≥ 6 months) in a way that they links themn to the development process, identify missing human oversight |


By attaching **in-toto-style evidence** to git commits, artifacts and application releases, agentic activity becomes first-class release provenance—collectable SLSA-style evidence.

---

## High-level model

Agentic development process example: 

```
on commit:
┌─────────────────────┐       persist         ┌──────────────────────────┐
│  Agent runtime tool │ ─────────────────────►│  Session log (BOM)       │
│  (monitors session) │                       │  Full agent timeline     │
└──────────┬──────────┘                       │  Stored as an artifact   │
           │                                  └───┬────────┬─────────────┘
           │ uploads evidence                     │        │ 
           ▼                                      │        │ 
┌──────────────────────────┐   referenced by      │        │
│ Agentic session evidence │  ────────────────────┘        │
│  on gitCommit.           │                               │
└──────────────────────────┘                               │
                                                           │
PR/build/other alignemnt check (optional)                  │
                                                           │
┌──────────────────────────┐   referenced by               │
│ Agentic alignment check  │  ─────────────────────────────┘
│  on gitCommit.           │                  
└───────────┬──────────────┘                  
            │
            │ uploads evidence 
            ▼
┌─────────────────────────────┐
│     Alignment evidence      │
│     ALIGNED / MISALIGNED    │
└─────────────────────────────┘

                                                        
On Agentic PR review:
┌──────────────────────────┐ persist       ┌─────────────────────────────┐
│ Agentic PR review        │──────────────►│     Session log (BOM)       │
│                          │               │     Full agent timeline     │
└──────────┬───────────────┘               │     Stored as an artifact   │
           │                               └───┬─────────────────────────┘
           │ uploads evidence                  │ 
           ▼                                   │
┌──────────────────────────┐  referenced by    │
│ Agentic session evidence │  ─────────────────┘
│  on gitCommit.           │
└──────────────────────────┘
```

### 1. Session log (BOM)

The **agent session trace** relevant to the agentic session — prompts, tool uses, responses, timestamps.

| Attribute                 | Guidance                                                                                           |
| ------------------------- | -------------------------------------------------------------------------------------------------- |
| **Cardinality**           | Many session logs per commit; one commit per session log (the commit that session created/updated) |
| **Type**                  | Generic artifact                                                                                   |
| **Location**              | Persistent storage with search capabilities                                                        |
| **Searchable Attributes** | `agent`, `tools`, `commit`, `session_id` (and optionally `parent_session_id`)                      |
| **Used for**              | Deep troubleshooting                                                                               |
| **Retention**             | Aligned to release retention (minimum of 6 months accoring to EU regulation for certain software)  |


#### Relationships

- Produced and uploaded by the [Agent runtime tool](spec/agent-runtime-tool.md) on commit
- Referenced from AI Process evidence via `sessionsLogs[].uri` + `sessionsLogs[].digest`
- Payload schema: [Agentic session log](spec/agentic-session-log.md)


### 2. Agentic Session evidence

**Provenance evidence** whose subject is the SDLC entity the agent handled—typically a **git commit**; for release approval, the **application release,** and for an artifact, the **artifact digest**.


| Attribute         | Guidance                                                                                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Schema**        | `https://in-toto.io/Statement/v1`                                                                                                                                               |
| **predicateType** | e.g. `https://jfrog.com/evidence/agentic-code-review`, `https://jfrog.com/evidence/agentic-dev-process`                                                                         |
| **Contains**      | Provider (harness / agent / LLMs), session IDs, session logs, tools, context artifacts, result, intents, summary, owner, reviewers, timestamps and process relevant custom data |
| **Used for**      | Provenance on the agentic process allowing for Policy checks and auditing                                                                                                       |
| **Retention**     | Aligned to release retention                                                                                                                                                    |


### 3. Session logs and evidence creators

The below are potential implementation examples.

#### 3.1 Development Agent runtime tool

Runs alongside the development agent. On a **commit event** it should create an evidence:
`predicateType`: `https://jfrog.com/evidence/agentic-dev-process/v1` 

High level process flow:

1. Collect relevant session logs
2. Extract provenance fields
3. Upload session logs to durable storage and ensure searchability
4. Upload git-commit agentic session evidence that references those logs and process provenance

The evidence should also contain any context documents used for developing the code. 

The runtime tool should be active for every agentic SDLC flow you intend to govern.

#### 3.2. Alignment Violations evidence

Runs at desired pipeline steps (Code Commit/PR review/release promotion) for agentically checking alignemnt of the developemnt process to organization policies and for flagging high-risk intents.

`predicateType`: `https://jfrog.com/evidence/agentic-alignment/v1` 
This process collects existing relevant session logs (git commit/PR commits/application release session logs) compares session logs against an intent/policy resource(s) and records `ALIGNED` | `MISALIGNED` plus violation summaries. 
The alignment evidence can then be used in policy checks for blocking a release or requiring additional approvals and oversight.

#### 3.3. PR Approval

Runs once an agentic PR approval completes, either by a homegrown agentic approval tool or by a 3rd party PR Approve product.

`predicateType`: `https://jfrog.com/evidence/agentic-code-review/v1` 
This process collects all relevant session logs generated in the review process and uploads them, then generates the agentic review provenance adding also the review verdict and the git information for the code diff that was reviewd and requirements against the code was reviewed.  

---

## Evidence shape (summary)

Evidence follows [in-toto Statement v1](https://in-toto.io/). Minimum subject + envelope:


| Field                                | Role                                                                                |
| ------------------------------------ | ----------------------------------------------------------------------------------- |
| `subject[].uri` / `subject[].digest` | Immutable identity of the subject and a download link                               |
| `_type`                              | `https://in-toto.io/Statement/v1`                                                   |
| `predicateType`                      | Process kind (`agentic-dev-process`, `agentic-code-review`, `agentic-alignment`, …) |
| `createdAt` / `createdBy`            | Freshness and creator identity                                                      |
| `predicate.`*                        | See field tables in `[spec/agentic-session-evidence.md](./spec/agentic-session-evidence.md)`  |


We recommand signing the evidence using DSSE ([https://github.com/secure-systems-lab/dsse](https://github.com/secure-systems-lab/dsse)).

**Predicate highlights for policy and humans:**

- **provider stack** — harness, agent id/name/version, requested vs resolved language models
- **sessionId** + **sessionsLogs** — correlate and download full chat/tool timeline  
- **tools** — list of used tools for allowing for blacklist / allowlist checks  
- **contextArtifacts** — policies, guidelines, prior logs (uri and/or inline `data` + digest)  
- **result**, **intents**, **processSummary** — to be used by automation gates and human review  
- **owner** / **reviewers** — accountability and oversight  
- **custom** — process-specific data (e.g. `baseCommit`, requirements issues, change profile)

---

## How to use this standard

### Adopt in an organization

1. **Pick subjects** — Start with `gitCommit` for development and review; extend to application release promotion or approval.
2. **Instrument the runtime** — Ensure the agent harness emits a session timeline (hooks or equivalent) and the Agent runtime tool flushes logs + evidence appropreately (e.g. on commit for development process).
3. **Store session logs as artifacts** — Attach scannable properties (`tools`, `agent`, `commit`, `session_id`) so you can find all sessions that used a compromised tool or flagged policy issue.
4. **Publish AI Process evidence** — Sign and attach the in-toto statement to the commit.
5. **Create policy-as-code** — Whitelist harnesses, agents, LLMs; require owners/reviewers; gate on `result` and alignment verdicts.
6. **Route exceptions to humans** — Use `reviewers`, and when avaialble `intents` and `processSummary`, and session log URIs for rapid approval when policy cannot decide.
7. **Retain** — Keep session logs and evidence at least as long as release is relevant.

### Integrate in a pipeline (typical flow)

```text
Agent session (IDE / CI)
        │
        │  timeline events (prompt, tool use, response, stop)
        ▼
Agent runtime tool  ──►  upload Session log artifact(s)
        │
        │  on commit
        ▼
Build Agentic session evidence (subject = gitCommit)
        │
        ├─► optional: Alignment evidence vs intents policy
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


| Path                             | Description                                                                                                           |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `[spec/](./spec/)`               | Normative entities: session log, process log, AI process evidence, agent identifier, alignment evidence, runtime tool |
| `[README.md](./README.md)`       | Orientation and adoption guide (this file)                                                                            |


---

## Out of scope

- Agent authentication methods and credentials  
- Algorithms for intents analysis (only the evidence shape for alignment verdicts)

---

## Status

Working draft toward a shared **agentic session evidence** practice for SDLC governance. Feedback and implementations should align field names and predicate types with `[spec/](./spec/)` so evidence remains interoperable across harnesses and aggregators.