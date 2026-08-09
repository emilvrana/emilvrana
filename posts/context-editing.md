# Your Agent Doesn't Need a Bigger Context Window — It Needs Better Editing

Every time a model vendor announces a larger context window, agent developers celebrate. Finally, we can fit the entire codebase. The full conversation history. All the documentation. Everything the agent could possibly need, all at once.

This is exactly backwards. More context doesn't make agents smarter. It makes them slower, more expensive, and more prone to losing the signal in their own noise.

The real skill in building production agents isn't stuffing context in — it's cutting context out.

## The Context Tax Is Real

Every token in your agent's context window carries three costs:

**Latency.** Processing 100K tokens takes meaningfully longer than processing 10K. In a multi-step agent where each step adds context, this compounds. Your agent that "feels slow" isn't slow because of the model — it's slow because it's carrying 80K tokens of baggage from previous steps.

**Cost.** Input tokens are billed. Output tokens are billed. A 128K context window that produces a 200-token answer still paid for 128K tokens of input. Multiply that by a pipeline with 15 steps and you've spent the price of a small cluster just on context processing.

**Coherence.** This is the hidden one. LLMs don't read the way humans do. They attend to everything simultaneously, and the more irrelevant context you include, the more the model's attention is spread across noise. A 2K prompt with exactly the right context will outperform a 50K prompt that happens to include the answer somewhere in paragraph 47. Not because the model can't find it — because the surrounding noise degrades the quality of the retrieval.

I've seen this play out repeatedly. A team adds more context to "help" the agent, and accuracy goes down. They assume the model isn't smart enough. The model was fine. The problem was the noise.

## The Editing Mindset

The solution isn't better retrieval or bigger windows. It's treating context like a budget and editing ruthlessly.

An editor at a publishing house doesn't add paragraphs to make a manuscript clearer. They cut. They remove what doesn't serve the reader. They tighten. The same discipline applies to agent context.

**Before each agent step, ask: what does this step actually need?** Not "what might be useful." Not "what do we have room for." What is the minimum context required to produce a correct answer for this specific step?

If the answer is "the full conversation history," your step is probably too broad. Break it down. A step that needs 50K tokens of context is a step that's trying to do too much.

## Practical Editing Strategies

**1. Summarize, don't accumulate.**

After every N steps, replace the raw conversation history with a structured summary. Not "the user asked about X and then Y and then Z." That's just compression. The summary should capture: what decisions were made, what facts were established, what's still unresolved. The next step gets the summary, not the transcript.

**2. Retrieve only what the step needs.**

If your agent has access to a knowledge base, don't dump the top 20 results into context. Retrieve aggressively — 3 results, not 20 — and use the step's specific question as the retrieval query, not the full conversation. The question "what's the API endpoint for creating a user?" needs different context than the question "why did the last API call fail?" even though they're part of the same conversation.

**3. Cut previous tool outputs.**

The single biggest context hog in agent systems is previous tool call results. The agent called a database, got 500 lines of JSON, read two fields, and now carries those 500 lines forward for the rest of the conversation. Extract what was needed. Discard the rest. After step 3, the agent doesn't need the raw output from step 1 — it needs the conclusion it drew from it.

**4. Separate reasoning from reference.**

The model's reasoning about a problem and the reference material it uses to solve it are different things. The reasoning should be concise and structured. The reference material should be cited, not quoted verbatim. "The API returns a 429 on rate limit (see rate-limits.md §3)" is better than pasting the entire rate limits documentation into context.

**5. Use the window for depth, not breadth.**

A 128K context window used well means you can include a complete, detailed specification for the one task the agent is solving right now. It doesn't mean you should include summaries of twelve tasks the agent might solve later. Depth for the current task. Zero breadth for hypothetical ones.

## The Editing Discipline

Editing context is not a one-time task. It's a discipline you apply at every step of the pipeline. The agent that starts with a clean 2K context and maintains it through summarization and pruning will outperform the agent that starts with 50K and lets it grow.

This is also the discipline that makes longer context windows genuinely useful. A 128K window isn't for carrying 128K of accumulated noise. It's for giving the agent 128K of carefully curated, directly relevant context for a complex task. The window is the budget. Editing is how you stay within it.

## The Uncomfortable Truth

Most "context problems" in agent systems aren't context problems at all. They're scoping problems. The agent has too much context because it's trying to do too much in one run. The fix isn't a bigger window or better retrieval. It's smaller, more focused steps with less context each.

If your agent needs the full conversation history to make a decision, your agent's steps are too big. Break them down. Each step should need only what came immediately before, plus a compact summary of everything earlier.

The best context window is the one you barely use. Not because you can't fill it — because you've edited so well you don't need to.

---

*Emil Vrána is an independent tech consultant based in Prague, specializing in AI systems, automation, and data architecture. He writes about the gaps between AI demos and production reality at [emil.aiadoption.cz](https://emil.aiadoption.cz).*