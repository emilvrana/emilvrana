---
title: "Your Agent's State Is a Liability"
date: 2026-06-18
description: "Accumulated context makes agents brittle. Why stateless agent designs outperform stateful ones, and how to prune what you carry."
---

# Your Agent's State Is a Liability

Every agent starts clean. Empty context, fresh instructions, ready to work. Then it runs — and accumulates state. Conversation history, tool results, intermediate decisions, cached facts. Each step adds weight.

Most agent frameworks treat this accumulation as a feature. "The agent remembers everything!" But in production, state is a liability. Here's why, and what to do about it.

## The compounding problem

State degrades non-linearly. An agent with 10k tokens of context is fine. At 50k, it starts making subtle errors — referencing outdated information from earlier in the conversation, conflating two similar tool calls, losing track of what it already decided. At 100k, you're flying blind.

This isn't a model limitation. It's a fundamental property of information: **the more you carry, the harder it is to find what matters.** Your agent doesn't need everything it's seen. It needs the right thing at the right time.

## Three patterns that make state worse

**1. Verbatim tool outputs.** A search returns 2000 tokens. Your agent dumps the whole thing into context "just in case." Multiply by 5-10 tool calls per task. You've now got 10-20k tokens of raw output that the model has to navigate every single inference.

**2. Conversational meandering.** The user changes direction, corrects themselves, explores tangents. The agent faithfully carries the entire conversation. But half of it is irrelevant to the current task.

**3. Implicit state drift.** The agent makes a decision early ("use database X"), carries it implicitly, then contradicts itself later because the original decision is buried in 30k tokens of context and nobody can find it.

## The stateless-by-default pattern

The fix isn't "better prompts." It's architectural:

**Summarize, don't carry.** After each major step, extract the decision and discard the reasoning. "Queried the database. Result: 3 active users." Not the 2000-token JSON response.

**Re-derive instead of remember.** If you need to know which database to use, re-read the config. It's one tool call and 50 tokens, versus carrying 500 tokens of context "in case" you need it again.

**Partition state by scope.** Global state (user identity, permissions) stays. Task state (current step, intermediate results) gets a shelf life. Conversational state (the user said "actually, use the other one") gets applied and then discarded.

**Reset aggressively.** Between tasks, wipe everything except what you explicitly chose to keep. A clean agent is a reliable agent.

## When state is worth keeping

Not all state is bad. Some state is genuinely useful:

- **User preferences** that apply across sessions
- **Schemas and tool definitions** that don't change within a session
- **Compressed summaries** of long conversations

The difference: **state you keep should be small, stable, and verified.** If it's large, changing, or assumed — prune it.

## The practical test

Ask yourself: "If I restarted this agent from scratch with only the current task description, would it produce a worse result?" If the answer is no, your state isn't helping. If the answer is yes, identify exactly what it needs from the past, compress that to its minimal form, and carry only that.

State is a cost. Treat it like one.

---

*More on building reliable agent systems at [emil.aiadoption.cz](https://emil.aiadoption.cz)*