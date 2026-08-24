# Customer success - Overview

## Scenario

An autonomous agent handles a customer interaction end-to-end

**Process = single session**

Process ends at a **case resolution**

## Whats the point of adding this

Control the support process wrt to questions like:

- Did the agent stay aligned with our policies (refunds, privacy, escalation)?
- Did it get the mandatory context — policy, ticket, order?
- Did it stay within the boundaries of its task?
- Which agent, model and tools handled this customer?
- Who is accountable, and was a human anywhere in the loop?

Control the supply chain risk introduced by the support agent (tools, models, MCP)

Keep the session long enough to audit and troubleshoot after the fact

## Mental Model

### Accountability

The support platform owns the process. A human is optional — only when policy routes the case to one

### Agentic session

The session is the support agent run on that ticket. Process and session are the same unit here

### Agentic process evidence

Its subject points to the **session log**
It holds the outcome of the case
It holds the data for policy checks and for answering the various checks
It holds the human owner

1. alignment check wrt support policies
2. mendatory context present in the session
3. alignment check wrt the ticket / order
4. human accountable information

## Artifacts

- **agentic-session-log** — one per support session
- **agentic-support-process-evidence** — on that session log
- **agentic-session-alignment-evidence** — optional, per session log; produced on the server

## See also

- `[implementation_details.md](./implementation_details.md)`
- `[objects_examples/](./objects_examples/)`
