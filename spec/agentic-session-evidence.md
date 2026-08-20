# Agentic Session Evidence

Provenance evidence on the agentic process. Enables rapid human and policy-as-code approval.

References Agentic process session logs and attests on the process in which they were generated, with the evidence **subject** typically a `gitCommit`.

JSON field names defined by this standard use **camelCase**.

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

## Example (Development session evidence on a Code Commit subject)

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [
    {
      "uri": "https://github.com/MYORG/evidence/commit/bf02510c8ce0b804a099797510af...c19325acfb979538c5b521304c83cde63892",
      "digest": {
        "gitCommit": "bf02510c8ce0b804a099797510af...c19325acfb979538c5b521304c83cde63892"
      }
    }
  ],
  "predicateType": "https://myorg.com/evidence/<agentic-code-review|agentic-dev-process>/v1",
  "predicate": {
    "providers": [
      {
        "harness": {
          "name": "cursor",
          "version": "3.5.33"
        },
        "agent": {
          "id": "stable-agent-id",
          "name": "cursor-agent",
          "version": "1.0"
        },
        "languageModels": [
          {
            "inferenceProvider": "anthropic/claude-opus-4-8",
            "resolved": "anthropic/claude-opus-4-8-20260701"
          }
        ]
      }
    ],
    "traceId": "trace id | ci_job_run_uri | vendor session id",
    "sessionsLogs": [
      {
        "uri": "session log url",
        "digest": {
          "sha256": "session log sha..."
        }
      }
    ],
    "tools": [{"name": "tool-name", "version": "tool version"}],
    "contextArtifacts": [
      {
        "tags": ["policy"],
        "uri": "link-to-development-guideline",
        "data": "The complete artifact document inline",
        "digest": {
          "sha256": "policy-sha256"
        }
      },
      {
        "tags": ["guideline"],
        "uri": "link-to-architecture-guidelines",
        "digest": {
          "sha256": "architecture-sha256"
        }
      }
    ],
    "custom": {
      "baseCommit": {
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

**Required** in the tables below: `yes` = must be present and non-empty; `no` = optional; `conditional` = see Constraints. **MUST**, **SHOULD**, **MAY**, and related words follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) as stated in [README Conventions](../README.md#conventions).

## Digest set

`digest` is a **DigestSet**: a JSON object mapping algorithm name to a lowercase hex-encoded digest string. At least one entry is required.

On `subject[].digest` this is the in-toto Statement v1 type. Predicate fields that name a digest (`sessionsLogs[].digest`, `contextArtifacts[].digest`, `baseCommit.digest`) reuse the same shape so every digest in this spec is one type.

```json
{
  "gitCommit": "bf02510c8ce0b804a099797510afc19325acfb979538c5b521304c83cde63892"
}
```

```json
{
  "sha256": "9f2b8c1d7a4530e6b8d0c7f1a2934ee41a"
}
```

| Algorithm key | Value | When to use |
|---|---|---|
| `gitCommit` | Git object id, as `git rev-parse` reports it | Git commit subjects |
| `sha256` | SHA-256 of the artifact bytes | Generic artifacts (session logs, files, blobs) |

Additional algorithm keys MAY be present (`sha512`, `sha1`, …) with the same meaning as in-toto. Unknown keys MUST be ignored by consumers that do not understand them.

A DigestSet is not a single string.

## Resource descriptor

A digest-linked reference to an artifact. Same idea as in-toto `ResourceDescriptor` (`uri` + `digest`); this spec uses only those two fields.

```json
{
  "uri": "https://myorg.jfrog.io/artifactory/agentic-session-logs/a1b2c3d4-e5f6-7890-abcd-ef1234567890.json",
  "digest": {
    "sha256": "9f2b8c1d7a4530e6b8d0c7f1a2934ee41a"
  }
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `uri` | String | yes | URI | Location of the artifact |
| `digest` | DigestSet | yes | ≥1 algorithm key; see [Digest set](#digest-set) | Immutable identity of the bytes at `uri` |

Used as `predicate.sessionsLogs` (ResourceDescriptor array) and as `custom.baseCommit`. `subject[]` uses the same `uri` + `digest` pair because that is in-toto Statement v1.

## Tool

A tool used during the agentic process (IDE action, MCP server, CLI, etc.).

```json
{
  "name": "Write",
  "version": "1.0"
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `name` | String | yes | non-empty | Tool name as the harness reports it |
| `version` | String | no | non-empty when present | Tool version as the producer reports it (not necessarily SemVer) |

Used as `predicate.tools` (Tool array) on [agentic session evidence](./agentic-session-evidence.md), and as a searchable attribute on [agentic session log](./agentic-session-log.md).

## Context artifact

A policy, guideline, instruction, or other document the process used as input. Extends a [Resource descriptor](#resource-descriptor) with labels and an optional inline body.

```json
{
  "tags": ["policy"],
  "uri": "https://myorg.jfrog.io/artifactory/policies/intents-policy-v4.md",
  "digest": {
    "sha256": "41c0e93b6d287fa5029b8c1d64"
  }
}
```

| Field | Type | Required | Constraints | Description |
|---|---|---|---|---|
| `tags` | String array | no | values SHOULD be from [Context artifact tags](#context-artifact-tags); additional tags allowed | Labels of the artifact |
| `uri` | String | conditional | URI; exactly one of `uri` or `data` | Location of the artifact |
| `data` | String | conditional | exactly one of `uri` or `data` | Inline content of the artifact |
| `digest` | DigestSet | no | ≥1 algorithm key when present; SHOULD include `sha256`; see [Digest set](#digest-set) | Immutable identity of the artifact bytes |

Used as `predicate.contextArtifacts` (ContextArtifact array).

### Context artifact tags

Recommended values for `tags`. Other tags MAY be used.

| Value | Meaning |
|---|---|
| `policy` | Organizational policy the process was expected to follow |
| `guideline` | Architecture or development guideline |
| `instructions` | Prompt, system, or agent instructions |
| `priorLog` | Prior session log used as input |

## Evidence subject object

| Field | Type | Required | Constraints | Description | Usages |
|---|---|---|---|---|---|
| `uri` | String | yes | URI | URL of the subject | Locating the subject |
| `digest` | DigestSet | yes | ≥1 algorithm key; see [Digest set](#digest-set) | Unique immutable identifier of the subject | Signature verification and immutability checks |

## Evidence classification and general information

| Field | Type | Required | Constraints | Description | Usages |
|---|---|---|---|---|---|
| `predicateType` | String | yes | URI identifying this evidence kind | Type of this evidence | Filtering and existence checks |
| `_type` | String | yes | exactly `https://in-toto.io/Statement/v1` | Evidence schema | Schema type and version |
| `createdAt` | Timestamp | yes | ISO 8601 | Evidence creation timestamp | Freshness checks |
| `createdBy` | String | yes | non-empty | Evidence creator | Optional permissions checks (prefer signing keys) |

## Evidence predicate object

| Field | Type | Required | Constraints | Description | Usages |
|---|---|---|---|---|---|
| `providers` | Provider array | yes | ≥1; see [Provider](./agent-identifier.md) | Agentic provider stacks used in the process | Verify approved / whitelisted harness |
| `traceId` | String | no | non-empty when present | Unique identifier of the agentic process | Correlate to original logs; identify runner |
| `sessionsLogs` | ResourceDescriptor array | no | 0..*; see [Resource descriptor](#resource-descriptor); each `digest` SHOULD include `sha256` | Links to session logs (full agentic chat context) | Download logs for review |
| `tools` | Tool array | no | 0..*; see [Tool](#tool) | Used tools | Check for blacklisted tools |
| `contextArtifacts` | ContextArtifact array | no | 0..*; see [Context artifact](#context-artifact) | Artifacts used by the process (policies, instructions, prior logs, guidelines, …) | Input provenance |
| `custom` | Object | no | process-specific keys | Custom information for the specific agentic process | Process-specific checks |
| `result` | String | yes | [Session result](#session-result) | Short process completion state | Validate positive completion / verdict |
| `intents` | String array | no | 0..* | Short descriptions of intents identified in the process | Human oversight; agentic policy validation |
| `processSummary` | String | no | | Summarized description of the process | Human oversight; agentic policy validation |
| `owner` | String | yes | login or email | Login or email of the accountable user | Accountability; permissions |
| `reviewers` | String array | no | 0..*; each login or email | Logins or emails of human overseers (e.g. developer chatting during development or PR review) | Human oversight checks |
| `startTimestamp` | Timestamp | yes | ISO 8601 | Process start | Duration checks |
| `endTimestamp` | Timestamp | yes | ISO 8601 | Process end | Duration checks |

## Session result

Closed set for `predicate.result` on agentic session evidence (development, review, promotion, and similar process evidence — not alignment evidence).

| Value | Meaning |
|---|---|
| `COMPLETED` | Process finished successfully |
| `FAILED` | Process ended unsuccessfully |
| `CANCELLED` | Process stopped before completion |

## Development process custom data

Optional. Custom attributes should allow adding any process-specific data (review, development, promotion, etc.).

| Field | Type | Required | Constraints | Description | Usages |
|---|---|---|---|---|---|
| `baseCommit` | ResourceDescriptor | no | see [Resource descriptor](#resource-descriptor); `digest` SHOULD include `gitCommit` | Base code commit | Code repo / SHA immutability |
| `requirements` | Object array | no | 0..* | Requirements issues | Checking that requirements exist and were used |
| `requirements[].issue` | String | yes | URI | Task URL | Requirements system is approved |
| `requirements[].title` | String | no | | Task title | Display / review |
