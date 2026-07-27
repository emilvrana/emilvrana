# Permanently lost posts

On **2026-07-20** an automated deploy ran `rsync --delete` against the web root and removed 22
post directories that existed on the server but not in this repository. They had no backup: the
host's nightly job has never covered `/opt/www`, and the site was not archived by any public
crawler.

On **2026-07-27** the loss was worked through post by post. 8 of the 22 were recovered from
surviving copies of their source text and are live again. The **14 below could not be recovered**
and their links have been removed from `/blog/` so readers no longer hit a 404.

Nothing here was rewritten or regenerated. A post is listed as lost precisely because its original
text no longer exists anywhere — writing a fresh article under the same title would have been a
forgery, not a restoration.

## Lost (14)

| Date | Title | URL that is now gone |
|---|---|---|
| 2026-03-12 | Running a Local LLM on Your Own Server | `/blog/local-llm-server/` |
| 2026-03-14 | How I Built Persistent Memory for AI Agents | `/blog/ai-agent-memory/` |
| 2026-03-17 | n8n for Developers: When Workflow Automation Beats Writing Code | `/blog/n8n-for-developers/` |
| 2026-03-19 | RAG Pipeline Deep-Dive: From Document Ingestion to Answer Generation | `/blog/rag-pipeline-deep-dive/` |
| 2026-03-23 | Prompt Caching: Cut LLM Costs by 80% on Context-Heavy Workloads | `/blog/prompt-caching/` |
| 2026-03-24 | What to Log When Your AI Agent Goes Rogue | `/blog/agent-observability/` |
| 2026-03-26 | Tool Schema Design for AI Agents | `/blog/tool-schema-design/` |
| 2026-03-28 | Context Window Management for Long-Running Agents | `/blog/context-window-management/` |
| 2026-05-12 | When Your AI Agent Goes Quiet: Detecting Silent Failures | `/blog/silent-failures/` |
| 2026-06-08 | Stop Building Agents. Start Building Protocols. | `/blog/stop-building-agents/` |
| 2026-06-16 | Why Your RAG Pipeline Needs Reranking | `/blog/reranking-rag/` |
| 2026-06-17 | Context Engineering > Prompt Engineering | `/blog/context-engineering/` |
| 2026-06-19 | Structured Outputs Are the Skeleton, Not the Decoration | `/blog/structured-outputs/` |
| 2026-07-03 | Fallback Patterns: What Happens When Your Agent Can't | `/blog/fallback-patterns/` |

For each of these, all that survives is the title, the date and the one-paragraph summary that used
to appear in the blog index — recorded above and in the publish-event log. The article bodies are
gone.

## Recovered (8)

| Title | URL | Recovered from |
|---|---|---|
| Agent Evaluation Is Not Optional — It's the Feature | `/blog/agent-eval/` | `posts/agent-evaluation-is-not-optional.html` |
| Your Agent's API Is a Contract | `/blog/ai-agent-contract/` | `posts/your-agents-api-is-a-contract.html` |
| Don't Build an Agent. Build a Pipeline. | `/blog/dont-build-an-agent/` | `dont-build-an-agent-build-a-pipeline.html` |
| MCP Is Not Magic — It's a Plumbing Standard | `/blog/mcp-in-practice/` | `mcp-is-not-magic.html` |
| Multi-Agent Patterns: When One Agent Isn't Enough | `/blog/multi-agent-patterns/` | `multi-agent-patterns.html` |
| The Retry Trap: Why Retrying Failed AI Calls Makes Things Worse | `/blog/retry-trap/` | `retry-trap.html` |
| The Specification Gap | `/blog/the-specification-gap/` | an older working copy of this repo (the 2026-07-01 original; the version at `/the-specification-gap.html` is a different, later article that reuses the title) |
| The Tool Count Trap: Why More Tools Make Your Agent Worse | `/blog/tool-count-trap/` | the author's draft, `blog-post-tool-count-trap.md` |

## Sources searched

Every place the text could plausibly have survived was checked. Those that yielded nothing are
listed too, so the search is not repeated from scratch:

| Source | Result |
|---|---|
| This repository, working tree | 7 posts |
| This repository, full git history (all blobs, all refs) | no additional posts; no `blog/<slug>/` directory for any of the 22 ever existed here |
| Two older working copies of this repo | 1 post (the 2026-07-01 "Specification Gap") |
| Author's draft/scratch directories | 1 post ("The Tool Count Trap") |
| Remote git mirrors (2) | 0 — both fully contained in local history |
| Web server: docroot, siblings, nightly backups, Docker volumes, filesystem snapshots, nginx cache | 0 — `/opt/www` was never a backup target and the host has no snapshot capability |
| Search index / memory store (38 indices) | 0 — publish-event log lines and one-line summaries only, never article bodies |
| Wayback Machine, archive.today, Cloudflare cache, web search for syndicated copies | 0 — the site was never archived; its HTML is served uncached |

*Recorded 2026-07-27.*
