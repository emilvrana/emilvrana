# The Replay Problem: Why You Can't Reproduce a Production Agent Failure

Your agent did something wrong in production. A customer complained. You have the logs. You want to reproduce the issue, fix it, and make sure it never happens again. So you take the prompt, the context, the tool inputs, and you run it again.

You get a different result.

Not a different error. Not the same answer with a slightly different tone. A completely different trajectory. The agent picks a different tool, takes a different path, and arrives at a different conclusion. You can't reproduce the bug. You can't verify the fix. You're left with a postmortem that says "we think this happened, probably."

This is the replay problem, and it's the reason most agent debugging is theatre.

## Why Agents Aren't Reproducible

Three things break reproducibility, and only one of them is fixable.

**Model non-determinism.** Same prompt, same context, same temperature — different output. Even at temperature 0, providers don't guarantee identical outputs across calls. The model isn't a function; it's a distribution. You get a sample, not the same sample every time. You can't fix this. It's the nature of the system.

**Context reconstruction.** To replay a run, you need the exact context the model saw. That includes the conversation history, the system prompt, the tool results, and every intermediate step. Most teams log the final output but not the full context window at each step. Without that, you're replaying a guess, not the run.

**Tool state drift.** Your agent called a database that returned 47 rows on Tuesday. On Wednesday, the database has 52 rows. The tool result is different, so the agent's context is different, so the agent's behavior is different. The world moved. Your replay is running against a different world.

The first problem is fundamental. The second and third are engineering problems with engineering solutions.

## What You Actually Need to Log

If you want to replay agent runs, you need to capture three things at every step:

**1. The exact input to the model.** Not a reconstruction. Not "we built the prompt like this." The literal bytes. Every token the model saw, in the order it saw them. This means logging the full message array — system prompt, conversation history, tool results, everything — at every LLM call, not just at the start of the run.

**2. The exact model output.** The raw completion, including tool calls, reasoning, and any metadata the provider returns. This is your replay target: if you feed the same input and get a different output, you know non-determinism is the issue, not your pipeline.

**3. The exact tool results.** Not a description of what the tool did. The actual return value, serialized, with timestamps. When you replay, you don't call the real tools — you replay the recorded tool results. This isolates the agent's decision-making from the world's state changes.

This is a lot of data. A 12-step agent run with 8k context per step is ~100KB of logs per run. That's not nothing, but storage is cheap and debugging is expensive. Log it.

## The Replay Workflow

When a production run fails, here's what you do:

**Step 1: Capture.** Pull the full execution trace from your logging system. This should be a single JSON document containing every model call, every tool call, and every intermediate result.

**Step 2: Isolate the failure.** Find the step where things went wrong. Not the step that produced the bad final output — the step where the agent made the wrong decision. This is usually two or three steps before the visible failure.

**Step 3: Replay with recorded tool results.** Re-run the agent from the failure step, feeding it the recorded tool results instead of calling real tools. If the agent makes the same decision, you've confirmed the bug. If it makes a different one, you've found a non-determinism issue — which is itself the bug.

**Step 4: Fix and verify.** Change your prompt, your tools, or your logic. Replay the same trace. If the agent now makes the right decision at the failure step, you've verified the fix. If it doesn't, you haven't fixed anything — you've added complexity without evidence.

## The Hard Part: Tool Calls

The trickiest part of replay is tool call handling. When you replay, you need to match tool calls to their recorded results. But the agent might make different tool calls on replay — different arguments, different order, different tools entirely.

The solution is to replay deterministically up to the failure step, then let the agent run free from there with real tools (or a snapshot of the tool state from the time of the failure). This gives you:

- **Confirmed reproduction** of everything up to the failure point.
- **Controlled experimentation** from the failure point forward.

You're not replaying the whole run. You're replaying the prefix and testing the branch.

## What Most Teams Do Instead

Most teams debug agents the way they debug traditional software: they look at logs, form a hypothesis, make a change, and deploy. They never replay. They never verify. They assume the fix works because the reasoning seems sound.

This is why the same agent bugs recur for months. The fix addressed a hypothesis, not the actual failure. Without replay, you're guessing. With replay, you're testing.

## The Infrastructure You Need

Building replay capability isn't a weekend project, but it's not a rewrite either. You need:

- **Structured execution traces.** Every run produces a JSON document with every model call, tool call, and result. No exceptions.
- **A trace storage system.** S3, local disk, a database — anything that lets you pull a trace by run ID.
- **A replay harness.** A script that takes a trace ID, replays the run with recorded tool results, and lets you inject changes at any step.
- **A comparison tool.** A diff between the original run and the replay that highlights where they diverge.

None of this is exotic. It's the same pattern that distributed systems engineers have used for decades — capture, replay, bisect. Agent systems just need it more, because they have an extra source of non-determinism that traditional systems don't: the model itself.

## The Payoff

Replay doesn't just help with debugging. It changes how you build:

- **Eval sets grow from real failures.** Every replayed bug becomes an eval case. Your test suite improves automatically.
- **Prompt changes become testable.** "Does this prompt change fix the bug?" becomes a question you answer with a replay, not a deployment.
- **Confidence replaces hope.** You stop deploying changes and hoping. You start deploying changes you've verified against recorded production traces.

The replay problem is the gap between "we think we fixed it" and "we know we fixed it." Most agent projects live in that gap permanently. Building replay infrastructure is how you leave it.

---

*Part of an ongoing series on building reliable AI systems. More at [emil.aiadoption.cz](https://emil.aiadoption.cz).*