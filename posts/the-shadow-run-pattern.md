# The Shadow Run Pattern

**August 15, 2026**

You deployed your agent. It works on the happy path. You ran your evals. The numbers look fine. But you still don't trust it with real customers, and you're right not to.

The gap between "evals pass" and "safe in production" is where most agent projects stall. You could deploy and hope. You could add more guardrails and deploy slightly less hopefully. Or you could run the agent alongside the system that already works — no impact, full observation, real data.

This is the shadow run. It's borrowed from traffic shadowing in service deployment, and it solves the specific problem that agent evals can't.

## Why Evals Aren't Enough

Evaluation sets tell you your agent performs well on the cases you thought to test. Production gives you the cases you didn't think to test. This is not a failure of evals — it's their nature. Every eval set is a sample, and production is the full distribution.

The gap isn't small. I've watched agents pass 95% of eval questions cleanly and then hit a pattern in production that nobody imagined: a customer name that looks like a SQL keyword, a tool response in an unexpected language, a date format that breaks the agent's parsing because the eval data used ISO and the real system uses something else.

You can't close this gap with more evals. You close it by running against real inputs before the outputs matter.

## What a Shadow Run Is

A shadow run means your agent processes the same inputs as your production system, in real time, but its outputs go nowhere. They're logged, compared, and scored — not acted on.

Your existing system handles the customer. Your agent watches the same incoming request, generates its response, and the response is compared to what the real system did. Nobody sees the agent's output except you.

The pattern:

1. **Intercept the input.** The same request that goes to your production handler goes to the agent. Real data, real timing, real edge cases.
2. **Run the agent.** Full pipeline — tool calls, context assembly, model inference. The agent doesn't know it's in shadow mode. It does what it would do.
3. **Compare outputs.** The agent's output against the production system's output. Manual review for a sample, automated scoring for the rest. Where they agree, you learn the agent is safe for that class of input. Where they diverge, you find your next eval cases.
4. **Log everything.** Latency, tool calls, token usage, failures. The shadow run is your observability layer for the agent you haven't deployed yet.

## What You Learn That Evals Don't Teach

**Latency under real load.** Your eval suite runs one request at a time. Production gives you bursts, queueing, and concurrent tool calls. Shadow runs expose the latency distribution you'll actually see.

**Tool selection in context.** Evals test tool selection in isolation. Production gives the agent a context window full of previous interactions, system state, and noise. The agent might pick a different tool when it has 4,000 tokens of context than when it has 200.

**Failure modes you didn't predict.** Every shadow run week produces at least one "I didn't think of that" case. These become eval cases. Your eval set grows from real failures instead of imagined ones.

**Cost distribution.** You'll see what the agent actually costs per request when it has freedom to call tools multiple times, retry, and think. The eval suite's cost numbers are almost always lower than reality.

## When to Stop Shadowing

You shadow until you have confidence. Confidence is not a feeling — it's a metric:

- **Agreement rate** with the production system stabilizes above your threshold (I use 90% for most systems, 95% for anything customer-facing).
- **No new failure modes** appear for a sustained period — I use two weeks of production traffic.
- **Latency profile** is within your budget at the 95th percentile, not the median.
- **Cost per request** is predictable and within budget.

When all four hold, you promote the agent. Not all at once — you route 5% of traffic to it, watch for a day, then 25%, then 100%. Shadow runs don't end at deployment; they end when the agent is the production system.

## The Hard Part: Comparison

The challenge of shadow runs is that your agent's output often isn't identical to your production system's output, even when both are correct. An agent might phrase an email differently, use a different tool sequence, or take a different path to the same result.

This means you can't just diff outputs. You need a comparison layer:

- **Semantic equivalence** for text outputs: same meaning, different words.
- **Outcome equivalence** for actions: same end state, different path.
- **Safety scoring** for anything that touches external systems: did the agent try to do anything the production system wouldn't?

Manual review for the first 1,000 cases. After that, you build a comparison model — an LLM call that judges whether the agent's output is equivalent to or better than the production output. It's not perfect, but it scales.

## The Trap

Shadow runs can become a permanent state. "Just one more week of observation." "We found a new edge case, let's extend." The agent is never good enough, the shadow run never ends, and the agent never ships.

Set a deadline before you start. "We shadow for four weeks. If agreement is above threshold, we deploy. If it's not, we fix the agent and shadow for two more weeks. Then we deploy or kill the project." A shadow run with no deadline is a project with no ending.

## The Point

Shadow runs aren't free. You're running a second system in parallel, paying for the compute, building the comparison infrastructure, and reviewing outputs. It's real work.

But it's cheaper than deploying too early. A bad agent in production costs you customers, trust, and the political capital to try again. A shadow run costs you compute and engineering time. The math is simple.

Ship when the data says ship. Not before, and not after.