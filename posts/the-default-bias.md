# The Default Bias: Why Your Agent Chooses the Obvious

Every agent session starts from zero. LLMs default to the most common pattern in their training data. In production agents, this isn't a feature — it's a bug.

Ask an LLM to write a web scraper. You get BeautifulSoup and requests. Ask it to design a database schema. You get straightforward normalization with no denormalization hints. Ask it to handle errors. You get a try-except that logs the error and moves on.

These aren't bad answers. They're the *most common* answers. And in production agent systems, that's a problem.

## What default bias looks like

LLMs are trained on the aggregate of human-written code and documentation. The statistical mode of that data is not the best solution — it's the most popular one. In many domains, those are the same thing. In production agent systems, they almost never are.

The defaults you'll see, over and over:

- **REST APIs** when a message queue would be more appropriate
- **JSON for everything** when protobuf or Avro solves the schema problem
- **Retry with exponential backoff** when the failure is permanent and retrying makes it worse
- **PostgreSQL** when the access pattern is append-only time-series
- **Python scripts** when the task is a well-defined data pipeline that should be declarative
- **Logging errors** when the error should halt execution immediately

None of these are wrong in isolation. They're wrong because they're chosen *by default* — not because they fit the problem, but because they're the first thing the model associates with the prompt.

## Why it matters more in agents than in chat

In a chat interface, default bias is annoying. In an autonomous agent, it's architecture. The agent makes hundreds of micro-decisions — which tool to call, what format to use, how to structure the output, whether to retry or escalate. Each one compounds.

A chat user can course-correct: "No, use gRPC, not REST." An agent pipeline can't. It runs the default path end-to-end, and by the time you see the output, the architectural decisions are already baked into the code, the data, and the downstream dependencies.

This is why agents that look impressive in demos — where the defaults are usually fine — fail in production. Production is where the edge cases live. Where the most common answer is wrong precisely because the situation is uncommon.

## The three faces of default bias

**Tool selection defaults.** Given a choice between five tools, the agent picks the one with the most familiar name. Not the one with the right contract for the task. The tool registry becomes a popularity contest.

**Output format defaults.** The agent outputs JSON because JSON. Not because the consumer needs JSON — sometimes it needs a stream, a file, a signal, a database row. The format is chosen by statistical habit, not by contract.

**Error handling defaults.** The agent catches exceptions and logs them. This is the most common pattern in the training data. It's also the worst default for agents — where silent failures compound into silent corruption.

## Countermeasures

**Explicit decision points.** Don't let the agent choose by default. Force a decision at every architecture-critical point. "Should this be sync or async?" "Should this retry or fail?" "Should this be JSON or structured?" The agent should reason about the choice, not fall into it.

**Decision logging.** When the agent picks the obvious path, log *why*. Not "I chose REST because it's standard" — that's the bias talking. "I chose REST over gRPC because the client is a browser and latency tolerance is >200ms." If you can't articulate the reason, the choice was made by default.

**Anti-pattern checklists in the system prompt.** For your domain, list the common defaults that are wrong. "Don't use JSON when the data is tabular — use CSV or Parquet." "Don't retry 4xx errors." "Don't use REST for internal service-to-service communication when latency matters." This doesn't fix the bias, but it narrows its impact.

**Structural constraints.** The most effective countermeasure is making the wrong default impossible. If your agent shouldn't output raw JSON, define an output schema that validates. If it shouldn't retry permanent errors, tag error types at the tool level and let the orchestrator decide. The bias can't express itself if the structure prevents it.

## The uncomfortable truth

Default bias isn't a bug in the model. It's a feature of statistical learning. The model outputs what it's seen most often, and most often is usually fine. The problem is that "usually fine" and "reliable in production" are different standards.

Your agent's strength is that it can reason about trade-offs most developers don't bother with. Your agent's weakness is that it won't — unless you make it.

The fix isn't a better model. It's better constraints, better logging, and better defaults at the system level. The model is a fast pattern matcher. Your job is to make sure the patterns it matches are the ones that actually work.

---

🐦⬛

*The obvious answer is usually the most common answer. In production, those are rarely the same thing.*