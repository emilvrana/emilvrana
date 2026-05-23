# Agent Observability: You Can't Fix What You Can't See

May 23, 2026

You built an agent. It works in dev. You deploy it. And then... what?

Most agent projects have the same blind spot: they log errors but not behavior. You know when the agent crashes. You don't know when it drifts — when it starts calling the wrong tool 40% of the time, when its responses get 2x longer over a week, when it skips a step in the pipeline because the prompt got trimmed.

This isn't monitoring. Monitoring tells you the process is running. Observability tells you what the process is *doing*.

## The three signals that matter

**Latency distribution.** Not average latency — that hides everything. You need p50, p95, p99. When an agent's p99 jumps from 8s to 45s, something changed. Maybe the model started generating longer outputs. Maybe a tool endpoint is slow. Maybe the context window filled up and the model is spinning. Average latency won't tell you any of this.

**Tool call patterns.** Every agent has a signature: which tools it calls, in what order, how often. Log tool names, not just outcomes. If your agent normally calls `search` → `summarize` → `respond` and suddenly starts calling `search` → `search` → `search` → `respond`, the pattern broke. You don't need to know *why* yet — you need to know *that it happened*.

**Output shape.** Track response length, structured output parse success rate, and key field presence. If your agent is supposed to return JSON with `decision` and `confidence` fields, log whether those fields exist and what values they take. A drift from `confidence: 0.85` to `confidence: 0.6` over a week is a signal no error log will catch.

## What most teams get wrong

**Logging everything is not observability.** Dumping raw LLM inputs/outputs into a database gives you data, not insight. You need to extract structured signals *before* storage. The log is the raw material; the metric is the signal.

**Alerting on errors only.** Most agent failures don't produce errors. They produce wrong answers that look right. If you only alert on exceptions, you'll miss the slow degradation that accounts for 80% of production problems.

**Ignoring the feedback loop.** Observability without action is just expensive logging. When you detect drift, you need a response: revert the prompt, switch the model, alert a human, throttle traffic. The best teams automate the easy responses and surface the hard ones.

## A minimal observability stack

You don't need Jaeger and Grafana and a three-person SRE team. Start here:

1. **Structured logs.** Every agent run gets a trace ID. Log tool calls with timestamps and durations. Log the model, prompt version, and token count. This is 90% of the value.

2. **A simple dashboard.** Even a shell script that aggregates last 1000 runs by tool call count and latency percentile. You're not building a platform — you're building a habit of looking.

3. **A drift alert.** One threshold: if any key metric deviates more than 2 standard deviations from the 7-day rolling mean, send a notification. One alert. That's it.

4. **Weekly review.** Spend 15 minutes looking at the dashboard. Not debugging — just observing. Patterns emerge that no alert will catch.

## The uncomfortable truth

The teams with the best agent systems aren't the ones with the most sophisticated architectures. They're the ones who can tell you, right now, what their agent did in the last hour, whether it's operating within normal parameters, and what changed since last week.

If you can't answer those questions, you're not running a system. You're running a demo.

And demos don't survive contact with production.