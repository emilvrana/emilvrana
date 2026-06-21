# Circuit Breakers for Agent Pipelines

*June 21, 2026*

You know the pattern. Your agent works flawlessly in testing. Then you deploy it, and three hours later it's stuck in a loop — calling the same failing API 200 times, burning through tokens, producing garbage output that downstream systems accept as valid.

The fix isn't better prompts. The fix is circuit breakers.

## Borrowed from Microservices

In microservice architecture, a circuit breaker watches for failures and trips when they exceed a threshold. While tripped, requests fail fast instead of attempting a doomed call. After a cooldown, it lets a few through to test if the problem resolved.

This pattern maps directly to agent pipelines, but most agent frameworks don't include it. You're expected to handle it yourself, and most people don't.

## Where Agents Break

Three failure modes that circuit breakers prevent:

**1. The Retry Spiral.** An LLM call fails (rate limit, timeout, malformed response). Your retry logic kicks in. The model returns the same error. You retry again. Each retry burns tokens and time. Without a breaker, this runs until your timeout kills the entire pipeline — if you have a timeout.

**2. The Cascading Failure.** Tool A returns degraded output. Agent processes it and calls Tool B with garbage input. Tool B returns more garbage. Three steps later, the output looks plausible but is completely wrong. The pipeline didn't fail — it succeeded at being wrong.

**3. The Cost Explosion.** A degraded model serves slow, verbose responses. Your agent, trying to be thorough, makes more calls than usual. Each call costs more and returns less. The bill triples while quality drops.

## Implementing Circuit Breakers

A circuit breaker for agents needs three things:

**Failure threshold.** How many failures before you trip? For LLM calls, 3–5 consecutive failures is reasonable. For tool calls, it depends on the tool — an idempotent read can tolerate more failures than a write.

**Cooldown period.** How long before you try again? 30–60 seconds for rate limits. Longer for upstream outages. Don't make this configurable at runtime — make it a parameter you set when you design the pipeline.

**Fallback behavior.** What happens when the breaker is tripped? Return a cached result. Use a cheaper model. Skip the step and flag the output as partial. Never silently proceed with stale data.

```python
class CircuitBreaker:
    def __init__(self, threshold=5, cooldown=60):
        self.failures = 0
        self.threshold = threshold
        self.cooldown = cooldown
        self.tripped_at = None
        self.state = "closed"  # closed, open, half-open

    def call(self, fn, fallback=None):
        if self.state == "open":
            if time.time() - self.tripped_at > self.cooldown:
                self.state = "half-open"
            else:
                if fallback:
                    return fallback()
                raise CircuitOpen("Circuit breaker is open")

        try:
            result = fn()
            if self.state == "half-open":
                self.state = "closed"
                self.failures = 0
            return result
        except Exception as e:
            self.failures += 1
            if self.failures >= self.threshold:
                self.state = "open"
                self.tripped_at = time.time()
            if fallback:
                return fallback()
            raise
```

This isn't exotic. It's the same pattern that's kept production systems alive for decades. The only difference is applying it to agent-specific failure modes.

## The Hard Part: Detecting Failure

Circuit breakers in HTTP are easy — you get a 500, you count it. In agent pipelines, failure is harder to detect.

A model call that returns a 429 is obviously a failure. But a model call that returns a plausible-sounding hallucination? A tool call that succeeds but returns data from last week because the upstream cache is stale?

You need two layers:

**Transport breakers** catch obvious failures — HTTP errors, timeouts, rate limits. These are easy and every pipeline should have them.

**Semantic breakers** catch degraded output. Validation rules on tool responses. Schema checks on model output. Maximum output length checks. If the model suddenly returns 10x longer responses, something is wrong — trip the breaker.

## What Breakers Don't Do

Circuit breakers don't fix the underlying problem. They prevent it from compounding. When a breaker trips, you still need to investigate why — is the model degraded? Is the API having issues? Is your prompt producing edge cases?

But without the breaker, you don't get a clean failure signal. You get a mess of cascading errors and inflated costs that you notice hours later. With it, you get a controlled degradation and a clear alert.

## The Bottom Line

If you're building agent pipelines that run unattended, circuit breakers aren't optional. They're the difference between "the system degraded gracefully and sent an alert" and "the system generated $400 of API calls producing unusable output."

Add them before you need them. You'll need them.