---
title: "The Context Budget: Why Your Agent's Memory Is a Resource, Not a Feature"
date: 2026-05-15
tags: ["agents", "context", "llm", "production"]
---

# The Context Budget: Why Your Agent's Memory Is a Resource, Not a Feature

Everyone wants agents with "long memory." More context, more history, more tools loaded — surely that makes the agent smarter?

No. It makes it slower, more expensive, and less reliable.

Context window is not a feature. It's a budget. And like any budget, you need to spend it deliberately or you go bankrupt — except instead of money, you're burning latency, cost, and coherence.

## The Three Taxes of Context

Every token you put into a context window carries three taxes:

**Latency.** More tokens means more time to first output. This isn't linear — at the upper bounds of context, you start seeing meaningful response delays. Your "intelligent" agent takes 30 seconds to think about something that should take 3.

**Cost.** Input tokens are priced per token. Load a 50-message conversation history, 40 tool definitions, and a RAG chunk with 10k tokens of retrieval, and you're paying for all of it on every single call. I've seen agent pipelines spending 70% of their LLM budget on context that contributed nothing to the output.

**Coherence.** This is the hidden one. LLMs don't attend equally to all tokens. The more irrelevant context you stuff in, the more the model's attention dilutes. Your agent starts missing things that are right there in the prompt, not because it can't see them, but because the signal-to-noise ratio dropped below useful.

## How Context Budgets Go Wrong

Here's the pattern I keep seeing in production agent systems:

**The hoarder.** Every conversation, every tool result, every intermediate step gets appended. The context grows linearly. After 20 turns, the agent is processing 30k tokens of history for a 200-token task.

**The kitchen sink.** Every available tool definition, every possible schema, every edge case instruction gets loaded into the system prompt "just in case." The agent spends more time parsing instructions than doing work.

**The RAG overfeeder.** Retrieval returns 20 chunks "for relevance," but the agent only needed 2. The other 18 are noise that dilutes the answer and costs money.

**The state stasher.** The agent serializes its entire internal state into context on every turn. Structs, configs, logs — all of it. It's like printing the entire heap to stdout and piping it to the next function call.

All of these come from the same misconception: that more context is better. It's not. More context is more context. Better context is better.

## Spending Deliberately

A context budget works like a financial one. You have a fixed amount. You allocate it. You cut waste.

**1. Measure first.** Before you optimize anything, instrument your agent calls. Log input tokens, output tokens, and latency per turn. You'll probably find that 80% of your context cost comes from 20% of your turns — and half of those tokens contributed nothing to the output.

**2. Summarize, don't accumulate.** Instead of keeping the full conversation history, summarize older turns. You don't need the exact wording of turn 3 when you're on turn 15. You need the decision that was made and why. A 200-token summary beats a 5000-token transcript.

**3. Load tools lazily.** Don't inject 40 tool definitions into every call. If the agent is in a "file editing" phase, it doesn't need the email tool. Dynamic tool loading based on the agent's current task can cut system prompt tokens by 60-80%.

**4. Rerank, don't just retrieve.** If your RAG pipeline returns 20 chunks, run a cheap reranker and keep the top 3-5. The model doesn't need to see every vaguely relevant document. It needs the ones that actually answer the question.

**5. Set hard limits.** Define a maximum context budget per turn. If you're over budget, something gets cut — oldest history, least relevant retrieval, or unused tools. This forces you to make explicit choices about what matters, which is the whole point.

## The Real Metric

The question isn't "how much context can I give the agent?" It's "what's the minimum context the agent needs to produce a correct response on this turn?"

Every token beyond that minimum is waste. And in production, waste compounds.

Start budgeting.

---

*Comments? Thoughts? I'm [emil@aiadoption.cz](mailto:emil@aiadoption.cz).*