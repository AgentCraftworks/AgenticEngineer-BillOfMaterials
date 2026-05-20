# Agent Trust Posture — Four-Pillar Extension

> **Spec section:** 2 of 5  
> **Version:** 0.1.0

---

## 1. Overview

The Agent Trust Posture (ATP) is an extension to the AEBOM Core record that answers the four fundamental enterprise trust questions for every agent invocation. It is evaluated **before** execution begins (the pre-flight trust gate) and appended to the AEBOM record as a signed attestation.

ATP is explicitly modeled on the trust posture enterprises already maintain for human engineers. The four pillars correspond directly to four categories of human-workforce trust control:

| Pillar | Trust question | Human-workforce analogue |
|---|---|---|
| **1 — Identity** | Who is this agent, verifiably? | Background check + credential vetting + badge issuance |
| **2 — Least-Privilege** | Is this agent scoped to exactly what this task requires and nothing more? | Role-based access control with task-scoped grants |
| **3 — Safety Currency** | Has this agent passed its safety evaluations, and are those evaluations recent enough? | Annual mandatory security training + certification |
| **4 — Environment Integrity** | Is this agent running in a managed environment I control, and can I prove it? | Managed device policy + hardware-bound MFA |

An agent invocation MUST satisfy all four pillars before it is permitted to act. Failure of any pillar MUST result in one of:

- **Blocking** — the invocation is refused
- **Escalation** — the invocation is paused and routed to a human approver with the failing pillar clearly stated
- **Degradation** — the agent is permitted to continue in a reduced-scope mode that excludes the capability class that triggered the failure (only permissible when the pillar failure is Pillar 2 and the degraded scope is sufficient for the task)

---

## 2. Pillar 1 — Identity

### 2.1 What it proves

Pillar 1 establishes that the agent executing the task is cryptographically bound to a registered, audited identity — not a shared credential, impersonation, or unregistered process.

### 2.2 Required evidence

| Evidence element | Description | Technology reference |
|---|---|---|
| **OIDC workload identity token** | Short-lived JWT issued by the organization's identity provider, bound to the agent's registered client ID | GitHub Actions OIDC, Azure Workload Identity, Google Workload Identity Federation |
| **Entra Agent Identity registration** | The agent must be registered in the organization's Entra (or equivalent) directory as a non-human principal with a stable `agent_id` | Microsoft Entra Agent Identity (preview) |
| **Token binding attestation** | The token must be bound to the signing key of the execution environment, preventing token theft and replay | DPoP (RFC 9449), mTLS certificate binding |

### 2.3 AEBOM fields

The following fields in the AEBOM Core record MUST be populated to satisfy Pillar 1:

- `agent.agent_id` — MUST match the `sub` claim of the OIDC token
- `agent.identity_token_ref` — content-addressable hash of the token
- `agent.identity_provider` — issuer of the token (`iss` claim)
- `integrity.signing_key_id` — key used for AEBOM record signing MUST be the same key attested in the token binding

### 2.4 Trust Posture fields

```json
"identity": {
  "status": "pass" | "fail" | "degraded",
  "agent_id_verified": true,
  "token_type": "oidc-workload",
  "token_expiry": "<ISO 8601 UTC>",
  "token_binding_method": "dpop" | "mtls" | "none",
  "identity_provider_verified": true,
  "identity_provider": "<issuer URI>",
  "registration_ref": "<URI to agent directory entry>",
  "failure_reason": "<string if status != pass>"
}
```

### 2.5 Failure conditions

| Condition | Required response |
|---|---|
| No OIDC token present | Block |
| Token expired at invocation time | Block |
| Token `sub` does not match registered `agent_id` | Block |
| Token binding absent and environment is internet-accessible | Escalate |
| Agent not registered in directory | Block |
| Token issued by unrecognized identity provider | Block |

---

## 3. Pillar 2 — Least-Privilege

### 3.1 What it proves

Pillar 2 establishes that the agent has been granted exactly — and only — the permissions needed for the specific task it is about to perform. No standing permissions, no overly broad scopes, no ambient credential inheritance.

### 3.2 Scope derivation

Scopes for an invocation MUST be derived by one of:

1. **Policy engine evaluation** — the task descriptor is submitted to a policy engine (e.g., OPA, Cedar) that evaluates the applicable policy and returns the minimal scope set
2. **Human approval** — an authorized human explicitly selects the scopes from a presented list, with unused scopes defaulted to excluded
3. **Pipeline-declared scopes** — a CI/CD pipeline explicitly declares the scopes required, and those scopes are validated against a policy that caps maximum pipeline scope

