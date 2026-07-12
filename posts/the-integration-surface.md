---
title: "The Integration Surface: Why Your Agent Lives at the Edges"
date: 2026-07-12
tags: ["agents", "integration", "architecture", "production"]
---

# The Integration Surface: Why Your Agent Lives at the Edges

Everyone focuses on the model. Which LLM, what context window, how many parameters. But here's the thing: in production, the model is the least interesting part of your agent.

The interesting part is everything the model touches.

I call it the **integration surface** — the sum of all APIs, databases, file systems, message queues, and human interfaces your agent reads from and writes to. And in every production agent system I've worked on, the integration surface is where the real complexity lives, where the real failures happen, and where the real engineering is required.

## The Model Is the Easy Part

You pick a model. You write a prompt. You wire up some tools. The agent works on the first demo. The model generates reasonable text. Tools get called. Results come back.

Then you deploy.

And suddenly the model is 5% of your problems. The other 95% are:

- The API you're calling changed its response format without versioning
- The database connection pool is exhausted because your agent holds transactions open across LLM calls
- The webhook your agent sends to has a 30-second timeout but your LLM takes 45 seconds to respond
- The file your agent wrote has UTF-8 BOM and the downstream parser chokes
- The user's timezone wasn't in the prompt and now the scheduled task fires at 3 AM their time

None of these are model problems. They're integration problems. And they're the ones that keep you up at night.

## Why the Integration Surface Is Hard

**Impedance mismatch.** LLMs deal in natural language. APIs deal in schemas. The translation between "the user wants last month's revenue" and `GET /api/reports?period=2026-06&metric=revenue&format=json` is where things break. Not because the model can't construct the request — it usually can — but because the error handling, pagination, rate limits, and data normalization are where the real logic lives.

**State management.** Most APIs are stateful. The agent needs to know that the order was already placed, that the file was already processed, that the user already confirmed. Where does that state live? In the agent's context window? In a database? In both? When they disagree, which wins?

**Failure modes multiply.** A single agent with 5 integrations has 5 independent failure surfaces. An agent that chains 5 integrations in sequence has up to 2^5 failure combinations. Your retry logic needs to understand which failure means "try again" and which means "the whole pipeline is in an inconsistent state, roll back."

**Observability gaps.** You can log the LLM's input and output. But can you see what happened at the boundary between your agent and the payment processor? Between the agent and the database? The integration surface is where agents go dark — and where you need the most visibility.

## The Contract Is the Architecture

Here's the shift: stop thinking about your agent as "an LLM with tools." Start thinking about it as **a system of contracts at the boundaries.**

Each integration point is a contract. It specifies:

- **Shape.** What goes in, what comes out. Schemas, types, required fields.
- **Semantics.** What a successful response means. What an error means. What idempotency guarantees exist.
- **Failure.** What happens when it breaks. Timeouts, retries, fallbacks, partial states.
- **Observability.** What gets logged, when, and where.

The LLM is the component that navigates between these contracts. It's a router, a translator, a decision engine. But the contracts are the architecture.

Design the contracts first. Then wire the model to them.

## Practical Patterns

**Schema-first integration.** Define the input/output schemas for every external call before you write a single prompt. JSON Schema, Zod, Pydantic — whatever your stack uses. The model's job is to produce data that conforms to the schema. The schema's job is to reject data that doesn't.

**Idempotent writes.** Every write operation your agent performs should be idempotent. If the agent calls `POST /api/orders` twice with the same idempotency key, the second call should be a no-op. This isn't optional — it's the difference between "sometimes creates duplicate orders" and "never does."

**Circuit breakers at every boundary.** If an external service is down, your agent shouldn't keep hammering it. A circuit breaker pattern at each integration point prevents cascade failures and gives you time to respond.

**Structured errors.** When an integration fails, return a structured error that the model can reason about. "HTTP 500" tells the model nothing. `{ "error": "payment_provider_timeout", "retryable": true, "suggested_delay_seconds": 60 }` gives the model a path forward.

**Observability at the seams.** Log every integration call: what went in, what came out, how long it took, whether it succeeded. Not the LLM's internal reasoning — the actual data that crossed the boundary. That's what you'll need when things break.

## The Uncomfortable Truth

Most agent projects fail at the integration surface. Not because the model isn't smart enough, but because connecting an LLM to real systems is systems engineering, and systems engineering is harder than prompt engineering.

The teams that ship reliable agents are the ones that spend 80% of their time on the integration surface and 20% on the model. Not the other way around.

Your agent's intelligence lives in the model. Your agent's reliability lives at the edges.

Design accordingly.

---

*Comments? Thoughts? I'm [emil@aiadoption.cz](mailto:emil@aiadoption.cz).*