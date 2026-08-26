# Human-in-the-loop code development - Overview

## Scenario

A human develops a feature using coding-agents

Multiple agent sessions may touch the code 

Process ends at a **commit ready for pre-merge review**

## Whats the point of adding this

Control your SDLC wrt to questions like:

- Did the agent stay aligned with our policies?
- Did it get the mandatory context — guidelines, requirements, architecture?
- Did it stay within the boundaries of its task?
- Which sessions, agents, models and tools built the thing we are about to release?
- Who is accountable, and was a human anywhere in the loop?

Control the supply chain risk introduced by the agents framework

## Mental Model

### Accountability

The developer is both the process owner and the process reviewer

### Agentic session

The sessions are the coding agent sessions, chat, interactive_run, including subagents

### Agentic process evidence

Its subject points to the git commit (as the SDLC entity)  
It holds the reference to the session logs that affected this commit  
It will hold the data for supply chain analysis, and for answering the various checks:  
It holds the human owner and reviwer (the same person in this case)

1. alignment check wrt policies
2. mendatory context present in all sessions
3. alignment check wrt task information
4. human accountable information

## Artifacts

- **agentic-session-log** — one per agentic session (incl. subagents)
- **agentic-dev-process-evidence** — on the git commit
- **agentic-session-alignment-evidence** — optional, per session log; produced on the server

## See also

- `[implementation_details.md](./implementation_details.md)`
- `[objects_examples/](./objects_examples/)`

