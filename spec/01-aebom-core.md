# AEBOM Core — Per-Execution Provenance Record

> **Spec section:** 1 of 5  
> **Version:** 0.1.0

---

## 1. Purpose

The AEBOM Core defines the **mandatory fields** of every Agent Execution Bill of Materials record. A Core record captures the minimum information required to:

1. Identify the agent that acted
2. Identify the task for which it was invoked
3. Identify the human principal who authorized the invocation
4. Establish a cryptographically verifiable chain from invocation to action
5. Record the outcome and any artifacts produced

Every AEBOM record MUST conform to the JSON Schema at [`../schema/aebom-core.schema.json`](../schema/aebom-core.schema.json).

---

## 2. Record Lifecycle

```
Invocation Request
    │
    ▼
┌──────────────────────────────────┐
│  Pre-flight Trust Gate           │  ← Trust Posture evaluated here
│  (see spec/02-trust-posture.md)  │
└──────────────────────────────────┘
    │  (gate passes)
    ▼
AEBOM Record Opened
    │  aebom_id assigned
    │  invocation_timestamp set
    ▼
Agent Executes
    │  actions appended incrementally
    ▼
Execution Completes or Is Interrupted
    │  completion_timestamp set
    │  outcome set
    ▼
Record Signed and Anchored
    │  record_signature computed
    │  (optionally: anchored to transparency log)
    ▼
Record Stored in Audit Ledger
```

---

## 3. Core Field Specification

### 3.1 Record Identity

| Field | Type | Required | Description |
|---|---|---|---|
| `spec_version` | `string` | MUST | AEBOM specification version this record conforms to. MUST be `"0.1.0"`. |
| `aebom_id` | `string` | MUST | Globally unique identifier for this record. MUST be a UUID v4. |
| `record_type` | `string` | MUST | MUST be `"execution"`. Reserved values: `"test"`, `"simulation"`. |
| `created_at` | `string` | MUST | ISO 8601 UTC timestamp at which this record was created (pre-flight). |
| `schema_uri` | `string` | SHOULD | URI of the JSON Schema this record was validated against. |

### 3.2 Agent Identity

| Field | Type | Required | Description |
|---|---|---|---|
| `agent.agent_id` | `string` | MUST | Stable identifier for this agent. MUST match the `sub` claim of the agent's OIDC credential. |
| `agent.agent_name` | `string` | MUST | Human-readable display name for the agent (e.g., `"CodeReview-Agent-v2"`). |
| `agent.agent_version` | `string` | MUST | SemVer version of the agent software. |
| `agent.model_id` | `string` | MUST | Identifier of the underlying foundation model (e.g., `"openai/gpt-4o-2024-05-13"`). |
| `agent.model_provider` | `string` | MUST | Organization providing the model (e.g., `"OpenAI"`). |
| `agent.agent_type` | `string` | MUST | One of: `"code-review"`, `"code-generation"`, `"dependency-update"`, `"test-generation"`, `"deployment"`, `"security-scan"`, `"orchestrator"`, `"other"`. |
| `agent.identity_token_ref` | `string` | MUST | Reference to the OIDC token or Entra Agent Identity credential used to authenticate the agent. MUST be a content-addressable hash (`sha256:<hex>`). |
| `agent.identity_provider` | `string` | MUST | Identity provider that issued the credential (e.g., `"https://login.microsoftonline.com/<tenant>"`). |

### 3.3 Task Context

| Field | Type | Required | Description |
|---|---|---|---|
| `task.task_id` | `string` | MUST | Identifier of the task or work item this invocation services (e.g., GitHub issue number, Jira ticket). |
| `task.task_type` | `string` | MUST | Structured task classification (e.g., `"pull-request-review"`, `"dependency-patch"`, `"ci-fix"`). |
| `task.task_description` | `string` | SHOULD | Human-readable description of the task (max 512 chars). |
| `task.repository` | `string` | SHOULD | Repository URI the task operates on (e.g., `"https://github.com/org/repo"`). |
| `task.branch` | `string` | SHOULD | Git branch in scope. |
| `task.commit_sha` | `string` | SHOULD | Git commit SHA at invocation time. |

### 3.4 Authorization

