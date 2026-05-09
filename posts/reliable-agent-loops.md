---
title: Three Patterns for Reliable Agent Loops
date: 2026-05-09
summary: Most agent systems spiral. Here are three patterns that keep them on track.
---

# Three Patterns for Reliable Agent Loops

Agent loops sound simple: give the model a goal, let it call tools, repeat until done. In practice, they spiral — calling the same API with the same bad arguments, hallucinating tool names, or wandering through irrelevant tangents until the context window fills up.

After running agent systems in production for the past year, three patterns keep coming back. None are novel. All are underrated.

## 1. Gate the Loop, Don't Just Cap It

Everyone sets `max_iterations: 10` or `max_tokens: 50000`. That's a safety net, not a strategy. The real problem is that agents don't know when to stop — they treat iteration as progress.

Instead, gate each iteration with a **relevance check**. Before the model acts, ask: does the proposed action directly advance the original goal? This can be as simple as including the original task in every prompt and requiring the model to explain *why* the next step matters.

```python
# Before each tool call, inject:
original_task = "Fix the failing CI pipeline on branch main"
rationale = "This SSH into the CI runner directly addresses the build failure"

# If rationale is weak → skip, re-prompt with tighter constraints
```

The goal isn't to add overhead — it's to make the model justify its trajectory. Agents that can't articulate why they're doing something are agents that are about to spiral.

## 2. Make Failures Structured, Not Narrative

When a tool call fails, the typical response is to dump the error into context and let the model "figure it out." This works for simple cases. For recurring failures, it creates a narrative of retries that consumes context without converging.

Structure your error feedback:

```json
{
  "tool": "ssh",
  "error_type": "connection_refused",
  "attempts": 3,
  "last_attempt": "2026-05-09T14:00:00Z",
  "suggestion": "Check if the host is reachable on port 22"
}
```

This does two things. First, it gives the model a clear signal about *what kind* of failure occurred, not just the raw output. Second, it lets you implement **escalation rules** — if `attempts >= 3` and `error_type == "connection_refused"`, skip to a different strategy instead of retrying.

Narrative errors are for humans. Structured errors are for agents.

## 3. Checkpoint State, Not Context

Agent systems that rely on accumulating context are fragile. A single hallucinated fact early in the loop poisons every subsequent decision. The fix isn't better prompts — it's external state.

After each meaningful step, write the *result* to a structured store, not the *process*:

```yaml
# checkpoint after step 3:
current_state:
  ci_status: failing
  root_cause: "missing dependency: libssl-dev"
  attempted_fixes: ["reinstall libssl-dev"]
  next_action: "commit fix and push"
```

When context gets long, summarize from checkpoints — not from the raw conversation. This is the difference between "the agent remembers what it tried" and "the agent remembers what happened."

Context windows are getting bigger. That's a trap. More context means more room for the model to latch onto irrelevant details from 20 steps ago. Checkpoints give you clean, compressible state.

---

These patterns share a principle: **don't trust the model to manage itself.** That's not a limitation — it's good engineering. We don't let databases manage their own consistency either. We build constraints that make correct behavior the path of least resistance.

The best agent systems aren't the ones with the smartest models. They're the ones where the architecture makes it hard to fail.