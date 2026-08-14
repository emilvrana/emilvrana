# The Blameless Agent Postmortem

**August 14, 2026**

Your agent sent a wrong email to a customer. Or it deleted a production resource. Or it spent $200 on API calls looping through a hallucinated tool chain. The incident is over. Now what?

Most teams do one of two things. They either shrug it off ("AI is unpredictable, what can you do") or they overhaul the entire prompt ("we'll add more rules so this never happens again"). Both responses are wrong. The first treats agent failures as acts of God. The second treats them as prompt engineering problems.

SRE teams figured this out decades ago. The practice is called the blameless postmortem. It works for agent systems too, but the shape of it is different.

## Why Blameless

The principle is simple: blame the system, not the person. Or in our case: blame the system, not the model. When you write "the agent was confused," you've already failed. The agent wasn't confused — your system allowed a confused agent to take an irreversible action.

Blameless doesn't mean consequence-free. It means the postmortem focuses on what broke in the system, not who to punish. For agents, "who" is often the model. "The model hallucinated" is the AI equivalent of "the intern made a mistake." Both might be true. Neither is actionable.

## The Five Questions

Every agent postmortem should answer these five questions, in order:

**What happened?** A factual timeline. Not "the agent went rogue" — "at 14:32, the agent called `send_email` with recipient `user@wrong.com` and subject 'Your order is cancelled.' The email was delivered. The customer replied at 15:10 asking why their order was cancelled."

**What was the impact?** Not just "a wrong email was sent." Who was affected? What did it cost — in money, trust, time? Did it cascade? The impact determines the severity, and the severity determines how deep the analysis goes.

**Why did the system allow it?** This is the key question. Not "why did the model do that" — models do things. The question is why your system let a wrong action through to completion. Where were the guardrails? Where was the human checkpoint? Where was the dry run?

**What would have caught it?** A specific control that, if present, would have prevented or detected the failure. Not "better prompts." A concrete mechanism: a confirmation step before outbound emails, a validation layer on tool arguments, a circuit breaker on repeated API calls.

**What did we change?** The postmortem ends with actions. Not "we'll be more careful." Specific changes, with owners and deadlines.

## What Agent Postmortems Look Different From

Traditional software postmortems deal with bugs — code that does the wrong thing. Agent postmortems deal with emergent behavior — code that does an unpredictable thing in a predictable way. The model followed its instructions. The instructions were incomplete.

This means the root cause is almost never "the model is bad." It's one of:

- **Missing guardrail.** The agent had the authority to take an action it shouldn't have been able to take alone. You gave `send_email` to an agent with no confirmation step. That's not a model problem — that's a system design problem.

- **Ambiguous context.** The agent interpreted context in a way that was reasonable but wrong. It saw "cancel the order" in the user's message and cancelled the wrong order because two orders matched the query. The fix isn't "be smarter" — it's disambiguation logic.

- **Cascading error.** The agent made a small mistake early, then built subsequent actions on top of that mistake. Each step was individually plausible. The chain was the problem. The fix is a checkpoint between steps.

- **Tool design.** The tool API was ambiguous — parameters weren't typed, error responses looked like success responses, or the agent couldn't tell that it had failed. This is the most common root cause and the most ignored one.

## The Template

Write postmortems in plain text. No fancy tools. Five sections:

```
## Summary
One paragraph. What happened, impact, severity.

## Timeline
Bullet points with timestamps. Only facts.

## Root Cause
What in the system allowed this to happen. Not "the model."

## Action Items
- [ ] Specific change | owner | deadline
- [ ] Specific change | owner | deadline

## Lessons
What we learned that changes how we build agents.
```

Keep it to one page. If it's longer, you're writing a novel, not a postmortem.

## The Anti-Patterns

**"We updated the prompt."** This is the most common postmortem action and the weakest. It's true that sometimes the prompt needs fixing, but if your only action is "added a rule to the system prompt," you've patched a symptom. The agent will find another way to fail because the system structure hasn't changed.

**"We'll add more tests."** Tests for agents are harder than tests for functions. If you're not specific about what the test checks and how it runs, this is a wish, not an action.

**"The model improved in the latest version."** Upgrading your model and declaring the problem solved is not a postmortem. The old failure mode may be gone, but a new one will appear. If you didn't add a guardrail, you're relying on luck.

## The Cultural Part

Postmortems only work if the team trusts the process. If writing an honest postmortem gets someone blamed, you'll get postmortems that hide the real cause. This is as true for agent systems as it is for traditional software.

The person who deployed the agent isn't the problem. The person who wrote the prompt isn't the problem. The system that let an uncertain agent take a irreversible action is the problem. Fix that.

## The Point

Your agent will cause incidents. The postmortem is how you turn each one into a system that's harder to break next time. Not by adding more rules to the prompt — by changing the architecture so the failure mode is impossible.

A blameless postmortem doesn't mean nobody is responsible. It means the responsibility is to the system, not to the blame.