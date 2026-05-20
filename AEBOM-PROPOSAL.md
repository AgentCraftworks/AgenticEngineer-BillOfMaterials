# Agent Execution Bill of Materials (AEBOM)
## A Provenance and Trust Standard for Autonomous Agentic Workflows

**Status:** DRAFT — Proposal for Industry Review  
**Authors:** AgentCraftworks / AICraftworks, LLC  
**Date:** May 2026  
**Contact:** standards@agentcraftworks.com

---

## Executive Summary

The rapid deployment of autonomous AI agents in enterprise software engineering workflows has outpaced the governance tooling available to assess, audit, and assure the trustworthiness of those agents and their actions. Existing standards — Software Bills of Materials (SBOM), SPDX 3.0 AI Profile, and supply-chain provenance frameworks like SLSA — address *what was built* and *what model was deployed*. None address the question enterprises actually face in production: **can I prove, after the fact, that the agent which took action on my systems was who it claimed to be, had only the permissions the task required, was current on its safety evaluations, and ran in an environment I control?**

These are not new questions. They are precisely the questions enterprise security and compliance teams have answered for human engineers for decades: background vetting before access is granted, principle of least privilege enforced and continuously verified, annual security training currency, and hardware-bound multi-factor authentication to prevent impersonation. **The Agent Execution Bill of Materials (AEBOM) is the structural answer to those same four questions, applied to agentic engineering partners.**

This proposal defines:
1. A schema specification for per-execution provenance records (the AEBOM core)
2. A four-pillar **Agent Trust Posture** extension that answers the four human-equivalent trust questions for every agent invocation
3. Capability thresholds and trust gates that determine when an agent may operate
4. Organizational considerations for governance, separation of duties, and human accountability
5. Alignment to existing regulatory frameworks (EU AI Act, NIST AI RMF, ISO 42001) and relationship to frontier AI safety frameworks published by FMF member firms

AEBOM is designed to be implementable today using available technology — Microsoft Entra Agent Identity, OIDC workload federation, MLCommons AILuminate benchmarks, and existing handoff state machine infrastructure — while defining an extensible schema that can incorporate future advances in trusted execution environments and interpretability methods.

---

## 1. Introduction and Problem Statement

### 1.1 The Governance Gap in Agentic AI

Frontier AI frameworks, as documented by the Frontier Model Forum, have established a foundational vocabulary for managing risks from frontier models: risk identification, capability thresholds, capability assessments, risk mitigation, and risk governance. These frameworks focus appropriately on *model-level* properties — what a model can do, whether it crosses capability thresholds, and what safeguards are required before deployment.

However, a new risk surface has emerged that frontier AI frameworks do not address: **the per-execution risk profile of autonomous AI agents operating within enterprise workflows**. When an agent executes a handoff — accepting a task, invoking tools, making governance decisions, and producing outputs — no existing standard captures whether:

- The agent that ran was the one registered to run (identity integrity)
- The agent operated with only the permissions the task required (least-privilege verification)
- The agent's safety evaluations were current at the time of execution (certification currency)
- The execution environment was protected and audited (environmental attestation)

This gap exists not because the questions are new, but because the *artifacts* to answer them for agents do not yet exist. AEBOM is that artifact.

### 1.2 Why Existing Standards Are Insufficient

| Standard | Scope | Gap |
|---|---|---|
| **SBOM / SPDX 3.0** | Software component inventory | Static; describes what was built, not what executed |
| **SPDX 3.0 AI Profile** | AI model and dataset provenance | Model-level; no per-execution or governance-decision record |
| **CycloneDX ML-BOM** | ML model artifacts | Describes models, not agent decisions or permission chains |
| **SLSA (Supply-chain Levels)** | Build-time artifact provenance | Verifies *how something was built*; silent on runtime |
| **W3C PROV** | General provenance | Generic; no agent-role, governance-gate, or trust-posture semantics |
| **OpenTelemetry / Traces** | Observability spans | Operational telemetry; not a compliance artifact, no accountability chain |
| **FMF Frontier Frameworks** | Frontier model risk governance | Model-level thresholds; no per-invocation accountability record |

