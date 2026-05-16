# The Configuration Drift Problem

You deploy an agent. It works. You move on. Two weeks later, someone asks why it's doing something weird.

You check the logs. The agent is running. It's producing outputs. Nothing crashed. But the outputs are... wrong. Not broken — just off. The tone shifted. It started calling tools it shouldn't. It's making assumptions it wasn't making before.

Welcome to configuration drift. The silent killer of agent systems.

## What Drifts

In traditional software, configuration drift means servers diverging from their intended state. In agent systems, it's worse — because the "configuration" isn't just a file. It's a whole stack of things that can silently change:

**Model behavior.** The same model name doesn't guarantee the same behavior. Providers update models without warning. A prompt that worked perfectly on gpt-4-0613 might produce subtly different results on gpt-4-1106. I've seen agents go from concise to verbose overnight because of a silent model update.

**Tool definitions.** Your agent calls an API. The API changes. Maybe a field is now required. Maybe the response format shifted. The agent doesn't crash — it just starts producing worse results because it's working from an outdated understanding of what the tool returns.

**Prompt context.** You wrote the system prompt three months ago. Since then, three people have edited it. Each change seemed small. But the cumulative effect is a prompt that says three slightly contradictory things. The model resolves the contradiction differently each time, producing inconsistent behavior.

**Environment assumptions.** The agent was built assuming a certain data layout, a certain set of available services, a certain scale. Those assumptions erode. The database got migrated. The queue grew 10x. The monitoring dashboard was removed during a cleanup sprint.

## Why It's Hard to Detect

Configuration drift is insidious because the system keeps running. No alerts fire. No errors appear in logs. The agent is technically healthy — it's just no longer doing what you intended.

Most monitoring is designed for crashes, not for "doing the wrong thing successfully." Your uptime is 99.9%. Your agent is available 24/7. And it's been slowly going off the rails for two weeks.

The detection problem is also a measurement problem. If you don't have a clear spec for what correct behavior looks like — and most agent projects don't — you can't measure drift because you don't have a baseline to drift from.

## What Works

I've found three practices that meaningfully reduce drift problems:

**1. Pin everything you can.** Pin the model version, not just the model family. Pin tool API versions. Pin library versions. The more specific your configuration, the less room there is for silent shifts. Yes, this means you'll need to explicitly upgrade. That's the point — upgrades should be intentional, not accidental.

**2. Run regression tests, not just integration tests.** Integration tests check that things connect. Regression tests check that behavior is what you expect. For agents, this means having a set of input/output pairs that represent correct behavior, and running them periodically. Not just at deploy time — continuously. A weekly cron that runs your agent against known scenarios and checks the outputs against expectations will catch drift that nothing else will.

**3. Version your prompts.** Every prompt change should be a commit with a reason. Not in a wiki. Not in a Slack thread. In git, where you can see the history, revert when needed, and diff between versions. If you can't answer "what changed between v3 and v4 of this prompt?" in 30 seconds, you're not versioning properly.

## The Uncomfortable Truth

Configuration drift isn't a bug you fix. It's a maintenance cost you accept. Agent systems are living systems — they interact with services, models, and data that change independently. Drift is the natural result.

The question isn't whether drift will happen. It's whether you'll notice it before your users do.

---

*Part of an ongoing series on building reliable AI agent systems. Previous posts on [agent evaluation](/blog/the-evaluation-gap-in-ai-systems/), [the retry trap](/blog/retry-trap/), and [silent failures](/blog/silent-failures/) cover related ground.*