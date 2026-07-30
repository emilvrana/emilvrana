# Run Your Agent Like You Run Your Production Service

You wouldn't deploy a web service without uptime targets, error budgets, or a runbook. But that's exactly how most AI agents go live.

An agent in production is a service. It has callers, dependencies, failure modes, and users who depend on it. The fact that one of its dependencies is an LLM doesn't change the fundamentals. If anything, it makes operational discipline more important, not less.

## The SRE playbook translates directly

**SLIs become agent metrics.** Instead of p99 latency, track time-to-first-token, tool-call success rate, and response validity. Instead of error rate, track the percentage of interactions that require human intervention. These are your agent's service-level indicators.

**SLOs become reliability targets.** "The agent should produce a useful response" is not an SLO. "90% of queries resolved without escalation, within 30 seconds, with valid output format" is. Write them down. Make them measurable. Hold yourself to them.

**Error budgets become release criteria.** If your agent's SLO allows 5% degraded responses per week, you have an error budget. Spend it intentionally — on a new model version, a prompt change, a tool upgrade. When the budget is gone, freeze deployments until you're back in bounds. This gives you a framework for balancing velocity and reliability that "vibes-based testing" never will.

**Runbooks become escalation paths.** When the agent fails, what happens? Who gets paged? Is there a fallback, or does the user stare at a spinner? If you can't answer these questions, you don't have an operational agent — you have a demo.

## What changes with LLMs

LLMs add a specific kind of nondeterminism that most services don't have. The same input can produce different outputs. Latency varies wildly. The "correct" output is often subjective.

This doesn't break SRE principles — it sharpens them:

**Version your prompts like you version your code.** Every prompt change is a deployment. Track it, roll it back if metrics degrade, and never change production prompts without measuring the impact.

**Canary your model upgrades.** New model version? Route 5% of traffic, compare against baseline, promote or roll back. Same process you'd use for any dependency upgrade.

**Define "correct" before you need to.** If you can't specify what a good response looks like in measurable terms, you can't measure whether your agent is working. Evaluation isn't optional — it's your monitoring.

## The uncomfortable truth

Most teams treat agent operations as an afterthought. They focus on the prompt, the model, the tools — and then deploy to production with no metrics, no SLOs, no runbook, no rollback plan.

The agent works. Until it doesn't. And when it doesn't, there's no framework for understanding why, no budget for deciding whether to push forward or pull back, no playbook for recovery.

SRE isn't about preventing all failures. It's about making failures visible, contained, and recoverable. That's exactly what production agents need.

Run your agent like you run your production service — because that's what it is.