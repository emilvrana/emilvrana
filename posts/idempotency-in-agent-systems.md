# Idempotency in Agent Systems: Why Your Agent Should Be Safe to Run Twice

You built an agent that sends emails. It runs. It sends the email. Then the orchestrator crashes and restarts. It runs the same task again. Your client gets two identical emails.

This isn't a hypothetical. It's the most common class of side-effect bug in production agent systems, and almost nobody talks about it because the demos never crash.

## What Idempotency Means for Agents

In traditional distributed systems, idempotency means: making the same request twice produces the same result as making it once. `PUT /user/42 {name: "Alice"}` is idempotent — setting it twice doesn't change anything. `POST /send-email` is not — sending twice sends twice.

Agent systems have the same problem, but worse. Agents don't just call APIs — they chain multiple actions, make decisions based on intermediate results, and operate in environments where crashes, timeouts, and re-runs are the norm, not the exception.

The question isn't "will my agent be asked to repeat a step?" The question is "what happens when it inevitably is?"

## Where It Breaks

**Database writes without deduplication.** Your agent inserts a row. The transaction succeeds, but the acknowledgment times out. The orchestrator retries. You now have duplicate rows. If your database has unique constraints, the retry fails with a constraint violation — and now the agent thinks the whole task failed.

**External API calls.** Sending a Slack message, creating a Jira ticket, posting a comment. These are almost never idempotent on the API side. The agent sent the message, but didn't record that it sent it. Re-run sends again.

**Stateful multi-step workflows.** Step 1 creates a resource. Step 2 uses it. Step 2 fails. The agent retries from Step 1 and creates a *second* resource. Now you have orphaned resources and your state is inconsistent.

**Payment and notification actions.** These are the worst class. An agent that charges a customer twice or notifies a team three times about the same incident isn't just buggy — it's a trust violation.

## Making Agent Actions Idempotent

The fix isn't complicated, but it requires deliberate design:

**1. Use idempotency keys for external calls.** Before sending that email or creating that ticket, generate a deterministic key from the task context (task ID + action type). Store it. Check it before acting. If the key exists, skip or resume — don't repeat.

**2. Make writes conditional.** `INSERT ... ON CONFLICT DO NOTHING` in PostgreSQL. `PUT` instead of `POST` when the API supports it. Create-if-not-exists patterns instead of blind creates.

**3. Record intent before action.** Write "I am about to send email X" to your state store *before* sending. If you crash and restart, you see the intent and can check whether the action completed — not guess.

**4. Design workflows with checkpoints.** Instead of one long chain of actions, break the workflow into idempotent stages. Each stage picks up where the last one left off, not from the beginning. This is what saga patterns do in distributed systems, and the same principle applies to agents.

**5. Separate reads from writes.** A common pattern: the agent reads current state, decides what to do, then does it. If the read and write aren't in the same transaction, state can change between them. For agents, this means: re-read state at the point of action, or use optimistic concurrency controls.

## The Mental Model Shift

Most agent developers think: *my agent runs once, completes, and that's it.*

Production reality: *your agent will be interrupted, retried, duplicated, and asked to re-run partial work. Plan for it.*

The shift is from "make it work" to "make it safe to repeat." It's not glamorous. It doesn't demo well. But it's the difference between an agent that works on your laptop and one that works in production at 3 AM when you're asleep.

Idempotency isn't an optimization. It's a correctness requirement. If your agent can't safely run the same task twice, it can't safely run once — because you can't guarantee it won't be asked to.