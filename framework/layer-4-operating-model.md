# Layer 4 - Operating Model

**The question this layer answers:** Who owns AI outcomes, how is the AI delivery team structured, and how are vendors and investment decisions governed?

**Why this layer matters:** The best data readiness, architecture, and governance controls in the world do not sustain themselves without an operating model behind them. This is the layer that determines whether good practice in the other four layers is a durable organisational capability or a set of habits that erode the moment the engineer who cared about them moves teams.

**Primary stakeholders:** Head of AI, CTO, PMO

---

## What Good Looks Like at Each Maturity Level

**Level 1 - Initial**
No single owner exists for AI programme outcomes - initiatives are pursued independently by whichever team wants to. AI work happens wherever an engineer decides to build it, with no coordination. Vendors and tools are adopted independently with no evaluation criteria. Leadership has no visibility into programme progress.

**Level 2 - Developing**
An informal owner exists but has no real authority or budget. Ad hoc pockets of AI work exist in a few teams with no coordination between them. Vendors are selected informally. Ad hoc updates reach leadership only when asked for.

**Level 3 - Defined**
A named owner exists with partial authority over some AI initiatives. A central team exists but has no formal mandate to govern AI work happening elsewhere. A basic vendor evaluation process exists but is applied inconsistently. Periodic status reports reach leadership but lack consistent metrics.

**Level 4 - Managed**
A defined owner - typically a Head of AI - has authority and budget across most AI initiatives. A hub-and-spoke model is in place: a central team sets standards, business units execute against them. A formal vendor selection and review process exists with a maintained scorecard. Regular reporting against defined KPIs reaches leadership.

**Level 5 - Optimising**
An accountable executive owns AI outcomes enterprise-wide with a clear mandate and board visibility. A mature AI Centre of Excellence coordinates federated delivery teams against shared standards. A vendor governance board reviews all AI vendors against risk, cost, and capability criteria on a defined cadence. Programme governance includes board-level reporting with standardised maturity and ROI metrics.

---

## Named Patterns

### Pattern 1 - AI Centre of Excellence (Hub-and-Spoke)

**What it is:** A central team that sets standards, provides shared platform capabilities, and reviews architecture, while business units retain their own delivery teams that build against those standards.

**When to use it:** Mid-size to large enterprises with more than a handful of AI use cases spread across multiple business units. It is the pattern that scales without either centralising all delivery (too slow) or leaving every unit fully independent (ungovernable).

**How it works:**
- A central team owns the platform layer: the gateway, shared patterns, governance tooling, and standards
- Business units retain delivery ownership and domain expertise for their own use cases
- New initiatives above a defined risk or spend threshold go through a lightweight central architecture review
- The central team publishes and maintains the pattern library that business units build against

**Trade-offs:**
- Requires genuine executive sponsorship to give the central team enough authority to matter - a CoE without teeth becomes a Level 3 "central team with no mandate"
- Business units may perceive central review as a bottleneck if review turnaround is not kept fast

**CloudKraft recommendation:** Start the CoE with a narrow, clearly-scoped mandate (platform + architecture review above a defined threshold) rather than attempting to centralise all AI delivery. Expand scope as the model proves itself, rather than over-scoping from day one and losing business unit buy-in.

---

### Pattern 2 - Federated Ownership with Central Standards

**What it is:** Full delivery autonomy for business units, with adherence to centrally published standards (architecture patterns, governance controls, tool allowlists) as the only central constraint - no mandatory central review.

**When to use it:** Organisations where a CoE with review authority is not currently achievable politically, or as a transitional step toward one.

**Trade-offs:**
- Relies on business units voluntarily adopting central standards - weaker guarantee than a CoE with review authority
- Easier to stand up quickly, but tends to drift back toward Level 1-2 without periodic reinforcement

**CloudKraft assessment:** This is a reasonable starting point, not an end state. Organisations that stay here long-term typically see standards compliance erode as the number of AI initiatives grows.

---

### Pattern 3 - Vendor Governance Board

**What it is:** A recurring, cross-functional review of AI vendors and tools against defined risk, cost, and capability criteria - rather than each team independently selecting and contracting with vendors.

**When to use it:** Any organisation with more than a small number of distinct AI vendor relationships, or operating in a regulated industry where third-party AI risk is a compliance concern.

