# Your Agent's Context Window Is Not a Database

There's a pattern I keep seeing in agent systems: someone builds a capable agent, it works great in a demo, and then in production it starts forgetting things. Not hallucinating — genuinely forgetting. Tasks it handled yesterday, decisions it made an hour ago, facts it looked up five turns back — gone.

The root cause is almost always the same: treating the context window like persistent storage.

## Context ≠ Memory

The context window is a working surface. It's RAM, not disk. It's where your agent thinks, not where it remembers.

Here's what actually happens as a conversation grows:

1. **Older context gets compressed.** Most production systems use some form of compaction — summarizing earlier turns to stay within limits. Each compression is lossy.
2. **Important details survive. But not all important details.** A summarizer might keep the conclusion but drop the reasoning. The "why" evaporates, leaving only the "what."
3. **Mid-conversation facts are the most vulnerable.** Too old to be in recent context, too recent to have been explicitly summarized. They simply fall through the cracks.

I've watched agents re-query the same API three times in one session because the first two results got compacted away. I've seen agents contradict themselves — making a decision, then later making the opposite decision because the first one was no longer in context.

## The Architecture That Works

After running agents in production for months, the pattern that actually works is embarrassingly simple:

**Explicit memory, separate from context.**

```
Context window → working memory (ephemeral, fast, lossy)
External store  → long-term memory (persistent, queryable, structured)
```

The agent writes important things to the external store. When it needs them, it queries. The context window stays lean — just the current task and recent turns.

This isn't novel. It's how every competent software system works. You don't store your application state in CPU registers and hope it's still there after a context switch.

## What to Store

Not everything deserves persistence. The signal-to-noise ratio matters.

**Store:** decisions (and why), key facts, task state, corrections, patterns.
**Don't store:** raw tool outputs, intermediate reasoning, anything you can recompute.

A good heuristic: if losing this information would cause the agent to make a different decision, store it. Otherwise, let it go.

## The Query Problem

The hard part isn't storage — it's retrieval. If your agent has to load everything back into context to find something, you've just moved the overflow problem.

What actually works:

- **Semantic search** over stored memories (embeddings + vector store)
- **Structured indices** for exact lookups (task ID → status, entity → attributes)
- **Tags and metadata** for filtered retrieval

The agent shouldn't need to "remember" where it put something. It should ask a question and get an answer — the same way you search your notes, not the way you recall a childhood memory.

## The Real Cost

Here's the uncomfortable truth: a memory system adds complexity. Another database to run, another failure mode, another thing to keep in sync. For a weekend project or a demo, the context window is fine.

But if your agent runs continuously — handling tasks over days and weeks, maintaining state across sessions, building on previous decisions — then you need real memory. And you need to design for it from the start, not bolt it on when things start breaking.

The context window is a fantastic scratchpad. It's a terrible filing cabinet.

---

*April 30, 2026*