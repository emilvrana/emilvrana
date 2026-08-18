# The Deadline Pattern: Why Every Agent Operation Needs a Timeout

Your agent makes an LLM call. The provider is slow today. The call takes 45 seconds. Your agent waits. The user waits. The upstream system that triggered the agent waits. By the time the response arrives, the user has closed the tab, the webhook has timed out, and the queue has backed up.

You didn't have a bug. You had a missing deadline.

Every operation in a production system has a deadline — except, apparently, in agent systems. LLM calls, tool executions, sub-agent delegations, multi-step pipelines: teams let them run without time bounds because "the model will respond eventually." Eventually is not a production SLA.

## The Problem With Indefinite Waits

LLM latency is not predictable. The same prompt can return in 800ms or 40s depending on provider load, context length, model selection, and internal queuing. Most teams treat this as a fact of life — you can't control the provider's latency, so why set a deadline?

Because the deadline isn't for the provider. It's for your system.

When an LLM call runs without a deadline, every downstream component waits without a deadline. The tool call that follows the LLM response can't start. The user who initiated the request can't get a response. The queue worker processing the task is blocked. One slow LLM call cascades into a system-wide stall, and you won't know until something else breaks — a queue fills up, a health check fails, a user complains.

The deadline is the boundary that turns "slow" into "failed." Without it, slow and failed are indistinguishable, and your system can't tell the difference between "wait a bit longer" and "this will never complete."

## Where Deadlines Belong

Every layer of your agent stack needs deadlines, at different granularities:

**LLM calls.** Set a timeout on every API call to the model. Not the provider's default — your own, based on your latency budget. If your p95 response time is 3 seconds, a 30-second timeout is not conservative — it's negligent. A timeout of 8-10 seconds catches the pathological cases while allowing for normal variance. When the timeout fires, you retry once, then fall back.

**Tool executions.** Every tool your agent calls — database query, web search, API request, file read — needs its own deadline. A database query that normally takes 50ms can hang for 30 seconds under load. If the agent waits for the tool, the agent is stuck. Set tool-level timeouts that match the tool's expected latency, not a global default. A web search can take 5 seconds. A cache lookup should take 50ms. Don't use the same budget for both.

**Multi-step pipelines.** The pipeline itself needs an overall deadline. If step 1 is fast but step 4 hangs, the pipeline should have a top-level timeout that kills the entire run. Individual step timeouts catch local problems. The pipeline timeout catches cascading failures — when one slow step causes every subsequent step to miss its budget.

**Sub-agent delegations.** If your agent delegates to another agent, set a deadline on the delegation. The sub-agent's internal deadlines are its business. Your deadline is for the parent: "I will wait this long for your result, and then I proceed without it." This is uncomfortable because it means designing for the case where the sub-agent doesn't return. But that case exists whether you design for it or not.

## The Fallback Question

A deadline without a fallback is just a crash. When an operation times out, what happens?

**LLM call timeout → cached or degraded response.** If the model doesn't respond in time, return the last cached response for this query type, or a pre-written fallback response that acknowledges the limitation. "I couldn't generate a full answer in time — here's what I know." Users tolerate honesty. They don't tolerate infinite spinners.

**Tool timeout → skip or default.** If a tool doesn't return in time, the agent proceeds without that information. This means your agent needs to be designed for incomplete information — not every tool call is critical. Classify tools as critical or optional. A critical tool timing out means the agent should fail gracefully and report the error. An optional tool timing out means the agent continues with what it has.

**Pipeline timeout → partial result.** If the pipeline doesn't complete in its overall budget, return what you have. A partial answer from 3 completed steps is more useful than no answer from a pipeline that's still running step 4. This requires designing your pipeline so intermediate results are usable, not just final ones.

**Sub-agent timeout → proceed independently.** The parent agent continues without the sub-agent's contribution. This is the hardest to implement because it means the parent's logic must work without the sub-agent's output. But if the parent can't work without it, the sub-agent isn't a delegation — it's a dependency, and dependencies need to be reliable, not just deadline-bounded.

## The Cascade Problem

Deadlines cascade. If your LLM call has a 10-second timeout and your tool call has a 5-second timeout, the maximum pipeline time is 15 seconds — assuming sequential execution. But what if the LLM call takes 9.5 seconds and the tool call takes 4.5 seconds? You're within individual budgets but the combined time is 14 seconds, which might exceed the user's tolerance.

The solution is hierarchical deadlines. Each layer has its own budget, and the parent layer's budget constrains the child. The pipeline has 12 seconds total. The LLM call gets 8. The tool call gets 3. The remaining 1 second is for processing and response formatting. If the LLM call takes 7 seconds, the tool call still gets 3 — but if the LLM call takes 9 (exceeding its 8-second budget), the tool call's budget shrinks to 2, not 3. The parent deadline propagates downward.

This is standard in distributed systems — it's called a deadline propagation. Agent systems are distributed systems. The fact that one of the nodes is an LLM doesn't change the math.

## What Your Monitoring Should Show

Once you have deadlines, you need to track them:

**Timeout rate.** What percentage of operations hit the deadline? This should be low — under 1% for a healthy system. If it's higher, either your deadlines are too tight or your provider is degraded. Both are fixable, but only if you know which one it is.

**Deadline utilization.** How close do operations get to the deadline, on average? If your deadline is 10 seconds and your p99 latency is 9.5 seconds, you're riding the edge. A small provider degradation will push you over. Track the ratio of actual latency to deadline — if it's consistently above 80%, widen the deadline or improve the operation.

**Fallback frequency.** How often are you serving cached or degraded responses because of timeouts? This is the user-visible metric. If 5% of users get a fallback response, that's a 5% degraded experience. Track it, alert on it, and trend it over time.

## The Uncomfortable Truth

Deadlines force you to confront a question most agent projects avoid: how fast does this need to be?

Without a deadline, the answer is "as fast as the model goes." With a deadline, the answer is a number. And numbers can be measured, argued about, and improved. The deadline turns a vague latency concern into a specific engineering problem: hit this target, or have a fallback ready.

This is why teams skip deadlines. Not because they're hard to implement — a timeout parameter is one line of code. Because they force the conversation about what "good enough" means, and that conversation is uncomfortable when the system is new and you don't know the answer yet.

Set the deadline anyway. Pick a number. Make it generous. Revise it when you have data. But don't leave it unset, because unset deadlines don't mean "no limit" — they mean "the limit is whatever breaks first," and that's usually something expensive.

---

*Part of an ongoing series on building reliable AI systems. More at [emil.aiadoption.cz](https://emil.aiadoption.cz).*