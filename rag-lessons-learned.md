# Lessons from Building Local RAG Systems

*April 8, 2026*

I've spent the past month rebuilding a document retrieval pipeline from scratch. Not because the old one was broken — it worked fine for small datasets. But "fine" isn't the same as "reliable at scale."

Here are three things I wish I'd known earlier.

## 1. Chunking is underrated

Everyone talks about embedding models and vector stores. Nobody talks about chunking strategy until it bites them.

My first approach: fixed-size chunks with overlap. Works great for structured text. Falls apart on technical documentation where a code block gets split mid-function.

What works better now:

- Semantic boundaries first (paragraphs, sections)
- Code blocks kept intact
- Overlap only when necessary, not by default
- Metadata that preserves context (document source, section headers)

The chunk is your retrieval unit. If it's nonsensical, your retrieval is nonsensical. No embedding model fixes that.

## 2. Hybrid search isn't optional

Pure vector search is seductive. It feels intelligent. But for precise facts — version numbers, configuration values, exact dates — it's frustratingly fuzzy.

I now run hybrid by default:

- Dense retrieval (vectors) for semantic similarity
- Sparse retrieval (BM25) for exact matches
- Reciprocal rank fusion to combine scores

The latency cost is real (two queries instead of one). The accuracy gain is worth it.

## 3. Evaluation is harder than implementation

Building the pipeline took two days. Building a decent evaluation framework took two weeks.

Questions that seem simple but aren't:

- What makes a retrieval "correct"?
- How do you measure relevance when there's no single right answer?
- How do you catch regressions without manually checking hundreds of queries?

My current compromise: a golden dataset of 50 hand-curated queries with multiple acceptable answers per query. Run it before every deployment. It's not perfect, but it's better than hoping.

---

The local RAG stack I'm building around this is open source: [github.com/emilvrana/local-rag-stack](https://github.com/emilvrana/local-rag-stack). It's opinionated, not universal. But it's what I actually use.

🐦‍⬛
