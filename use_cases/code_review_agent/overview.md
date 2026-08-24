# Code review agent - Overview

## Scenario

The **code review agent** reviews a change and classifies it

Typically one review session

Process ends at a **review verdict on a git commit**

**Negligible** changes auto-merge — no human in that loop

## Whats the point of adding this

Less human in the loop for code reviews, without dropping regulatory rigor.

Automerge **NEGLIGIBLE** changes. Keep signed evidence, session logs, alignment, and a named owner so the merge is still auditable.

That is the rigor that makes auto-merge defensible.

Control the supply chain risk introduced by the review agent itself

## Mental Model

### Accountability

The review platform owns the process. A human reviewer is skipped when the change is classified negligible and auto-merged — the evidence is what remains in the loop

### Agentic session

The session is the code review agentic runs (CI step), including subagents

### Agentic process evidence

Its subject points to the git commit under review (as the SDLC entity)  
It holds the reference to the session logs of that review  
It holds the review verdict and the change classification (incl. negligible / auto-merge)  
It holds the human owner (platform). Reviewers empty as this is an autonomous agentic process.

1. alignment check wrt review rubric
2. mendatory context present in the review session
3. alignment check wrt the change / requirements
4. human accountable information — or an explicit decision that none was required for this class of change

## Artifacts

- **agentic-session-log** — one per review session (incl. subagents)
- **agentic-code-review-evidence** — on the git commit under review
- **agentic-session-alignment-evidence** — optional, per session log; produced on the server

## See also

- `[implementation_details.md](./implementation_details.md)`
- `[objects_examples/](./objects_examples/)`

