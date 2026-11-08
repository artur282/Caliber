# Caliber — roadmap

> **Status: plan, not implemented.** Dates are intentionally relative: the project moves when the previous milestone is green, not when a calendar says so.

## M0 — Plan (current)

- [x] Product scope, non-goals and market fit.
- [x] Architecture and data model.
- [x] Demo script and acceptance criteria.
- [x] Metrics definitions and test strategy.
- [x] Planned decisions.
- [ ] Review by the owner.
- [ ] First code commit.

## M1 — Skeleton

- [ ] `pyproject.toml`, FastAPI app, settings, health endpoint.
- [ ] PostgreSQL + pgvector via Docker Compose.
- [ ] CI with lint, types and tests.
- [ ] `.env.example`, `.gitignore`, LICENSE.

**Exit:** `docker compose up` reaches healthy; CI is green.

## M2 — Corpus and golden set

- [ ] 30 technical documents.
- [ ] Deterministic chunker (120 words, 24 overlap).
- [ ] 24 golden questions with 28 validated evidence rows.
- [ ] Seed endpoint and idempotent dataset hash.
- [ ] First migration.

**Exit:** seeding twice produces the same counts and hash.

## M3 — Retrievers

- [ ] Full-text with GIN and `ts_rank_cd`.
- [ ] Vector with HNSW and cosine.
- [ ] Hybrid RRF with document collapse.
- [ ] Hash and local embedding providers.

**Exit:** each retriever returns ranked documents for every question; RRF has a hand-computed test.

## M4 — Metrics

- [ ] recall@5, recall@10, MRR, nDCG@10.
- [ ] Latency mean and nearest-rank p95.
- [ ] Hand-calculated tests.
- [ ] Same-config determinism through the API.

**Exit:** the metric tests are green and a reader can reproduce the arithmetic.

## M5 — Experiment API and UI

- [ ] `POST /experiments`, `GET /experiments/{id}`, `GET /leaderboard`.
- [ ] One-page UI with checkboxes, table, traces and “what you are seeing”.
- [ ] Seeded baseline; mobile layout.

**Exit:** the 60-second demo works end to end.

## M6 — Evidence

- [ ] Real table with the `local` provider.
- [ ] README first screen, architecture, review path, limitations.
- [ ] Four ADRs and three Spanish learning docs.
- [ ] Screenshot and CI status.
- [ ] A five-minute review by someone who has not seen the project.

**Exit:** a reviewer can explain what the project proves and what it does not.

## M7 — Follow-ups (not MVP)

- [ ] Harder golden set: adversarial paraphrases, unanswerable questions, graded relevance.
- [ ] Sentence-aware chunking experiment.
- [ ] CSV export.
- [ ] Hosted demo with a read-only token.
- [ ] A small post comparing the three retrievers with the measured numbers.
