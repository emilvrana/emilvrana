# Idempotency: Why Running Your Agent Twice Shouldn't Create Two Tickets

You build an agent that creates Jira tickets from Slack messages. It works perfectly in testing. Then a network hiccup causes a timeout — your agent retries and now you have duplicate tickets. A webhook fires twice and a customer gets charged twice. An agent loop re-runs and sends the same email three times.

This isn't a theoretical problem. It's the first thing that breaks in production.

## What Idempotency Actually Means

In HTTP, `PUT` is idempotent — calling it once or ten times produces the same server state. `POST` is not — each call creates a new resource.

Agent systems are almost entirely `POST`. They create things: tickets, emails, database rows, API calls. Every action that creates is a potential duplicate if the agent runs again.

The fix isn't "don't retry." The fix is making your actions idempotent by design.

## Three Patterns That Work

### 1. Deduplication Keys

Before creating anything, check if it already exists. Use a natural key — a Slack message timestamp, a hash of the input, a composite of relevant fields.

```python
def create_ticket_from_message(msg):
    dedup_key = f"slack-{msg.channel}-{msg.timestamp}"
    existing = jira.search(f"dedup_key = {dedup_key}")
    if existing:
        return existing[0]
    return jira.create_ticket(
        summary=msg.text[:100],
        dedup_key=dedup_key,
        ...
    )
```

This is boring and it works. Every external-facing action in your agent should have a dedup key.

### 2. Idempotency Keys at the API Level

Most payment and transactional APIs support idempotency keys natively — Stripe, Square, etc. Pass a key derived from your input, and the API guarantees at-most-once execution.

```python
charge = stripe.Charge.create(
    amount=2000,
    idempotency_key=f"invoice-{invoice_id}",
    ...
)
```

If your agent talks to external APIs, use their idempotency support. Don't reinvent it.

### 3. Checkpoint State Transitions

For multi-step agent workflows, persist the state after each step. If the agent crashes and restarts, it resumes from the last checkpoint — not from the beginning.

```
State: EMAIL_SENT → CHARGED → TICKET_CREATED
                ↑ crash here
Resume from CHARGED, not from scratch
```

This is what "reliable agent loops" look like in practice. Not retry logic. Not exponential backoff. State machines that pick up where they left off.

## What Doesn't Work

**"Just catch the error and don't retry."** Network errors are ambiguous. Did the request fail before or after the server processed it? You don't know. Not retrying means potentially losing work. Retrying without idempotency means potentially duplicating it.

**"Make the user confirm everything."** This defeats the purpose of automation. Confirmation fatigue is real and people click through anyway.

**"Just use a queue with exactly-once delivery."** Exactly-once delivery is a myth in distributed systems. At-least-once is real. Design for it.

## The Litmus Test

Before putting an agent action in production, ask:

> If this action runs three times with the same input, does the result look like it ran once?

If the answer is no, you have work to do. If the answer is "I don't know," you also have work to do.

## The Uncomfortable Truth

Most agent demos skip idempotency entirely because it's not interesting. It doesn't make for good conference talks. It's plumbing. But plumbing is what separates a demo from a system you can trust in production.

Your agent will retry. Your webhooks will fire twice. Your network will fail at the worst moment. The question isn't whether these things happen — it's whether your system handles them gracefully.

Make it idempotent. Run it twice. Check the result.