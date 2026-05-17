# Configuration as Code for AI Agents

Every AI agent has configuration. Prompts, model parameters, tool definitions, temperature settings, context window limits. At first it's a single YAML file or a few environment variables. Then it's three files. Then ten. Then someone changes the temperature in production and the agent that was reliably extracting invoices starts writing poetry.

If this sounds familiar, it's because we've solved this problem before — just not for agents.

## The Config Creep Problem

Agent configurations have a unique property that makes them harder to manage than traditional service configs: they're semi-code, semi-data.

Your temperature setting isn't just a number — it's a behavioral contract. Your system prompt isn't just text — it's a program written in natural language that gets interpreted by a non-deterministic runtime. Your tool definitions are both API specs and behavioral instructions.

This means config changes have unpredictable blast radius. Changing a database timeout from 30s to 60s is predictable. Changing "Be concise" to "Be brief" in a system prompt might shift your agent's entire decision-making pattern.

And yet most teams manage agent configs the way they managed server configs in 2010 — manual edits, no versioning, no review, no rollback.

## What Configuration as Code Means for Agents

The principle is simple: if a change affects agent behavior, it should go through the same process as code changes. Commits, reviews, tests, rollbacks.

Here's what this looks like in practice:

**Version control everything.** System prompts, tool descriptions, model parameters, even few-shot examples. If it changes behavior, it's in git. No exceptions.

**Diff-friendly formats.** Store prompts as Markdown files, not JSON strings embedded in code. You can't review a 2000-character escaped string in a diff. But you can review a Markdown file.

**Environment parity.** Your dev agent and production agent should differ only in the model endpoint and API keys. Same prompts, same tools, same parameters. If you need different behavior in production, that's a config flag, not a copy-paste.

**Deploy configs like code.** CI/CD for prompts. A merge to main deploys the new configuration. With rollback. With canary if you're serious.

## The Prompt-as-Code Pattern

Here's the pattern I use:

```
agents/
  invoice-extractor/
    system-prompt.md       # The system prompt — versioned, reviewed
    tools.json             # Tool definitions
    config.yaml            # Temperature, max_tokens, etc.
    tests/
      extract-invoice.yaml # Test cases
      edge-cases.yaml
```

Each agent has its own directory. The system prompt is a first-class file, not a string constant. Tool definitions are separate from the prompt so you can version them independently. Configuration is explicit and minimal.

When someone wants to change the extraction behavior, they edit `system-prompt.md`, open a PR, and the test suite runs against the new prompt. If tests pass, it ships. If tests fail, you catch it before users do.

## The Uncomfortable Truth About Prompt Changes

Here's something most teams learn the hard way: prompt changes are more like schema migrations than config updates.

When you change a database schema, you plan a migration. You test it. You have a rollback plan. When you change a system prompt, you should apply the same discipline — because a prompt change is effectively a behavioral migration. Your agent will behave differently. Downstream systems that consume its output may break.

Treat prompt changes with the same gravity you'd treat an API contract change. Because that's exactly what they are.

## What to Do Monday

1. **Find all your agent configs.** Prompts, parameters, tool definitions. All of them. Check the database, check the codebase, check the .env files, check the admin panel.

2. **Put them in git.** Extract them from wherever they're hiding. One file per logical unit. Markdown for prompts, YAML/JSON for structured config.

3. **Write three test cases per agent.** One happy path, one edge case, one failure mode. Run them manually if you have to. You can automate later.

4. **Stop editing configs in production.** If you're SSHing into a server or clicking around an admin panel to change a prompt, that's the habit to break first.

Configuration as code isn't exciting. It's plumbing. But it's the plumbing that keeps your agents from becoming unmaintainable messes. And unlike most agent infrastructure advice, you can actually start doing it on Monday.

---

*Published May 17, 2026*