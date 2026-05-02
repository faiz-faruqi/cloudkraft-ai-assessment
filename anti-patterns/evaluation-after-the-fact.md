# Anti-Pattern: Evaluation After the Fact

**Layer:** 4 — Evaluation & Trust
**Risk Level:** High
**How Common:** Very Common — the default path when delivery pressure is high

---

## What Happens

An AI system is built, tested informally by the development team, and shipped to production. Evaluation — systematic measurement of output quality against defined metrics and a representative test set — is deferred because "we'll add it later" or "we'll do it once we see what production traffic looks like."

Later never comes, or comes too late.

The typical progression:

1. **Development:** Team evaluates by trying queries manually. The system "seems to work." No golden dataset, no baseline metrics.
2. **Deployment:** System goes live. No evaluation framework. No baseline to compare against.
3. **Month 1–3:** System works reasonably well for the queries the team tested. Users find edge cases and report them informally. No systematic way to distinguish "always broken" from "recently regressed."
4. **Month 3–6:** Quality degrades due to one or more of: knowledge base staleness, upstream model change, prompt changes without evaluation, query distribution shift. No alert fires. The first signal is user complaints.
5. **Month 6+:** A significant quality incident occurs — a wrong answer that caused a downstream business problem. The post-mortem reveals that the system had been degrading for weeks and no one knew.

At this point, the team wants to add evaluation retroactively. The problem: there is no baseline. You cannot demonstrate a regression if you have no measurement of the original system state. You cannot defend "this used to work at 90% faithfulness and now it's at 70%" if you never measured the 90%.

---

## Why It Happens

Evaluation is consistently the first thing cut under delivery pressure, for reasons that seem reasonable in isolation:

- "Building the golden dataset takes two weeks and the deadline is next week"
- "We don't have enough production traffic yet to evaluate against real queries"
- "The system is clearly working — our developers tested it extensively"
- "We'll add proper evaluation in the next sprint"
- "Evaluation frameworks are complicated — let's ship and come back to it"

The underlying problem is that evaluation is not treated as a deployment prerequisite, in the same way that passing a test suite is a deployment prerequisite. When teams have discretion about whether to evaluate before shipping, they will consistently choose not to when under pressure.

---

## Why It Is Dangerous

**You cannot detect regression without a baseline.** If you shipped without measuring quality, you cannot prove the system has degraded or improved. Every quality incident becomes an argument rather than a measurement.

**Silent degradation is the most common AI failure mode.** LLM providers update models without announcement. Knowledge bases go stale without ingestion pipeline alerts. User query patterns drift outside the distribution the system was designed for. None of these changes generate errors or exceptions — they generate quietly wrong answers that users stop trusting.

**Compliance evidence requires a baseline.** Regulated organisations are increasingly required to demonstrate that AI systems met quality standards at deployment, not just that they are evaluated now. OSFI, in particular, expects evidence of pre-deployment validation for AI systems used in risk-relevant decisions. "We tested it and it seemed good" is not a documentable control.

**Retrospective evaluation is not a substitute.** You can add evaluation six months after deployment. What you cannot do is reconstruct the system state at deployment and prove it met standards that you never measured.

---

## Observable Symptoms

- No golden dataset exists for the production AI system
- The team cannot answer "what was the faithfulness score at deployment?"
- Quality issues are discovered through user complaints, not monitoring alerts
- When the development team makes a prompt change, there is no automated check that quality was maintained
- The LLM provider changed the model version and no one ran a quality check
- "We eyeballed it" is the evaluation methodology

---

## The Fix

**The minimum viable evaluation practice:**

1. **Before any production deployment:** Build a golden dataset of at minimum 50 representative queries with human-labelled expected answers. This takes 8–16 hours for a typical enterprise RAG system. It is not optional.

2. **Establish a baseline:** Run RAGAS evaluation against the golden dataset before the first deployment. Record the baseline scores for Answer Faithfulness, Answer Relevance, Context Precision, and Context Recall. Store these as the deployment baseline.

3. **Gate deployments on evaluation:** Add evaluation to the deployment pipeline. Every subsequent deployment runs the same evaluation suite. If scores drop more than 5 percentage points from baseline on any core metric, deployment is blocked pending investigation.

4. **Add production monitoring:** Deploy Langfuse (self-hosted) or LangSmith to capture production traces. Run LLM-as-judge evaluation on a 10% sample of production queries weekly. Alert on metric regression from the trailing 30-day average.

**The investment argument:**
The golden dataset build is the largest upfront cost — approximately 20–40 hours for a 100-question set with human-labelled answers. This is a one-time investment that pays back in the first quality incident it prevents. A single customer-facing quality failure in a regulated industry — a wrong policy interpretation communicated to a customer, an incorrect figure in a generated report — costs far more than 40 hours of dataset work to remediate.

**If you shipped without evaluation and want to add it now:**
1. Build the golden dataset from scratch — represent the current system's intended behaviour, not what you wish it did
2. Run evaluation on the current production system to establish the retroactive baseline
3. Document explicitly that this is a retroactive baseline, not a deployment-day measurement
4. Start the deployment gating process from the next deployment forward
5. You will not be able to prove the original system met standards, but you can prove that everything from this point forward does

---

## What Not to Do

**Do not use the development team's judgment as evaluation.** Developers are systematically biased toward their own system performing well. They test the queries they know work. A golden dataset built by people who were not involved in development produces a more honest quality picture.

**Do not use a golden dataset that only has easy questions.** A system that scores 0.95 faithfulness on 50 carefully selected, easy, unambiguous queries may score 0.60 on the actual distribution of production queries. The dataset must include hard questions, ambiguous questions, and questions the system is expected to decline.

**Do not treat one-time evaluation as continuous evaluation.** A baseline establishes the starting point. Without ongoing evaluation, you know the system was acceptable at deployment but have no information about its current state.

---

## Related

- **Remediation pattern:** [Continuous Evaluation](../patterns/continuous-evaluation.md)
- **Framework layer:** [Layer 4 — Evaluation & Trust](../framework/layer-4-evaluation-trust.md)
- **Tools:** RAGAS (metric framework), Langfuse (observability), DeepEval (evaluation framework)

---

*Part of the [CloudKraft AI Control Tower framework](../framework/README.md). Take the [AI Governance Maturity Assessment](https://assessment.cloudkraft.com) to score your organisation on this and related evaluation gaps.*
