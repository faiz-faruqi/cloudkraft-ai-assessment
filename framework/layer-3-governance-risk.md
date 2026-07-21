# Layer 3 - Governance & Risk

**The question this layer answers:** Who is allowed to do what with which AI system, is every decision auditable, and can the organisation prove it is meeting its regulatory obligations?

**Why this layer matters:** This is the layer regulators, auditors, and boards actually ask about. Strong data readiness and clean architecture do not answer "how do you know an unauthorised user couldn't access this," or "show me the audit trail for this AI decision." Layer 3 is where AI governance becomes demonstrable, not just assumed.

**Primary stakeholders:** CISO, Risk, Legal & Compliance

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
Anyone with system access can use any AI capability or data source. No audit trail exists for AI decisions or outputs. Responsible AI controls - bias, toxicity, safety screening - are absent. Regulatory requirements for AI are not tracked anywhere.

**Level 2 - Developing**
Basic authentication exists but authorisation is not role-based. Logs exist but are not structured for audit or compliance review. Awareness of responsible AI risk exists but controls are manual and inconsistent. Relevant regulations are known informally but there is no formal compliance programme.

**Level 3 - Defined**
RBAC is applied to some AI systems but not consistently. Structured logs exist but audit readiness has not been tested. Automated screening exists for some risk categories on some systems. A compliance inventory exists but has not been independently reviewed.

**Level 4 - Managed**
RBAC is enforced consistently across AI systems, aligned to enterprise IAM roles. Audit trails are structured, retained, and have been reviewed in at least one audit. Automated responsible AI controls run on all production systems with defined thresholds. An active compliance programme maps controls to specific regulatory requirements with evidence collection.

**Level 5 - Optimising**
Fine-grained, policy-as-code RBAC is enforced dynamically across every AI system and data source. Continuous audit readiness runs with automated evidence collection mapped to regulatory requirements. Responsible AI testing includes automated red-teaming and regression gates before every release. Compliance assurance is continuous, with external audit readiness across every applicable framework.

---

## Named Patterns

### Pattern 1 - Policy-as-Code Enforcement

**What it is:** Access, content, and usage policies for AI systems expressed as code and enforced automatically at request time - not as a written policy document that relies on individual compliance.

**When to use it:** Any enterprise with more than a handful of AI systems or more than one team with independent AI access. Written-only policy does not scale past a small, trusted group.

**How it works:**
- Policies (who can access which model, with which data, under which conditions) are defined declaratively
- A policy engine evaluates every request against current policy before it is permitted
- Policy changes are version-controlled and deployed like code, not communicated via email or wiki update
- Denials and exceptions are logged for audit

**Trade-offs:**
- Requires an initial investment in defining policies precisely enough to encode - a valuable forcing function, but real up-front work
- Policy engines add a request-path dependency that must be highly available

**Tools that implement this pattern:**

| Tool | Best For | Limitations |
|---|---|---|
| Open Policy Agent (OPA) | General-purpose, cloud-native policy enforcement | Requires engineering effort to integrate at every enforcement point |
| AWS IAM / Azure AD Conditional Access | Organisations already standardised on one cloud's IAM | Less flexible for AI-specific policy logic (model, prompt, cost-based rules) |
| Custom policy middleware | AI-specific policy logic (model allowlists, cost budgets, content rules) | Build and maintenance cost |

**CloudKraft recommendation:** OPA at the AI gateway or middleware layer is the most flexible foundation for AI-specific policy-as-code, layered on top of - not replacing - the enterprise IAM system for identity and coarse-grained access.

---

### Pattern 2 - Continuous Responsible AI Testing

**What it is:** Automated testing for bias, toxicity, safety, and hallucination risk that runs continuously against production traffic and as a gate before every release - not a one-time pre-launch review.

**When to use it:** Every AI system with material user-facing or decision-influencing output, particularly in regulated industries.

**Trade-offs:**
- Automated testing catches known risk categories well but is not a substitute for periodic human red-teaming, which finds novel failure modes automated tests miss
- False positives from automated screening require a defined review and override process, or teams will route around the controls

---

### Pattern 3 - Regulatory Control Mapping

**What it is:** An explicit, maintained mapping from each applicable regulatory framework (OSFI E-23, NIST AI RMF, ISO 42001, EU AI Act, sector-specific requirements) to the specific technical and process controls that satisfy it.

