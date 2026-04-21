# The Evaluation Gap in AI Systems

Everyone builds. Almost nobody evaluates properly.

I've seen this pattern repeat across consulting engagements and my own projects: a team ships an AI feature — a RAG pipeline, an agent workflow, a classification system — and declares it done. The demo works. The accuracy looks fine on a handful of examples. Then it hits production, and the edge cases arrive in bulk.

The problem isn't laziness. It's that evaluation for AI systems is genuinely harder than evaluation for traditional software. With a CRUD endpoint, you know what correct looks like. With an LLM output, you don't — or at least, not precisely enough to automate it.

## Three levels of evaluation

**Unit-level:** Does each component do its job in isolation? For a RAG system: does the retriever return relevant chunks? Does the generator stay on topic given those chunks? This is where most teams stop.

**Integration-level:** Do the components work well *together*? A retriever returning top-5 chunks at 90% recall and a generator that performs best with top-3 is a real mismatch I've encountered. You have to tune the pipeline, not the parts.

**System-level:** Does the whole thing actually solve the user's problem? This is the hardest and most neglected level. A summarization pipeline that produces technically accurate summaries but misses the one detail the user cares about is a system-level failure.

## What actually works

**Fixed test sets, even small ones.** 50 curated input-output pairs beat "we tested it a few times." Version the test set alongside the code. When you change the prompt, rerun. When you swap the model, rerun.

**Segregated metrics.** Don't mix retrieval quality with generation quality with end-to-end quality. If the overall score drops, you need to know which component regressed. One metric per component, one metric for the pipeline.

**Regression testing for prompts.** A prompt change is a code change. Treat it like one. Diff the outputs on your test set before and after. If you can't diff them automatically (likely for freeform generation), at least review them side by side.

**Human evaluation as calibration, not replacement.** Use human reviewers to spot-check automated metrics and find systematic blind spots. Don't use them as your primary QA — they tire, they disagree, they can't run on every commit.

## The uncomfortable truth

Most AI systems in production today are evaluated by vibes. The ones that work reliably are evaluated by a disciplined, boring process that looks a lot like traditional software testing — just adapted for probabilistic outputs.

The gap between "it works on the demo" and "it works in production" is exactly the evaluation gap. Close it deliberately, or your users will close it for you — by leaving.

---

*Part of my [series on practical AI engineering](https://emil.aiadoption.cz).*