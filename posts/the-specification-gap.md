# The Specification Gap

Most AI system failures aren't model failures. They're specification failures.

You built the pipeline. You tested the happy path. The demo worked. Then someone asked the system to do something you never explicitly told it *not* to do — and it complied, confidently, destructively.

Sound familiar?

## The Gap Nobody Talks About

We spend enormous energy on prompt engineering, model selection, and retrieval architecture. These matter. But they're downstream of a more fundamental problem: **what you tell a system to do is always less than what you need it to do.**

This is the specification gap. It's not unique to AI — it's the oldest problem in software engineering. Requirements are always incomplete. The difference is that traditional software fails *noisily* (crashes, exceptions, wrong output), while AI systems fail *silently* (plausible output that misses an unstated constraint).

A traditional system given an underspecified task returns an error. An AI system given an underspecified task returns an answer — confidently. That's the danger.

## Why It's Worse with AI

Three properties make the specification gap uniquely treacherous in AI systems:

**1. Competence without understanding.** A language model can follow instructions it doesn't comprehend. It'll process your constraint about "no PII in responses" while having no model of what PII actually *is*. The instruction is syntactically present but semantically absent.

**2. Adversarial inputs are the norm.** In traditional software, edge cases are rare. In AI systems, edge cases arrive with every prompt. Every user interaction is an implicit test of your specification's completeness — and users are creative.

**3. The illusion of specification.** A well-written prompt *feels* complete. It reads well. It covers obvious cases. But the distribution of possible inputs is effectively infinite, and your specification covers a vanishing fraction of it. You don't know what you didn't specify until it hurts.

## What Works

I've seen three patterns that meaningfully narrow the specification gap:

**Observability over anticipation.** You can't specify every constraint upfront. But you *can* observe what the system actually does in production and tighten specifications based on real failures. Log inputs, outputs, and decisions. Review them regularly. The specification is a living document, not a deployment artifact.

**Failure modes as first-class citizens.** Before you write a single prompt, enumerate the ways the system can fail. Not the happy-path variations — the actual failures. What does "wrong but plausible" look like? What data could the system leak? What actions are irreversible? Write specifications that address these *first*, because they're where the gap is widest.

**Invariant checks, not output checks.** Instead of validating that outputs look correct, validate that invariants hold. "Response contains no PII" is an invariant. "Response answers the question" is a quality check. Invariants are fewer, more testable, and more likely to catch specification gaps than any amount of output validation.

## The Hard Truth

The specification gap can be narrowed, not closed. This is not a problem that better models solve — a more capable model simply finds more creative ways to comply with incomplete specifications. Capability amplifies the gap.

The systems that work reliably in production are the ones that assume the specification is incomplete and are designed to degrade gracefully in the face of that incompleteness. Not the ones that assume the specification is sufficient and break when it isn't.

Stop trying to write the perfect prompt. Start building systems that survive imperfect specifications.

---

*Emil Vrána is an independent tech consultant based in Prague, specializing in AI systems, automation, and data architecture.*