---
title: "Self-Hosting LLMs: The Real Math"
date: 2026-06-29
tags: ["infrastructure", "self-hosting", "economics", "production"]
---

# Self-Hosting LLMs: The Real Math

Everyone has an opinion about self-hosting LLMs. Most of those opinions are based on vibes, not numbers. I've been running local models in production for over a year. Here's the actual math, and where self-hosting genuinely makes sense vs. where it's just ideology.

## The API Baseline

Before you can evaluate self-hosting, you need to know what you're comparing against. Here's what APIs actually cost at mid-2026 volumes for a typical agent workload (60% input, 40% output, average 2k tokens per turn):

- **GPT-4o-class:** ~$5-8 per 1M tokens → roughly $0.01-0.02 per turn
- **Claude Sonnet-class:** ~$3-6 per 1M tokens → roughly $0.008-0.015 per turn
- **Smaller models (GPT-4o-mini, Haiku):** ~$0.15-0.60 per 1M tokens → roughly $0.0003-0.001 per turn

For a system doing 10,000 turns/day, that's roughly $3-6/day for a capable model, or $0.10-0.30/day for a smaller one. Not nothing, but also not the budget-breaker people imagine.

## The Self-Hosting Cost

Now the real numbers. I run local models on a Mac mini M4 Pro with 48GB RAM and on a VPS with an RTX 4090. Here's what it actually costs:

**Hardware.** The Mac mini was ~$2,000. The VPS with GPU is ~$150/month. If you're running 24/7, you need to compare against the API cost of the workload, not against zero.

**Electricity.** The Mac mini draws ~30-60W under load. At Czech electricity prices (roughly €0.15/kWh), that's €0.10-0.20/day. Negligible. The VPS cost is fixed regardless.

**Model quality.** This is the real variable. A self-hosted Qwen 2.5 32B or Llama 3.3 70B (quantized) is genuinely competitive with GPT-4o-mini for many tasks. It is not competitive with GPT-4o or Claude Opus for complex reasoning. This isn't ideology — it's benchmarkable, and I've benchmarked it on my actual workload.

**Operational cost.** Model updates, monitoring, failure recovery, cold start latency, context window limits. These are real costs that API users don't pay. I spend roughly 2-3 hours/week on local model maintenance.

## Where Self-Hosting Wins

**High-volume, simple tasks.** If you're doing 100k+ classification/extraction/summarization turns per day on a model that a Qwen 7B can handle, self-hosting saves real money. The crossover point is roughly $200-300/month in API costs — below that, the operational overhead isn't worth it.

**Privacy-sensitive workloads.** If the data can't leave your infrastructure, self-hosting isn't a choice — it's a requirement. But be honest about whether this is a real constraint or a preference. Most "privacy concerns" about LLM APIs are FUD. The API providers don't train on your data (check the actual terms), and your data is more likely to leak through your own infrastructure mistakes than through OpenAI's API.

**Latency-critical loops.** A local model on the same machine as your agent has sub-100ms first-token latency. An API call has 300-800ms network overhead. If your agent does 5+ LLM calls per turn (common in agentic loops), that's 1.5-4 seconds of pure network latency saved. This matters.

**Custom fine-tuning.** If you need a model tuned on your specific domain data, self-hosting the inference is a natural extension. You're already committed to the model lifecycle.

## Where Self-Hosting Loses

**Low-volume workloads.** If you're doing under 10k turns/day with a capable model, APIs are cheaper. The hardware alone takes months to amortize.

**Complex reasoning.** The best local models are good. They're not GPT-4o or Claude Opus good for tasks requiring deep multi-step reasoning, complex code generation, or nuanced understanding. If your workload needs top-tier reasoning, self-hosting means accepting lower quality.

**Team productivity.** Every hour spent maintaining local models is an hour not spent building your product. If your team is small, this opportunity cost dominates.

**Reliability.** APIs have 99.9% uptime. Your Mac mini has whatever uptime you personally ensure. I've had local model outages from OOM kills, disk space exhaustion, and macOS updates that broke inference. The API never has these problems.

## The Honest Framework

Here's how I decide:

| Factor | Self-Host | API |
|--------|-----------|-----|
| Volume >100k turns/day | ✓ | |
| Volume <10k turns/day | | ✓ |
| Simple tasks (classify, extract, summarize) | ✓ | |
| Complex reasoning | | ✓ |
| Privacy requirement (real, not assumed) | ✓ | |
| Latency-critical agentic loops | ✓ | |
| Small team, limited ops capacity | | ✓ |
| Need latest model capabilities | | ✓ |

And the hybrid approach that actually works: **use APIs for complex reasoning and self-hosted models for high-volume simple tasks in the same system.** Route based on task type, not ideology. A RAG pipeline that does retrieval with a local model and synthesis with an API model gets the best of both.

## The Takeaway

Self-hosting LLMs is a legitimate strategy for specific situations: high volume, simple tasks, privacy requirements, latency sensitivity. It's not a universal good, and the "stick it to the API companies" motivation leads to bad architecture decisions.

Run the math for your actual workload. Be honest about model quality differences. Account for operational overhead. And if you self-host, measure the real cost — hardware, electricity, time, opportunity cost, reliability — against the API alternative.

The best infrastructure decision is the one based on numbers, not narratives.

---

*Part of an ongoing series on building AI systems that work in production. Related: [self-hosted agent infrastructure](/blog/self-hosted-infrastructure/), [the context budget](/blog/the-context-budget/), and [compound latency](/blog/compound-latency/).*