The structural absence is not a criticism of existing standards — each addresses its intended scope correctly. AEBOM is not a replacement for any of them. It is a *complementary layer* that sits at the execution record level, consuming outputs from identity systems (Entra Agent ID), safety evaluation systems (AILuminate), and governance gates (engagement level framework) to produce a unified per-handoff trust record.

### 1.3 The Human Engineer Accountability Analogy

Enterprise security and compliance for human engineers rests on four pillars that are well understood, widely implemented, and regularly audited:

| Human Engineer Requirement | Purpose | Verification Mechanism |
|---|---|---|
| **Pre-hire background check** | Verify identity and history before granting access | Third-party screening, credential verification |
| **Principle of least privilege + continuous access review** | Ensure employees only have access required for their role | PAM systems, quarterly access reviews, JIT access |
| **Annual security training** | Maintain current knowledge of threats and policies | Training completion records, certification expiry |
| **Multi-factor authentication** | Prevent credential theft and impersonation | Hardware-bound FIDO2, device binding |

Agentic engineering partners operating in enterprise software workflows require equivalent assurances. The AEBOM Agent Trust Posture maps each human pillar to an agent equivalent, using mechanisms appropriate to the nature of software agents rather than human employees.

---

## 2. The AEBOM Framework

### 2.1 Core Principles

**P1 — Per-Execution Granularity.** An AEBOM record is generated for every agent handoff. Aggregate or batch records are insufficient for compliance purposes; accountability must trace to individual execution events.

**P2 — Immutability After Completion.** Once an AEBOM record is finalized (handoff reaches `completed` or `failed`), it must not be modifiable. Hash-chaining or append-only storage is required.

**P3 — Human Accountability Is Non-Negotiable.** Every AEBOM record must identify the human principal who authorized the agent's engagement level. The agent is accountable for its actions; the human is accountable for granting the agent the level at which those actions were permitted.

**P4 — Trust Gates Are Blocking, Not Advisory.** If any pillar of the Agent Trust Posture fails (certification expired, environment unverified, identity tampered), the agent MUST NOT execute. Trust gates are preconditions, not warnings.

**P5 — Least Disclosure.** AEBOM records contain the minimum information necessary to establish accountability. They do not capture raw inputs, outputs, or data processed — only provenance, permission, certification status, and attestation.

**P6 — Iterative Schema Evolution.** As assessment methodologies advance, the AEBOM schema will evolve. Version tagging is required on all records. Consumers must handle forward-compatibility gracefully.

### 2.2 Scope and Applicability

AEBOM applies to any autonomous or semi-autonomous agent that:
- Accepts a task from a human or another agent
- Invokes tools or takes actions with external effects
- Operates within a defined engagement level (L1–L5 / Observer–Autonomous)
- Produces outputs that influence downstream decisions or systems

AEBOM does not apply to:
- Purely read-only information retrieval (no external effects)
- Human-in-the-loop interactions where the human reviews every action before execution
- Model training or evaluation runs (covered by SPDX 3.0 AI Profile and FMF frameworks)

### 2.3 Relationship to Existing Standards

```
SLSA             → Verifies how the agent software was built (build provenance)
SPDX 3.0 AI Profile → Describes the model the agent is based on (model provenance)
FMF Frameworks   → Defines what the model is capable of and what safeguards apply (model governance)
OpenTelemetry    → Captures what the agent did at the operation level (observability)
──────────────────────────────────────────────────────────────────────────────────
AEBOM            → Records who the agent was, whether it was trustworthy to act,
                   what governance decisions were made, and what the outcome was (execution accountability)
```

AEBOM is designed to *consume* artifacts from the layers above it, not replace them. The `model_fingerprint` field references the SPDX record. The `safety_certification.benchmarks_passed` field references AILuminate results. The `provenance.registry_source` references the Foundry allowlist entry. AEBOM is the binding layer.

---

## 3. Schema Specification

### 3.1 Core AEBOM Record

