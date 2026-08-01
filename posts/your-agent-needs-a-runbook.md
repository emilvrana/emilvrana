# Your Agent Needs a Runbook

You have observability. You have alerts. Something just paged you at 2 AM: your agent's error rate spiked. Now what?

You open the dashboard. You see red lines. You check the logs. There's a tool call failing. You look at the code. The tool hasn't changed. The API it calls hasn't changed. The model version hasn't changed. You scroll through 400 lines of agent reasoning to find the one sentence where it started constructing the wrong arguments.

Twenty minutes in, you're still reading. The alert is still firing. Your agent is still failing in production.

This is the runbook gap. Your service has a runbook. Your agent does not.

## What a runbook gives you

A runbook is not documentation. Documentation explains what the system does. A runbook explains what **you do** when it breaks.

For a traditional service, a runbook covers: how to restart, how to roll back, how to disable a feature flag, how to scale, who to escalate to. The answers are deterministic. Step 3 follows step 2. The system is predictable enough that predictability can be encoded.

Agents are less predictable, which makes runbooks more important, not less. When the failure mode is "the model started calling the wrong tool," you need a predefined response, not a late-night debugging session.

## What an agent runbook looks like

**Degradation modes.** What happens when each dependency fails? If the search API returns errors, should the agent skip it, retry with backoff, or fall back to a cached result? Define this ahead of time. At 2 AM, "just let it fail" is a valid answer, but it should be a conscious choice, not an accident.

**Circuit breakers.** What triggers a circuit breaker? How many failures in what window? What does the agent do when the breaker is open? A broken agent looping on retries is worse than an agent that gracefully stops and tells the user something is wrong.

**Rollback procedures.** Can you revert to a previous model version? A previous prompt version? How? Model updates are the most common cause of silent agent degradation. You need a one-command rollback, and you need to have tested it before you need it.

**Escalation paths.** When should a human be pulled in? What does the agent do while waiting? Define the thresholds: three consecutive tool failures, any instance of data exfiltration risk, confidence below a threshold on a critical path. The agent should never silently struggle in a loop that a human could resolve in seconds.

**Communication templates.** When the agent degrades, what does the user see? "I'm having trouble with X, let me try a different approach" is better than a spinning loader or, worse, a hallucinated answer. Write these messages in advance, when you're calm and thinking clearly.

## The pattern

Every incident you respond to, write down what you did. Not in a post-mortem that nobody reads — in a short, actionable runbook entry. "When tool X returns error Y, do Z." Over time, you build a decision tree that any on-call engineer can follow.

This is how you get from "I hope it works" to "I know what to do when it doesn't." Your agent will break. The question is whether you'll be reading logs or following a playbook.

---

*The best runbook is the one you wrote before the incident, not after.*