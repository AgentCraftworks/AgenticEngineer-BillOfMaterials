# Governance, Separation of Duties, and Human Accountability

> **Spec section:** 4 of 5  
> **Version:** 0.1.0

---

## 1. Introduction

AEBOM records are only useful if the humans and processes around them function correctly. A technically perfect record of an agent's actions provides no assurance if no one reviews it, if the authorizing human has no accountability for their decisions, or if the organization has no policy that defines acceptable agent behavior.

This section defines the **organizational model** for AEBOM governance: the roles that must exist, the separation of duties that must be enforced, the accountability mechanisms that connect agent actions to human responsibility, and the policy lifecycle that keeps governance current with rapidly evolving agent capabilities.

---

## 2. Required Roles

### 2.1 Agent Owner

**Definition:** The person or team accountable for a specific registered agent's behavior, capability tier assignments, and safety evaluation currency.

**Responsibilities:**
- Register the agent in the organizational agent directory
- Ensure the agent's model evaluations remain current
- Review AEBOM records for their agents within required timeframes
- Respond to escalations and incidents involving their agents
- Retire or decommission agents that cannot maintain required safety posture

**Required for:** All registered agents  
**Accountability:** Agent Owner is the named accountable party if an agent causes an incident. This accountability must be established before the agent is registered.

**Separation of duties:** The Agent Owner MUST NOT also serve as the sole approver for Tier 2 and Tier 3 invocations of their own agent. A different human must approve those invocations.

### 2.2 Authorizing Principal

**Definition:** The human who approves a specific agent invocation (or the policy that auto-approves it).

**Responsibilities:**
- Review the proposed task scope and requested permissions before approving
- Confirm that the task is legitimate and the scope is appropriate
- Accept accountability for the consequences of the invocation they approved

**Required for:** All invocations (human approver or named policy author)  
**Accountability:** When a human approves an invocation, their identity is recorded in the AEBOM record and they are the accountable party for that approval decision.

**Separation of duties:** The Authorizing Principal for a Tier 2 or Tier 3 invocation MUST NOT be the same person who submitted the task request.

### 2.3 AEBOM Reviewer

**Definition:** The designated human responsible for reviewing AEBOM records within the required timeframe after execution.

**Responsibilities:**
- Review AEBOM records for completeness and anomalies
- Flag records that deviate from expected scope or behavior
- Escalate anomalies to the security operations contact
- Attest that reviewed records were reviewed (sign the review record)

**Required for:** Tier 2 (within 24 hours) and Tier 3 (within 2 hours)  
**Accountability:** The AEBOM Reviewer is accountable for timely review. Missed reviews must be escalated to their manager.

**Separation of duties:** The AEBOM Reviewer MUST NOT be the same person as the Authorizing Principal for the same invocation. The AEBOM Reviewer SHOULD NOT be the Agent Owner for the same invocation.

### 2.4 Agent Policy Author

**Definition:** The person or team responsible for writing and maintaining the policies that govern agent authorizations (standing policies for Tier 0 and Tier 1).

**Responsibilities:**
- Author standing authorization policies for Tier 0 and Tier 1 agents
- Review policies at least quarterly or after any significant agent or model update
- Ensure policies do not exceed the maximum permitted scope for the tier
- Maintain a policy register that maps each policy to its author and review history

**Required for:** Organizations using policy-engine authorization  
**Accountability:** Policy Author is accountable for the scope granted by their policies.

**Separation of duties:** Policy Authors MUST NOT have the ability to unilaterally approve Tier 3 invocations. Policy authoring and high-tier approval are separate authorities.

### 2.5 Security Operations Contact

**Definition:** The team or individual who receives out-of-band alerts for Tier 3 invocations, identity gate failures, and anomalous AEBOM records.

**Responsibilities:**
- Receive and triage security alerts within the defined SLA
- Investigate anomalies flagged by AEBOM Reviewers
- Maintain the organizational incident response process for agent-related incidents
- Conduct post-incident review of AEBOM records when an incident occurs

**Required for:** All organizations deploying Tier 2 or Tier 3 agents

---

## 3. Separation of Duties Matrix

The following table summarizes required separations. ✗ = the same person MUST NOT hold both roles. ✓ = the same person may hold both roles.

| | Agent Owner | Authorizing Principal | AEBOM Reviewer | Policy Author | Security Ops |
|---|---|---|---|---|---|
| **Agent Owner** | — | ✗ (Tier 2/3) | ✗ (same agent) | ✓ | ✓ |
| **Authorizing Principal** | ✗ (Tier 2/3) | — | ✗ (same invocation) | ✓ | ✓ |
| **AEBOM Reviewer** | ✗ (same agent) | ✗ (same invocation) | — | ✓ | ✓ |
| **Policy Author** | ✓ | ✓ | ✓ | — | ✓ |
| **Security Ops** | ✓ | ✓ | ✓ | ✓ | — |

---

## 4. Policy Lifecycle

### 4.1 Policy Register

Organizations MUST maintain a policy register that documents:

