# Caliber — implementation plan

> **Status: plan, not implemented.** This document is the source of truth for the first implementation. It is written before the code, deliberately.

## 1. Goal

Build the smallest complete version of Caliber that a reviewer can understand in **60 seconds** and that proves one capability: *the same corpus and the same golden questions produce comparable retrieval metrics across three strategies.*

The MVP is a lab, not a product. It runs with one command, seeds itself and shows the evidence on one page.

## 2. Definition of done

- [ ] `docker compose up` starts PostgreSQL 16 + pgvector and the API with seeded data; the page is never empty.
- [ ] The three retrievers (full-text, vector, hybrid RRF) run over the same 24 golden questions.
- [ ] The results table shows recall@5, recall@10, MRR, nDCG@10 and latency per configuration.
- [ ] The same configuration hash produces the same ranking metrics through the public API.
- [ ] `ruff`, `mypy` and `pytest` pass locally and in CI against a real `pgvector/pgvector:pg16` service.
- [ ] The README has the 60-second demo, the measured table (only when measured), architecture, review path and limitations.
- [ ] Four ADRs and three learning docs in Spanish exist.
- [ ] No secrets; `.env.example` only.

## 3. Phases and tasks

### P0 — Skeleton (foundation)

| ID | Task | Acceptance |
|---|---|---|
| C0.1 | Create `pyproject.toml` with `uv`, Python 3.12, FastAPI, SQLAlchemy async, Alembic, pgvector, Pydantic, pytest, ruff, mypy | `uv sync` resolves; `uv run python -c "import app"` works |
| C0.2 | Create `app/main.py`, `app/config.py`, `app/db.py` with settings and a `/health` endpoint | `GET /health` returns 200 and database status |
| C0.3 | Create `docker-compose.yml` with `db` (pgvector/pgvector:pg16) and `api` | `docker compose config` is valid; `docker compose up` reaches healthy |
| C0.4 | Create `.github/workflows/ci.yml` with the Postgres service | CI runs lint, types and an empty test suite |
| C0.5 | Create `.env.example`, `.gitignore`, `LICENSE` | No secret is committed; `.env.example` is visible |

### P1 — Corpus and golden set

| ID | Task | Acceptance |
|---|---|---|
| C1.1 | Write 30 technical documents about Python, FastAPI, PostgreSQL, pgvector and RAG | Each document has a stable slug and a title |
| C1.2 | Implement the deterministic chunker: 120-word windows, 24-word overlap | Same input → same chunks; unit tests cover boundaries |
| C1.3 | Write 24 golden questions, each with an expected document and a passage | The passage is validated to exist in the document; 28 evidence rows |
| C1.4 | Implement `POST /seed` and the startup seed | Seeding twice produces the same document, chunk and question counts and the same dataset hash |
| C1.5 | Add `documents`, `chunks`, `golden_questions`, `golden_evidence` tables and the first Alembic migration | `alembic upgrade head` works on an empty database |

### P2 — Retrievers

| ID | Task | Acceptance |
|---|---|---|
| C2.1 | Full-text retriever: generated `tsvector`, GIN index, `ts_rank_cd`, OR of lexemes | Returns a ranked list of documents for every question |
| C2.2 | Vector retriever: 384-dim embeddings, HNSW cosine index, `ef_search` per transaction | Returns a ranked list with an `id` tie-breaker |
| C2.3 | Hybrid retriever: 50 candidates per list, RRF with `k = 60`, collapse chunks to documents | Hand-computed RRF test passes; duplicates do not inflate results |
| C2.4 | Embedding provider interface: `hash` (deterministic, no download) and `local` (all-MiniLM-L6-v2) | Tests use `hash`; the demo can use `local`; the provider is visible in the API and README |
| C2.5 | Shared retriever interface and configuration objects | All three retrievers are interchangeable in the experiment runner |

### P3 — Metrics

| ID | Task | Acceptance |
|---|---|---|
| C3.1 | Implement recall@k, MRR, nDCG@k and nearest-rank p95 from first principles | Hand-calculated unit tests pass |
| C3.2 | Collapse chunks to documents before every cutoff | A document with several chunks is counted once |
| C3.3 | Macro-average per question; latency measured around the full retrieval call | A question with no relevant document scores zero, not an error |
| C3.4 | Add a test that same config → same ranking metrics through the API | Test passes with the seeded corpus |
| C3.5 | Document the formulas in `docs/learning/02-metricas-de-recuperacion.md` | A reader can reproduce each number on paper |

### P4 — Experiment API

