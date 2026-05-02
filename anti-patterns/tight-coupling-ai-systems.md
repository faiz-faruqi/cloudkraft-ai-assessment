# Anti-Pattern: Tight Coupling of AI Systems

**Layer:** 5 — Integration
**Risk Level:** Medium
**How Common:** Common — the natural consequence of moving fast without integration architecture

---

## What Happens

AI systems are integrated directly and synchronously into business-critical workflows with no abstraction, no fallback, and no isolation between the AI component and the surrounding enterprise systems. The result is a fragile architecture where an AI provider outage, a model version change, or a prompt regression becomes a production incident that affects systems far beyond the AI component itself.

Common manifestations:

**Direct provider calls in business logic:** The enterprise application calls the OpenAI API directly from the same code path as a critical business transaction. When OpenAI has a 15-minute partial outage (which happens several times per year), the business transaction fails entirely.

**No model abstraction:** The application has `model="gpt-4o"` hardcoded throughout. Switching to a different model, a different provider, or a self-hosted option requires code changes across dozens of files and a full deployment.

**Synchronous AI blocking business transactions:** A customer-facing checkout flow calls an AI recommendation service synchronously. The checkout cannot complete until the AI responds. When the AI is slow (as it often is under load), the checkout experience degrades for all users.

**Shared schema between AI and enterprise systems:** The AI system writes directly to the production database tables used by the core business application. A schema change in the business application breaks the AI system. A bad AI write corrupts business data.

**No timeout or circuit breaker:** The application waits indefinitely for an AI response. Under load, connections pile up, memory is exhausted, and the entire application fails — not just the AI component.

---

## Why It Happens

Tight coupling is the path of least resistance in the short term. Calling an API directly is simple. Adding an abstraction layer, an event bus, a fallback, and a circuit breaker takes more time upfront. Teams under delivery pressure consistently choose the simple path, planning to "refactor later." Later rarely arrives before the first production incident.

The second driver is AI system exceptionalism — treating AI components as fundamentally different from other third-party services. Every experienced engineer knows that a direct, unguarded call to an external API in a critical code path is an antipattern. That knowledge is somehow suspended when the external API is an LLM provider.

---

## Why It Is Dangerous

**Single points of failure:** LLM API availability is not 100%. In 2024–2025, every major LLM provider (OpenAI, Anthropic, Google, Cohere) experienced meaningful outages. A tightly coupled AI system converts a provider partial outage into a business system outage.

**Cascading failures:** When AI components are synchronously embedded in request-response paths, slow AI responses cause thread pool exhaustion, connection pool exhaustion, or timeout avalanches that take down entire services — not just the AI feature.

**Vendor lock-in at the worst time:** When a provider has an outage, you want to route around it. If the model name and API format are hardcoded throughout the application, routing around the outage requires a code change and deployment — exactly when you do not have time for that.

**Uncontrolled data flow:** Direct AI calls with no gateway layer mean that what is sent to the AI provider and what is returned is not logged, not attributable, and not inspectable. This is a compliance gap and a debugging nightmare.

**AI regression becomes a business incident:** When an AI system is tightly coupled to business logic, a prompt regression or model update that degrades quality immediately affects business outcomes. With appropriate abstraction and a fallback, an AI quality regression degrades the AI feature while leaving the business process intact.

---

## Observable Symptoms

- `model="gpt-4o"` or equivalent is hardcoded directly in application code rather than in configuration
- There is no fallback behaviour if the LLM call fails — the entire request fails
- The application times out when the LLM provider is slow
- A 30-minute OpenAI outage would take down a customer-facing feature
- The AI system has direct write access to production database tables shared with non-AI systems
- Switching to a different LLM provider would require significant code changes throughout the application

---

## The Fix

**Abstraction layer:** Never call an LLM API directly from business logic. All LLM calls go through a gateway (see [Centralised AI Gateway](../patterns/centralised-ai-gateway.md)) or at minimum an internal abstraction that can be swapped without application code changes.

**Async over sync for non-critical paths:** AI enrichment, classification, summarisation, and recommendation should be asynchronous wherever the user experience allows. Submit the request, continue the business workflow, and apply the AI output when it arrives. This decouples AI latency from business transaction latency.

**Circuit breakers on every AI call:** Implement a circuit breaker (Resilience4j, Polly, or a custom implementation) around every external AI call. When the AI service is failing, the circuit breaker opens and fast-fails to the fallback path rather than waiting for timeouts to cascade.

**Explicit fallback paths:** Every AI feature must have a defined fallback for when the AI component is unavailable. The fallback may be a degraded experience (recommend nothing), a rule-based substitute (show the most recent 5 items), or a human escalation. Silence and hangs are not acceptable fallbacks.

**Model routing abstraction:** Use a gateway that abstracts the model selection. Application code requests a capability tier ("fast", "standard", "high-quality") rather than a specific model. The gateway maps capabilities to models and can reroute during outages without application changes.

**Isolated data stores:** AI systems should write to their own tables or collections. Writes to core business tables require an explicit, reviewed integration contract and should go through the same data validation as any other business data write.

---

## What Not to Do

**Do not add abstraction as a "nice to have" for future refactoring.** The abstraction layer pays for itself the first time you need to switch providers during an outage. Add it before the first production deployment.

**Do not use timeouts as your only resilience mechanism.** A 30-second timeout prevents infinite hangs but still degrades user experience for 30 seconds per failed request. A circuit breaker with a half-open state gives much better user experience by fast-failing immediately once failure patterns are detected.

---

## Related

- **Remediation pattern:** [AI Workflow Orchestration Bridge](../patterns/ai-workflow-orchestration.md)
- **Supporting pattern:** [Centralised AI Gateway](../patterns/centralised-ai-gateway.md)
- **Framework layer:** [Layer 5 — Integration](../framework/layer-5-integration.md)

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this and related integration gaps.*
