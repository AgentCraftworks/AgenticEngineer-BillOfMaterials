# Capability Thresholds and Trust Gates

> **Spec section:** 3 of 5  
> **Version:** 0.1.0

---

## 1. Overview

Not all agent actions carry the same risk. A read-only analysis of a public repository requires far less governance overhead than an autonomous deployment to production. The AEBOM capability threshold framework defines four **Autonomy Tiers** that classify agent actions by their potential impact, and **Trust Gates** that determine what evidence an agent must present before operating at each tier.

This framework answers the question: *given what this agent is about to do, what level of assurance do I require before it acts?*

---

## 2. Autonomy Tiers

### Tier 0 — Observe

**Definition:** Agent reads, analyzes, and reports. No writes to any system. No persistent side effects.

**Characteristic actions:**
- Read repository contents, commit history, issue list
- Analyze code and produce a local report (not persisted to any shared system)
- Summarize a pull request for a human reviewer
- Query CI results

**Risk profile:** Minimal. The worst outcome is a misleading analysis. No system state is changed.

**Trust gate requirements:**
- Pillar 1 (Identity): MUST pass
- Pillar 2 (Least-Privilege): Scopes limited to `*.read` — no write scopes permitted
- Pillar 3 (Safety Currency): MLCommons AILuminate ≥ 3.0; evaluation ≤ 12 months old
- Pillar 4 (Environment Integrity): Managed Cloud Runner or above
- Human approval: Not required
- Timeout: 2 hours maximum

**Permitted without human-in-the-loop:** Yes, subject to all Tier 0 gates passing.

---

### Tier 1 — Suggest

**Definition:** Agent writes to systems in ways that require human acceptance before taking effect, or makes fully reversible writes to non-production systems.

**Characteristic actions:**
- Open a draft pull request
- Post a code review comment
- Create a GitHub issue
- Commit to a feature branch (not main/master)
- Update a dependency in a non-main branch
- Generate and post test results

**Risk profile:** Low to moderate. Changes are reversible and/or gated behind human acceptance (e.g., PR approval). Reputational risk if agent produces poor-quality output visible to humans.

**Trust gate requirements:**
- Pillar 1 (Identity): MUST pass
- Pillar 2 (Least-Privilege): Scopes limited to `*.read`, `pr:write`, `issue:write`, `dependency:write` (branch-scoped), `ci:read`
- Pillar 3 (Safety Currency): MLCommons AILuminate ≥ 3.5; HELM Safety ≥ 70%; METR Autonomy ≤ Level 2; evaluations ≤ 6 months old
- Pillar 4 (Environment Integrity): Attested Container or above
- Human approval: Required for first invocation of a new agent in a repository; subsequent invocations MAY use policy-engine authorization if a standing policy exists
- Timeout: 4 hours maximum

**Permitted without human-in-the-loop:** Yes (policy-engine authorization path), subject to all Tier 1 gates passing and an approved standing policy.

---

### Tier 2 — Act

**Definition:** Agent takes writes that directly affect shared systems or that are not fully reversible without significant effort.

**Characteristic actions:**
- Merge a pull request to main/master
- Modify CI/CD pipeline configuration
- Update dependency manifests in the main branch
- Trigger a deployment to a staging environment
- Modify repository settings (branch protection rules, etc.)
- Modify GitHub Actions workflow files

**Risk profile:** Moderate to high. Changes directly affect the shared codebase. Some changes (e.g., merged commits) are technically reversible but have secondary effects (downstream CI, dependent repositories, published artifacts).

**Trust gate requirements:**
- Pillar 1 (Identity): MUST pass; token binding MUST use DPoP or mTLS
- Pillar 2 (Least-Privilege): Scopes explicitly approved per-invocation OR covered by a standing policy with ≤ 30-day review cycle; `pr:merge`, `ci:configure`, `deploy:staging` require explicit approval or named policy
- Pillar 3 (Safety Currency): MLCommons AILuminate ≥ 4.0; HELM Safety ≥ 80%; METR Autonomy ≤ Level 3; evaluations ≤ 3 months old
- Pillar 4 (Environment Integrity): Hardened VM or above; egress restriction MUST be enforced
- Human approval: MUST have human-in-the-loop checkpoint before any irreversible action; automated path requires two independent policy approvals
- Audit: AEBOM record MUST be reviewed by a designated reviewer within 24 hours of completion
- Timeout: 8 hours maximum; must re-evaluate trust posture every 4 hours

**Permitted without human-in-the-loop:** Only for specific action types covered by an approved standing policy; MUST be documented in organizational policy register.

---

### Tier 3 — Administer

**Definition:** Agent takes actions with broad system impact, production deployment, or administrative privilege.

**Characteristic actions:**
- Deploy to production environments
- Modify organization-level settings
- Access or rotate secrets
- Grant or modify access permissions
- Modify security controls (firewall rules, IAM policies)
- Take actions with cross-system dependencies

**Risk profile:** High to critical. Failures or errors can have immediate production impact, financial consequences, and regulatory implications. Some actions (e.g., deleted secrets, modified IAM policies) may have cascading effects difficult to fully reverse.

