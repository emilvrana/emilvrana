# Testing Agent Systems: Why Your Unit Tests Aren't Enough

You wrote tests for your agent's tools. You tested the LLM call with a mocked response. You're feeling good. Then your agent goes to production and fails in ways your tests never caught — wrong tool chosen, context overflowed, the model hallucinated a parameter, or the agent decided to call the same tool seven times in a loop.

Here's the problem: traditional testing assumes deterministic systems. Agents are probabilistic systems wrapped in deterministic code. Testing them like a CRUD app gives you false confidence.

## Three Levels of Agent Testing

### 1. Unit Tests — Necessary, Not Sufficient

Test your tools, your parsers, your validation logic. This is table stakes. If your tool can't handle edge cases, your agent certainly can't.

```python
def test_search_tool_handles_empty_results():
    result = search_tool("obscure query with no matches")
    assert result == []  # not None, not an error
    assert search_tool.call_count == 1  # no retries
```

This catches broken tools. It doesn't catch an agent that calls the wrong tool.

### 2. Integration Tests — The Agent in the Loop

Give the agent a task, let it use real tools (or realistic stubs), and check the outcome. Not the path — the outcome.

```python
def test_agent_finds_document():
    agent = create_agent(tools=[search, retrieve, summarize])
    result = agent.run("Find the Q1 revenue report and summarize it")
    
    assert "€4.2M" in result  # the answer is correct
    assert agent.tool_calls <= 5  # it didn't spiral
    assert agent.cost < 0.10  # it didn't burn tokens
```

Key insight: test the **contract**, not the **implementation**. The agent can take any path it wants. You care that it arrives.

### 3. Adversarial Tests — Where the Interesting Failures Live

These test what happens when things go wrong:

- **Tool returns garbage.** Does the agent retry sensibly or loop?
- **Context gets large.** Does the agent start losing the plot at 8k tokens?
- **Multiple correct tools exist.** Does the agent pick the efficient one?
- **The task is ambiguous.** Does the agent ask for clarification or guess?

```python
def test_agent_handles_tool_failure_gracefully():
    # Make search return an error
    agent = create_agent(tools=[failing_search, retrieve, summarize])
    result = agent.run("Find the Q1 revenue report")
    
    assert result is not None  # didn't crash
    assert "couldn't find" in result.lower() or "unavailable" in result.lower()
    assert agent.tool_calls <= 3  # didn't hammer the failing tool
```

## The Metric That Matters: Failure Mode Coverage

You can't test every possible agent path — there are too many. What you can do is enumerate failure modes and test each one:

| Failure Mode | Test |
|---|---|
| Wrong tool selected | Task that requires tool A, agent has tools A and B |
| Tool failure | Stub that returns errors |
| Context overflow | Task requiring information from 5+ documents |
| Infinite loop | Task that could trigger repetitive behavior |
| Hallucinated parameters | Task with ambiguous inputs |
| Silent wrong answer | Task with verifiable ground truth |

Write one test per failure mode. This gives you more coverage than fifty happy-path tests.

## What About Flaky Tests?

Agent tests *will* be somewhat flaky — the model is nondeterministic. Mitigation strategies:

- **Run tests multiple times.** If 8 out of 10 runs pass, the failure mode is rare, not gone.
- **Use temperature 0.** Reduces variance without eliminating it.
- **Test failure modes, not exact outputs.** "Agent doesn't loop infinitely" is testable. "Agent produces exactly this text" is not.
- **Track trends, not individual runs.** A test that goes from 95% pass to 70% pass is telling you something.

## The Uncomfortable Truth

Agent testing is harder than agent building. Most teams spend 90% of their time on the agent and 10% on testing. The ratio should be closer to 50/50.

If your test suite only has unit tests for tools and a couple of happy-path integration tests, you're testing that your agent works when everything goes right. The interesting question is what happens when things go wrong.

---

*Testing agent systems is a subset of agent evaluation — but evaluation is broader (measuring quality in production), while testing is about catching failures before they ship. For the evaluation perspective, see [Agent Evaluation Is Not Optional](agent-evaluation-is-not-optional.md).*