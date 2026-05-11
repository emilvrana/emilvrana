---
title: "Prompt Engineering Is Just Programming"
date: 2026-05-11
tags: ["agents", "llm", "engineering", "architecture"]
---

# Prompt Engineering Is Just Programming

We gave it a fancy name and built a cottage industry around it. But "prompt engineering" is just programming — with all the discipline that implies, and none of the rigidity people fear.

## The Naming Problem

When you call something "prompt engineering," you imply it's a new discipline. A special skill. Something separate from software engineering that requires its own certifications, frameworks, and conference tracks.

Here's what it actually is: writing specifications for a very flexible, very unreliable, very powerful interpreter.

If you've ever written a config file, designed an API contract, or debugged a distributed system, you've already done the core work. The medium changed. The discipline didn't.

## What Makes It Hard (And What Doesn't)

The hard part isn't writing English instructions to a model. The hard part is:

**Managing state.** The context window is memory. When it overflows, your program loses scope. Sound familiar? It's the same problem as memory management in any system — you need explicit strategies for what to keep, what to discard, and what to externalize.

**Handling failure.** LLMs fail silently and creatively. They don't throw exceptions. They produce plausible wrong answers. This is exactly the problem of dealing with untrusted inputs, except your compute layer is also untrusted.

**Controlling non-determinism.** Temperature, sampling, model version drift — these are sources of variance. In traditional systems, you design for variance with idempotency, retries, and circuit breakers. Same tools apply here.

The easy part, ironically, is "writing prompts." The text is the least important component. The architecture around it — routing, validation, fallbacks, evaluation — is where the engineering happens.

## Patterns That Transfer

If you're a developer moving into agent systems, you already know the patterns:

- **Input validation** → Structured output parsing with schemas, not regex on free text
- **Error boundaries** → Retry with backoff + model downgrade, not just "try again"
- **Caching** → Cache semantic intent, not exact strings
- **Monitoring** → Log inputs, outputs, latency, and tool calls — not just "it worked"
- **Testing** → Eval suites with labeled examples, not "I tried it and it seemed fine"

The companies struggling with AI in production aren't struggling because they lack prompt engineers. They're struggling because they skipped the engineering.

## The Anti-Pattern

The most common anti-pattern I see: a brilliant prompt, no architecture.

Someone crafts the perfect system message. The model nails the demo. Then they deploy it, and reality arrives — edge cases, inconsistent outputs, cost overruns, and no way to debug why.

The prompt was never the problem. The problem was building a system that depends on a single, fragile, untested component. We wouldn't accept that in any other part of our stack. We shouldn't accept it here.

## What I Actually Do

When someone asks me to "optimize the prompt," here's what I typically do:

1. **Map the failure modes** — what goes wrong, how often, why
2. **Add structure** — schemas, typed outputs, validation layers
3. **Reduce scope** — narrower tasks, clearer boundaries, less ambiguity
4. **Build evaluation** — automated checks, not manual review
5. **Then** tweak the text

Steps 1-4 are programming. Step 5 is the prompt engineering part, and it's usually the smallest change.

## The Takeaway

Call it what you want. The discipline that makes AI systems reliable is the same discipline that makes any system reliable: clear contracts, explicit failure handling, observable state, and automated verification.

The prompt is the interface. The engineering is everything around it.

---

*Written in Prague, where the coffee is strong and the context windows are short.*