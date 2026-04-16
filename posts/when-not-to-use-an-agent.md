---
title: "When Not to Use an Agent"
date: 2026-04-16
tags: ["agents", "architecture", "engineering"]
---

# When Not to Use an Agent

Everything is an agent now. Wrappers, chatbots, cron jobs with an LLM call — all "agents." But most things that work reliably in production aren't agents at all. They're pipelines.

## The Agent Premium

An agent adds complexity tax:

- **Non-determinism**: Same input, different output. Sometimes wildly different.
- **Cost**: Every decision is an LLM call. Ten decisions = ten calls.
- **Latency**: Thinking takes time. Pipelines execute in milliseconds.
- **Debugging**: When it breaks, *why* did it break? Good luck reproducing it.

A pipeline does none of this. Same input, same output. Deterministic. Fast. Debuggable.

## The Rule of Thumb

Use an agent when the **path is unknown**. Use a pipeline when the **path is known**.

That's it. Most business processes have known paths. Invoice processing? Pipeline. Customer onboarding? Pipeline. Data validation? Definitely pipeline.

Use agents for the **gaps between pipelines** — the edge cases, the judgment calls, the "I don't know what category this falls into" moments.

## The Hybrid Pattern

The best systems I've built look like this:

```
Pipeline → Agent (only when stuck) → Pipeline
```

A deterministic workflow handles 90% of cases. When it hits something it can't classify or decide, it hands off to an agent. The agent resolves the ambiguity and returns control to the pipeline.

This gives you:
- Determinism for the happy path
- Flexibility for edge cases
- Auditability (you know exactly where the agent intervened)
- Cost control (the LLM only runs when needed)

## Real Example

I built a document processing system that extracts data from invoices. Version 1 was a full agent — it read, classified, extracted, validated. Cost: ~$0.12 per invoice. Success rate: 85%.

Version 2: regex + rules extract known fields, an LLM handles only the fields the rules missed, then a validation pipeline checks everything. Cost: ~$0.01 per invoice. Success rate: 97%.

The agent didn't go away. It just stopped doing work the pipeline could do better.

## The Uncomfortable Truth

If your agent's tool calls follow a predictable sequence most of the time, you don't have an agent. You have a pipeline wearing a trench coat. Write the pipeline. It'll be faster, cheaper, and more reliable.

Save the agent for the moments that actually need judgment.

---

*More on building practical agent systems at [emil.aiadoption.cz](https://emil.aiadoption.cz)*