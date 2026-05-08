# Your Agent's API Is a Contract

May 8, 2026

Most AI agent projects focus on what the agent can do. Very few focus on what the agent promises to do. The difference matters more than you think.

When you build an agent that "responds to natural language," you've built a demo. When you build an agent that accepts a structured request, produces a typed response within a latency bound, and degrades gracefully when it can't — you've built a system other software can depend on.

## The Demo-to-Product Gap

Here's a pattern I've seen repeated:

1. Build an agent that works great on happy-path inputs
2. Deploy it behind a chat interface
3. Real users send edge-case inputs
4. Agent produces inconsistent, slow, or broken responses
5. Add more prompts, more guardrails, more model calls
6. System becomes fragile and expensive

The root cause isn't the model. It's that the agent never had a contract.

## What a Contract Looks Like

A real API has:

- **Input schema** — what fields are required, what types, what ranges
- **Output schema** — what the caller gets back, every field typed
- **Error contract** — what happens when things go wrong, with structured error types
- **Latency bounds** — how long the caller should wait before giving up
- **Versioning** — what changes are safe, what require migration

Your agent needs all five. Not as documentation — as enforceable code.

**Input schema** means you validate before the model sees the request. Catch the obviously broken inputs at the API layer. "I need a customer ID and a date range" is a contract. "Ask me anything" is a demo.

**Output schema** means you validate after the model responds. If the agent is supposed to return a list of actions with confidence scores, validate that structure. If the model returns free text instead, that's not "creative" — it's a contract violation.

**Error contract** means your agent has well-defined failure modes. A timeout is different from a validation error is different from "I don't have enough information." Each maps to a specific HTTP status, a specific error code, a specific recovery path for the caller.

## The Pattern That Works

Here's what I use:

```
Request → Validate → Route → Execute → Validate → Respond
              ↓                       ↓
          400 Bad Request        Retry or Fallback
```

The two validation steps are the contract. The first catches bad inputs before they waste model tokens. The second catches bad outputs before they reach the caller.

Between them, routing decides: is this a task the agent should handle, or should it be sent to a deterministic pipeline? Not every request needs an LLM. The contract makes this routing explicit.

## What This Costs

Structure isn't free. You spend time defining schemas instead of tweaking prompts. You write validation code instead of adding more examples. You think about failure modes before they happen.

But you gain something most agent projects never have: the ability to reason about your system's behavior without running it. You can look at the contract and know what's guaranteed, what's best-effort, and what's undefined. You can write tests against the contract, not against the model's mood.

And when the model changes — and it will — the contract catches regressions before your users do.

## Start Small

You don't need a full OpenAPI spec on day one. Start with:

1. A Pydantic model (or Zod schema) for your inputs
2. A Pydantic model for your outputs
3. Three error types: validation, timeout, and unknown
4. A timeout on every model call

That's it. That's your v1 contract. Everything else builds on top of this foundation.

The model is the engine. The contract is the chassis. You can swap engines. You can't swap a missing chassis.

---

*More on building reliable AI systems at [emil.aiadoption.cz](/).*