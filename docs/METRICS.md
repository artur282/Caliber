# Caliber — metrics plan

> **Status: plan.** No metric in this repository is published yet. The numbers below are definitions and acceptance criteria, not results.

## Ranking metrics

Every metric operates on a ranked list of **documents** and a set of relevant documents. Chunks must be collapsed to their parent document before the cutoff.

### recall@k

```text
recall@k = |relevant documents in the top k distinct results| / |relevant documents|
```

- Cutoffs: 5 and 10.
- A question with no relevant document scores 0, not an error.
- Duplicate chunks from the same document count once.

### MRR (mean reciprocal rank)

```text
RR = 1 / rank of the first relevant document (0 if none)
MRR = mean(RR) over all questions
```

### nDCG@10

Binary relevance, gain 1 for a relevant document, discount `1 / log2(rank + 1)`:

```text
DCG@10  = Σ gain / log2(rank + 1)
IDCG@10 = Σ 1 / log2(rank + 1) for the ideal top 10
nDCG@10 = DCG@10 / IDCG@10
```

### Latency

- Wall time around one full retrieval call, including query embedding.
- Report mean and nearest-rank p95.
- A single-run measurement is not an SLO. The README will say so.

## What will be published

After the MVP runs, the README table will include, for each configuration:

| Column | Source |
|---|---|
| recall@5, recall@10 | Aggregate over 24 questions |
| MRR | Mean of reciprocal ranks |
| nDCG@10 | Mean over 24 questions |
| mean and p95 latency | Server-side measurement |
| config hash | Canonical JSON of the configuration |
| dataset hash | Corpus + provider |

The README will also state:

- embedding provider and model;
- corpus size, chunker and golden-set size;
- machine and date;
- whether latency is a single run;
- the limitations (small sample, recall saturation, document-level binary relevance).

## What will never be published

- A number without a run.
- A metric mixed across providers (`hash` vs `local`).
- A p95 presented as an SLO with 24 samples.
- A comparison with a system that was not measured under the same conditions.

## Test plan for the metrics

- Hand-calculated cases for recall, MRR and nDCG, with the arithmetic in the docstring.
- Duplicate documents do not inflate any metric.
- Relevant documents below the cutoff are ignored for nDCG.
- Empty relevant sets score zero.
- Same configuration → same ranking metrics through the public API.
- A macro average weights each question once.

## Baseline and follow-ups

The baseline is the three retrievers on the seeded corpus. A harder set with adversarial paraphrases, unanswerable questions and graded relevance is a follow-up, not part of the MVP.

The first published table will be honest about its size. A small, reproducible result is better than a large, unverifiable one.
