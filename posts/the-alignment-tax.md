---
title: "The Alignment Tax: Why Good Enough AI Costs More Than Great AI"
date: 2026-06-20
tags: ["agents", "production", "cost", "reliability", "economics"]
---

# The Alignment Tax: Why Good Enough AI Costs More Than Great AI

Most teams budget for AI costs as if they're linear. One LLM call costs X. A pipeline with five calls costs 5X. Simple.

It isn't simple. The real cost of an AI system isn't the API calls that work. It's the calls that don't — and the human labor they generate downstream. I call this the alignment tax, and it's the reason that "good enough" AI often costs more than getting it right.

## The Math That Nobody Does

Here's a typical scenario. You build an extraction pipeline that processes 1,000 documents per day. The accuracy is 95%. Sounds great.

But 5% of 1,000 is 50 documents per day that need manual review. Each review takes a human 10 minutes. That's 8.3 hours of manual labor per day — more than a full-time employee just handling the errors.

Now improve accuracy to 98%. The error rate drops from 5% to 2%. That's 20 documents per day. Three hours of review. You just cut your manual labor by 64%.

The API cost difference between 95% and 98% accuracy? Maybe 20% more per call — better prompts, more context, maybe a larger model. The labor savings? 64%.

This is the alignment tax: the compounding cost of outputs that aren't aligned with what you actually need.

## Where the Tax Hides

The alignment tax shows up in places most teams don't track:

**Manual corrections.** Every time a human fixes an AI output, that's alignment tax. The API call was cheap. The correction is expensive — and it's rarely budgeted under "AI costs."

**Retry cascades.** The extraction fails on a tricky document. The agent retries with a different approach. Still fails. Tries again. Three LLM calls later, you've spent 3X the API cost and still need a human. The retry wasn't free — it cost both API tokens and time.

**Verification overhead.** If you can't trust the output, you verify it. Verification is alignment tax. A system that's 99% reliable needs spot checks. A system that's 90% reliable needs full review. The verification cost scales inversely with reliability.

**Downstream failures.** The extraction output feeds into a summary. The summary feeds into a report. The report goes to a client. When the extraction was wrong, everything downstream is wrong. The cost isn't just fixing the extraction — it's re-running the entire pipeline, re-generating the report, and managing the client relationship.

**Opportunity cost.** The team member reviewing 50 documents per day could be building something new. The engineering hours spent patching edge cases could be spent on features. The alignment tax doesn't just cost money — it costs progress.

## Why "Good Enough" Isn't

The common argument: "95% is good enough for a v1. We'll improve it later."

Later never comes. Here's why:

1. **The 5% errors establish the review process.** You build a review queue. You assign a reviewer. The reviewer builds a workflow around checking. The workflow becomes institutional. Now the system is optimized for review, not for accuracy.

2. **The 5% errors shape user expectations.** Users learn the output is unreliable. They start checking everything — even the 95% that's correct. The verification overhead balloons beyond just the errors.

3. **The 5% errors compound in pipelines.** Each step has its own error rate. Step A: 95%. Step B: 95%. Step C: 95%. The combined accuracy: 0.95 × 0.95 × 0.95 = 85.7%. Three "good enough" steps and you're at 14% failure rate.

4. **The improvement work gets deprioritized.** The system "works." There's a review process. The business has adapted. Investing in accuracy improvement now means disrupting the review workflow that people depend on. The incentive is to maintain the status quo, not improve it.

## The Inversion

Here's the counterintuitive part: investing in reliability early is cheaper than adding it later.

Building a 98% accurate system from scratch might take 2X the initial effort. But it avoids:
- Months of manual review labor
- The institutional weight of a review process that resists change
- The compounding error rates of multi-step pipelines
- The trust deficit that makes users verify everything

The teams I've seen succeed with AI in production didn't start with "good enough." They started with "reliable enough that humans don't need to check every output." That threshold is higher than most people think — and reaching it costs less than living below it.

## How to Reduce the Tax

**1. Measure the full cost.** API calls + human review time + retry costs + downstream failure costs. The total is always higher than you think. Until you measure it, you can't make informed tradeoffs.

**2. Set reliability targets, not just accuracy targets.** "95% accuracy" is meaningless without context. 95% on what distribution? On easy cases or hard ones? On your actual production data or a test set? Define what "reliable enough to skip review" looks like, then measure against that.

**3. Invest in evaluation early.** The evaluation gap isn't just about quality — it's about economics. Without evals, you can't tell if a prompt change improved things or made them worse. You're flying blind on costs you can't measure.

**4. Reduce pipeline steps.** Each step adds error rate. If you can merge two steps into one, the combined accuracy goes up even if each individual step stays the same. Simpler pipelines have lower alignment taxes.

**5. Fail loudly, not quietly.** A system that says "I'm not confident about this" is cheaper than one that produces a wrong answer with high confidence. Loud failures trigger review at the point of failure. Quiet failures propagate and multiply their cost downstream.

## The Bottom Line

The alignment tax is a progressive tax. The worse your accuracy, the higher the rate. And unlike most taxes, you can opt out — by building systems that are reliable enough to trust.

95% sounds impressive. In production, it's expensive. The path to cheaper AI isn't cheaper models. It's more reliable systems.

---

*Emil Vrána builds production AI systems in Prague. This is part of a series on the economics and engineering of reliable AI automation. Related: [compound latency](/blog/compound-latency/), [the retry trap](/blog/retry-trap/), and [agent evaluation](/blog/the-evaluation-gap-in-ai-systems/).*