**How it works:**
- New AI vendors go through an intake process covering security, data handling, and cost before contracting
- Existing vendors are reviewed on a defined cadence (typically annually, or on contract renewal)
- A maintained scorecard captures risk tier, cost trend, and capability fit for every active vendor
- Findings feed both procurement decisions and the Layer 3 third-party risk programme

**Typical findings:** In CloudKraft assessments, organisations are frequently surprised by how many active AI vendor relationships exist once a proper inventory is done - many originated as a single team's pilot and were never centrally reviewed.

---

## Operating Model Selection Guide

| Model | Best For | Watch-outs |
|---|---|---|
| Centralised delivery | Small organisations, early-stage AI programmes, or highly regulated single-function environments | Becomes a bottleneck as demand for AI capability grows |
| Hub-and-spoke (CoE) | Mid-size to large enterprises with multiple business units building AI | Requires real executive sponsorship to avoid becoming a toothless committee |
| Federated with central standards | Organisations not yet ready for formal review authority | Standards compliance tends to erode without periodic reinforcement |
| Fully federated, no central function | Not recommended for any regulated enterprise | Produces the fragmentation this framework exists to prevent |

---

## Anti-Patterns

### No Single Owner

**What happens:** AI initiatives are pursued independently across the organisation with no one accountable for enterprise-wide outcomes, cost, or risk. When something goes wrong, or the board asks a pointed question, there is no single person positioned to answer.

**Why it happens:** AI capability often emerges bottom-up from individual teams before the organisation has decided who should own it enterprise-wide.

**Why it is dangerous:** Without ownership, none of the other four layers get sustained investment - governance, architecture, and data readiness all decay without someone accountable for maintaining them.

**The fix:** Name an accountable owner - even before that role has full authority or budget - and grow the role's mandate as the AI programme matures.

---

### Vendor Sprawl Without a Scorecard

**What happens:** Dozens of AI tool and vendor relationships accumulate over time, each originating from an individual team's pilot, with no central inventory, risk review, or cost visibility.

**Why it happens:** Signing up for a new AI SaaS tool is fast and often does not require procurement involvement for smaller contracts.

**Why it is dangerous:** Beyond the direct cost, each ungoverned vendor relationship is a potential data handling and compliance gap that the organisation cannot even enumerate, let alone manage.

**The fix:** Vendor Governance Board (Pattern 3) with a mandatory intake process for any new AI vendor above a defined spend or data-sensitivity threshold.

---

### The Committee Without a Charter

**What happens:** An AI governance committee or steering group is formed, meets periodically, and discusses AI strategy - but has no defined decision rights, no budget, and no authority to enforce anything it decides.

**Why it happens:** Forming a committee feels like progress and is politically easier than granting an individual or team real authority.

**Why it is dangerous:** It creates the appearance of governance without the substance, which can be more damaging than no committee at all - it slows decisions down while providing no actual control.

**The fix:** Any governance body needs a written charter with explicit decision rights, escalation paths, and - critically - budget or veto authority over something real.

---

## Key Questions for a Client Engagement

1. If the board asked who is accountable for AI programme outcomes, who would answer that question?

2. Walk me through how a new AI initiative gets funded, staffed, and approved in your organisation today.

3. Can you produce a current list of every AI vendor your organisation has an active relationship with, and when each was last reviewed?

4. What authority does your central AI team (if one exists) actually have over what business units build?

5. What does your AI programme report to leadership, how often, and against what metrics?

The quality of the answers to these five questions is a reliable predictor of the overall Layer 4 maturity score.

---

## Relationship to Other Layers

**Layer 4 sustains Layer 1 (Data Readiness) and Layer 3 (Governance & Risk):** Data contracts and policy enforcement both require a named, empowered owner - without an operating model, they exist on paper only.

**Layer 4 enables Layer 2 (Architecture Patterns):** Shared, governed architecture patterns require a central function with the mandate to define and maintain them across teams.

**Layer 4 is tested by Layer 5 (Integration & Scale):** Scaling AI from pilot to enterprise-wide production is where operating model gaps become visible - a programme without clear ownership struggles to scale coherently.

---

*Part of the [CloudKraft AI Control Framework](./README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this layer.*
