# The Agent Error Budget

**August 13, 2026**

Production teams have a concept from site reliability engineering: the error budget. If your SLA promises 99.9% uptime, you get 0.1% error budget — roughly 43 minutes of downtime per month. You can spend it on deployments, on experiments, on known risks. When it's gone, you stop shipping and focus on reliability.

Agent systems need the same concept. Here's why.

## You Already Have One

Your agent fails. Not sometimes — regularly. It misclassifies intents, calls the wrong tool, produces plausible-sounding nonsense. Right now you treat every failure as an incident. You're living in permanent firefighting mode with no budget framework.

The error budget flips the question. Instead of "why did it fail?" you ask "do we still have budget left?" If yes, the failure is within tolerance. If no, it's time to harden.

## Setting the Budget

Start with the business impact of a wrong action. Not the technical error rate — the actual cost.

- An agent that drafts internal summaries can tolerate 10% inaccuracy. A human reviews anyway.
- An agent that sends customer emails can tolerate maybe 1%. Wrong message to wrong person is a trust violation.
- An agent that executes financial transactions can tolerate near zero. One wrong transfer is a crisis.

Your error budget isn't a percentage you pick from thin air. It's derived from who catches the mistake and what it costs when they don't.

## Spending the Budget

Error budget is a resource. Spend it deliberately.

**Testing in production.** Every A/B test on your agent prompt or model version costs budget. That's fine — that's what it's for. But track it. If your test caused 15 more errors this week and you only had 10 in budget, roll back.

**Known rough edges.** Your agent can't handle edge cases in date parsing. You know this. It's in the budget. Document it. When budget runs out, fix it.

**New capabilities.** Adding a new tool? New domain? The error rate will spike. Budget for it. Don't pretend the first week will be smooth.

## When the Budget Runs Out

This is where most teams fail. They blow past the error budget and keep shipping. The SRE playbook is clear: when the budget is exhausted, you freeze non-essential changes and focus on reliability.

For agents this means:

1. Stop adding features, tools, or prompt changes.
2. Run the failure analysis you've been deferring.
3. Fix the top three error patterns.
4. Re-measure. Only resume changes when you're back under budget.

Most teams skip step 2-4 and go straight back to shipping. The error budget becomes a suggestion, not a constraint. This is how you accumulate prompt debt.

## The Dashboard You Need

Track these weekly:

- **Total agent actions** (denominator)
- **Actions that needed human correction** (numerator)
- **Budget remaining** (target minus actual)
- **Top 3 failure patterns** (specific, not "it was wrong")

One number on a dashboard changes the conversation from "the agent isn't working" to "we're at 73% of our error budget this month and the biggest pattern is tool selection on ambiguous queries." The second sentence leads to action. The first leads to frustration.

## The Point

You can't eliminate agent errors. You can budget for them, track them, and decide when they matter. The error budget turns reliability from a vague aspiration into a management discipline. Your agent will fail. The question is whether you planned for it.