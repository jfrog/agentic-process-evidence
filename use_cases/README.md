# Use cases

Business goals an agentic **process** achieves. Normative field definitions stay in `[spec/](../spec/)`.


| Use case                                                               | Process shape                                 | Subject               |
| ---------------------------------------------------------------------- | --------------------------------------------- | --------------------- |
| [Code development](./code_development/overview.md) | Human + IDE agent; many sessions → one commit | git commit            |
| [Code review agent](./code_review_agent/overview.md)                   | Review agent; automerge negligible; one session | git commit (reviewed) |
| [Customer success](./customer_success/overview.md)                     | Autonomous agent; process = one session       | session log           |


Each folder:

1. `overview.md` → scenario, mental model, artifacts
2. `implementation_details.md` → how to implement APE for this scenario (runtime tool + server)
3. `objects_examples/` → examples for the output objects (sample artifacts)