```typescript
interface AgentExecutionBOM {
  // Identity
  handoff_id: string;             // UUID — globally unique
  aebom_version: '1.0';
  schema_hash: string;            // SHA-256 of this schema version (schema integrity)
  generated_at: string;           // ISO 8601 timestamp
  finalized_at?: string;          // Set when handoff reaches terminal state

  // Agent Participation
  agents: AgentParticipation[];

  // Tool Invocations
  tool_invocations: ToolInvocation[];

  // Governance Events
  governance_events: GovernanceEvent[];

  // Cross-Squad Handoff Chains (S2S)
  s2s_chains?: S2SChain[];

  // Outcome
  outcome: ExecutionOutcome;

  // Trust Posture (see §3.2)
  agent_trust_posture: AgentTrustProfile[];

  // Human Accountability
  authorizing_principal: {
    human_id: string;             // Entra Object ID or equivalent
    role: string;                 // Job title or role
    authorization_timestamp: string;
    engagement_level_granted: 1 | 2 | 3 | 4 | 5;
    justification: string;        // Why this level was appropriate for this task
  };

  // Chain Integrity
  previous_aebom_hash?: string;   // For sequential handoffs: links to prior record
  record_hash: string;            // SHA-256 of this record (computed last, excluding this field)
}

interface AgentParticipation {
  agent_id: string;
  role: string;                   // e.g., "security-specialist", "code-reviewer"
  engagement_level: 1 | 2 | 3 | 4 | 5;
  entry_timestamp: string;
  exit_timestamp?: string;
}

interface ToolInvocation {
  tool_name: string;
  agent_id: string;
  timestamp: string;
  tier: 'T1' | 'T2' | 'T3' | 'T4' | 'T5';
  authorized: boolean;
  success: boolean;
  duration_ms: number;
}

interface GovernanceEvent {
  event_type: 'level_check' | 'tier_gate' | 'compliance_attestation' |
              'promotion_decision' | 'trust_gate_check' | 'scope_creep_detected';
  timestamp: string;
  decision: 'allowed' | 'denied' | 'escalated';
  agent_id: string;
  reason?: string;
  human_override?: boolean;       // true if a human manually approved past a gate
}

interface ExecutionOutcome {
  status: 'completed' | 'failed';
  completion_timestamp: string;
  failure_reason?: string;        // Prefixed: 'rejected:*', 'abandoned:*', 'error:*', 'timeout:*'
  output_summary?: string;
}
```

### 3.2 Agent Trust Posture Extension

The `agent_trust_posture` array contains one `AgentTrustProfile` per agent that participated in the handoff. Each profile answers the four human-equivalent trust questions.

