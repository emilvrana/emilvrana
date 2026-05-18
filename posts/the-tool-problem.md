---
title: "The Tool Problem: Why More Tools Make Agents Worse"
date: 2026-05-18
tags: ["agents", "tools", "production", "architecture"]
---

# The Tool Problem: Why More Tools Make Agents Worse

Every new capability you add to an agent makes it weaker. Not stronger.

This sounds wrong. More tools = more power, right? An agent that can search the web, query a database, send emails, update a CRM, and generate reports is more capable than one that can only search the web.

Except it isn't. In practice, an agent with 15 tools performs worse than an agent with 5. The reasoning model spends its context budget deciding *which* tool to use instead of *how* to solve the problem. It picks the wrong tool. It calls three tools when one would do. It forgets what it was doing halfway through a tool chain because the context window is cluttered with tool definitions.

This is the tool problem, and I've watched it degrade every multi-tool agent system I've worked on.

## The Context Tax

Every tool you give an agent has a cost. The cost isn't the API call — it's the space in the context window that the tool definition occupies. A typical tool definition (name, description, parameters, examples) takes 200-500 tokens. Fifteen tools? That's 3,000-7,500 tokens before the user even says anything.

On a 128K context window, that's not a lot. But on the models most production systems actually use — the fast, cheap ones — you're working with 8K-32K effective context. Spend 7K of that on tool definitions, and your agent has less room for the actual task, conversation history, and reasoning.

The result: shorter reasoning chains, more errors, more retries, more tokens burned on failures.

## The Selection Problem

More tools means more decisions. Every time an agent needs to act, it has to choose from a larger menu. This is where things get subtly bad.

With 3 tools, the agent almost always picks the right one. The decision space is small, the descriptions are distinct, and the model can distinguish between them reliably.

With 15 tools, several will have overlapping descriptions. "Search the knowledge base" and "Query the documentation index" sound different to you. To a language model trying to pick one in 50 milliseconds, they're nearly identical. The agent picks the wrong one, gets a bad result, and either retries (burning tokens) or proceeds with garbage input (producing garbage output).

I've measured this. In one system, going from 5 to 12 tools dropped first-call tool selection accuracy from 92% to 71%. The remaining 29% weren't failures — they were the agent using a slightly wrong tool and producing slightly wrong results. The worst kind of failure.

## The Chaining Problem

Tools interact. An agent with 3 tools has 3 possible single-tool calls and 6 possible two-tool sequences. An agent with 12 tools has 12 single-tool calls and 132 possible two-tool sequences. The combinatorics explode.

In practice, this means the agent tries tool chains that shouldn't exist. It queries the CRM, then searches the web for information it already has, then sends an email based on the web search instead of the CRM data. Not because the logic is wrong — because the option space is too large and the model can't prune it effectively.

## What Actually Works

After building and debugging too many over-tooled agents, here's what I've learned:

**1. Start with fewer tools than you think you need.** Begin with 3-5. Add tools only when you can measure that the agent actually needs them. Not "might need them." *Needs them.* The default answer to "should we add another tool?" is no.

**2. Group tools by task, not by capability.** Instead of giving the agent 15 tools at once, create toolsets for specific tasks. An order-processing agent gets order tools. A research agent gets search tools. The routing layer — which agent handles which request — is a simple, deterministic switch. Not another LLM call.

**3. Make tool descriptions unambiguous.** Each tool description should make it crystal clear when to use *this* tool and when *not* to. "Searches the internal knowledge base for product documentation. Use this when the question is about product features, specifications, or troubleshooting. Do not use for general web searches or customer data." The more tools you have, the more important specificity becomes.

**4. Measure tool selection accuracy.** Don't just measure task completion. Measure whether the agent picked the right tool for the job. If selection accuracy drops below 90%, you have too many tools or poorly written descriptions. Fix one or both.

**5. Delete tools.** When you add a tool and task completion doesn't measurably improve, remove it. Dead tools still cost context. They still create decision noise. The best tool list is the shortest one that gets the job done.

## The Counter-Argument

"But what about GPT-4 / Claude / Gemini? They can handle lots of tools."

Some can, sometimes. The latest frontier models are better at tool selection than the models most production systems run on. But:

- You're probably not running the latest frontier model in production (cost, latency).
- Even frontier models degrade with tool count — just at a higher threshold.
- Your users don't care about the model's theoretical capability. They care about whether the system works.

Design for the model you're deploying, not the model you wish you were deploying.

## The Principle

Tool count is a design constraint, not a feature list. Every tool should justify its existence with measured improvement. Every tool that doesn't is making your agent worse.

The best agent I ever built had three tools. It did one thing and did it reliably. The worst agent I ever built had twenty-three. It did everything poorly.

Start small. Add deliberately. Measure relentlessly. Cut ruthlessly.

---

*Related: [Tool Selection Patterns](./tool-selection-patterns.md) · [When Not to Use an Agent](./when-not-to-use-an-agent.md) · [Your Agent's API Is a Contract](./your-agents-api-is-a-contract.md)*