---
title: "The Hidden Cost of Context: Why Your Agent Forgets"
date: 2026-04-22
tags: ["agents", "context", "memory", "llm", "architecture"]
---

# The Hidden Cost of Context: Why Your Agent Forgets

You've built an agent. It works great for five minutes. Then it starts repeating itself, losing track of earlier decisions, or hallucinating facts it already confirmed. The model hasn't changed. The tools haven't changed. What happened?

The answer is almost always the same: **your context window filled up, and nobody told you.**

## The Budget You Didn't Know You Had

Every LLM has a fixed context budget — 8K, 32K, 128K, 200K tokens. It feels enormous until you start counting:

- System prompt: ~1-2K tokens
- Tool definitions: 2-5K for a well-equipped agent
- Conversation history: grows linearly, 500-2K tokens per turn
- Tool outputs: the silent killer — a single API response or file read can eat 3-10K tokens

By turn 15, you're often spending 40-60% of your budget on history. By turn 30, you're rationing. The model doesn't warn you. It just starts performing worse — less accurate retrieval, more generic responses, skipped steps.

This isn't a model quality problem. It's an economics problem.

## Three Strategies (Pick Two)

### 1. Compaction: Shrink the Past

Instead of keeping raw conversation history, periodically summarize older turns. Turn 1-10 becomes a 200-token summary. You lose detail, you gain runway.

The tradeoff is real: compaction is lossy. Once you summarize a debugging session into "explored X and Y, found issue in Z", you can't recover the specific error messages or intermediate hypotheses. For task execution, this is usually fine. For open-ended exploration, it hurts.

**Rule of thumb:** Compact aggressively for task agents, conservatively for research agents.

### 2. External Memory: Move State Out of Context

Write important facts to a database, vector store, or even a JSON file. On each turn, retrieve only what's relevant.

This is how humans work — we don't keep every conversation we've ever had in working memory. We write things down and look them up. Your agent should do the same.

The hard part isn't storage. It's **knowing what to store and what to retrieve**. Store decisions, not process. Retrieve context, not history.

- ✅ "User prefers dark mode, timezone UTC+1, deploys via Docker"
- ❌ "I asked user about dark mode, user said yes, I confirmed..."

### 3. Budgeted Execution: Plan Before You Spend

Before the agent starts, estimate how many turns the task needs and allocate context accordingly. A simple classification task? 5 turns, keep full history. A multi-step debugging session? 30 turns, compaction at turn 10, external memory for intermediate findings.

This requires the agent (or its orchestrator) to have a rough task model. Not a full planner — just enough to say "this is a 10-turn job" vs "this could go long."

## The Compaction Trap

Here's a subtle failure mode: you compact, the model forgets an earlier constraint, violates it, and you spend 5 turns recovering. Net context cost: higher than if you hadn't compacted.

This happens when compaction drops *constraints* — rules the user specified early on. The fix: separate constraint storage from conversation history. Keep constraints in the system prompt or a dedicated "always-retrieved" section. Never compact them.

## What I Actually Do

For my own agent systems, I use a hybrid approach:

1. **System prompt** holds identity, rules, and critical constraints (~2K tokens, fixed)
2. **Working memory** holds the last N turns verbatim (N=8-12, rolling window)
3. **Compacted history** summarizes everything older into a ~500 token block
4. **External store** holds facts, decisions, and task state — retrieved per-turn via semantic search

Total per-turn overhead: ~3-4K tokens for the memory layer. The rest of the budget goes to actual work. On a 128K context model, this buys 50+ turns of effective agent work without degradation.

## The Uncomfortable Truth

Most "agent reliability" problems are context management problems in disguise. The model didn't forget — you never gave it a way to remember.

If your agent works in a demo but degrades over time, check your context budget before you check your prompt. The fix is usually architectural, not linguistic.

---

*Building agent systems in Prague. More notes at [emil.aiadoption.cz](https://emil.aiadoption.cz).*