```typescript
interface AgentTrustProfile {
  agent_id: string;

  // ─────────────────────────────────────────────────────
  // PILLAR 1: AGENT PROVENANCE RECORD
  // Human equivalent: Pre-hire background check
  // Question answered: Was this agent vetted, unmodified, and from a trusted source?
  // ─────────────────────────────────────────────────────
  provenance: {
    model_id: string;
    model_version: string;
    model_fingerprint: string;          // SHA-256 of model config / API identity
    system_prompt_hash: string;         // SHA-256 — detects prompt injection or tampering
    registry_source: string;            // e.g., "azure-ai-foundry/allowlist-v2"
    registry_verified: boolean;
    registry_verification_timestamp: string;
    spdx_ai_profile_ref?: string;       // Reference to SPDX 3.0 AI Profile record
    known_issues: string[];             // CVE-style entries if any apply to this model version
    tamper_evidence: {
      system_prompt_matches_registered: boolean;
      tool_manifest_matches_registered: boolean;
      config_integrity_verified: boolean;
    };
  };

  // ─────────────────────────────────────────────────────
  // PILLAR 2: PERMISSION PROOF
  // Human equivalent: Least-privilege access + continuous access review
  // Question answered: Did the agent only use what the task actually required?
  // ─────────────────────────────────────────────────────
  permission_proof: {
    requested_level: 1 | 2 | 3 | 4 | 5;
    granted_level: 1 | 2 | 3 | 4 | 5;
    task_minimum_required: 1 | 2 | 3 | 4 | 5;
    effective_level: number;            // min(agent, squad_ceiling, environment_cap)
    overprivileged: boolean;            // granted > task_minimum_required (policy violation flag)
    tool_tiers_authorized: ('T1'|'T2'|'T3'|'T4'|'T5')[];
    tool_tiers_actually_used: ('T1'|'T2'|'T3'|'T4'|'T5')[];
    scope_creep_detected: boolean;      // Agent attempted tools outside its granted tier
    scope_creep_events?: string[];      // Tool names that triggered scope creep
    jit_verification_timestamps: string[]; // Timestamps of per-operation access rechecks
    squad_ceiling: 1 | 2 | 3 | 4 | 5;
    environment_cap: 1 | 2 | 3 | 4 | 5;
    environment_tier: 'local' | 'dev' | 'staging' | 'production';
  };

  // ─────────────────────────────────────────────────────
  // PILLAR 3: SAFETY CERTIFICATION
  // Human equivalent: Annual security training (current, not expired)
  // Question answered: Is this agent current on its safety evaluations?
  // ─────────────────────────────────────────────────────
  safety_certification: {
    certification_id: string;
    issued_at: string;
    expires_at: string;                 // Certification window (e.g., 90 days)
    is_current: boolean;                // false = agent MUST NOT execute (blocking)
    certification_authority: string;    // e.g., "mlcommons-ailuminate", "nist-airt"
    benchmarks: {
      benchmark_name: string;           // e.g., "AILuminate-2026Q1", "ART-v2", "b3-2026"
      version: string;
      evaluated_at: string;
      hazard_categories_tested: number;
      hazard_categories_passed: number;
      overall_passed: boolean;
      disqualifying_failures: string[]; // Hazard categories where agent failed outright
    }[];
    recertification_trigger?: string;   // What would require early recertification
    red_team_conducted: boolean;
    red_team_date?: string;
  };

  // ─────────────────────────────────────────────────────
  // PILLAR 4: EXECUTION ATTESTATION
  // Human equivalent: Two-factor authentication / anti-impersonation
  // Question answered: Is the agent running where and how it claims,
  //                    in an environment it cannot escape?
  // ─────────────────────────────────────────────────────
  execution_attestation: {
    environment_id: string;             // e.g., "azure-container-apps/prod/eastus2"
    environment_tier: 'local' | 'dev' | 'staging' | 'production';
    managed_identity_used: boolean;     // No stored credentials
    identity_non_exportable: boolean;   // Credential bound to execution environment
    entra_agent_id?: string;            // Microsoft Entra Agent Identity Blueprint ID
    entra_blueprint_principal?: string; // BlueprintPrincipal ID
    oidc_token_verified: boolean;       // Workload identity federation verified
    execution_context_hash: string;     // Hash of runtime context at invocation time
    attestation_signature: string;      // Cryptographic proof signed by execution environment
    attestation_algorithm: string;      // e.g., "RS256", "ES256"
    tamper_detection: {
      system_prompt_at_execution_matches_registered: boolean;
      tool_list_at_execution_matches_authorized: boolean;
      environment_matches_declared: boolean;
    };
    tee_used?: boolean;                 // Trusted Execution Environment (future capability)
    tee_provider?: string;              // e.g., "azure-confidential-computing"
  };

  // ─────────────────────────────────────────────────────
  // TRUST POSTURE SUMMARY
  // Computed from the four pillars — the overall gate result
  // ─────────────────────────────────────────────────────
  trust_posture_result: {
    all_pillars_passed: boolean;
    blocking_failures: ('provenance' | 'permission' | 'certification' | 'attestation')[];
    trust_score?: number;               // 0–100 composite, optional future capability
    human_override_applied: boolean;    // A human manually approved past a blocking failure
    human_override_approver?: string;   // Entra Object ID of the human who overrode
    human_override_justification?: string;
  };
}
```

---

## 4. Capability Thresholds and Trust Gates

Adapted from the Frontier Model Forum's framework for model-level capability thresholds, AEBOM defines execution-level trust gates that operate in sequence before any agent action is permitted.

### 4.1 The Two-Gate Structure

Drawing from the FMF's "if-then" threshold structure:

**Gate 1 — Enabling Trust Threshold:** Does this agent meet the minimum trust posture to execute *at all*? All four pillars of the Agent Trust Posture must pass. If any pillar fails, the agent is blocked from execution — equivalent to the FMF concept of an enabling capability threshold that, if not met, prevents deployment.

**Gate 2 — Acceptable Engagement Threshold:** Even if the agent passes Gate 1, does the *level of access requested* meet the acceptable deployment standard for this environment? This maps to the FMF's acceptable training or deployment threshold — a second, outcome-focused assessment that determines whether the safeguards in place are adequate for the specific execution context.