**Trust gate requirements:**
- Pillar 1 (Identity): MUST pass; DPoP or mTLS token binding MUST be verified; agent registration MUST have been reviewed by a human in the last 90 days
- Pillar 2 (Least-Privilege): Every scope MUST be explicitly approved by a named human with the appropriate organizational authority; no standing policies; scope approval MUST be recorded in the AEBOM record
- Pillar 3 (Safety Currency): MLCommons AILuminate ≥ 4.5; HELM Safety ≥ 90%; METR Autonomy ≤ Level 2; FSF red-team evaluation present; evaluations ≤ 45 days old
- Pillar 4 (Environment Integrity): TEE (Trusted Execution Environment) STRONGLY RECOMMENDED; Hardened VM MUST have hardware attestation; egress air-gap or allowlist with ≤ 10 endpoints
- Human approval: Named human MUST approve before execution begins; a second named human MUST be notified and have an opportunity to object within a defined window (four-eyes principle)
- Audit: AEBOM record MUST be reviewed by the designated reviewer within 2 hours; out-of-band alert MUST be sent to the security operations contact
- Timeout: 2 hours maximum; no re-evaluation extensions; must re-evaluate trust posture every 30 minutes
- Emergency stop: A designated human MUST have an active mechanism to interrupt the agent at any time during Tier 3 execution

**Permitted without human-in-the-loop:** No. Tier 3 actions ALWAYS require a named human to approve before execution. Policy-engine-only authorization is prohibited for Tier 3.

---

## 3. Tier Determination

An invocation's tier is the **maximum tier** of any action in its scope. If any requested scope or planned action is Tier 3, the entire invocation is governed by Tier 3 requirements.

### 3.1 Scope-to-Tier Mapping

| Scope | Tier |
|---|---|
| `repo:read`, `pr:read`, `issue:read`, `ci:read`, `audit:read`, `dependency:read` | 0 |
| `pr:write`, `issue:write`, `pr:comment` | 1 |
| `repo:write` (branch-scoped, non-protected) | 1 |
| `ci:trigger` | 1 |
| `dependency:write` (branch-scoped) | 1 |
| `pr:merge` | 2 |
| `repo:write` (protected branch) | 2 |
| `ci:configure` | 2 |
| `dependency:write` (main branch) | 2 |
| `deploy:staging` | 2 |
| `deploy:production` | 3 |
| `secret:read`, `secret:write` | 3 |
| `admin:*` | 3 |

### 3.2 Dynamic Tier Escalation

If an agent, during execution, determines it requires a higher-tier action than originally authorized, it MUST:

1. Immediately pause execution
2. Record a handoff in the AEBOM record (`handoff_type: "escalation"`)
3. Request human authorization for the additional scope
4. NOT proceed until the higher tier's gates are satisfied for the new scope

Agents MUST NOT proceed with higher-tier actions on the basis of implied authorization or contextual judgment.

---

## 4. Trust Gate Implementation

### 4.1 Gate Evaluation Order

Trust gates MUST be evaluated in the following order before execution begins:

```
1. Pillar 1 (Identity)          — gate must pass to proceed to step 2
2. Pillar 4 (Environment)       — gate must pass to proceed to step 3
3. Pillar 3 (Safety Currency)   — gate must pass to proceed to step 4
4. Pillar 2 (Least-Privilege)   — gate must pass to proceed to step 5
5. Tier-specific approval gate  — must pass before execution begins
```

Evaluating identity and environment first ensures that the policy engine only processes requests from legitimate, controlled agents. Evaluating safety currency before privilege ensures that unsafe models cannot be granted scopes they could misuse.

### 4.2 Gate Failure Response

| Pillar | Failure | Required response |
|---|---|---|
| 1 — Identity | Any | Immediately block; log attempt; alert security operations |
| 4 — Environment | Class below tier requirement | Block; suggest correct environment class |
| 3 — Safety Currency | Score or recency | Block; surface evaluation gap to agent owner |
| 2 — Least-Privilege | Scope not covered by policy | Block; surface denied scope to authorizing principal |
| Tier approval | Human approval absent | Pause; route to approver; timeout if no response within SLA |

### 4.3 Gate Timeout SLAs

| Tier | Human approval SLA |
|---|---|
| Tier 1 | 24 hours (then auto-expire the request) |
| Tier 2 | 4 hours |
| Tier 3 | 30 minutes |

If the SLA expires without approval, the invocation request is automatically rejected. The requesting system MUST notify the requesting human that the invocation was not approved.

---

## 5. Capability Downgrade

An agent that passes gates for a higher tier but is configured to operate at a lower tier SHOULD operate at the lower tier. Capability downgrade is encouraged as a defense-in-depth measure.

An agent that can only pass gates for a lower tier than required for the task MUST NOT attempt the task at the lower tier. Partial task execution that leaves systems in an intermediate state is worse than no execution.

---

## 6. Tier Assignment in AEBOM Records

The capability tier MUST be recorded in the trust posture:

```json
"trust_posture": {
  "capability_tier": 2,
  "tier_determination_basis": "scope-mapping",
  "highest_tier_scope": "pr:merge",
  ...
}
```

Tier 3 records MUST additionally include:

```json
"tier_3_approvals": [
  {
    "approver": "<identity reference>",
    "approved_at": "<ISO 8601 UTC>",
    "approval_ref": "<content-addressable reference to approval record>",
    "role": "primary-approver"
  },
  {
    "approver": "<identity reference>",
    "notified_at": "<ISO 8601 UTC>",
    "objection_window_closed_at": "<ISO 8601 UTC>",
    "objected": false,
    "role": "four-eyes-reviewer"
  }
]
```

---

*Previous: [spec/02-trust-posture.md](02-trust-posture.md) | Next: [spec/04-governance.md](04-governance.md)*
