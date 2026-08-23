# Agent runtime tool

Monitors agent sessions and, on a **commit event**:

1. Collects all relevant [Session logs](./agentic-session-log.md)
2. Extracts provenance information
3. Uploads the session logs into remote persistence storage
4. Uploads git-commit [agentic process evidence](./agentic-process-evidence.md) referencing the uploaded session logs

## Requirements

- Should be active on every agentic SDLC flow intended for governance
- Emits or attaches [agentic process evidence](./agentic-process-evidence.md) with subject = the commit produced by the process (all sessions that contributed to that commit)

## Related entities

| Entity | Relationship |
|---|---|
| [Agentic session log](./agentic-session-log.md) | Payload / timeline schema for the session artifacts |
| [Agentic process evidence](./agentic-process-evidence.md) | Provenance statement uploaded on the SDLC entity the process produced (commit, artifact, or release) |
| [Alignment evidence](./alignment-evidence.md) | An optional second statement on a session log from the same commit-time collection indicating policy violations |
| [Agent identifier](./agent-identifier.md) | Provider stack recorded on logs and evidence |
