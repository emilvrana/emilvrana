---
title: "Compound Latency: Why Your Agent Feels Slow Even When Nothing Is"
date: 2026-06-13
tags: ["agents", "performance", "production", "latency"]
---

# Compound Latency: Why Your Agent Feels Slow Even When Nothing Is

You benchmark each component. The LLM call? 800ms. The tool execution? 200ms. The RAG retrieval? 300ms. Everything looks fine. Individual latencies are well within acceptable range.

Then a user asks your agent a question and waits 12 seconds for an answer. Nothing timed out. Nothing retried. Nothing failed. So where did the time go?

Compound latency. Each step is fast. The pipeline is slow. And fixing individual components won't fix the pipeline.

## How Latency Compounds

A typical agent turn isn't one LLM call. It's a chain:

1. Receive user input → validate → route (50ms)
2. RAG retrieval → rerank → inject into context (500ms)
3. LLM call — decide which tool to use (800ms)
4. Tool execution (200ms)
5. Tool result injection back into context (50ms)
6. LLM call — process tool result and decide next step (800ms)
7. Repeat steps 4-6 for each tool call
8. Final LLM call — generate response (800ms)

Two tool calls and you're already at 3.2 seconds of LLM inference alone, plus retrieval, plus overhead. Three tool calls? Pushing 5 seconds. And that's the happy path — no retries, no reflection loops, no "let me think about this more carefully" steps.

The math is brutal because it's multiplicative, not additive. Each tool call adds a full LLM round-trip. Each reflection step doubles the inference cost. Each branching decision adds latency that can't be parallelized away because the next step depends on the previous result.

## Where the Time Actually Goes

When I profile slow agent systems, the distribution usually looks like this:

**LLM inference: 40-60% of total time.** Not because the model is slow — because it's called three, four, five times per turn. Multi-step agents are multi-call agents. Each call has its own latency floor.

**Context processing: 15-25%.** Large contexts take longer to process. A 4k-token system prompt with 20 tool definitions adds 200-300ms before the model even starts generating. Most teams never measure this separately.

**Tool execution: 10-20%.** The part everyone optimizes first. Usually the least impactful to optimize, because it's already the fastest component.

**Overhead and serialization: 5-15%.** JSON parsing, schema validation, state management, logging. Individually trivial. Collectively noticeable.

The surprising part: the biggest latency savings rarely come from optimizing the slowest component. They come from eliminating unnecessary components from the pipeline entirely.

## The Anti-Patterns

**The overthinking agent.** The agent calls three tools to gather information, then reflects on the results, then calls another tool to verify, then summarizes before responding. Five LLM calls for a question that needed one. The reflection step felt like a good idea in development. In production, it's a 3-second tax on every turn.

**The kitchen-sink context.** Every tool definition, every possible instruction, every edge case loaded on every call. The model spends 300ms just reading the system prompt before it can start thinking about the user's question. Most of those tools won't be called. Most of those instructions won't apply.

**The sequential-where-parallel-would-work.** The agent retrieves documents, then searches the web, then checks a database — all sequentially, because the pipeline was easier to write that way. Two of those three calls are independent and could run concurrently. Nobody noticed because each call is "fast enough" on its own.

**The retry-everything default.** A tool call fails. The agent retries. Same prompt, same context, same parameters. Of course it fails again — and now you've doubled the latency for that step. Naive retries don't fix latency problems. They compound them.

## What Actually Reduces Latency

**1. Reduce LLM calls, not LLM call time.** Can the agent answer in one call instead of three? Can you merge tool results before feeding them back, instead of making the model process each one separately? Every removed round-trip saves 800ms+ of inference and context processing. That's worth more than any model speedup.

**2. Parallelize independent operations.** If the agent needs data from two sources, fetch both simultaneously. If it needs to validate and retrieve, start both at once. This is the single highest-impact optimization in most agent pipelines, and the one most often missed because the sequential version "works fine."

**3. Trim context aggressively.** The agent doesn't need 20 tool definitions for a query about scheduling. It doesn't need the full conversation history for a simple lookup. Context is processed on every call. Trimming context to what's needed for this specific turn reduces both latency and cost.

**4. Cache ruthlessly.** Same user, same query structure, same tool results? Return the cached answer. Same RAG retrieval for similar queries? Cache the chunks. The best latency optimization is not doing the work at all.

**5. Set latency budgets per step.** Define how long each step is allowed to take. If RAG retrieval exceeds 300ms, return what you have rather than waiting. If the model hasn't started generating in 2 seconds, you're probably sending too much context. Hard budgets force hard decisions, and hard decisions produce faster systems.

## The Uncomfortable Measurement

Here's the exercise: add timing instrumentation to your agent pipeline. Log the time for each step — not just total, but per-call. Then ask yourself:

What percentage of total latency comes from steps that produce information the final answer actually uses?

In most systems I've profiled, the answer is 40-60%. The rest is overhead, redundant calls, over-inclusive context, and steps that produce information the model already had or didn't need.

Compound latency isn't a speed problem. It's an architecture problem. And you can't optimize your way out of an architecture problem — you have to simplify your way out.

Fewer calls. Smaller context. Parallel where possible. Cache where repeatable. The fastest agent step is the one you don't execute.

---

*This is part of an ongoing series on building AI agent systems that actually work in production. Related posts on [the context budget](/blog/the-context-budget/), [the retry trap](/blog/retry-trap/), and [silent failures](/blog/silent-failures/) cover different aspects of the same problem space.*