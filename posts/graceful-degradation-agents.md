# Graceful Degradation in Agent Systems

Your agent calls an API. The API is down. What happens?

If you're like most teams I've worked with, the answer is: the agent crashes, retries blindly, or returns a generic error that tells you nothing. The whole pipeline stops because one component failed.

This is the wrong default.

## The Pattern: Fail Open, Not Closed

Production systems degrade. Databases slow down, APIs rate-limit, models return gibberish. The question isn't whether these things happen — it's whether your agent can continue doing *something useful* when they do.

The principle is borrowed from network design: **fail open, not closed**. When a component is unavailable, the system should do the best it can with what remains, not refuse to operate at all.

For agent systems, this means:

- **If the search tool is down**, the agent should still be able to answer from its training data, clearly noting the limitation
- **If the database is slow**, the agent should serve cached results with a staleness indicator
- **If a sub-agent times out**, the parent should continue with partial results, not block indefinitely

## Three Practical Strategies

### 1. Capability Advertising with Health Checks

Every tool in your agent's registry should report its own health. Not just "up/down" — but degraded states:

```python
tool_status = {
    "web_search": "healthy",
    "database": "degraded",  # slow, 5s latency
    "email": "down"
}
```

The agent then reasons about which tools are available *this request* and adjusts its plan. This is more useful than a health endpoint that returns 200 while the service is melting.

### 2. Tiered Response Quality

Instead of binary success/failure, define quality tiers:

- **Full fidelity**: all tools available, real-time data
- **Degraded**: some tools unavailable, cached or approximate data, clearly labeled
- **Minimal**: no external tools, model knowledge only, explicit disclaimer

The agent's response should always indicate which tier it operated at. Users respect "I couldn't verify this against live data" more than a silent guess.

### 3. Timeout Budgets, Not Infinite Waits

Set a budget for how long the agent will wait for any single tool. Not a global timeout — a *budget* that the agent allocates across steps:

```python
total_budget = 30  # seconds
spent = 0
for step in plan:
    remaining = total_budget - spent
    if remaining < step.estimated_time:
        step.degrade()  # use faster, less accurate alternative
    result = step.execute(timeout=min(remaining, step.max_time))
    spent += result.time
```

This ensures the agent respects the user's time even when tools don't respect theirs.

## When Degradation Isn't Appropriate

Not everything should degrade gracefully. Some operations are atomic:

- **Financial transactions** — partial execution is worse than failure
- **Security decisions** — deny by default when uncertain
- **Data mutations** — a partial write is often worse than no write

The trick is knowing which category your operation falls into. If the cost of being wrong is higher than the cost of being slow, don't degrade — fail explicitly.

## The Mindset Shift

Most agent development focuses on the happy path: perfect inputs, available tools, responsive models. The happy path is the demo path.

Production is the unhappy path. The API that returns 503. The model that hallucinates. The database that's 10x slower than usual.

Designing for degradation isn't pessimism — it's engineering. Your agent's value isn't measured by how well it works when everything's perfect. It's measured by how much it can still deliver when everything isn't.

---

*This is part of a series on building AI agents that survive contact with reality. Previously: [Agent Evaluation Is Not Optional](https://emil.aiadoption.cz/blog/agent-evaluation/), [When Your AI Agent Goes Quiet](https://emil.aiadoption.cz/blog/silent-failures/).*