| ID | Task | Acceptance |
|---|---|---|
| C4.1 | `POST /experiments` accepts one or more retrievers and runs the golden set | Returns 201 with per-config metrics and per-query traces |
| C4.2 | `GET /experiments/{id}` returns the full result | 404 for unknown IDs; the response includes the configuration |
| C4.3 | `GET /leaderboard` returns the best completed row per configuration hash for the current dataset | Sorted by nDCG@10; ties are stable |
| C4.4 | Stable config hash: canonical JSON of retriever + chunker + embedding + top_k | Same settings → same hash; the hash is stored with the result |
| C4.5 | Failure handling: unknown retriever → 422; empty retriever list → 422; database error → 503 | Tests cover each case |

### P5 — Demo UI

| ID | Task | Acceptance |
|---|---|---|
| C5.1 | One HTML page, one stylesheet, one vanilla-JS file | No frontend build; served by FastAPI |
| C5.2 | Config checkboxes, “Run experiment” button, results table | The table shows the five metrics and the config hash |
| C5.3 | Expandable per-question trace: expected evidence, ranked documents, matching passages | A reviewer can see why a hit happened |
| C5.4 | “What you are seeing” panel and leaderboard | The page explains recall, MRR and nDCG without requiring the README |
| C5.5 | Seeded baseline on startup | Opening the page shows a completed baseline, not an empty state |
| C5.6 | Mobile layout: the table becomes cards, no horizontal overflow | Screenshot at 390×844 is readable |

### P6 — Evidence and closure

| ID | Task | Acceptance |
|---|---|---|
| C6.1 | Run the demo with the `local` provider and record the real table | The table includes provider, corpus, golden set, machine and date |
| C6.2 | Write the README first screen, architecture, review path and limitations | A reviewer can follow it in five minutes |
| C6.3 | Write four ADRs: PostgreSQL as the only substrate; document-level evaluation; two embedding providers; JSONB traces | Each ADR has context, decision, alternatives, consequences |
| C6.4 | Write three learning docs in Spanish | They explain the system, the metrics and reproducibility |
| C6.5 | Verify CI locally with the same commands as the workflow | `ruff`, `mypy`, `pytest` clean; CI pending the first push |
| C6.6 | Add a screenshot of the final UI | The image matches the current page, not an earlier version |

## 4. Test strategy

- **Unit:** metrics with hand-calculated cases, chunker boundaries, RRF, hashing.
- **Integration:** seed determinism, each retriever against PostgreSQL, API contract, same-config determinism.
- **Manual:** the 60-second demo, the mobile layout, the error cases (`400/404/422`).
- **CI:** the same commands against `pgvector/pgvector:pg16`; no model download and no paid API.

The test suite must not depend on a downloaded model. The `hash` provider exists for that reason.

## 5. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Recall saturates on a small corpus | Publish MRR and nDCG@10; state the limitation; plan a harder set as a follow-up |
| The model download makes the demo slow | Bake the model into the image and run offline; keep `hash` for CI |
| Scope creep into a search product | The non-goals are explicit; no auth, no ingestion, no generation |
| The metrics look impressive but are wrong | Hand-calculated tests; collapse chunks to documents; publish the formulas |
| The UI becomes a frontend project | One HTML file and vanilla JS; no framework, no build step |
| The README overclaims | A number is published only when the run exists; limitations are mandatory |

## 6. Non-goals

- Production search, multi-tenancy, auth, billing.
- LLM answer generation or a chat UI.
- Elasticsearch/OpenSearch or an external vector database.
- A statistically significant benchmark or a large corpus.
- A hosted demo in the first milestone.

## 7. Deliverables

```text
caliber/
  pyproject.toml
  README.md
  docker-compose.yml
  Dockerfile
  .env.example
  .github/workflows/ci.yml
  migrations/
  app/
    main.py  config.py  db.py  models.py  schemas.py
    seed.py  metrics.py  experiments.py  api.py
    retrieval/ (embeddings.py, fulltext.py, vector.py, hybrid.py, service.py)
    templates/index.html
    static/app.css  static/app.js
  tests/
    test_metrics.py  test_chunking.py  test_hashing.py
    test_embeddings.py  test_seed.py  test_retrieval.py  test_api.py
  docs/
    decisions/0001..0004
    learning/01..03
```

## 8. First implementation order

1. P0.1 → P0.5 (skeleton and CI).
2. P1.1 → P1.5 (corpus and seed; verify the dataset hash).
3. P2.1 → P2.5 (retrievers; verify each against PostgreSQL).
4. P3.1 → P3.5 (metrics; verify the hand calculations).
5. P4.1 → P4.5 (API; verify determinism).
6. P5.1 → P5.6 (UI; verify the 60-second path).
7. P6.1 → P6.6 (evidence; publish only measured numbers).

Do not start P4 before P3 is green: an API over wrong metrics is worse than no API.
