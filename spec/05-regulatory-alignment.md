# Regulatory Alignment and Framework Relationships

> **Spec section:** 5 of 5  
> **Version:** 0.1.0

---

## 1. Overview

AEBOM is designed to support compliance with existing and emerging AI governance frameworks. This section maps AEBOM elements to specific requirements in:

- **EU AI Act** (Regulation (EU) 2024/1689)
- **NIST AI Risk Management Framework** (AI RMF 1.0)
- **ISO/IEC 42001:2023** (AI management systems)
- **Frontier AI Safety Frameworks** from FMF member firms (Anthropic ASL, Google DeepMind FSF, OpenAI Preparedness Framework)
- **Related supply-chain standards** (SPDX 3.0 AI Profile, SLSA, CycloneDX)

This mapping is informational and does not constitute legal advice. Organizations should engage qualified legal and compliance counsel to assess their specific obligations.

---

## 2. EU AI Act

### 2.1 Applicability

The EU AI Act classifies AI systems by risk level. Agentic engineering systems that operate on shared infrastructure, production systems, or that can affect safety-relevant software (e.g., embedded systems, medical device software) may fall under the **High-Risk** category (Annex III, particularly categories 6 and 8).

Autonomous AI agents that take actions with real-world consequences also implicate the **General-Purpose AI (GPAI) model** obligations for the model provider, and the **deployer** obligations for the organization operating the agent.

### 2.2 AEBOM Alignment to EU AI Act Requirements

| EU AI Act Requirement | Article / Recital | AEBOM Element |
|---|---|---|
| Technical documentation of AI system | Art. 11 | AEBOM Core record; `agent.model_id`; `agent.agent_version` |
| Record-keeping of AI system operations | Art. 12 | AEBOM Core record; `execution.actions`; record retention requirements |
| Transparency to users and affected persons | Art. 13 | `task.task_description`; `execution.outcome_detail`; `authorization.authorizing_principal` |
| Human oversight capability | Art. 14 | `execution.human_handoffs`; Tier 3 four-eyes requirement; emergency stop requirement |
| Accuracy, robustness, cybersecurity | Art. 15 | Pillar 3 (Safety Currency); Pillar 4 (Environment Integrity) |
| Quality management system | Art. 17 | Policy lifecycle (Section 4 of spec); agent registration lifecycle |
| Conformity assessment documentation | Art. 43 | AEBOM record as conformity evidence; `trust_posture.trust_posture_signature` |
| Incident reporting to supervisory authority | Art. 73 | Incident response procedure (Section 5 of governance spec) |
| Logging of operations for high-risk AI | Art. 12(1) | `execution.actions`; tamper-evident audit log requirement |
| Post-market monitoring | Art. 72 | Periodic AEBOM record review; agent registration review cadence |

### 2.3 GPAI Model Transparency Obligations

For organizations using GPAI models (Art. 53), AEBOM supports compliance with:

- **Model capability documentation** — `agent.model_id` and `agent.model_provider` reference the specific model version and its published capability documentation
- **Evaluation results disclosure** — Pillar 3 `safety_currency.evaluations` references publicly available benchmark results
- **Copyright policy** — While not directly addressed in AEBOM, the `extensions` field can accommodate copyright policy references

---

## 3. NIST AI Risk Management Framework (AI RMF 1.0)

### 3.1 Framework Structure

The NIST AI RMF is organized around four functions: **GOVERN**, **MAP**, **MEASURE**, and **MANAGE**. AEBOM contributes most directly to the MEASURE and MANAGE functions, with supporting contributions to GOVERN.

### 3.2 GOVERN Function

| AI RMF Category | Subcategory | AEBOM Contribution |
|---|---|---|
| GOVERN 1 | Policies, processes, and procedures are in place | Policy lifecycle and register (Section 4 of governance spec) |
| GOVERN 2 | Accountability structures are in place | Roles, separation of duties, accountability chain (Section 2–6 of governance spec) |
| GOVERN 4 | Org teams are committed to AI risk management | Agent Owner and AEBOM Reviewer roles; mandatory review cadences |
| GOVERN 6 | Policies and procedures for AI risk are established | Trust gates; capability threshold framework |

### 3.3 MAP Function

