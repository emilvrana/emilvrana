# The Retry Trap: Why Retrying Failed AI Calls Makes Things Worse

Your agent calls an LLM. It fails. What do you do?

If you're like most people, you retry. Maybe with exponential backoff. Maybe up to three times. It's the obvious thing — it works for databases, for APIs, for everything else in distributed systems.

But LLM calls aren't databases. And in agent systems, naive retries don't just fail to help — they make things actively worse.

## Why Standard Retries Backfire

A database query fails because the database is down or the network blipped. The query is deterministic: same input, same output. Retrying makes sense because the failure is transient.

An LLM call fails for fundamentally different reasons:

**1. The prompt is the problem, not the service.** Most LLM "failures" in agent systems aren't timeouts or rate limits — they're bad outputs. The model returned something you can't parse. It called the wrong tool. It went off on a tangent. Retrying the same prompt gives you the same class of failure, maybe with cosmetic differences.

**2. Context compounds on retry.** You retry inside a conversation. The failed attempt is already in the context window. The model sees its own failure and tries to "fix" it — which often means doubling down or going in circles. Each retry adds tokens without adding value.

**3. Cost compounds silently.** Three retries on a 4K-token prompt = 12K tokens of inference. On a 32K-token long-running agent session, three retries eat through your context budget fast. And you're paying for all of them — failures included.

## The Three Failure Modes

Not all failures are equal. Here's how I classify them:

**Transient failures** — rate limits, timeouts, service errors. These are the only ones where retry makes sense, and even then, only with backoff and a circuit breaker.

**Prompt failures** — the model misunderstood, hallucinated, or produced unparseable output. Retrying the same prompt rarely helps. You need to reformulate.

**State failures** — the agent got stuck in a loop, lost track of its goal, or drifted into irrelevant territory. Retrying the last step is like pressing the gas when you're in a ditch.

Most agent systems conflate all three. They shouldn't.

## What to Do Instead

### For transient failures: Retry, but cap it

Standard exponential backoff with a maximum of 2–3 retries. Add jitter. Track the error rate — if you're hitting rate limits consistently, you need to slow down globally, not just locally.

```
retry with: backoff = min(2^attempt + random_ms, 30s)
max retries: 3
circuit breaker: if >50% errors in 5min window, stop entirely
```

This is boring, well-understood infrastructure. Do this part right, then stop.

### For prompt failures: Reformulate, don't repeat

When the model gives you garbage, change the input. This sounds obvious, but most agent frameworks don't do it — they just re-throw the same prompt at the model and hope for a different result.

Concrete patterns:

- **Add a constraint.** "Respond with valid JSON only. No markdown, no explanation." The model loves to be helpful — sometimes you need to tell it what *not* to do.
- **Show an example.** Few-shot works. One example of the format you want beats three retries with no example.
- **Reduce the task.** If the model can't produce the full output, ask for less. Break the task down. A model that fails at "extract all entities and relationships from this 10K-token document" might succeed at "list the person names in paragraph 3."
- **Switch models.** If you're hitting a wall with a cheap model, try a better one for that step. This is not failure — it's routing.

### For state failures: Checkpoint and recover

Long-running agents drift. It's not a bug — it's the nature of autoregressive generation over long contexts. The fix isn't retry; it's checkpointing.

Every N steps, capture:
- The agent's current goal
- What it's accomplished so far
- What it's about to do next

When things go wrong, you don't retry from the failure point — you roll back to the last checkpoint and try a different path. This is the same pattern we use in distributed systems: write-ahead logs, not retries.

## The Pattern I Actually Use

In my agent systems, the retry logic looks like this:

```
attempt 1: original prompt
attempt 2: reformulated prompt (add constraints + example)
attempt 3: simpler task / different model
after 3: checkpoint recovery or graceful degradation
```

The second and third attempts are *different prompts*, not the same prompt repeated. Each step reduces scope or changes the approach. The model isn't doing the same thing and hoping for a different result — it's doing a progressively simpler version of the task.

And if all three fail? The agent logs what happened, reports the failure to the user (or calling system), and stops. No infinite loops. No context window full of retries. No runaway costs.

## The Real Lesson

Retries in agent systems are a symptom of a deeper problem: treating an LLM like a deterministic service. It's not. It's a model that interprets prompts, and when the interpretation is wrong, sending the same prompt again is the least effective thing you can do.

The fix isn't better retry logic. It's better failure classification, better prompt reformulation, and checkpointing over retrying.

Stop retrying. Start recovering.

---

*Emil Vrána builds AI systems and automation in Prague. More at [emil.aiadoption.cz](https://emil.aiadoption.cz).*