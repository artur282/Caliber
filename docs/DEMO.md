# Caliber — the 60-second demo

> **Status: plan.** This is the script the finished MVP must make true. It is also the acceptance test for the UI.

## Before the demo

```bash
docker compose up --build
# wait for the API health check
open http://localhost:8000
```

The database is seeded and a baseline run is already present. If the page is empty, the demo fails.

## The script

### 0:00 — The screen

The reviewer sees:

- the corpus summary: 30 documents, 60 chunks, 24 golden questions;
- three configuration checkboxes: full-text, vector, hybrid;
- a baseline table with recall@5, recall@10, MRR, nDCG@10 and latency;
- a panel that explains what each column means.

**What it proves:** the lab starts with evidence, not with a setup wizard.

### 0:10 — Run an experiment

Press **Run experiment** with all three configurations selected.

The table refreshes with the three rows. The same config hashes produce the same ranking metrics; latency may move with CPU load.

**What it proves:** the comparison is controlled and repeatable.

### 0:25 — Read the table

Point at the differences:

- recall@5 and recall@10 show whether the expected evidence is in the top 5 / top 10;
- MRR shows how high the first correct document ranks;
- nDCG@10 shows the quality of the whole top 10;
- latency shows the cost of each strategy.

The README states the provider, corpus, golden set and machine. The table is a real run, not an estimate.

**What it proves:** the reviewer can evaluate the trade-off in one screen.

### 0:40 — Expand a trace

Expand the hybrid configuration, then one question. The reviewer sees:

- the expected document and passage;
- the ranked documents;
- the passage that caused each hit;
- the per-query contribution to the aggregate.

**What it proves:** the aggregate is auditable; the lab does not hide the rows behind a number.

### 0:55 — The review path

Open `README.md` and `docs/decisions/`. The reviewer sees the architecture, the four decisions, the limitations and the exact commands to reproduce the table.

**What it proves:** the project is documented like an engineering artifact, not a demo.

## Failure cases the demo must not have

- An empty page on first load.
- A table with `TODO`, `NaN` or invented numbers.
- A run that silently uses a different provider than the one in the README.
- A mobile layout with horizontal overflow.
- A browser console error.

## What the demo deliberately does not show

- Ingestion of arbitrary documents.
- Authentication or multi-tenancy.
- LLM answers.
- A production deployment.

Those are non-goals, not omissions. The README says so.