**When to use it:** Any regulated enterprise, and any organisation that expects to face regulatory scrutiny of its AI systems within the next planning cycle.

**Typical findings:** In CloudKraft assessments, organisations frequently have most of the underlying controls in place already but cannot produce the mapping that shows an auditor which control satisfies which requirement - turning a straightforward audit into a scramble.

---

## Tool Selection Matrix - Governance & Risk Layer

| Capability | OPA | Presidio | Vault | Credo AI / Holistic AI | Custom |
|---|---|---|---|---|---|
| Policy-as-code enforcement | Excellent | No | No | Limited | Full control |
| PII / sensitive data detection | No | Excellent | No | Limited | Custom |
| Secrets and credential management | No | No | Excellent | No | Custom |
| Responsible AI / bias testing | No | No | No | Excellent | Custom |
| Regulatory control mapping | No | No | No | Good | Custom |
| Setup complexity | Medium | Low-Medium | Medium | Medium | High |
| Canadian data residency | Self-hosted | Self-hosted | Self-hosted | Depends on plan | Full control |

---

## Anti-Patterns

### Governance by Policy Document

**What happens:** A comprehensive AI usage policy is written, approved, and circulated - and then relies entirely on individual employees reading and complying with it. There is no technical enforcement.

**Why it happens:** Writing a policy document is far faster than building enforcement, and it satisfies an audit checkbox in the short term.

**Why it is dangerous:** Written-only policy fails exactly when it matters most - under time pressure, with a new hire who never read it, or with a team that quietly decides the policy does not apply to their use case.

**The fix:** Policy-as-code enforcement (Pattern 1) for anything that matters. Reserve written policy for guidance that genuinely cannot be encoded.

---

### The Audit Trail Gap

**What happens:** Logs exist somewhere, but nobody has verified that they contain enough detail to reconstruct an AI decision after the fact, or that they are retained long enough to matter.

**Why it happens:** Logging is implemented for debugging, not for audit, and the two have different requirements that only diverge visibly when an actual audit happens.

**Why it is dangerous:** Discovering a logging gap during a live regulatory inquiry is far more damaging than discovering it in a self-assessment.

**The fix:** Structure audit logs deliberately - what decision was made, on what data, by which model version, under which policy - and test audit readiness before a real audit forces the question.

---

### Compliance Theatre

**What happens:** A compliance inventory or control framework exists on paper, was populated once during a certification push, and has not been reviewed or updated since. Controls listed as "in place" no longer reflect what the systems actually do.

**Why it happens:** Compliance documentation and the systems it describes drift apart over time unless something forces them to stay synchronised.

**Why it is dangerous:** A false sense of compliance is worse than a known gap - it means the gap is discovered by an external auditor or a regulator, not internally.

**The fix:** Regulatory control mapping (Pattern 3) tied to automated evidence collection, so the mapping reflects what the systems actually do rather than what a document once claimed.

---

## Key Questions for a Client Engagement

1. If I asked you to prove that only authorised users can access your most sensitive AI system, what would you show me?

2. Walk me through the audit trail for a specific AI-assisted decision made last month. What can you reconstruct, and what is missing?

3. What responsible AI testing runs automatically before a model change reaches production, if any?

4. Which specific regulatory requirements apply to your AI systems, and which control satisfies each one?

5. When was your AI compliance documentation last independently reviewed against what the systems actually do?

The quality of the answers to these five questions is a reliable predictor of the overall Layer 3 maturity score.

---

## Relationship to Other Layers

**Layer 3 depends on Layer 1 (Data Readiness):** Data classification decisions made at Layer 1 are the input that Layer 3's access and handling controls actually enforce.

**Layer 3 constrains Layer 2 (Architecture Patterns):** Which integration and orchestration patterns are permissible depends on the governance and risk requirements attached to the data and use case involved.

**Layer 3 requires Layer 4 (Operating Model):** Policies and controls need a named owner and an enforcement mandate - without an operating model, governance requirements exist on paper only.

**Layer 3 is validated by Layer 5 (Integration & Scale):** Production observability is often the actual source of evidence that governance controls are working as designed, not just documented.

---

*Part of the [CloudKraft AI Control Framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
