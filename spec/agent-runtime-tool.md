# Agent runtime tool

Monitors an agentic process and its sessions and on completion:

1. Collects all relevant [Session logs](./agentic-session-log.md) and enriches them with process data
2. Extracts provenance information
3. Uploads the session logs into remote persistence storage
4. Uploads [agentic process evidence](./agentic-process-evidence.md) referencing the uploaded agentic session logs

## Requirements

- Should be active on every agentic process intended for governance
- Emits or attaches [agentic process evidence](./agentic-process-evidence.md) with subject = the target entity of the process 

## Related entities


| Entity                                                    | Relationship                                                                                                       |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [Agentic session log](./agentic-session-log.md)           | Payload / timeline schema of the agentic session                                                                   |
| [Agentic process evidence](./agentic-process-evidence.md) | Provenance statement uploaded on the SDLC entity the process produced (commit, artifact, or release)               |
| [Alignment evidence](./alignment-evidence.md)             | An optional second statement on a session log(s) from the same commit-time collection indicating policy violations |
| [Agent identifier](./agent-identifier.md)                 | Provider stack recorded on logs and evidence                                                                       |


