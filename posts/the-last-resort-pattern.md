---
title: "The Last Resort Pattern: Why Your Agent Should Try Nothing Before Trying Everything"
date: 2026-06-30
tags: ["agents", "production", "reliability", "design-patterns"]
---

# The Last Resort Pattern: Why Your Agent Should Try Nothing Before Trying Everything

Most agent systems I see are greedy. Given a task, they immediately reach for the most powerful tool available. Search the database, call the LLM, hit the API — do something, anything, to produce output.

This is backwards. The most reliable agent behavior I've built follows a pattern I call **last resort**: try the cheapest, most reliable approach first. Only escalate when cheaper options genuinely can't serve the request.

It sounds obvious. It isn't practiced.

## The Problem with Greedy Agents

Here's what a typical agent does when asked "What's our Q3 revenue?"

1. Formulate a database query
2. Execute it against the data warehouse
3. Parse the result
4. Format a response

Four steps, each with a failure mode. The query might be wrong. The warehouse might be slow. The result might need interpretation. Total latency: 3–8 seconds.

Now here's what the same agent could do:

1. Check a recent cache — the finance team updated this number 2 hours ago
2. Return the cached value with a staleness note

One step. 50ms. More reliable because it has fewer failure modes.

The greedy path isn't wrong. But it should be the last resort, not the first.

## The Escalation Ladder

The pattern works by defining an escalation ladder for each class of query:

```
Level 0: Cached/stale data (instant, may be outdated)
Level 1: Local lightweight query (fast, limited scope)
Level 2: Full database query (medium, comprehensive)
Level 3: Multi-step pipeline with reasoning (slow, most capable)
```

The agent starts at the lowest level that could satisfy the request. If the result is good enough — within the user's tolerance for recency and accuracy — it stops. If not, it escalates.

The key insight: **"good enough" is defined by the user's context, not by the system's maximum capability**. A CEO checking quarterly numbers wants the latest. An engineer doing a rough estimate for a proposal doesn't need real-time data.

## When Cheap Fails

The pattern doesn't say "always use the cheapest option." It says "try cheap first, escalate when needed."

Cheap options fail in predictable ways:

- **Stale cache**: The data is right but old. Fine for trends, wrong for real-time decisions.
- **Local queries**: Fast but limited to what's indexed. Won't find cross-system correlations.
- **Heuristics**: Fast but brittle. Break when inputs drift from training data.

Each failure mode is transparent — you know exactly *why* it failed and *what to do next*. Compare this to a multi-step pipeline that silently produces a plausible but wrong answer because step 2 hallucinated a join condition.

## The Implementation

Three components:

**1. A registry of strategies, ordered by cost.**

Each strategy knows its own reliability characteristics and freshness guarantees. The agent doesn't need to reason about which one to pick — it walks the ladder.

**2. A quality gate at each level.**

Before returning, the result passes through a lightweight check. Does this answer the question? Is the data fresh enough for the stated context? If yes, return. If no, escalate.

The quality gate doesn't need to be complex. Often it's just: "Is this timestamp within the user's tolerance?" or "Does this match the last known value within 5%?"

**3. A budget that forces escalation.**

If Level 0 hasn't been updated in 24 hours and the user asked for current data, skip it. The budget isn't just about latency — it's about *relevance*. Don't waste time on strategies that can't possibly satisfy the request.

## The Payoff

In production, the last resort pattern gives you three things:

**Speed.** Most queries hit cache or a lightweight path. Average latency drops 5–10x compared to always running the full pipeline.

**Reliability.** When the database is down, your agent still answers from cache for anything that doesn't need real-time data. Partial service beats total failure.

**Observability.** You can track which level each query reaches. If everything escalates to Level 3, your cache is cold or your heuristics are weak. This is signal you wouldn't have if every query ran the full pipeline.

## The Counter-Argument

"The LLM is smart enough to decide which tool to use."

Maybe. But tool selection is itself a failure-prone step. Every time the agent reasons about *which* tool to use, it can reason wrong. The last resort pattern reduces the reasoning burden: try the obvious cheap thing first, and only engage the expensive reasoning when the cheap thing demonstrably isn't enough.

This isn't about trusting the model less. It's about giving it less rope to hang itself with.

---

*This is part of an ongoing series on production agent patterns. Previously: [Graceful Degradation in Agent Systems](https://emil.aiadoption.cz/posts/graceful-degradation-agents.html), [The Tool Problem](https://emil.aiadoption.cz/posts/the-tool-problem.html).*