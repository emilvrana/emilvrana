---
title: "When Agents Break: Debugging Patterns for LLM Pipelines"
date: 2026-04-29
tags: ["agents", "debugging", "llm", "patterns"]
---

# When Agents Break: Debugging Patterns for LLM Pipelines

Agent systems don't crash like traditional software. They drift. A pipeline that worked yesterday silently produces worse output today. A tool call that should take one step spirals into five. The model "forgets" context that's clearly in the prompt.

After debugging enough of these, patterns emerge. Here's what I reach for.

## 1. Check the Prompt, Not the Code

My first instinct is always to look at the code. Wrong. In agent systems, the prompt *is* the code. Nine times out of ten, the model is doing exactly what you asked — you just asked something ambiguous.

Before touching code, dump the full prompt the model received. Not what you think you sent. What actually arrived. Log the raw request.

## 2. Replay, Don't Rerun

When a pipeline fails, don't just run it again. Rerunning changes timing, API load, even model version on some providers. Instead, capture the exact inputs and replay them.

I log every model call with input hash, model version, and output. When something breaks, I replay that exact call. If it reproduces — real bug. If it doesn't — nondeterminism or provider change. Either way, you've learned something.

## 3. The Silent Degradation Pattern

The scariest bugs aren't failures. They're silent quality drops. Your agent still runs, still produces output, just... worse. Maybe the model provider changed a default. Maybe a rate limit is causing truncated responses. Maybe context window pressure is silently dropping earlier messages.

Defense: baseline metrics. Run a small eval suite periodically. If your agent normally extracts 3-5 key points from a document and suddenly it's extracting 1-2, something changed. You won't notice without measuring.

## 4. Tool Call Spirals

The classic: agent calls a tool, doesn't like the result, calls it again with slightly different params, loops. This isn't a bug in the tool — it's a feedback loop in the agent's decision loop.

Fix: hard limits. Every tool gets a max retry count. Every pipeline gets a max step budget. Not as error handling — as *architecture*. If the agent can't solve it in N steps, escalate or fail gracefully.

## 5. Context is Not Memory

This one bit me hard. Your agent has access to a 128k context window. It can read everything. But "can read" ≠ "will use." Models have attention patterns — they weight recent and prominent tokens higher. Critical context at position 40,000 might as well not exist for many models.

Practical fix: put the most important instructions at the top and the bottom of the prompt. Repeat critical constraints. It feels wasteful. It isn't.

## The Meta-Pattern

All of these share a root cause: we treat LLM pipelines like deterministic software. They're not. They're probabilistic systems that need probabilistic debugging — logging, measurement, baselines, and replay.

Debug the system, not the code.

---

*Comments? Find me at [emil.aiadoption.cz](https://emil.aiadoption.cz)*