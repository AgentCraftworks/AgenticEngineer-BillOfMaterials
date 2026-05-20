# Agent Execution Bill of Materials (AEBOM)

> **Status:** Public RFC — v0.1.0 Draft  
> **Purpose:** Proposal for feedback and inclusion in Agentic Engineering governance forums

---

## The Problem

The rapid deployment of autonomous AI agents in enterprise software engineering workflows has outpaced the governance tooling available to assess, audit, and assure the trustworthiness of those agents and their actions.

Existing standards address **what was built** and **what model was deployed**:

| Standard | What it covers |
|---|---|
| SBOM (CycloneDX / SPDX) | Software component inventory |
| SPDX 3.0 AI Profile | Model provenance and training data |
| SLSA | Build-time supply-chain integrity |

None address the question enterprises actually face **in production**:

> *Can I prove, after the fact, that the agent which took action on my systems was who it claimed to be, had only the permissions the task required, was current on its safety evaluations, and ran in an environment I control?*

These are not new questions. They are precisely the questions enterprise security and compliance teams have answered for human engineers for decades:

| Human engineer | Agentic equivalent |
|---|---|
| Background vetting before access is granted | Identity attestation at invocation time |
| Principle of least privilege, continuously verified | Scoped capability set per task |
| Annual security training currency | Safety-evaluation recency and benchmark passage |
| Hardware-bound MFA to prevent impersonation | Cryptographically bound workload identity |

## The Proposal: AEBOM

The **Agent Execution Bill of Materials (AEBOM)** is the structural answer to those four questions applied to every agentic engineering invocation.

AEBOM defines:

1. A **schema specification** for per-execution provenance records (the AEBOM Core)
2. A **four-pillar Agent Trust Posture** extension that answers the four human-equivalent trust questions for every agent invocation
3. **Capability thresholds and trust gates** that determine when an agent may operate autonomously
4. **Organizational considerations** for governance, separation of duties, and human accountability
5. **Alignment to existing regulatory frameworks** (EU AI Act, NIST AI RMF, ISO 42001) and relationship to frontier AI safety frameworks published by FMF member firms

AEBOM is designed to be **implementable today** using available technology — Microsoft Entra Agent Identity, OIDC workload federation, MLCommons AILuminate benchmarks, and existing handoff state machine infrastructure — while defining an extensible schema that can incorporate future advances in trusted execution environments and interpretability methods.

---

## Repository Structure

```
.
├── README.md                          ← This file
├── spec/
│   ├── 01-aebom-core.md              ← Core schema: per-execution provenance record
│   ├── 02-trust-posture.md           ← Four-pillar Agent Trust Posture extension
│   ├── 03-capability-thresholds.md   ← Trust gates and autonomy tiers
│   ├── 04-governance.md              ← Org governance, separation of duties
│   └── 05-regulatory-alignment.md    ← EU AI Act, NIST AI RMF, ISO 42001, FMF
├── schema/
│   ├── aebom-core.schema.json        ← JSON Schema for AEBOM core record
│   └── trust-posture.schema.json     ← JSON Schema for Agent Trust Posture
└── examples/
    ├── sample-aebom-record.json      ← Reference AEBOM execution record
    └── sample-trust-posture.json     ← Reference trust posture payload
```

---

## Four Pillars of Agent Trust Posture

| Pillar | Question answered | Human analogue |
|---|---|---|
| **Identity** | Who is this agent? | Background check + badge |
| **Least-Privilege** | What is this agent allowed to do for this task? | Role-based access control |
| **Safety Currency** | Is this agent current on its safety evaluations? | Annual security training |
| **Environment Integrity** | Is this agent running in a controlled environment? | Managed device + MFA |

---

## Quick Start: Reading the Specification

| If you want to… | Start here |
|---|---|
| Understand the complete data model | [`spec/01-aebom-core.md`](spec/01-aebom-core.md) |
| Understand the trust posture pillars | [`spec/02-trust-posture.md`](spec/02-trust-posture.md) |
| Understand autonomy tiers and gates | [`spec/03-capability-thresholds.md`](spec/03-capability-thresholds.md) |
| Understand governance roles | [`spec/04-governance.md`](spec/04-governance.md) |
| Understand regulatory alignment | [`spec/05-regulatory-alignment.md`](spec/05-regulatory-alignment.md) |
| Validate a record against the schema | [`schema/aebom-core.schema.json`](schema/aebom-core.schema.json) |
| See a concrete example | [`examples/sample-aebom-record.json`](examples/sample-aebom-record.json) |

---

## Design Principles

- **Implementable today.** Every field in the schema maps to a capability available in current enterprise tooling.
- **Tamper-evident.** Records are cryptographically signed; the signing key is attested to a hardware root of trust.
- **Composable.** AEBOM records are SPDX 3.0-compatible and can be embedded in existing SBOM workflows.
- **Auditor-legible.** Field names and values are chosen so that a compliance auditor — not just a security engineer — can read and reason about a record.
- **Extensible.** The schema uses a `extensions` envelope so vendors and researchers can add fields without breaking parsers.

---

## Relationship to Existing Standards

```
SLSA (build provenance)
    └── establishes model artifact integrity
        └── SPDX 3.0 AI Profile (model inventory)
            └── AEBOM (execution provenance)
                └── Agent Trust Posture (runtime assurance)
```

AEBOM sits at the **execution layer** — the layer existing standards do not yet reach.

---

## Contributing

This is an open proposal. Feedback is welcome via:

- **GitHub Issues** — for specific objections or missing requirements
- **GitHub Discussions** — for broader design questions
- **Pull Requests** — for proposed schema changes or additional examples

Please read [`spec/04-governance.md`](spec/04-governance.md) for the proposed governance model for this specification itself.

---

## License

This proposal is published under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). Schema files are additionally available under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) to facilitate implementation without attribution burden.

---

*AEBOM v0.1.0 — Initial public draft*