# Why Your Agent Loops (And How to Make It Converge)

You've seen it. Your agent calls a tool, reads the result, decides it needs more information, calls another tool, reads that result, and somehow ends up back where it started. Three iterations in, the context window is bloated, the bill is climbing, and the agent is no closer to done.

Agent loops aren't a mystery. They're a design problem — and the fix is structural, not prompt-level.

## Why Agents Loop

**The context trap.** Each iteration adds context without removing dead ends. The agent sees its previous attempts, interprets them as progress, and builds on them — even when they're noise. More context means more surface area for the model to justify "one more try."

**The missing termination condition.** Most agent prompts say "complete the task." They don't say what completing looks like, what constitutes failure, or how many attempts are reasonable. The model optimizes for completion, not for convergence.

**Tool feedback loops.** An agent queries a database, gets an empty result, tries a different query, gets another empty result, and loops. The tool output doesn't encode "you've exhausted this path" — it just returns `[]`.

## Three Structural Fixes

### 1. Budgeted Execution

Give every agent run a hard budget before it starts:

- **Maximum steps** — 10, 15, 20. Pick a number and enforce it.
- **Maximum tokens** — cap total output, not just context.
- **Maximum cost** — if the run exceeds $X, stop.

The budget isn't a hack. It's a specification of how much effort this task is worth. If an agent can't finish in 12 steps, the problem statement is wrong, not the agent.

### 2. Convergence Checks

After each iteration, check: **did the state actually change?**

```python
def converged(previous_state, current_state):
    if current_state == previous_state:
        return True  # We're going in circles
    if current_state.answer != None:
        return True  # We found it
    return False
```

State comparison doesn't have to be fancy. Hash the agent's key observations after each step. If two consecutive hashes match, you're looping. Break out, report the partial result, and let the orchestrator decide what happens next.

### 3. Forcing Functions

When an agent is stuck, it needs a push in a new direction — not more of the same.

- **Step 5:** "Summarize what you've learned so far and list three different approaches."
- **Step 10:** "Pick the best partial answer you have and return it with confidence level."
- **Step 15 (budget):** "Return whatever you have. Mark it incomplete."

These aren't retries. They're deliberate perspective shifts that break the loop pattern.

## The Uncomfortable Truth

Most agent loops happen because the task was underspecified. If you can't write down what "done" looks like — in a form a deterministic program could check — the agent can't either. 

Before you reach for better prompts or smarter models, write the completion criteria. If you can't, the agent was never going to converge anyway.

---

*Part of an ongoing series on building reliable AI systems. More at [emil.aiadoption.cz](https://emil.aiadoption.cz).*