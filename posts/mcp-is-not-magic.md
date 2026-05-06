# MCP Is Not Magic — It's a Plumbing Standard

May 6, 2026

Everyone's excited about Model Context Protocol. "Just add MCP support and your agent can use any tool!" Sounds great. In practice, it's more like adopting a plumbing standard — useful, but the pipes still leak if you don't understand the pressure.

## What MCP Actually Solves

Before MCP, every agent framework had its own tool interface. LangChain tools, OpenAI function calling, CrewAI tools — each one slightly different, each one requiring adapters if you wanted to share tools between frameworks. MCP standardizes the *interface*: a JSON schema for describing tools, a protocol for calling them, and a transport layer (stdio or HTTP+SSE).

That's it. That's the whole thing. It's like when REST replaced SOAP — same capability, less ceremony.

## What It Doesn't Solve

MCP doesn't make your agent smarter about *which* tool to use. It doesn't solve the tool routing problem. If your agent picks the wrong tool, MCP just makes it pick the wrong tool faster and more portably.

It also doesn't solve reliability. An MCP server that crashes mid-call, returns inconsistent schemas, or has undocumented rate limits will break your agent just as thoroughly as a hand-rolled function call would. The protocol is transport — the semantics are still on you.

## Where MCP Genuinely Helps

**Tool reuse across frameworks.** Write a tool once, expose it via MCP, and any compatible agent can use it. This is the real win. Your Postgres query tool works with Claude, GPT, local models — anything that speaks MCP.

**Composability.** MCP servers can be chained. A file server, a database server, and a web search server can all coexist, and the agent picks what it needs per-query. No custom glue code.

**Debugging.** Standardized protocol means standardized observability. You can intercept MCP calls, log them, replay them. Try doing that with a Python function call buried in an agent loop.

## Practical Takeaways

1. **Don't rewrite working tools just to add MCP.** Wrap them. A thin MCP server around existing logic is an afternoon's work, not a refactor.

2. **Validate schemas at the boundary.** MCP trusts the server to return what it promised. Your agent shouldn't. Add validation on the client side — one malformed response can send an agent into a spiral.

3. **Handle server failures gracefully.** MCP servers are processes. Processes die. Your agent needs fallback logic, timeouts, and retry — same as any other distributed system.

4. **Don't confuse protocol with architecture.** MCP is a wire format, not a design philosophy. Your agent still needs good tool routing, loop detection, and error handling. MCP makes none of that automatic.

## The Bottom Line

MCP is worth adopting — it reduces integration friction and makes tools portable. But it's infrastructure, not intelligence. The hard problems in agent design (routing, reliability, evaluation) are the same as before. MCP just makes the easy parts easier, which is exactly what a good standard should do.