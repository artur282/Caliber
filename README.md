# Caliber — plan

**A reproducible evaluation lab for search and RAG retrieval.**
One corpus, one golden set, three retrievers and real PostgreSQL — measured, not asserted.

> **Status: plan, not implemented.** This repository contains the product plan, the architecture, the demo script and the acceptance criteria. There is no application code yet. No metric is published until it is measured.

[![status](https://img.shields.io/badge/status-plan-6C63FF?style=flat-square)](#status)
[![stack](https://img.shields.io/badge/stack-Python%20%C2%B7%20FastAPI%20%C2%B7%20PostgreSQL%20%2B%20pgvector-4169E1?style=flat-square)](#planned-stack)
[![license](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## The problem

Every RAG team eventually asks: *is this retriever actually better, or does it just feel better?* Changing the chunker, the embedding model or the fusion strategy without a controlled experiment produces anecdotes, not evidence.

Caliber is a small lab that answers the question with the same corpus, the same golden questions and the same metrics for every configuration:

- full-text retrieval;
- vector retrieval with pgvector;
- hybrid retrieval with Reciprocal Rank Fusion;
- recall@5, recall@10, MRR, nDCG@10 and latency;
- one stable configuration hash so the same input produces the same ranking.

It is a lab, not a search product. There is no ingestion pipeline, no auth, no multi-tenancy and no LLM generation.

## Why this project

The market study behind the portfolio (100 backend postings in Spain and Switzerland/Germany, plus 58 live postings reviewed in November 2026) found:

- **Python is the first language of the Spanish backend market**: 26.5% of postings.
- **AI/LLM appears in 11%** of Spanish postings and grows inside backend roles.
- **Hiring managers read evidence, not plans**: a README with a demo, real metrics, architecture and honest limitations.

Caliber targets the Spanish market: Python + FastAPI + PostgreSQL + RAG, with evaluation as the differentiator. Continuum, the second project, targets the Rust/systems market.

## The 60-second demo (what the finished MVP will show)

```text
docker compose up  ->  http://localhost:8000
Run experiment      ->  three configurations, one table
Expand a config     ->  expected evidence, ranked documents, per-query trace
```

The demo starts with a deterministic seed, so the page is never empty. It runs three retrieval configurations over 24 golden questions and shows recall@5, recall@10, MRR, nDCG@10 and latency. The point is visible in one screen: the same corpus and the same questions, three strategies, one comparable table.

See [docs/DEMO.md](docs/DEMO.md) for the exact script and the expected observable result.

## Scope (MVP)

- **Corpus:** 30 technical documents, deterministic chunking, 24 golden questions with expected document and passage.
- **Retrievers:** PostgreSQL full-text (`tsvector` + GIN), pgvector HNSW with cosine distance, hybrid RRF.
- **Metrics:** recall@5, recall@10, MRR, nDCG@10, mean and p95 latency, computed from first principles.
- **Reproducibility:** stable config hash, dataset hash and a deterministic seed; same configuration → same ranking metrics.
- **API:** health, seed, run experiment, get result, leaderboard, single-page UI.
- **Demo:** `docker compose up` with seeded data; one page, no frontend build.
- **Evidence:** CI from the first commit, four ADRs, learning docs in Spanish, README with real numbers only when measured.

## Non-goals

- A production search engine or a hosted SaaS.
- Authentication, multi-tenancy, rate limiting or billing.
- LLM answer generation; Caliber evaluates retrieval, not generation.
- A large corpus or a statistically significant benchmark.
- Elasticsearch, OpenSearch or an external vector database. PostgreSQL is the only substrate.

## Planned stack

Python 3.12 · FastAPI · SQLAlchemy 2 async · Alembic · PostgreSQL 16 + pgvector · Pydantic v2 · pytest · ruff · mypy · Docker Compose · GitHub Actions. Managed with `uv`.

The embedding provider is an interface: `hash` for deterministic CI (no model download) and `local` (`sentence-transformers/all-MiniLM-L6-v2`) for the semantic demo. The README will always state which provider produced a published number.

## Planned architecture

```text
browser (vanilla JS)
   |
   v
FastAPI / uvicorn
   |-- experiment runner: config hash, metrics, per-query traces
   |-- embedding provider: hash | local MiniLM (384 dimensions)
   |
   v
PostgreSQL 16 + pgvector
   documents and chunks
     chunks.tsv        -> GIN index (full-text)
     chunks.embedding  -> HNSW index (cosine)
   golden questions and evidence
   experiments and per-query results
```

One process, one database, one compose file. The full plan is in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Milestones

| Phase | Deliverable | Exit criterion |
|---|---|---|
| **P0 — Skeleton** | Repository, CI, Docker Compose, settings, health endpoint | CI green with lint, types and an empty test suite |
| **P1 — Corpus** | 30 documents, deterministic chunker, 24 golden questions with validated evidence | Seeding twice produces the same counts and dataset hash |
| **P2 — Retrievers** | Full-text, vector and hybrid RRF | Each retriever returns ranked documents through the same interface |
| **P3 — Metrics** | recall@5, recall@10, MRR, nDCG@10, latency | Hand-calculated unit tests pass |
| **P4 — Experiment API** | Run, fetch, leaderboard, stable config hash | Same config → same ranking metrics through the public API |
| **P5 — Demo** | One-page UI, seeded baseline, 60-second path | `docker compose up` shows the table without a manual seed |
| **P6 — Evidence** | README with real numbers, four ADRs, three Spanish learning docs | A reviewer can follow the demo and the decisions in five minutes |

The detailed task list and acceptance criteria are in [docs/PLAN.md](docs/PLAN.md). The roadmap is in [ROADMAP.md](ROADMAP.md).

## Evidence contract

This is the definition of done for the implementation:

1. **CI from the first commit** — ruff, mypy and pytest against a real `pgvector/pgvector:pg16` service.
2. **README first screen** — what it is, the 60-second demo, the measured table (only after it exists), then architecture and trade-offs.
3. **Metrics only when measured** — no invented numbers, no skill bars. The README states the provider, corpus, golden set, machine and limitations.
4. **One-command demo** with seeded data; never an empty screen.
5. **Four ADRs** in `docs/decisions/` and three learning docs in Spanish in `docs/learning/`.
6. **Hygiene** — no secrets, `.env.example` only, correct `.gitignore`, LICENSE, topics and a single `main` branch.
7. **Honest limitations** — recall saturation on a small set, single-run latency, document-level binary relevance and the difference between the `hash` and `local` providers.

## How to review this plan

1. Read this README (2 minutes).
2. Read [docs/PLAN.md](docs/PLAN.md) — phases, tasks and acceptance criteria (5 minutes).
3. Read [docs/DEMO.md](docs/DEMO.md) — what the reviewer will see in 60 seconds.
4. Read [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/DECISIONS.md](docs/DECISIONS.md) — the hard choices before the code.
5. Check the [wiki](https://github.com/artur282/Caliber/wiki) for the same material in a navigable form.

## Limitations of the plan

- The MVP is intentionally small: 30 documents and 24 questions. Recall may saturate; MRR and nDCG@10 carry the comparison. A harder set with adversarial paraphrases and unanswerable questions is the next step, not the first one.
- The sample is not statistically significant. The README will say so.
- Latency is a single-machine measurement, not an SLO.
- The `hash` provider exists for deterministic CI; its numbers are not comparable to the `local` model and will never be mixed in the leaderboard.
- The project is not deployed. A hosted demo is a later milestone, not a promise.

## License

MIT — see [LICENSE](LICENSE).

## Links

- Portfolio hub: [github.com/artur282/Portafolio](https://github.com/artur282/Portafolio)
- Second project: [Continuum](https://github.com/artur282/Continuum) — durable execution in Rust
- Wiki: [github.com/artur282/Caliber/wiki](https://github.com/artur282/Caliber/wiki)