| AI RMF Category | Subcategory | AEBOM Contribution |
|---|---|---|
| MAP 1 | Context is established | `task.task_type`; `task.task_description`; Autonomy Tier determination |
| MAP 2 | Scientific understanding is applied | Pillar 3 benchmark requirements; recognized benchmark list |
| MAP 3 | AI risks are identified | Scope vocabulary with risk-tiered mapping; trust gate failure conditions |
| MAP 5 | Impacts to individuals and society are identified | Irreversibility tracking (`execution.actions[].reversible`); human handoff records |

### 3.4 MEASURE Function

| AI RMF Category | Subcategory | AEBOM Contribution |
|---|---|---|
| MEASURE 1 | AI risk metrics are identified | Pillar 3 benchmark scores; Autonomy Tier |
| MEASURE 2 | AI risks are evaluated | Trust posture evaluation; gate failure conditions |
| MEASURE 2.5 | AI system performance is monitored | AEBOM record completeness; reviewer attestation |
| MEASURE 2.6 | Risk to third parties is evaluated | `execution.artifacts_produced`; scope audit trail |
| MEASURE 4 | Risk metrics are shared | AEBOM record shareability; `schema_uri` for interoperability |

### 3.5 MANAGE Function

| AI RMF Category | Subcategory | AEBOM Contribution |
|---|---|---|
| MANAGE 1 | AI risks are mitigated | Trust gates; dynamic tier escalation; capability downgrade |
| MANAGE 2 | Responses are planned | Incident response procedure; emergency stop requirement |
| MANAGE 3 | Risks are tracked | AEBOM record as audit trail; anomaly detection basis |
| MANAGE 4 | Risk responses are communicated | AEBOM Reviewer role; security operations alert requirements |

---

## 4. ISO/IEC 42001:2023

### 4.1 Scope of Alignment

ISO 42001 defines requirements for an **AI management system (AIMS)**. AEBOM supports an organization's AIMS implementation by providing the operational evidence layer — the records that demonstrate the AIMS is functioning as designed.

### 4.2 Clause Alignment

| ISO 42001 Clause | Requirement | AEBOM Contribution |
|---|---|---|
| 6.1.2 | AI risk assessment | Autonomy Tier framework; trust gate failure conditions |
| 6.1.4 | AI impact assessment | Irreversibility tracking; handoff records |
| 7.4 | Communication | `authorization.authorizing_principal`; handoff records |
| 7.5 | Documented information | AEBOM record as documented information; retention requirements |
| 8.2 | AI risk treatment | Trust gates; policy lifecycle |
| 9.1 | Monitoring, measurement, analysis | AEBOM record review; reviewer attestation |
| 9.2 | Internal audit | AEBOM records as audit evidence; policy register |
| 10.1 | Continual improvement | Post-incident review process; policy update cadence |

### 4.3 Annex A Controls

ISO 42001 Annex A defines AI-specific controls. Relevant mappings:

| Control | Description | AEBOM Contribution |
|---|---|---|
| A.2.2 | AI policy | Policy register; standing authorization policies |
| A.3.3 | Roles and responsibilities | Agent Owner, Authorizing Principal, Reviewer, Security Ops roles |
| A.4.1 | Resources | Environment Integrity pillar; environment descriptor |
| A.5.1 | Stakeholder needs | Authorizing Principal role; task context in AEBOM |
| A.6.1 | AI system documentation | AEBOM Core record; schema documentation |
| A.6.2 | Data management for AI | `agent.model_id`; Pillar 3 evaluation references |
| A.7.2 | AI system lifecycle | Agent registration lifecycle |
| A.8.1 | Logging and monitoring | `execution.actions`; tamper-evident audit log |
| A.9.1 | Human oversight of AI systems | Human handoff records; Tier 3 four-eyes requirement |
| A.9.2 | Responsibilities and authorities | Separation of duties matrix |
| A.10.1 | Third-party relationships | `agent.model_provider`; Pillar 3 external benchmark references |

---

## 5. Frontier AI Safety Frameworks

### 5.1 Role in AEBOM

FMF (Frontier Model Forum) member firms — Anthropic, Google DeepMind, Microsoft, and OpenAI — have each published frontier safety frameworks that define thresholds above which models are not deployed without additional controls. AEBOM's Pillar 3 safety currency requirement is designed to be compatible with these frameworks.

