# Layer 5 - Integration

**The question this layer answers:** How does the organisation connect AI systems to its existing enterprise applications, identity infrastructure, data platforms, and business processes in a way that is secure, maintainable, and compliant?

**Why this layer matters:** An AI system that exists in isolation is a prototype. An AI system integrated with your enterprise IAM, your core applications, your data platform, and your business processes is a production capability. The integration layer is where AI stops being a standalone tool and becomes part of how the organisation actually operates.

**Primary stakeholders:** Enterprise Architecture, Integration Engineering, Information Security, Operations

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
AI systems are not integrated with enterprise systems. Users copy-paste outputs between the AI tool and their business applications. Identity is managed separately in each AI system. Data residency for AI processing is unknown.

**Level 2 - Developing**
Basic API integrations built individually for each use case. Some AI systems use SSO but not all. Data residency requirements are known but enforcement is manual. No shared integration patterns.

**Level 3 - Defined**
Shared integration patterns documented and reused. SSO is standard for all AI-facing applications. Data residency requirements are documented and enforced for production systems. Integration decisions documented in ADRs.

**Level 4 - Managed**
Full IAM integration with role-based access control propagated to AI systems. Governed integration library with versioned, reusable adapters. Automated data residency controls. AI system availability monitored with defined SLAs.

**Level 5 - Optimising**
Zero-trust AI access model. End-to-end process automation with AI outputs driving workflow state transitions. Comprehensive data sovereignty framework. Continuous resilience testing including AI-specific failure scenarios.

---

## Named Patterns

### Pattern 1 - AI-Augmented API Layer

**What it is:** An API layer that adds AI capabilities to existing enterprise APIs without replacing them. The AI augmentation sits alongside the existing integration points - it does not become a dependency in the critical path of core business operations.

**When to use it:** Adding AI to existing workflows where the underlying business process must continue to function if the AI component is unavailable. The AI enhances the API response but is not required for the response to be valid.

**Example:** A document management API that normally returns document metadata. The AI-augmented version also returns a summary and key entity extraction - but if the AI service is unavailable, the API still returns the base metadata without failing.

**Trade-offs:**
- The non-blocking pattern adds complexity to the API implementation
- Users must understand that AI-enhanced fields are best-effort, not guaranteed
- Versioning becomes more complex when AI-enhanced fields are added to existing schemas

---

### Pattern 2 - Workflow Orchestration Bridge

**What it is:** Using an integration platform (n8n, Microsoft Power Automate, MuleSoft, or equivalent) to connect AI capabilities to existing business workflows without requiring changes to core enterprise systems.

**When to use it:** Organisations with stable core systems that cannot be modified easily, or where the AI use case is a workflow addition rather than a core system change. Also appropriate when the AI workflow logic needs to be accessible to non-developers for modification.

**Tool selection:**

| Tool | Best For | AI Integration | Self-Hosted |
|---|---|---|---|
| n8n | Technical teams, self-hosted, flexibility | Native LLM nodes, OpenAI-compatible | Yes |
| Power Automate | Microsoft 365 organisations, non-technical users | Copilot integration, Azure AI connectors | No |
| MuleSoft | Large enterprise, existing MuleSoft investment | AI extensions via custom connectors | No |
| Zapier | Simple integrations, non-technical users | Basic AI actions | No |
| Apache Camel | Java/enterprise, complex integration logic | Custom AI processors | Yes |
| Custom FastAPI | Full control, complex logic, performance | Native | Yes |

**CloudKraft recommendation:** n8n for technically sophisticated teams requiring self-hosted integration with data residency requirements. Power Automate for Microsoft 365 organisations where non-technical staff need to build and modify workflows. Custom FastAPI when integration logic is complex enough that a visual workflow tool adds friction rather than reducing it.

---

### Pattern 3 - Event-Driven AI Consumer

**What it is:** AI processing triggered by enterprise events published to a message bus or event stream, rather than by direct API calls or user requests.

**When to use it:** High-volume, asynchronous AI processing where results do not need to be returned synchronously. Document classification on upload, anomaly detection on data streams, automated triage of incoming requests.

**Key design considerations:**
- Event schema design: the event must contain all information the AI workflow needs, or a reference to where it can be retrieved
- Idempotency: the AI workflow must handle duplicate events gracefully
- Dead letter queue: events that fail processing must be captured for investigation, not silently dropped
- Back-pressure: the AI processing rate must be controllable to prevent runaway spend during traffic spikes

---

## Tool Selection Matrix - Integration Layer

### IAM Integration

| Approach | Best For | Complexity | Enterprise Support |
|---|---|---|---|
| Azure AD / Entra ID | Microsoft-committed organisations | Medium | Excellent |
| Okta | Multi-cloud, best-of-breed IAM | Medium | Excellent |
| AWS IAM + Cognito | AWS-committed organisations | Medium | Good |
| On-premise AD with federation | Legacy enterprise environments | High | Varies |
| Custom OIDC | Full control, specific requirements | High | Internal |