```
Gate 1: Trust Posture    Gate 2: Engagement Level
─────────────────────    ────────────────────────────────────
Provenance verified  ┐   Requested level ≤ effective_level cap
Permission bounded   ├──►  AND
Certification current│   Tool tiers authorized ⊇ tool tiers needed
Attestation valid    ┘   AND environment_cap ≥ required level
         │                         │
         ▼                         ▼
   BLOCK if any fail         BLOCK if constraints violated
         │                         │
         └──────────┬──────────────┘
                    ▼
              Agent may execute
              (AEBOM record begins)
```

### 4.2 Certification Validity Windows

Analogous to annual security training requirements for human engineers, safety certification has a mandatory validity window. Current proposed defaults (subject to industry calibration):

| Deployment Context | Certification Window | Red Team Requirement |
|---|---|---|
| Local / Dev | 180 days | Recommended |
| Staging | 90 days | Required |
| Production (L1–L3) | 90 days | Required |
| Production (L4–L5) | 30 days | Required + independent |

Certification expiry is a **blocking condition** — an expired certificate prevents execution regardless of other factors. This mirrors the human requirement: an employee whose annual security training has lapsed cannot access certain systems until current.

### 4.3 Scope Creep as a Threshold Event

When `scope_creep_detected: true` is recorded in a completed AEBOM record, it triggers:
1. An automatic flag for human review
2. A recertification evaluation for the agent's next engagement
3. A governance event of type `scope_creep_detected` in the record

Persistent scope creep across multiple AEBOM records for the same `agent_id` should trigger suspension of that agent's engagement level pending review — equivalent to revoking access pending an employee security investigation.

---

## 5. Risk Identification and Assessment

### 5.1 Threat Modeling for Agentic Workflows

Drawing on the Frontier Model Forum's methodology, threat modeling for AEBOM covers four risk pathways specific to agentic execution:

**Pathway A — Identity Compromise:** An agent's identity is spoofed or stolen. A malicious actor presents valid credentials but runs a different system prompt or tool set than was registered. *AEBOM mitigates this through Pillar 4 (execution attestation) and the tamper detection fields in Pillar 1.*

**Pathway B — Privilege Escalation:** An agent is granted an engagement level higher than the task requires, and exploits the excess access to take actions outside its mandate. *AEBOM mitigates this through Pillar 2 (permission proof) and scope creep detection.*

**Pathway C — Stale Safety Profile:** An agent's safety evaluation was conducted against an earlier model version or an earlier hazard taxonomy. A new hazard category has emerged since certification, but the agent continues to operate with a "current" certification based on an outdated evaluation. *AEBOM mitigates this through Pillar 3 (safety certification currency) with mandatory recertification windows and benchmark version tracking.*

**Pathway D — Environmental Escape:** An agent executes outside its declared environment — for example, code that was evaluated against a staging environment profile runs in production with different constraints. *AEBOM mitigates this through Pillar 4's `environment_tier` field and the attestation that the declared and actual environments match.*

### 5.2 Assessment Methodology Parallels

The FMF defines three assessment types for frontier models. AEBOM adopts equivalent concepts for agent trust posture:

| FMF Assessment Type | AEBOM Equivalent |
|---|---|
| **Relative Capability Assessment** — compare new model against known-safe reference | **Relative Trust Assessment** — compare agent's current certification scores against the registered baseline for its model version |
| **Bottleneck Assessment** — test whether the model removes specific barriers to harm | **Permission Bottleneck Assessment** — test whether the agent's granted tool tiers, in combination, could enable actions beyond its stated mandate |
| **Threat Simulation Assessment** — simulate end-to-end threat scenarios | **Red Team Execution** — simulate adversarial agent invocations to verify that trust gates correctly block execution when any pillar fails |

### 5.3 Leading Indicators

Analogous to the FMF's leading indicator assessments during training, AEBOM records enable continuous monitoring for emerging trust posture degradation:

- Increasing `overprivileged: true` rate across an agent's AEBOM records → engagement level calibration needed
- Increasing `scope_creep_detected` frequency → agent's tool authorization model needs review
- Approaching certification expiry without recertification scheduled → automated alert to human principal
- Repeated `tamper_detection` field failures → potential infrastructure integrity issue requiring incident response