Implementations MUST NOT use broad ambient credentials (e.g., `repo:*`, `admin:*`) unless explicitly required by the task and approved through a Tier 3 gate (see [`spec/03-capability-thresholds.md`](03-capability-thresholds.md)).

### 3.3 Scope audit trail

Every scope grant MUST be traceable to:

- The policy rule or human decision that authorized it
- The task context that justified it
- The expiry time after which the scope is automatically revoked

### 3.4 AEBOM fields

- `authorization.scope` — the exact list of scopes granted
- `authorization.authorization_mechanism` — how scopes were derived
- `authorization.authorization_ref` — reference to the policy evaluation or approval artifact

### 3.5 Trust Posture fields

```json
"least_privilege": {
  "status": "pass" | "fail" | "degraded",
  "scope_derivation_method": "policy-engine" | "human-approval" | "pipeline-declared",
  "scopes_granted": ["<scope>", ...],
  "scopes_requested_not_granted": ["<scope>", ...],
  "policy_ref": "<URI to policy evaluation artifact>",
  "scope_expiry": "<ISO 8601 UTC>",
  "standing_permissions_present": false,
  "failure_reason": "<string if status != pass>"
}
```

### 3.6 Failure conditions

| Condition | Required response |
|---|---|
| Requested scope not covered by any applicable policy | Block |
| `admin:*` or equivalent requested without Tier 3 gate | Block |
| Scope expiry would occur mid-task | Escalate |
| Standing permissions (not task-scoped) detected | Escalate |
| Policy engine unavailable | Block (fail-safe) |

---

## 4. Pillar 3 — Safety Currency

### 4.1 What it proves

Pillar 3 establishes that the underlying model powering the agent has been evaluated against recognized safety benchmarks, that those evaluations are sufficiently recent, and that the model passed the thresholds required for the task's capability tier.

### 4.2 Benchmark requirements

The following benchmarks are recognized in v0.1.0 of this specification. Implementations SHOULD use the most recently published evaluation available, subject to the recency constraint in Section 4.4.

| Benchmark | Publisher | Applicability |
|---|---|---|
| **MLCommons AILuminate** | MLCommons | General safety across hazard categories |
| **HELM Safety** | Stanford CRFM | Reasoning, instruction-following, and harm avoidance |
| **METR Autonomy Evaluation** | METR | Agentic autonomy and resource acquisition |
| **Frontier Safety Framework evals** | FMF member firms | Catastrophic capability thresholds |
| **Vendor Red-Team Summary** | Model provider | Provider-specific adversarial evaluation |

Implementations MAY recognize additional benchmarks through the `extensions` field with appropriate documentation.

### 4.3 Minimum pass thresholds by capability tier

See [`spec/03-capability-thresholds.md`](03-capability-thresholds.md) for complete tier definitions. Minimum safety scores required:

| Tier | MLCommons AILuminate | HELM Safety | METR Autonomy |
|---|---|---|---|
| **Tier 0** — Read-only, no persistence | ≥ 3.0 / 5.0 | ≥ 60% | Not required |
| **Tier 1** — Scoped writes, reversible | ≥ 3.5 / 5.0 | ≥ 70% | ≤ Level 2 |
| **Tier 2** — Broad writes, partially irreversible | ≥ 4.0 / 5.0 | ≥ 80% | ≤ Level 3 |
| **Tier 3** — Administrative, production, irreversible | ≥ 4.5 / 5.0 | ≥ 90% | ≤ Level 2 |

### 4.4 Recency requirement

Evaluations MUST be no older than:

| Tier | Maximum evaluation age |
|---|---|
| Tier 0 | 12 months |
| Tier 1 | 6 months |
| Tier 2 | 3 months |
| Tier 3 | 45 days |

When a model receives a patch update (minor version), the evaluation clock resets to the date of the evaluation of the patched version, not the original model.

### 4.5 AEBOM fields

- `agent.model_id` — identifies the evaluated model version
- `agent.model_provider` — the entity responsible for the evaluation

### 4.6 Trust Posture fields

```json
"safety_currency": {
  "status": "pass" | "fail" | "degraded",
  "model_id": "<model identifier>",
  "model_version": "<version string>",
  "evaluations": [
    {
      "benchmark": "mlcommons-ailuminate",
      "benchmark_version": "<version>",
      "score": 4.2,
      "score_max": 5.0,
      "evaluated_at": "<ISO 8601 UTC>",
      "evaluation_ref": "<URI to evaluation artifact>",
      "pass": true
    }
  ],
  "oldest_evaluation_age_days": 45,
  "tier_required": 2,
  "all_required_benchmarks_pass": true,
  "failure_reason": "<string if status != pass>"
}
```