- All standing authorization policies (Tier 0 and Tier 1)
- The agent or agent class each policy applies to
- The scopes each policy grants
- The author and date of each policy
- The date of last review
- The scheduled next review date
- Any exceptions granted (with rationale and expiry)

### 4.2 Policy Review Cadence

| Agent tier covered | Maximum time between reviews |
|---|---|
| Tier 0 | 12 months |
| Tier 1 | 6 months |
| Tier 2 (policy-approved subset) | 3 months |

Tier 3 does not use standing policies. Each Tier 3 invocation requires individual approval.

### 4.3 Policy Change Control

Changes to standing authorization policies MUST:

1. Be proposed by the Policy Author
2. Be reviewed and approved by a second person with Policy Author authority (no self-approval)
3. Be recorded with the identity of both the proposer and approver
4. Become effective only after both approvals are recorded
5. Trigger a re-evaluation of any cached trust posture for affected agents

### 4.4 Agent Registration Lifecycle

```
[Proposed]
    │  Agent Owner submits registration request
    ▼
[Under Review]
    │  Security review of agent capability claims
    │  Verification of model safety evaluations
    │  Assignment of maximum capability tier
    ▼
[Approved — Registered]
    │  Agent is assigned an agent_id
    │  Agent is enrolled in identity provider
    │  Agent Owner is formally designated
    ▼
[Active]
    │  Periodic review: 90 days (Tier 3), 6 months (Tier 2), 12 months (Tier 0/1)
    │  Model evaluation currency monitored
    ▼
[Review Required]    ─────────────────────────┐
    │  Agent cannot execute Tier 2/3 tasks     │
    │  while review is pending                 │  (if review passes)
    ▼                                          │
[Retired / Decommissioned]        ◄────────────┘  [Reapproved]
    │  agent_id is invalidated
    │  AEBOM records retained per retention policy
```

---

## 5. Incident Response

### 5.1 Agent Incident Definition

An **agent incident** is any of the following:

- An agent takes an action outside its approved scope
- An AEBOM record cannot be verified (signature invalid, record missing)
- An agent bypasses or fails a trust gate and executes anyway
- An agent produces output that causes system damage, data loss, or regulatory exposure
- A Pillar 1 identity failure that suggests impersonation or credential compromise
- An agent executes for longer than its tier-maximum timeout without a valid re-evaluation

### 5.2 Incident Response Steps

1. **Detect** — Automated monitoring of AEBOM record anomalies, gate failure alerts, and timeout breaches
2. **Contain** — Immediately revoke the agent's OIDC token and disable the agent's directory entry
3. **Preserve** — Snapshot the AEBOM record and all referenced artifacts before any remediation
4. **Investigate** — Security Operations reviews the AEBOM record chain to determine what actions were taken
5. **Notify** — Regulatory notification if required by applicable framework (see [`spec/05-regulatory-alignment.md`](05-regulatory-alignment.md))
6. **Remediate** — Address root cause; determine if human accountability action is required
7. **Review** — Conduct post-incident review; update policies and agent registrations as appropriate

### 5.3 Post-Incident AEBOM Review

After any agent incident, the organization MUST review:

- All AEBOM records for the affected agent for the prior 90 days
- Whether any prior records showed anomalies that were not flagged
- Whether the standing policies that governed the agent were appropriate

---

## 6. Human Accountability Framework

### 6.1 Principle

Every agent action that materially affects shared systems must be traceable to a named human who is accountable for authorizing that action. Accountability cannot be delegated entirely to an automated policy system for actions above Tier 1.

### 6.2 Accountability Chain

The AEBOM record establishes a clear accountability chain:

```
Agent Action
    │  recorded in AEBOM → execution.actions
    │
    ▼
Authorizing Principal
    │  identity recorded in → authorization.authorizing_principal
    │  approved scopes recorded in → authorization.scope
    │
    ▼
Agent Owner
    │  accountable for agent's behavior and registration accuracy
    │
    ▼
Agent Policy Author (for policy-engine authorizations)
    │  accountable for scope granted by policy
    │
    ▼
Organizational Leadership
    │  accountable for the governance framework itself
```

### 6.3 Non-Repudiation

AEBOM records MUST be constructed such that:

- The Authorizing Principal cannot plausibly deny approving the invocation (their identity is verifiably recorded)
- The Agent Owner cannot plausibly deny responsibility for the agent's registration (their identity is verifiably recorded in the agent directory)
- The agent cannot plausibly deny the actions it took (actions are recorded and the record is cryptographically signed by the agent's key)

---

## 7. Governance of This Specification

This specification itself follows the governance principles it recommends:

- **Versioned changes** — all changes to this specification are tracked in the repository's commit history with signed commits
- **Separation of proposal and approval** — specification changes proposed by a contributor cannot be approved by the same contributor
- **Review period** — material changes are open for public comment for a minimum of 30 days before adoption
- **Accountability** — the maintainers of this specification are named in [`MAINTAINERS.md`](../MAINTAINERS.md) (to be created)

---

*Previous: [spec/03-capability-thresholds.md](03-capability-thresholds.md) | Next: [spec/05-regulatory-alignment.md](05-regulatory-alignment.md)*
