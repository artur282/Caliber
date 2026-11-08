# Caliber — planned architecture

> **Status: plan.** This is the architecture the MVP will implement. It is deliberately small: one process, one database, one page.

## Components

```text
┌──────────────────────────────────────────────┐
│ Browser — one HTML page + vanilla JS          │
│ config checkboxes · results table · traces   │
└──────────────────────┬───────────────────────┘
                       │ fetch (JSON)
┌──────────────────────▼───────────────────────┐
│ FastAPI / uvicorn                            │
│ /health /seed /experiments /leaderboard /    │
└───────┬──────────────────────────┬───────────┘
        │                          │
┌───────▼──────────────┐  ┌────────▼────────────┐
│ Experiment runner     │  │ Embedding provider   │
│ config hash · metrics │  │ hash │ local MiniLM  │
│ per-query traces      │  │ 384 dimensions       │
└───────┬──────────────┘  └────────┬────────────┘
        │                          │
┌───────▼──────────────────────────▼───────────┐
│ PostgreSQL 16 + pgvector                     │
│ documents ─ chunks                           │
│   chunks.tsv       → GIN (full-text)         │
│   chunks.embedding → HNSW (cosine)           │
│ golden questions + evidence                  │
│ experiments + per-query results (JSONB)      │
└──────────────────────────────────────────────┘
```

## Data model (planned)

- `documents(id, slug, title, body, created_at)` — 30 rows.
- `chunks(id, document_id, ordinal, content, tsv, embedding vector(384))` — 60 rows.
- `golden_questions(id, slug, question)` — 24 rows.
- `golden_evidence(id, question_id, document_id, passage)` — 28 rows, validated to exist in the document.
- `experiments(id, status, dataset_hash, embedding_provider, config jsonb, created_at, completed_at)`.
- `experiment_configs(id, experiment_id, retriever, config_hash, config jsonb, metrics jsonb, per_query jsonb)`.

The experiment result is stored as aggregate metrics plus per-query traces. The traces make the demo explainable; they are not a separate analytics system.

## Retrieval flows

**Full-text.** The question is converted to lexemes and an OR `tsquery`. The query ranks chunks by `ts_rank_cd` with normalization 32 and returns the top 50 candidates. Chunks are collapsed to documents before the cutoff.

**Vector.** The question is embedded with the configured provider. The query sets `hnsw.ef_search` for the transaction and orders by cosine distance with an `id` tie-breaker. It returns the top 50 candidates.

**Hybrid.** Both lists are fused with RRF: `score = Σ 1 / (k + rank)` with `k = 60`. The fused list is collapsed to documents, then the top 10 are returned. The candidate pool and `k` are part of the configuration hash.

## Reproducibility

A configuration is hashed as canonical JSON of:

- retriever type and parameters;
- chunker name, target words and overlap;
- embedding provider, model and dimension;
- `top_k`.

The dataset hash adds the corpus and the provider. The same hashes must produce the same ranking metrics. Latency is stored but is not part of the determinism claim.

## Failure modes

| Failure | Behavior |
|---|---|
| Database unavailable | `/health` returns 503; the UI shows an actionable error |
| Model unavailable | The API logs it and falls back to `hash`; the UI badge changes; the README table is marked stale |
| Unknown retriever | `422` with the allowed values |
| Empty retriever list | `422` |
| Experiment exceeds the request timeout | The MVP runs synchronously; the API returns an error, not a half-written result |
| Duplicate seed | Idempotent: same counts and dataset hash |

## What is intentionally missing

- A queue or background worker: 24 questions × 3 retrievers is a synchronous request.
- Authentication and rate limiting: local demo only.
- A separate vector database: PostgreSQL + pgvector is the only substrate; that is the point.
- LLM generation: the lab evaluates retrieval, not answers.

## Open decisions

- Exact embedding model: `all-MiniLM-L6-v2` is the default candidate; the final choice is recorded in ADR 0003.
- Chunker: word windows with overlap are the baseline; sentence-aware chunking is a follow-up.
- Whether to expose a CSV export in the UI: useful for reviewers, not required for the MVP.
