---
title: "Model Portability: Why Your Agent Shouldn't Care Which LLM It Runs"
date: 2026-05-04
tags: ["agents", "architecture", "llm", "patterns"]
---

# Model Portability: Why Your Agent Shouldn't Care Which LLM It Runs

Everyone picks a model first, then builds around it. Wrong order. Build your agent so it doesn't care which model runs it, and you'll save yourself months of pain when you inevitably need to switch.

## The Lock-In You Didn't Notice

It starts innocently. You pick GPT-4, write prompts that work with GPT-4, tune tool descriptions for how GPT-4 interprets them, structure your context window around GPT-4's attention patterns. Everything works great.

Then the pricing changes. Or the API version shifts. Or you need a local model for a client with data residency requirements. And you discover your "model-agnostic architecture" has model-specific assumptions baked into every prompt.

Three months of work to rewrite prompts, retune tool schemas, restructure context. Three months where your system is half-broken.

## The Abstraction Layer That Actually Works

Not an ORM-style abstraction — those just add complexity. A thin contract between your agent logic and the model:

**1. Separate prompt templates from code.** Prompts live in their own files, versioned alongside your evals. When you switch models, you switch prompt sets. Your agent code doesn't change.

**2. Standardize on a tool schema, not a model's dialect.** Every provider has opinions about how tools should be described. Pick one format (OpenAI's function calling schema is the de facto standard) and translate at the adapter layer. Your agent defines tools once. The adapter handles the rest.

**3. Model config, not model identity.** Your agent shouldn't know if it's running GPT-4, Claude, or Gemini. It knows it has a context budget, a preferred output format, and a latency tolerance. The model name is a config value, not an architectural pillar.

## What Breaks When You Switch Models

The stuff nobody talks about until they try it:

- **Tool calling reliability drops.** Models interpret tool schemas differently. A parameter that GPT-4 fills reliably, Claude might skip half the time. Solution: minimal schemas with clear descriptions, and fallback handling in your tool executor — not in the prompt.

- **Context attention shifts.** GPT-4 and Claude weight context positions differently. Instructions at position 40k that GPT-4 respects, Claude might ignore. Solution: critical instructions at top and bottom. Redundancy is a feature.

- **Output format consistency.** Even with explicit format instructions, models drift differently. Solution: validate outputs against a schema before processing. Parse failures are normal, not exceptional.

- **Reasoning depth varies.** The same prompt that gets a thorough analysis from one model gets a surface-level summary from another. Solution: explicit step requirements in the prompt ("analyze, then evaluate, then recommend") rather than open-ended requests.

## The Portability Test

Before you're locked in, run this: take your agent, swap the model adapter, run your eval suite. If more than 30% of cases fail — you're locked in. Start decoupling.

The goal isn't to make every model perform identically. It's to make switching a configuration change, not an architecture rewrite. The model is a component. Treat it like one.

---

*Emil Vrána is an AI systems engineer based in Prague, building production automation infrastructure. He writes about the gaps between AI demos and production reality.*