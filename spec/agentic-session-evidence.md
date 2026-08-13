# Agentic Session Evidence

Provenance evidence on the agentic process. Enables rapid human and policy-as-code approval.

References Agentic process session logs and attests on the process in which they were gnerated, with the evidence **subject** typically a `gitCommit`.

## Subject

The entity produced or handled by the agentic process:

| Process | Subject |
|---|---|
| Code development / review | git commit |
| Version release approval | Application version |
| Artifact promotion | Artifact digest |

## Storage attributes

| Attribute | Value |
|---|---|
| **Type** | Evidence, Attestation artifact |
| **Contains** | Provenance on the agentic process, with verdict information when relevant and intents or summary when extracted|
| **Used for** | Compliance, policy checks, auditability, review, troubleshooting|
| **Usage** | Provenance evidence when auditors drill down into randomly selected releases |
| **Retention** | As long as release is required |

## Example (Developemnt session evidence on a Code Commit subject)

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    {
      "uri": "https://github.com/MYORG/evidence/commit/bf02510c8ce0b804a099797510af...c19325acfb979538c5b521304c83cde63892",
      "digest": {
        "gitCommit|sha256": "bf02510c8ce0b804a099797510af...c19325acfb979538c5b521304c83cde63892"
      }
    }
  ],
  "predicateType": "https://myorg.com/evidence/<agnetic-code-review|agnetic-dev-process>/v1",
  "predicate": {    
    "providers": [
      {
      "harness": 
        {"name": "cursor","version": "3.5.33"}
      ,
      "agent": 
        {"id": "stable-agent-id","name": "cursor-agent", "version": "1.0"}
      ,
      "languageModel": 
        {"inferenceProvider": "anthropic/claude-opus-4-8","resolved": "anthropic/claude-opus-4-8-20260701"}              
    }],
    "traceId": "trace id | ci_job_run_uri | vendor session id",
    "sessionsLogs": [
      {
        "uri": "session log url",
        "digest": "session log sha..."
      }
    ],
    
    "tools": [{"name":"tool-name", "version": "tool version"}],
    "contextArtifacts": [
      {
        "tags": ["policy"],
        "uri": "link-to-development-guideline",
        "data": "The complete artifact document inline", 
        "digest": "policy-sha256"
      },
      {
        "tags": ["policy"],
        "uri": "link-to-architecture-guidelines",
        "digest": "architecture-sha256"
      }
    ],
    "custom": {
        "uri": "https://github.com/MYORG/evidence/commit/bf02510c8ce0b804a099797510af...c19325acfb979538c5b521304c83cde63892",
        "digest": {
          "gitCommit": "bf02510c8ce0b804a099797510af...c19325acfb979538c5b521304c83cde63892"
        }
      },
      "requirements": [
        {
          "issue": "https://myorg.atlassian.net/browse/FIN-3678",
          "title": "my requirements subject"
        }
      ]
    },
    "result": "COMPLETED",
    "intents": ["short description"],
    "processSummary": "long text",
    "owner": "login|email",
    "reviewers": ["login|email"],
    "startTimestamp": "ISO 8601 timestamp",
    "endTimestamp": "ISO 8601 timestamp"
  },
  "createdAt": "2026-04-08T08:55:17.242Z",
  "createdBy": "dev-auto-reviewer"
}
```

`contextArtifacts[].data` should be omitted when `uri` is provided (and vice versa).

Provider identity fields are defined in [`agent-identifier.md`](./agent-identifier.md).

## Evidence subject object

| Field | Type | Description | Usages |
|---|---|---|---|
| `uri` | String | Required. URL of the subject | Locating the subject |
| `subject.digest` | Object | Required. Unique immutable identifier of the subject (e.g. gitCommit, artifact sha256) | Signature verification and immutability checks |

## Evidence classification and general information

| Field | Type | Description | Usages |
|---|---|---|---|
| `predicateType` | String | Required. Type of this evidence | Filtering and existence checks |
| `_type` | String | Required. Evidence schema: `https://in-toto.io/Statement/v1` | Schema type and version |
| `createdAt` | ISO 8601 timestamp | Required. Evidence creation timestamp | Freshness checks |
| `createdBy` | String | Required. Evidence creator | Optional permissions checks (prefer signing keys) |

## Evidence predicate object

| Field | Type | Description | Usages |
|---|---|---|---|
| `providers[]` | Object Array | Required. details of the agentic provider harness, agent and language model | Verify approved / whitelisted harness |
| `providers[].harness.name` | String | Required. Name of the agentic provider harness | Verify approved / whitelisted harness |
| `providers[].harness.version` | String | Required. Version of the agentic harness | Verify approved harness versions |
| `providers[].agent.id` | String | Stable agent id | Traceability |
| `providers[].agent.name` | String | Agent name | Check whitelisted agents |
| `providers[].agent.version` | String | Agent version | Check whitelisted agent versions |
| `providers[].languageModel.inferenceProvider` | String | Name and version of the LLM requested, if known | Check whitelisted LLMs |
| `providers[].languageModel.resolved` | String | Name and version of the LLM used, if known | Resolved model attribution |
| `traceId` | String | Unique identifier of the agentic process | Correlate to original logs; identify runner |
| `sessionsLogs` | Object array | Links to session logs (full agentic chat context) | Download logs for review |
| `sessionsLogs[].uri` | String | Location of the session log artifact | Download logs for review |
| `sessionsLogs[].digest` | String | Digest of the session log artifact | Mutability check |
| `tools` | Object array | Used tools | Check for blacklisted tools |
| `tools[].name` | String | Used tool name | Check for blacklisted tools |
| `tools[].version` | String | Used tool version | Check for blacklisted tool versions |
| `contextArtifacts` | Object array | Artifacts used by the process (policies, instructions, prior logs, guidelines, …) | Input provenance |
| `contextArtifacts[].tags` | String array | Labels of the artifact | Check use of corporate sources |
| `contextArtifacts[].uri` | String | Required unless `data` is given. Location of the context artifact | Check specific inputs used |
| `contextArtifacts[].data` | String | Required unless `uri` is given. Content of the context artifact | Review inputs inline |
| `contextArtifacts[].digest` | String | Digest of the context artifact | Mutability check |
| `custom` | Object | Custom information for the specific agentic process | Process-specific checks |
| `result` | String | Required. Short process completion state | Validate positive completion / verdict |
| `intents` | String array | Short descriptions of intents identified in the process | Human oversight; agentic policy validation |
| `processSummary` | String | Summarized description of the process | Human oversight; agentic policy validation |
| `owner` | String | Required. Login or email of the accountable user | Accountability; permissions |
| `reviewers` | String array | Logins or emails of human overseers (e.g. developer chatting during development or PR review) | Human oversight checks |
| `startTimestamp` | ISO 8601 timestamp | Required. Process start | Duration checks |
| `endTimestamp` | ISO 8601 timestamp | Required. Process end | Duration checks |

## Development process custom data

Optional. Custom attributes should allow adding and process-specific data (review, development, promotion, etc.).

| Field | Type | Description | Usages |
|---|---|---|---|
| `baseCommit.uri` | String | Code commit URI | Code repo validation |
| `baseCommit.digest.gitCommit` | String | Code commit SHA | Code SHA immutability |
| `requirements` | Array | Requirements issues | Checking that requirements exists and were used |
| `requirements.issue` | String | Task URL | Requirements system is approved |
| `requirements.title` | String | Task title | Display / review |