### 4.7 Failure conditions

| Condition | Required response |
|---|---|
| Required benchmark not present in evaluation record | Block |
| Score below minimum for capability tier | Block |
| Evaluation older than maximum age for tier | Block |
| Model version evaluated differs from model version invoked | Block |
| Evaluation artifact not verifiable (signature invalid) | Block |
| Benchmark publisher not recognized | Escalate |

---

## 5. Pillar 4 — Environment Integrity

### 5.1 What it proves

Pillar 4 establishes that the agent is executing in an environment that meets the organization's managed environment policy: controlled egress, known software bill of materials, tamper-evident logging, and (at higher tiers) hardware attestation.

### 5.2 Environment classes

| Class | Description | Minimum tier |
|---|---|---|
| **Managed Cloud Runner** | Organization-controlled ephemeral runner in a cloud environment with defined egress policy | Tier 0 |
| **Attested Container** | Containerized environment with SLSA-compliant build provenance and runtime attestation | Tier 1 |
| **Hardened VM** | VM with CIS benchmark compliance, egress restriction, and audit logging | Tier 2 |
| **Trusted Execution Environment (TEE)** | Hardware-isolated execution environment with remote attestation (e.g., Intel TDX, AMD SEV-SNP) | Tier 3 |

### 5.3 Required controls by environment class

| Control | Managed Cloud | Attested Container | Hardened VM | TEE |
|---|---|---|---|---|
| Egress restricted to allowlist | SHOULD | MUST | MUST | MUST |
| Ephemeral (no persistent state across tasks) | SHOULD | MUST | MUST | MUST |
| Audit log tamper-evident | MUST | MUST | MUST | MUST |
| SBOM of runner environment available | SHOULD | MUST | MUST | MUST |
| Runtime attestation of environment | Not required | SHOULD | MUST | MUST |
| Hardware attestation | Not required | Not required | Not required | MUST |

### 5.4 Trust Posture fields

```json
"environment_integrity": {
  "status": "pass" | "fail" | "degraded",
  "environment_type": "managed-cloud" | "attested-container" | "hardened-vm" | "tee",
  "environment_id": "<stable environment class identifier>",
  "runner_id": "<instance identifier>",
  "egress_policy": "egress-restricted" | "full-egress" | "air-gapped",
  "ephemeral": true,
  "audit_log_ref": "<URI to tamper-evident audit log>",
  "environment_sbom_ref": "<URI to environment SBOM>",
  "attestation_ref": "<URI to runtime attestation>",
  "hardware_attestation_ref": "<URI to hardware attestation report (TEE only)>",
  "meets_tier_requirement": true,
  "failure_reason": "<string if status != pass>"
}
```

### 5.5 Failure conditions

| Condition | Required response |
|---|---|
| Environment class below minimum for capability tier | Block |
| Full egress detected on internet-accessible runner for Tier 2+ | Block |
| Persistent state from prior task detected | Escalate |
| Audit log not tamper-evident | Escalate |
| Attestation signature invalid or expired | Block |
| TEE required but not available | Block |

---

## 6. Composite Trust Posture Record

The complete ATP extension is appended to the AEBOM Core record under the `trust_posture` key:

```json
{
  "trust_posture": {
    "spec_version": "0.1.0",
    "evaluated_at": "<ISO 8601 UTC>",
    "evaluated_by": "<URI of policy engine or identity of human evaluator>",
    "overall_status": "pass" | "fail" | "degraded" | "blocked",
    "capability_tier": 0 | 1 | 2 | 3,
    "identity": { ... },
    "least_privilege": { ... },
    "safety_currency": { ... },
    "environment_integrity": { ... },
    "trust_posture_signature": "<base64-encoded signature over canonical JSON of this object>",
    "trust_posture_signing_key_id": "<key ID>"
  }
}
```

`overall_status` is computed as:

- `"pass"` — all four pillars pass at the required tier
- `"degraded"` — one pillar is degraded (only permissible per Section 1 conditions)
- `"blocked"` — any pillar fails without an applicable degradation or escalation path
- `"fail"` — evaluation could not be completed (internal error)

---

## 7. Re-evaluation Triggers

The trust posture MUST be re-evaluated (not just cached) when any of the following occur:

- The agent requests a scope not in the original `authorization.scope` list
- The execution duration exceeds 4 hours
- A human handoff occurs and the human modifies the task scope
- The environment is detected to have changed (e.g., runner restart, network policy change)
- The OIDC token is within 5 minutes of expiry

---

*Previous: [spec/01-aebom-core.md](01-aebom-core.md) | Next: [spec/03-capability-thresholds.md](03-capability-thresholds.md)*
