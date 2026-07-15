# Stop Optimizing the Happy Path

Most AI agent engineering focuses on what happens when everything works. The prompt is clear, the model responds correctly, the tool returns what you expect, the output is valid. On that path, everything looks great.

But that's not where your agent lives most of the time.

## The happy path is the demo path

Every agent demo follows the happy path. You show the prompt, the model calls the right tool, the result comes back clean, and the agent produces a polished answer. Impressive. Convincing. And fundamentally misleading.

Because in production, the happy path might be 60% of your traffic. Or 40%. Or less. The rest — the model calls the wrong tool, the API returns a 503, the context window fills up, the user asks something ambiguous — that's where your agent actually earns its keep.

## What breaks, and where

The unhappy paths aren't random. They cluster into predictable categories:

**Model failures.** The model misunderstands the intent, picks the wrong tool, or produces output that doesn't match your schema. These are the failures everyone worries about and few handle well.

**System failures.** The tool is down, the database is slow, the API returns unexpected data. These are the failures everyone assumes someone else is handling.

**Edge cases.** The user asks something just outside the agent's scope, provides incomplete information, or contradicts themselves mid-conversation. These are the failures that reveal whether your agent has graceful boundaries or just collapses.

Each category needs a different response. But they share one thing: the happy path code won't help you.

## The 80/20 inversion

Here's the uncomfortable truth: the happy path is the easy 20% of your engineering effort. The unhappy paths — error handling, fallbacks, validation, retry logic, circuit breakers, observability — that's the 80% that determines whether your agent is reliable or just impressive.

Most teams get this backwards. They spend 80% of their time polishing the prompt, tuning the happy path, making the demo better. Then they bolt on error handling as an afterthought.

The result: an agent that looks great in testing and falls apart in production. Because production is where the unhappy paths live.

## What optimizing the unhappy path looks like

Start with failure modes, not features:

**Define what "broken" means.** Before you write a single prompt, write down the five most likely failure modes and what your agent should do in each. This is your failure spec, and it matters as much as your feature spec.

**Make failure visible.** If your agent can fail silently, it will. Log decisions, not just outputs. Track whether each tool call succeeded, whether the output matched expectations, whether the response was delivered within your time budget.

**Design fallbacks, not just retries.** Most LLM failures aren't transient — retrying a bad prompt with the same context produces the same bad result. Have a real fallback: degrade to a simpler response, escalate to a human, or return an honest "I can't do this" instead of a confident hallucination.

**Test the unhappy paths.** Write tests for your failure modes. Not "does the agent work when everything is fine" — "does the agent handle a timeout, a malformed response, an ambiguous request." If you only test the happy path, you only know your agent works in demos.

## The real metric

The measure of a production agent isn't how well it works when everything goes right. It's how gracefully it handles everything that goes wrong.

A reliable agent doesn't avoid failure — it's transparent about it, contains it, and recovers from it. The happy path is necessary but insufficient. The unhappy path is where reliability lives.

Stop optimizing the demo. Start optimizing for the 40% that actually tests your architecture.