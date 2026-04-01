---
title: "Tool Selection Patterns in Agent Systems"
date: 2026-04-01
tags: ["agents", "llm", "architecture"]
---

# Tool Selection Patterns in Agent Systems

After working with autonomous agents for a while, you notice that "intelligence" isn't just about the model — it's about how the system decides *what* to do.

## The Problem

Give an agent 20 tools and watch it struggle. Each decision point becomes a routing problem. The naive approach ("describe all tools, let the LLM pick") breaks down quickly:

- **Token bloat**: Tool descriptions eat context
- **Ambiguity**: Similar tools confuse the model
- **Latency**: More options = slower decisions

## What Actually Works

### 1. Intent Classification First

Don't ask the LLM to pick from 20 tools. Ask it to classify intent first.

```
User request → Intent classifier (3-5 categories) → Tool picker (subset)
```

This two-stage approach cuts cognitive load dramatically. You go from "choose one of 20" to "choose one of 3" twice.

### 2. Semantic Tool Retrieval

Store tool descriptions in a vector index. Retrieve top-k relevant tools based on the query, not the full registry.

Key insight: Most queries only need 2-4 tools. The trick is knowing *which* 2-4.

### 3. Tool Grouping by Domain

Structure matters. Group tools by function:
- `file:*` — file operations
- `db:*` — database queries  
- `web:*` — external APIs
- `sys:*` — system commands

When the agent detects it's working with files, only `file:*` tools get exposed. Context window stays lean.

## The Trade-off

More routing layers = more latency per call, but fewer tokens per decision and fewer misfires. In practice, a 2-stage router with ~5 intents beats a flat 20-tool registry on both accuracy and cost.

## Practical Notes

- Keep tool descriptions under 50 words. Be concrete about inputs/outputs.
- Add examples to the prompt, not to the schema.
- Monitor tool selection accuracy. When it drifts, your descriptions are stale.

---

This is how I've been structuring the tool layer in my own agent setups. Not revolutionary, but the difference between "works sometimes" and "works reliably."

*Written from Prague, while debugging a misbehaving RAG pipeline.*
