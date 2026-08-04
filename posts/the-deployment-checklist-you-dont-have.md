# The Deployment Checklist You Don't Have

You've got an agent. It works on your machine. You're about to ship it to production. Stop.

Ask yourself: what happens when it breaks? Not *if* — *when*. Because it will break. The model will timeout, the API will change its response format, the third-party service you depend on will rate-limit you at 3 AM on a Sunday.

Most AI agent deployments I see fail not because of bad prompts or wrong models, but because nobody thought about what happens after the "hello world" moment. Here's the checklist I wish every team had before they ship.

## 1. Can you observe it?

If your agent makes a decision and nobody can see *why*, you're flying blind. You need:

- **Structured logs** for every tool call, model response, and state transition
- **Trace IDs** that follow a request from trigger to completion
- **Metrics** on latency, cost per invocation, and success rate

Not "we'll add logging later." Later never comes. If you can't answer "what did the agent do last Tuesday at 2 PM?" right now, you're not ready.

## 2. Can you stop it?

An agent in a loop is a credit card on fire. You need:

- A **kill switch** — one command that stops all running agent instances
- **Execution timeouts** — hard limits, not polite suggestions
- **Budget caps** — maximum tokens or dollars per run, per day, per user

If your answer to "how do you stop a runaway agent?" involves logging into a dashboard and clicking three things, you don't have a kill switch. You have a prayer.

## 3. Can you roll it back?

You changed the prompt. The agent broke. Can you go back to the version that worked?

- **Version your prompts** like code. Git, not Google Docs.
- **Version your tool schemas.** A new required field is a breaking change.
- **Deploy incrementally.** One user first, then ten, then everyone.

The first time you need to roll back an agent, it will be 2 AM and you will be tired. Make the rollback path obvious before you need it.

## 4. Can you fix it without redeploying?

Hardcoding configuration into your agent binary is a trap. You need:

- **External prompt templates** you can edit without rebuilding
- **Feature flags** for new tool integrations
- **Model routing** you can swap without a code change

The half-life of an AI deployment is measured in weeks before something external changes. Build for that reality.

## 5. Does it fail gracefully?

What happens when the LLM returns garbage? When the API is down? When the context window overflows?

- **Validate outputs** before acting on them. An LLM that returns `null` is better than one that returns confidently wrong data.
- **Retry with backoff**, but with a limit. Three retries, then fail.
- **Return meaningful errors** to the caller, not stack traces.

A good agent fails like a good service: clearly, predictably, and without taking other systems down with it.

## 6. Do you know what "normal" looks like?

Before you ship, you need baseline metrics:

- What's the average latency? The 95th percentile?
- What does a successful run cost in tokens/dollars?
- How often does it succeed on the first try?

If you can't define "normal," you can't detect "abnormal." And abnormal is where all the interesting problems live.

## The point

None of this is rocket science. It's the same operational discipline we apply to any production service. But somehow, when we add an LLM to the stack, we forget everything we learned about reliability.

Don't. Your agent is software. Treat it like software. Ship it with the same care you'd ship any other production system — or don't be surprised when it breaks at the worst possible moment.

---

*Emil Vrána is an independent tech consultant in Prague, helping teams ship AI systems that actually work in production.*