---

## 6. Implementation Guidance

### 6.1 Integration with Handoff Lifecycle

AEBOM record creation is integrated into the handoff state machine:

```
create_handoff()
  │
  ├─► Run Trust Gate 1 (all four pillars)
  │     └─► BLOCK if any pillar fails
  │         (handoff → 'failed', reason: 'rejected:trust-gate')
  │
  ├─► Run Trust Gate 2 (engagement level)
  │     └─► BLOCK if level constraints violated
  │
  ├─► Initialize AEBOM record (status: 'open')
  │     └─► Snapshot all four trust posture pillars at invocation time
  │
  ├─► accept_handoff() → AEBOM records agent participation
  ├─► [tool invocations] → AEBOM records each invocation
  ├─► [governance events] → AEBOM records each decision
  │
  └─► complete_handoff() / failed
        └─► Finalize AEBOM record
              ├─► Set finalized_at
              ├─► Compute record_hash (SHA-256 of full record)
              └─► Write to append-only persistence
```

### 6.2 Persistence and Retention

AEBOM records are compliance artifacts, not operational telemetry. Retention requirements:

| Environment | Minimum Retention | Storage Class |
|---|---|---|
| Local / Dev | 30 days | Standard |
| Staging | 1 year | Immutable |
| Production | 7 years | Immutable, encrypted at rest |

Records must be stored in append-only infrastructure. Azure Blob Storage with immutability policies, AWS S3 Object Lock, or equivalent is appropriate. Hash-chaining via `previous_aebom_hash` enables detection of gaps or deletions in the audit trail.

### 6.3 API Surface

Minimum required API:

```
GET  /api/aebom/:handoff_id       — Retrieve AEBOM record for a handoff
GET  /api/aebom/agent/:agent_id   — List all AEBOM records for an agent
GET  /api/aebom/audit/summary     — Aggregate trust posture metrics
POST /api/aebom/:handoff_id/verify — Verify record integrity (recompute hash)
```

---

## 7. Organizational Considerations

### 7.1 Separation of Duties

Drawing from the Frontier Model Forum's organizational considerations, AEBOM governance requires separation between:

- **The team that operates agents** — cannot modify their own AEBOM records
- **The team that manages certification** — cannot approve their own agents' certifications
- **The team that reviews AEBOM records for compliance** — cannot also hold the authorizing principal role for the agents being reviewed

This mirrors the FMF finding that "most frontier model developers maintain assessment teams that are separate from model development teams" — the same principle applies at the enterprise level for agent governance.

### 7.2 Human Accountability Requirements

Every AEBOM record must carry a named human `authorizing_principal`. This is non-negotiable. Three questions must be answerable from every record:

1. **Who is accountable?** — The `authorizing_principal.human_id`, named by role. Never "the system."
2. **What's the evidence?** — The AEBOM record itself: attestations, certification status, audit trail.
3. **Which framework controls does this satisfy?** — Mapped in §8.

Agents are accountable for their actions. Humans are accountable for granting agents the level at which those actions were permitted. This distinction is the governance foundation.

### 7.3 Framework Update Process

AEBOM schema versions are tagged. Organizations adopting AEBOM should:
- Define trigger conditions for schema updates (e.g., new hazard categories, new assessment methodologies)
- Require that existing records remain readable under newer schema versions (backward compatibility)
- Publish schema changes with adequate notice for implementing organizations

This mirrors the FMF's approach of iterative, versioned frameworks that "accommodate changes to ecosystem risk."

---

## 8. Regulatory Framework Alignment

| Requirement | Framework | Specific Controls | AEBOM Field |
|---|---|---|---|
| Human accountability for AI decisions | EU AI Act | Art. 14, Art. 22 | `authorizing_principal` |
| Transparency and traceability | EU AI Act | Art. 13 | Core AEBOM record + `record_hash` |
| Risk management and documentation | NIST AI RMF | GOVERN 1.1, MAP 1.1 | Full AEBOM record |
| Measurement and evaluation | NIST AI RMF | MEASURE 2.5, 2.6 | `safety_certification` |
| Transparency of AI system behavior | ISO 42001 | §8.4, §9.1 | `governance_events`, `outcome` |
| Organizational controls for AI | ISO 42001 | §6.1.2, §7.2 | `authorizing_principal`, `permission_proof` |
| Capability threshold documentation | FMF Frameworks | All member firm frameworks | `permission_proof.effective_level` + trust gates |
| Separation of assessment duties | FMF Org Considerations | Section 6 | Governance structure (§7.1) |