### 5.2 Anthropic Responsible Scaling Policy (RSP) / Alignment Science

| RSP Concept | AEBOM Alignment |
|---|---|
| AI Safety Levels (ASL-1 through ASL-4) | Pillar 3 recognizes ASL classification as a relevant evaluation dimension; Tier 3 requires ASL-2 or below for autonomous operation |
| Model evaluation before deployment | Pillar 3 recency requirements; evaluation reference anchoring |
| Dangerous capability evaluations | METR Autonomy Evaluation in recognized benchmark list |

### 5.3 Google DeepMind Frontier Safety Framework (FSF)

| FSF Concept | AEBOM Alignment |
|---|---|
| Critical Capability Levels (CCLs) | Tier 3 requires FSF red-team evaluation reference in Pillar 3 |
| Deployment safeguards | Tier requirements and trust gates |
| Incident response | Agent incident response procedure in governance spec |

### 5.4 OpenAI Preparedness Framework

| Preparedness Framework Concept | AEBOM Alignment |
|---|---|
| Risk categories (Cybersecurity, CBRN, Persuasion, Model Autonomy) | Recognized in METR Autonomy Evaluation; Tier structure limits autonomy |
| Deployment decisions based on safety scores | Pillar 3 minimum scores by tier |
| Human oversight for high-capability deployments | Tier 3 four-eyes requirement; emergency stop |

### 5.5 Microsoft Responsible AI Standard

| Microsoft RAI Concept | AEBOM Alignment |
|---|---|
| Accountability | Accountability chain; Agent Owner role |
| Transparency | AEBOM record shareability; `task.task_description` |
| Fairness | Out of scope for v0.1.0 (logged for future extension) |
| Reliability and safety | Trust gates; Pillar 3 safety currency |
| Privacy and security | Environment Integrity; Pillar 1 identity |
| Inclusiveness | Out of scope for v0.1.0 |

---

## 6. Supply-Chain Standard Relationships

### 6.1 Position in the Supply-Chain Stack

AEBOM occupies the **execution layer** of the AI supply chain. It is downstream of model artifact standards and upstream of downstream consumer systems:

```
Source Code (SLSA Build Provenance)
    │
    ▼
Software Artifact (SBOM — CycloneDX / SPDX)
    │
    ▼
AI Model (SPDX 3.0 AI Profile)
    │
    ▼
Agent Software (AEBOM agent registration)
    │
    ▼
Agent Execution (AEBOM Core Record + Trust Posture)
    │
    ▼
Produced Artifacts (AEBOM artifact references → downstream SBOM)
```

### 6.2 SPDX 3.0 AI Profile

AEBOM is designed to be composable with SPDX 3.0 AI Profile documents. The `agent.model_id` field SHOULD reference the SPDX element ID of the model's SPDX 3.0 AI Profile document when one is available.

AEBOM does not duplicate fields present in SPDX 3.0 AI Profile (training data, model architecture, dataset provenance). AEBOM references the SPDX document for that information.

### 6.3 SLSA (Supply-chain Levels for Software Artifacts)

The `execution.environment.isolation_attestation_ref` and `agent.identity_token_ref` fields are designed to reference SLSA provenance attestations where available. An agent executing in a SLSA Level 3 build environment SHOULD produce SLSA provenance for any software artifacts it creates or modifies.

### 6.4 CycloneDX

AEBOM records can be embedded as `externalReference` elements in a CycloneDX BOM document. Organizations that use CycloneDX for software supply-chain management can link AEBOM execution records to the software components the agent produced or modified.

---

## 7. Interoperability

### 7.1 Schema Portability

The AEBOM JSON Schema is published under Apache 2.0 and designed for portability. Implementations are encouraged to:

- Implement the schema in their preferred language
- Publish AEBOM records to shared transparency logs
- Build tooling that queries AEBOM records across organizational boundaries for supply-chain assurance

### 7.2 Federation

Organizations with inter-organizational agent collaborations (agents calling agents across trust boundaries) SHOULD exchange AEBOM records at handoff points. A receiving organization's trust gate SHOULD require a valid AEBOM record from the originating organization as part of Pillar 1 identity verification.

---

*Previous: [spec/04-governance.md](04-governance.md)*
