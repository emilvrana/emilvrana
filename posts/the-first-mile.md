# The First Mile Problem

Everyone talks about the last mile in AI — getting from prototype to production. But the first mile is where most projects die, and nobody talks about that.

The first mile is the distance between "it worked once" and "it works consistently." It's the gap between your notebook demo and a system you'd trust to run while you sleep.

## What the First Mile Looks Like

You built something. A RAG pipeline. An agent. An automation. It worked. You showed your team. They nodded. Someone said "let's put this in production."

And then:

- The retrieval misses the right chunk one time out of five
- The agent loops on edge cases you didn't test
- The model changes its output format between runs
- The API you depend on rate-limits you at 2pm because that's when everyone else's cron jobs fire too
- The prompt that worked in English falls apart in Czech

None of these are exciting problems. None of them make for good conference talks. But each one is a potential project killer.

## Why We Skip It

The first mile is boring. It's logging. It's error handling. It's writing tests for paths you're sure nobody will take (and then someone does). It's documenting what "normal" looks like so you can detect abnormal.

We skip it because:

**The incentive gradient points the wrong way.** Demos get attention. Reliability gets ignored until it breaks. Nobody posts on LinkedIn about their retry logic. But the retry logic is what lets you sleep at night.

**We confuse "works" with "works reliably."** A system that returns the right answer 90% of the time isn't 90% as good as one that returns it 99.9% of the time. It's a fundamentally different category. The first system requires human oversight. The second doesn't. The cost difference isn't linear — it's the difference between "expensive tool" and "actual automation."

**We underestimate entropy.** The real world generates more variety than any test suite. Users will find paths you didn't consider. Data will drift. Models will update. APIs will change. The first mile is about building for the reality that everything will be slightly different tomorrow.

## What Actually Helps

**Guardrails, not guardrails-as-prompts.** Don't tell the model "don't do X." Write code that makes X impossible. If the model should never return more than 3 items, truncate the output. If it should never call a write endpoint on weekends, enforce that in the routing layer. Prompts are suggestions. Code is law.

**Structured output from day one.** If you're parsing model output with regex, you've already lost. Use structured output (JSON mode, function calling, whatever your provider calls it) from the very first prototype. Not because the first version needs it — but because the 100th version will, and retrofitting is expensive.

**Observe before you optimize.** Log everything. Input, output, latency, token count, model version, temperature. Not because you'll analyze it all, but because when something breaks at 3am on a Saturday, you'll want to know what "normal" looked like.

**Fail loudly and early.** A silent failure is worse than a loud one. If the agent can't complete the task, it should say so clearly, not return a plausible-looking wrong answer. Partial results with honest confidence scores beat confident hallucinations every time.

**Budget everything.** Time, tokens, cost, retries. Set limits before you start, not after the bill arrives. A budget isn't pessimism — it's a specification of how much you care about this particular task.

## The Uncomfortable Part

The first mile isn't a phase you complete and move past. It's ongoing. Systems decay. Models change. Data drifts. What worked last month might not work next month.

The teams that ship reliably aren't smarter. They're just more honest about what "working" means, and more disciplined about the boring work that keeps it working.

Build the first mile. Or accept that your demo was always just a demo.

---

*Part of an ongoing series on building reliable AI systems. More at [emil.aiadoption.cz](https://emil.aiadoption.cz).*