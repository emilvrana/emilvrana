# Your Agent Doesn't Need More Tools

**August 11, 2026**

Every agent project reaches the same crossroads. The agent works for the happy path. It fails on edge cases. Someone suggests: "What if we give it another tool?"

This is almost always the wrong answer.

## The Tool Proliferation Trap

Here's what happens when you add tools to fix reliability:

1. The agent calls the wrong tool because two tools overlap in capability.
2. The agent calls the right tool with wrong parameters because the tool surface area grew.
3. The agent calls three tools sequentially when one would do, because it can.
4. The agent hallucinates a tool that doesn't exist, but close enough to a real one that it tries.
5. You add another tool to handle the new failure mode.

Each tool you add multiplies the decision space. The model doesn't get smarter — it gets more options to be wrong about.

I've seen agents with 30+ tools where 8 of them were essentially the same operation with different parameter shapes. The model picked the wrong variant roughly 40% of the time. Refactoring to 2 tools with union-typed parameters brought accuracy from 60% to 94%.

## Fewer Tools, Better Contracts

The fix isn't better prompts. It's fewer, more precise tools.

**Merge tools with overlapping responsibility.** If `search_customers_by_email` and `search_customers_by_name` both query the same table, they should be one tool: `search_customers` with an optional `field` parameter. The model now has one thing to remember, not two.

**Make tools reject early.** A tool that silently accepts bad input and returns garbage is worse than a tool that rejects invalid input immediately. The model learns from clear errors. It can't learn from plausible-looking wrong data.

**Remove tools you don't use.** Audit your tool registry every sprint. If a tool wasn't called in the last 1000 agent runs, it's noise. Remove it. If you need it later, add it back. Unused tools are free in code but expensive in model attention.

**Prefer composition over configuration.** Instead of `create_user_with_permissions_and_settings`, offer `create_user` and let the agent call `set_permissions` and `configure_settings` in sequence. Smaller tools are easier for the model to understand and combine. But — and this is the key — only if the smaller tools have clear contracts. If the model needs to pass an ID from step 1 into step 2, make that explicit in the tool description.

## The Real Problem

Tool proliferation is a symptom. The real problem is unclear boundaries.

When you don't know what an agent should be responsible for, you give it more tools and hope it figures out the scope. It won't. The agent will try to use every tool available, because the model has no concept of "not my job." It has a hammer collection and everything looks like a nail.

Before adding a tool, ask:

- Does this tool belong to this agent's responsibility?
- Can an existing tool handle this with a minor parameter change?
- Will this tool create ambiguity with another tool?
- Am I adding this because the agent needs it, or because I haven't defined what the agent *shouldn't* do?

## The Metric That Matters

Track tool-selection accuracy. Not call success rate — that's downstream. Measure how often the agent picks the right tool for the job before you look at whether it used the tool correctly.

If tool-selection accuracy is below 90%, adding more tools will make it worse. Remove tools until selection accuracy is high, then add new ones one at a time, measuring the impact.

Your agent doesn't need more tools. It needs fewer, better ones. And it needs you to decide what it's not responsible for.