### Data Residency and Sovereignty

**Canadian-specific guidance:**

PIPEDA and provincial privacy laws (Quebec Law 25 in particular) require that personal data processed by AI systems is subject to appropriate safeguards. For AI systems processing personal data, this means:

- LLM API calls containing personal data must go to providers with Canadian or adequately protected data residency
- AWS ca-central-1 (Montreal) and Azure canadacentral (Toronto) are the primary options for hyperscaler-hosted AI with Canadian data residency
- Self-hosted models (Ollama, vLLM) on Canadian infrastructure are the highest-assurance option for sensitive personal data
- OpenRouter, direct Anthropic API, and direct OpenAI API do not offer Canadian data residency - route personal data through Azure OpenAI (canadacentral) or AWS Bedrock (ca-central-1) instead

**Financial services specific (OSFI B-13):**

OSFI Guideline B-13 (Technology and Cyber Risk Management) requires that federally regulated financial institutions maintain adequate oversight and control over technology risk, including AI systems. Key integration requirements:

- AI systems used in material business processes must be subject to the same change management and operational risk controls as other critical technology
- Third-party AI providers must be assessed under OSFI third-party risk management requirements
- Audit trails for AI-assisted decisions must meet the same standards as other regulated decision records

---

## Anti-Patterns

### Tight Coupling Between AI and Core Systems

**What happens:** An AI component is integrated directly into the critical path of a core business process. When the AI component is unavailable (provider outage, rate limit, model error), the entire business process fails.

**Example:** A loan application system that cannot process applications when the AI document classification service is unavailable. A customer service platform that cannot route tickets when the AI triage system is down.

**Why it is dangerous:** LLM providers have outages. Rate limits get hit. Models get updated and behave unexpectedly. A tightly coupled AI integration translates provider unreliability into business process downtime.

**The fix:** AI components in business-critical paths must have fallback behaviour. If the AI is unavailable, the process falls back to a rule-based approach, a human queue, or a degraded-but-functional mode. The AI enhances the process - it does not own it.

---

### Ignoring Existing Integration Middleware

**What happens:** A team builds a custom integration layer for their AI workflow, duplicating functionality that already exists in the organisation's integration platform. The custom integration becomes technical debt that is neither supported by the platform team nor maintained consistently with enterprise integration standards.

**Why it happens:** AI projects often start outside normal enterprise architecture governance. By the time architecture review is involved, the custom integration is already in production.

**The fix:** AI integration architecture should go through the same enterprise integration review as any other system integration. If the organisation has an integration platform, AI workflows should use it. Custom integration should require a documented exception with a justification that the platform cannot support the requirement.

---

### Rebuilding What iPaaS Already Does

**What happens:** A developer builds a custom event routing system, retry logic, dead letter queue, and workflow monitoring for their AI pipeline - features that are standard in any enterprise integration platform.

**The fix:** Evaluate the organisation's existing integration capability before building custom infrastructure. n8n, Power Automate, MuleSoft, and equivalent tools solve the reliability and routing problems that developers tend to rebuild from scratch. Use them.

---

## Key Questions for a Client Engagement

1. Pick your most business-critical AI workflow. What happens to the business process it supports when the AI component is unavailable? Is that acceptable?

2. How does a new employee get access to your AI-enabled tools? Is that process integrated with your standard IAM provisioning, or is it a separate manual process?

3. Walk me through what happens to data that a user submits to your AI system. Where does it go? What processing does it undergo? Where is it stored? Is that consistent with your data residency obligations?

4. If you needed to produce an audit trail of all AI-assisted decisions made by your organisation in the last 90 days, in a format suitable for a regulatory inspection, what would you give them? How long would it take to produce?

5. When was the last time you tested what happens to your AI-integrated systems when the LLM provider has an outage? What did you learn?

---

## Relationship to Other Layers

**Layer 5 connects to Layer 1:** IAM integration decisions (how calling applications authenticate to the gateway) are defined in Layer 5. The gateway enforces them.

**Layer 5 connects to Layer 2:** The ingestion pipeline that feeds the knowledge base is an integration concern - it connects source systems to the vector store through enterprise integration patterns.

**Layer 5 connects to Layer 3:** Event-driven AI triggers and workflow orchestration bridges are integration patterns that enable the orchestration use cases defined in Layer 3.

**Layer 5 is informed by Layer 4:** Evaluation findings may reveal integration failures (stale data from a broken ingestion pipeline, missing context from an incomplete IAM integration) that are root causes of quality degradation.

---

*Part of the [CloudKraft AI Control Tower framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
