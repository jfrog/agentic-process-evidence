# Agent runtime tool

Monitors the agent session and, on a **commit event**:

1. Collects all relevant [Session logs](./session-log.md)
2. Extracts provenance information
3. Uploads the session logs into remote persistence storage
4. Uploads git-commit agentic evidence referencing the uploaded session logs

## Requirements

- Should be active on every agentic SDLC flow intended for governance
- Emits or attaches [AI Session evidence](./agnetic-session-evidence.md) with subject = the commit produced by the session

## Related entities

| Entity | Relationship |
|---|---|
| [Agentic session log](./agentic-session-log.md) | Payload / timeline schema for the session artifacts |
| [Agentic session evidence](./agnetic-session-evidence.md) | Provenance statement uploaded on SDLC entity such as commit, artifact or release |
| [Alignment evidence](./alignment-evidence.md) | An optional second statement on a session log from the same commit-time collection indicating policy violations |
| [Agent identifier](./agent-identifier.md) | Provider stack recorded on logs and evidence |