---

## 9. Industry Prior Art and Differentiation

AEBOM draws from, and explicitly does not duplicate, the following prior work:

**Software Bill of Materials (SBOM / SPDX / CycloneDX):** AEBOM consumes SPDX AI Profile records via the `spdx_ai_profile_ref` field. It does not re-specify model or dataset provenance — that is SPDX's domain.

**SLSA (Supply-chain Levels for Software Artifacts):** AEBOM consumes SLSA build provenance via the `model_fingerprint` field. It does not re-specify build-time provenance — that is SLSA's domain.

**FMF Frontier AI Frameworks:** AEBOM adopts the FMF's two-gate threshold structure and threat modeling methodology at the execution level. It does not address model-level capability thresholds — that is the FMF's domain.

**Microsoft Entra Agent Identity:** AEBOM's Pillar 4 (Execution Attestation) is designed to consume Entra Agent Identity Blueprint IDs and BlueprintPrincipal attestations natively. Entra provides the identity infrastructure; AEBOM provides the compliance record.

**What is genuinely novel:** No existing standard ties these layers together into a single per-execution compliance record with certification currency, scope-creep detection, and human accountability chain. AEBOM is that binding layer — the artifact that answers the question: *"Can I prove this agent invocation was trustworthy, and can I show my auditors the evidence?"*

---

## 10. Open Questions and Continuing Work

The following areas require further research, industry input, and empirical calibration:

**Q1 — Certification Window Calibration.** The proposed windows (90 days for production, 30 days for L4–L5) are first estimates. Empirical data on how rapidly safety evaluation results become stale as model capabilities evolve is needed. The FMF's ongoing work on assessment timing provides a useful methodological foundation.

**Q2 — Trust Score Computation.** The optional `trust_score` field (0–100 composite) is not yet specified. Developing a principled weighting across the four pillars — and validating that composite scores correlate with actual agent failure modes — is a priority for v1.1.

**Q3 — Multi-Agent Trust Inheritance.** When Agent A hands off to Agent B (S2S), does Agent B's AEBOM record inherit anything from Agent A's trust posture? The current proposal treats each agent independently. The FMF notes that Bottleneck Assessments "typically evaluate individual models in isolation, potentially missing risks that emerge when multiple models with complementary capabilities are used together" — the same challenge applies here.

**Q4 — Trusted Execution Environment Integration.** The `tee_used` and `tee_provider` fields are specified but no implementation guidance is provided. As Azure Confidential Computing and equivalent technologies mature, this field will become increasingly relevant for high-assurance deployments.

**Q5 — Harmonization with Emerging Standards.** NIST AI 800-1 (Managing Misuse Risk for Dual-Use Foundation Models), the UK AISI's early lessons reports, and the EU AI Act's implementing measures are all evolving. AEBOM should be reviewed against each as they are finalized.

**Q6 — Open-Source Reference Implementation.** A reference implementation of AEBOM generation, persistence, and verification integrated with the AgentCraftworks handoff FSM will be published alongside this proposal to enable empirical testing.

---

## Acknowledgments

This proposal draws on the structure and methodology of the Frontier Model Forum's technical report series, particularly *Frontier Capability Assessments* (April 2025), *Risk Taxonomy and Thresholds for Frontier AI Frameworks* (June 2025), and *Components of Frontier AI Safety Frameworks* (November 2024). The human engineer accountability analogy that frames the four-pillar trust posture structure is original to AgentCraftworks.

The authors invite engagement from organizations implementing agentic engineering workflows, enterprise security and compliance teams, and standards bodies working on AI governance. Contact: standards@agentcraftworks.com

---

*Version 0.1 — Draft for Industry Review — May 2026*  
*© 2026 AICraftworks, LLC. Released for public comment under Creative Commons Attribution 4.0 International.*
