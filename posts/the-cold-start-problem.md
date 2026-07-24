# The Cold Start Problem in AI Adoption

Every company I talk to wants AI. Most of them shouldn't start with AI.

Not because the technology isn't ready — it is, for the right problems. But because the prerequisites are missing, and nobody wants to hear that.

## What Cold Start Means Here

In recommendation systems, the cold start problem is well-understood: you can't personalize without data. In AI adoption, it's the same principle, but the missing ingredient isn't data. It's **operational clarity**.

An AI system automates decisions. To automate a decision, you need to know what the right decision looks like — at least well enough to evaluate whether the AI's output is correct. This sounds obvious. In practice, it's where most adoption efforts stall.

Companies come to me saying "we want to use AI for customer support." I ask: what does a good response look like? What's your current resolution rate? How do you measure whether a conversation went well?

Silence.

They don't know. Not because they're incompetent — because they've never had to formalize it. Humans handle the ambiguity implicitly. The senior support person *just knows* what a good answer looks like. That knowledge exists, but it's locked in heads, not in processes.

## The Three Missing Prerequisites

**1. A measurable outcome.** Before you automate anything, you need to define success in terms that a machine can evaluate. "Good customer support" is not measurable. "First-response resolution rate above 70% with satisfaction score above 4.2" is. If you can't write down the metric, you can't build the system.

**2. A failure taxonomy.** What goes wrong today? Not in theory — in practice. What are the top 5 things your current process gets wrong? If you can't list them, you don't have a specification for the AI to follow. And you won't be able to tell whether the AI is doing better or just failing differently.

**3. A feedback loop.** The human process you're replacing — does it improve over time? How? If there's no mechanism for learning from mistakes today, adding AI won't create one. AI amplifies feedback loops. If the loop doesn't exist, AI amplifies nothing.

## Why This Is Hard to Accept

The cold start problem is uncomfortable because it means the first step of AI adoption isn't AI. It's operational hygiene. Process documentation. Metric definition. Failure analysis.

These are not glamorous. They don't make for exciting board presentations. But they're the difference between a system that works and a system that generates plausible outputs while quietly drifting away from what you actually need.

I've seen teams spend six months building an AI pipeline, only to discover they can't evaluate whether it's working. Six months of engineering on top of a specification gap they didn't know they had. The model was fine. The problem was never the model.

## The Unsexy Path That Works

Start with measurement. Before writing a single prompt or evaluating a single model:

1. **Instrument the current process.** Track what happens now, manually. Build the dashboard. Count the things that matter.
2. **Enumerate failure modes.** Not hypothetical ones — the ones that actually happen. Write them down. Categorize them. You'll find patterns.
3. **Define the contract.** What should the system do, specifically? What should it never do? What's the latency budget? What's the cost ceiling?

Only then do you start building. And when you do, you'll build faster — because you know what "done" looks like.

## The Pattern I Keep Seeing

The organizations that succeed with AI adoption are the ones that already had good operational hygiene. The ones that could tell you their metrics, their failure rates, their process bottlenecks before AI entered the picture.

AI didn't create their clarity. It amplified it.

The ones that struggle are the ones hoping AI would *provide* clarity — as if the model would somehow figure out what they actually wanted, without them having to articulate it.

It won't. The cold start problem doesn't resolve itself. You resolve it by doing the unsexy work first.

---

*Emil Vrána is an independent tech consultant based in Prague, specializing in AI systems, automation, and data architecture.*