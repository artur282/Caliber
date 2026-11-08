# Caliber — decisions to make before the code

> **Status: plan.** These are the decision records the implementation will turn into ADRs. They are written now so the code does not decide them by accident.

## 0001 — PostgreSQL is the only retrieval substrate

**Context.** The MVP needs full-text and vector retrieval, a golden set and experiment results. A separate vector database or Elasticsearch would add a service, a second consistency model and a second failure mode.

**Decision.** Use PostgreSQL 16 with `pgvector` for both retrieval paths. Full-text uses `tsvector` + GIN; vectors use `HNSW` + cosine distance.

**Alternatives.** Elasticsearch/OpenSearch for BM25; Qdrant/Weaviate/Chroma for vectors; SQLite for local tests.

**Consequences.** One service, one backup, one transaction boundary and a simple demo. The lexical ranker is `ts_rank_cd`, not BM25; the README must say so. A future comparison with a tuned BM25 engine is a separate experiment.

## 0002 — Evaluate at document level with binary relevance

**Context.** Chunks are the retrieval unit, but the golden questions are about documents. Counting chunk hits would inflate recall when several chunks come from the same document.

**Decision.** Collapse chunks to their parent document before every cutoff. Use binary relevance: a document is relevant or not. The golden set stores the expected document and the passage that supports the answer.

**Alternatives.** Chunk-level relevance; graded relevance; LLM-as-judge.

**Consequences.** The metrics are simple and auditable. Graded relevance and inter-annotator agreement are out of scope. The limitation is documented.

## 0003 — Two embedding providers

**Context.** CI must run without downloading a model or calling a paid API. The demo needs semantic embeddings to be credible.

**Decision.** Define an `EmbeddingProvider` interface with two implementations:

- `hash`: deterministic, no download, used by tests and CI;
- `local`: `sentence-transformers/all-MiniLM-L6-v2`, 384 dimensions, used by the demo.

The provider is part of the dataset hash and is visible in the API and README.

**Alternatives.** A single paid API; a single local model in CI; precomputed embeddings committed to the repository.

**Consequences.** The two providers are not comparable. The README must state which provider produced each published number. CI stays fast and offline.

## 0004 — Store aggregate metrics plus per-query traces in JSONB

**Context.** The UI needs to explain why a question hit or missed. A separate analytics schema would be overkill for 24 questions.

**Decision.** Store the aggregate metrics in typed columns and the per-query traces in JSONB on `experiment_configs`. The trace contains the question, expected evidence, ranked documents and matching passages.

**Alternatives.** A normalized `experiment_queries` table; a separate analytics database; no traces.

**Consequences.** The result is easy to fetch and render. JSONB is not a long-term analytics store; if the corpus grows, the traces move to a table or an object store.

## Open questions

- Sentence-aware chunking vs word windows: start with word windows, measure, then decide.
- CSV export: useful for reviewers; not required for the 60-second path.
- A hosted demo: decide after the local demo is stable.
