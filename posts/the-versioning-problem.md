---
title: "The Versioning Problem: Your Agent Has No Release Tags"
date: 2026-08-19
tags: ["agents", "versioning", "production", "reliability"]
---

# The Versioning Problem: Your Agent Has No Release Tags

Your code is versioned. Your data is versioned. Your infrastructure is versioned. Your agent? Your agent is a prompt file in a Google Doc that someone edited last Tuesday without telling anyone.

I've never walked into a team where the agent system had proper versioning. Not once. And I've walked into a lot of teams.

## What You're Not Versioning

An agent system has at least four layers that need versioning, and most teams version zero of them:

**Prompts.** The system prompt, the tool descriptions, the output format instructions. These are code — they determine behavior as much as any function in your codebase. But they live in unversioned files, get edited in place, and nobody knows what changed when.

**Tool definitions.** The schemas your agent uses to call APIs, databases, and external services. When an API adds a required field, does your tool definition track that? When you change a parameter name, is there a migration path? Or does the agent just start failing with a cryptic error from the tool executor?

**Model configuration.** Which model, which version, which parameters. Temperature, top-p, max tokens, retry counts. These are knobs that change behavior. If you can't answer "what model config was running when this bug was reported?" you can't reproduce the bug.

**Evaluation baselines.** Your eval suite — the set of test cases that tells you whether the agent is still working. If the eval suite changes at the same time as the agent, you can't tell whether a regression is from the agent change or the eval change. These need to be versioned independently.

## Why It Breaks

The failure mode is always the same. Something goes wrong in production. You investigate. You discover the prompt was changed. By whom? When? Why? What was the previous version? Nobody knows. The Google Doc has the full edit history, but good luck reconstructing what the prompt looked like when the bug happened.

Or worse: someone improves the prompt. The agent gets better on the test cases — great. But it silently gets worse on a class of inputs that isn't in the test suite. Nobody notices for two weeks because the monitoring only tracks errors, not quality regression. By the time someone reports degraded outputs, three other changes have been layered on top. Now you're debugging a stack of unversioned changes with no baseline to compare against.

This is how teams end up afraid to touch their own agent. Not because it's good — because they've lost the ability to safely change it. Every modification is a leap of faith because there's no rollback, no diff, no "what did this look like before."

## The Minimum Viable Versioning

You don't need a sophisticated MLOps platform. You need four things:

**1. Prompts in git, not in docs.** Every prompt is a file in your repository. Edits go through pull requests. The PR description says what changed and why. This alone solves 80% of the "who changed what and when" problem. It's the single highest-impact change you can make, and most teams haven't made it.

**2. A version manifest for every deploy.** A JSON file that records: prompt version (git SHA), model name and version, model parameters, tool definition versions, eval suite version. This gets logged with every agent run. When something breaks, you look up the manifest for that run and know exactly what configuration produced the behavior.

```json
{
  "prompt_version": "a3f2c1e",
  "model": "gpt-4o-2024-08-06",
  "temperature": 0.3,
  "tool_definitions_version": "b7d4e8a",
  "eval_suite_version": "c1a2b3d",
  "deployed_at": "2026-08-19T10:00:00Z"
}
```

**3. Tagged releases, not rolling deploys.** When you change the prompt, you create a tag. The tag is what deploys. If the new version is worse, you roll back to the previous tag. This is standard practice for code and somehow novel for agents.

**4. Eval results in the PR.** Before a prompt change merges, the eval suite runs against both the old and new versions. The PR shows the diff: which test cases improved, which regressed, which stayed the same. If any critical case regresses, the merge is blocked. This turns "I think this is better" into "here's what changed."

## The Hard Part: Model Versioning

Prompt versioning is straightforward — it's text in a file. Model versioning is harder because you don't control the model. Providers update models silently. The model called `gpt-4o` today is not the same model that was called `gpt-4o` six months ago. The behavior changes, the latency changes, the cost changes — and you didn't change anything.

You can't pin a model version forever. But you can:

- **Record which model version produced which outputs.** The manifest above handles this. When behavior shifts, you check whether the model version shifted.
- **Run a canary on model updates.** When your provider announces a model update, don't switch all traffic. Route 5% to the new model, compare outputs to the old model on the same inputs. If the diff is small, increase. If it's large, investigate.
- **Maintain a golden dataset.** A set of 50-100 representative inputs with known-good outputs. Run this dataset against any new model version before adopting it. If more than 10% of outputs change meaningfully, you're not upgrading — you're migrating.

## The Uncomfortable Question

If I asked you right now: "What prompt was your agent running on August 5th at 3 PM?" — could you answer?

If the answer is "I'd have to check the Google Doc history and cross-reference with the deploy log and maybe ask Sarah" — you don't have versioning. You have archaeology.

Your agent is a system. Systems need version control. Not because it's best practice, but because without it, you can't answer the most basic question about your own production system: what was running when it broke?

Put the prompts in git. Tag your releases. Log your manifests. Run your evals. It's not complicated, and it's the difference between an agent you can trust and an agent you're afraid to touch.

---

*Part of an ongoing series on building AI agent systems that work in production. Related posts on [configuration drift](/blog/configuration-drift/), [the deployment checklist you don't have](/blog/the-deployment-checklist-you-dont-have/), and [the replay problem](/blog/the-replay-problem/) cover adjacent aspects of the same challenge.*