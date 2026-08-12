# The Composition Gap

**August 12, 2026**

Your agent handles each task well in isolation. Task A passes tests. Task B passes tests. Ask it to do A then B in sequence, and things fall apart.

This is the composition gap — and it's where most agent projects actually fail.

## Why Isolation Lies

Unit tests for agents are seductive. You verify the agent can search, can summarize, can format. Green across the board. Then you deploy, and the first real workflow — search, then summarize, then format — produces garbage.

The reason: each task leaves residue. Context window pollution, state mutations, tool call artifacts. The agent that starts task B is not the same agent that passed the test for task B.

## The Residue Problem

Every action an agent takes changes its context:

- **Token residue.** The output of task A fills the window, leaving less room for task B's reasoning.
- **State residue.** Variables, working memory, half-formed plans from the previous task bleed into the next.
- **Tool residue.** The agent remembers which tools it just used and reaches for them again, even when inappropriate.

None of this shows up in isolated tests.

## How to Close the Gap

**1. Test sequences, not just steps.**

Don't just test "can the agent search?" and "can the agent summarize?" Test "can it search *and then* summarize?" If your test suite only has single-task scenarios, you have a composition gap in your testing, too.

**2. Explicit state resets between phases.**

Between logical steps in a workflow, wipe the slate. Clear working memory. Summarize the prior step's output into a clean handoff document. The overhead is minimal; the reliability gain is massive.

**3. Design for decreasing context.**

Structure your workflows so that earlier tasks produce *smaller* outputs, not larger ones. A 200-word summary is a better input to the next step than a 2000-word raw dump. Compression isn't just efficiency — it's composability.

**4. Validate the seams.**

The boundary between task A and task B is where things break. Add assertions there: does the output of A match the expected input shape of B? If not, catch it before B runs.

## The Uncomfortable Truth

Most agent failures aren't model failures. They're composition failures. The model is fine at each step. It's the transitions that kill you.

If your agent works in demos but fails in production, look at the seams. That's where the gap lives.

---

*Emil Vrána is an independent tech consultant based in Prague, working on reliable AI agent systems.*