| Field | Type | Required | Description |
|---|---|---|---|
| `authorization.authorizing_principal` | `string` | MUST | Identity of the human (or policy engine) that authorized this invocation. For human authorization, this MUST be a verified identity reference (e.g., Entra UPN). |
| `authorization.authorization_mechanism` | `string` | MUST | One of: `"human-approval"`, `"policy-engine"`, `"scheduled-pipeline"`, `"ci-trigger"`. |
| `authorization.authorization_timestamp` | `string` | MUST | ISO 8601 UTC timestamp of authorization. |
| `authorization.authorization_ref` | `string` | SHOULD | Content-addressable reference to the authorization artifact (approval record, policy evaluation log). |
| `authorization.scope` | `array<string>` | MUST | Explicit list of scopes granted for this invocation (e.g., `["repo:read", "pr:write", "ci:trigger"]`). See [Section 3.6](#36-scope-vocabulary). |

### 3.5 Execution Record

| Field | Type | Required | Description |
|---|---|---|---|
| `execution.invocation_timestamp` | `string` | MUST | ISO 8601 UTC timestamp at which the agent began execution. |
| `execution.completion_timestamp` | `string` | SHOULD | ISO 8601 UTC timestamp at which the agent completed or was interrupted. |
| `execution.outcome` | `string` | SHOULD | One of: `"success"`, `"failure"`, `"partial"`, `"interrupted"`, `"in-progress"`. |
| `execution.outcome_detail` | `string` | SHOULD | Human-readable summary of outcome (max 1024 chars). |
| `execution.actions` | `array<Action>` | MUST | Ordered list of discrete actions taken. See [Section 3.7](#37-action-record). |
| `execution.artifacts_produced` | `array<Artifact>` | SHOULD | List of artifacts created or modified. See [Section 3.8](#38-artifact-record). |
| `execution.human_handoffs` | `array<Handoff>` | SHOULD | Human-in-the-loop checkpoints triggered. See [Section 3.9](#39-handoff-record). |
| `execution.environment` | `object` | MUST | Runtime environment descriptor. See [Section 3.10](#310-environment-descriptor). |

### 3.6 Scope Vocabulary

Scopes follow the pattern `<resource>:<permission>`. Implementations SHOULD use the following base vocabulary and MAY extend it with `x-` prefixed values:

| Scope | Meaning |
|---|---|
| `repo:read` | Read repository contents |
| `repo:write` | Write to repository (commits, branches) |
| `pr:read` | Read pull requests |
| `pr:write` | Create, update, or comment on pull requests |
| `pr:merge` | Merge pull requests |
| `issue:read` | Read issues |
| `issue:write` | Create or update issues |
| `ci:read` | Read CI pipeline results |
| `ci:trigger` | Trigger CI pipeline runs |
| `ci:configure` | Modify CI pipeline configuration |
| `dependency:read` | Read dependency manifests |
| `dependency:write` | Modify dependency manifests |
| `secret:read` | Read secrets or environment variables |
| `deploy:staging` | Deploy to staging environment |
| `deploy:production` | Deploy to production environment |
| `audit:read` | Read audit logs |
| `admin:*` | Broad administrative permissions (requires Tier 3 gate) |

### 3.7 Action Record

Each entry in `execution.actions` MUST conform to:

| Field | Type | Required | Description |
|---|---|---|---|
| `action_id` | `string` | MUST | UUID v4 uniquely identifying this action within the execution. |
| `action_type` | `string` | MUST | Structured action type (e.g., `"git-commit"`, `"pr-comment"`, `"api-call"`, `"file-write"`, `"tool-invocation"`). |
| `timestamp` | `string` | MUST | ISO 8601 UTC timestamp. |
| `target` | `string` | SHOULD | Resource URI or identifier the action acted upon. |
| `summary` | `string` | MUST | Human-readable description of the action (max 256 chars). |
| `reversible` | `boolean` | MUST | Whether this action can be reversed. |
| `tool_name` | `string` | SHOULD | If a tool was invoked, the tool's registered name. |
| `tool_version` | `string` | SHOULD | Version of the tool invoked. |
| `input_hash` | `string` | SHOULD | `sha256:<hex>` of the input provided to this action. |
| `output_hash` | `string` | SHOULD | `sha256:<hex>` of the output produced by this action. |

### 3.8 Artifact Record

Each entry in `execution.artifacts_produced` MUST conform to:

| Field | Type | Required | Description |
|---|---|---|---|
| `artifact_id` | `string` | MUST | UUID v4. |
| `artifact_type` | `string` | MUST | One of: `"pull-request"`, `"commit"`, `"comment"`, `"file"`, `"report"`, `"test-result"`, `"sbom"`, `"other"`. |
| `uri` | `string` | MUST | Resolvable URI for the artifact. |
| `content_hash` | `string` | MUST | `sha256:<hex>` of the artifact content at time of creation. |
| `created_at` | `string` | MUST | ISO 8601 UTC timestamp. |
| `description` | `string` | SHOULD | Human-readable description (max 256 chars). |

### 3.9 Handoff Record

Each entry in `execution.human_handoffs` MUST conform to:

| Field | Type | Required | Description |
|---|---|---|---|
| `handoff_id` | `string` | MUST | UUID v4. |
| `handoff_type` | `string` | MUST | One of: `"approval-required"`, `"review-requested"`, `"escalation"`, `"interrupt"`. |
| `triggered_at` | `string` | MUST | ISO 8601 UTC timestamp when the handoff was triggered. |
| `trigger_reason` | `string` | MUST | Reason the handoff was triggered (max 512 chars). |
| `assignee` | `string` | SHOULD | Identity reference of the human assigned. |
| `resolved_at` | `string` | SHOULD | ISO 8601 UTC timestamp when the handoff was resolved. |
| `resolution` | `string` | SHOULD | One of: `"approved"`, `"rejected"`, `"modified"`, `"timed-out"`, `"pending"`. |
| `resolution_note` | `string` | SHOULD | Human-provided note on resolution (max 512 chars). |

### 3.10 Environment Descriptor

The `execution.environment` object MUST contain:

| Field | Type | Required | Description |
|---|---|---|---|
| `environment_id` | `string` | MUST | Stable identifier for the execution environment class. |
| `environment_type` | `string` | MUST | One of: `"managed-cloud"`, `"self-hosted-runner"`, `"container"`, `"vm"`, `"tee"` (Trusted Execution Environment). |
| `runner_id` | `string` | SHOULD | Instance identifier of the runner (e.g., GitHub Actions runner ID). |
| `platform` | `string` | MUST | Operating platform (e.g., `"linux/amd64"`, `"windows/amd64"`). |
| `network_policy` | `string` | SHOULD | One of: `"egress-restricted"`, `"full-egress"`, `"air-gapped"`. |
| `isolation_attestation_ref` | `string` | SHOULD | Reference to attestation that the environment meets the claimed isolation properties. |

### 3.11 Record Integrity

| Field | Type | Required | Description |
|---|---|---|---|
| `integrity.record_hash` | `string` | MUST | `sha256:<hex>` of the canonical JSON serialization of this record (excluding the `integrity` field itself). |
| `integrity.record_signature` | `string` | MUST | Base64-encoded signature over `integrity.record_hash`, produced by the signing key. |
| `integrity.signing_key_id` | `string` | MUST | Key ID of the signing key. MUST be resolvable to a public key via the agent's JWKS endpoint. |
| `integrity.signing_algorithm` | `string` | MUST | JWA algorithm identifier (e.g., `"ES256"`). |
| `integrity.transparency_log_ref` | `string` | SHOULD | Reference to the Rekor or equivalent transparency log entry for this record. |

---

## 4. Canonical Serialization

Records MUST be serialized as JSON following RFC 8259. When computing `integrity.record_hash`:

1. Serialize the record to JSON with all fields sorted alphabetically by key (recursive)
2. Remove the `integrity` top-level key entirely
3. Compute SHA-256 of the resulting UTF-8 byte string
4. Encode as `sha256:<lowercase-hex>`

---

## 5. Record Storage Requirements

Implementations MUST store AEBOM records such that:

- Records are **immutable** after the `integrity.record_signature` field is populated
- Records are **retained** for a minimum of 7 years (or the jurisdiction's applicable audit retention period, whichever is longer)
- Records are **queryable** by `aebom_id`, `agent.agent_id`, `task.task_id`, and `authorization.authorizing_principal` within 30 seconds
- Records are **accessible** to designated auditors without requiring access to the systems the agent acted upon

---

## 6. Versioning

The `spec_version` field identifies the version of this specification the record conforms to. This specification uses SemVer. Records conforming to a given major version MUST be processable by any compliant parser for that major version.

---

*Next: [spec/02-trust-posture.md](02-trust-posture.md)*
