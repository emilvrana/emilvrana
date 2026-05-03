---
title: "Don't Build an Agent. Build a Pipeline."
date: 2026-05-03
tags: ["agents", "architecture", "engineering"]
---

# Don't Build an Agent. Build a Pipeline.

You have a task that involves multiple steps — fetch data, transform it, make a decision, send a notification. The AI Twitter consensus says: build an agent. Give it tools. Let it reason.

Here's the thing: most multi-step tasks aren't ambiguous. You *know* the steps. You just want them done reliably.

## The Agent Tax

An autonomous agent brings real costs:

- **Non-determinism.** Same input, different path each time. Good for exploration, terrible for production.
- **Token overhead.** Every step burns tokens on reasoning about *what to do next* — even when there's only one reasonable thing to do.
- **Debugging hell.** When it fails, you don't get a stack trace. You get a 2,000-token chain-of-thought that ended in a wrong tool call.
- **Latency.** The agent deliberates at each step. A pipeline just runs.

## When Pipeline Wins

Use a deterministic pipeline when:

1. **The steps are known.** You can enumerate them before runtime.
2. **Branching is simple.** If/else at most, not "let the model figure it out."
3. **Reliability matters more than flexibility.** Production systems, scheduled jobs, customer-facing workflows.
4. **You need to debug it at 3 AM.** Pipelines have traces. Agents have vibes.

Example: an invoice processing workflow — extract fields, validate amounts, look up vendor, create payment record. Every step is deterministic except field extraction, which you can handle with a single LLM call wrapped in a schema. No agent needed.

## When Agent Wins

Use an agent when:

- The task requires genuine **exploration** — research, open-ended search, creative generation.
- You **cannot** enumerate the steps in advance.
- The environment changes unpredictably and the system needs to adapt at runtime.

But be honest: most "agent" use cases I see in production are just pipelines with a chat interface bolted on top.

## The Hybrid Pattern

Here's what actually works well:

```
Pipeline (deterministic orchestration)
  └─ Step 3: LLM call (scoped, single-purpose)
  └─ Step 7: Agent sub-task (exploratory, bounded)
  └─ Step 9: LLM call (classification)
```

The pipeline owns the flow. LLM calls handle the fuzzy parts. An agent sub-task handles the rare case where you genuinely need exploration — but it's *scoped*, *bounded*, and *optional*.

This is the architecture most production systems converge toward anyway. Save yourself the iteration.

## The Rule

**If you can draw the flowchart before writing code, you don't need an agent.** You need a pipeline with LLM steps.

Agents are for the parts you can't draw.

---

*More on agent architecture: [When Not to Use an Agent](/posts/when-not-to-use-an-agent.html) · [The Evaluation Gap in AI Systems](/posts/the-evaluation-gap-in-ai-systems.html)*