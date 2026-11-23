---
title: "When a vector search misses the runbook"
date: 2026-11-22
slug: "when-a-vector-search-misses-the-runbook"
description: "Diagnose filtered retrieval in pgvector before changing the model or prompt."
tags: ["ai", "data-engineering", "postgresql", "pgvector", "retrieval"]
draft: false
---

Imagine an on-call data engineer asking an AI assistant how to recover a failed invoice load. The assistant searches operational runbooks, retrieves relevant passages, and uses them to explain the recovery procedure. In this hypothetical system, the correct runbook exists, but the assistant says it found insufficient guidance.

Before changing the prompt, I would inspect the retrieval query. A filter for the production environment can leave an approximate vector search with too few results, even when plenty of production documents exist.

Suppose the team stores runbook chunks in PostgreSQL with pgvector and an HNSW index. Each chunk has an embedding and environment metadata. The application asks for the nearest eight production chunks. That sounds straightforward, but SQL's logical meaning does not tell you how the database finds those rows.

The [pgvector filtering documentation](https://github.com/pgvector/pgvector#filtering) explains that, with an approximate index, filtering happens after the index scan. Candidates from development and staging can be discarded by the production filter, leaving fewer than eight results. Increasing `LIMIT` alone does not establish that the search examined enough candidates.

First, capture the actual query and its bound parameters. Check that the missing runbook was ingested, has the expected metadata, and is visible to the retrieval role. Then inspect the plan with [PostgreSQL's EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html). Confirm whether it uses the HNSW index; the presence of an index does not mean the planner selected it.

For a controlled comparison, run an exact search over the same eligible rows, using the same embedding and a stable dataset. Compare returned chunk IDs with the approximate result. If exact search finds the recovery passage and approximate search misses it, you have isolated a retrieval problem worth fixing. If both miss it, investigate the embedding, chunk boundaries, and document content.

[pgvector 0.8.0 introduced iterative scans](https://www.postgresql.org/about/news/pgvector-080-released-2952/), which continue searching when filtering leaves insufficient results, up to configured limits. This illustrative, untested query assumes a compatible installed version, an HNSW cosine index, and an application-bound query vector:

```sql
BEGIN;
SET LOCAL hnsw.iterative_scan = strict_order;

SELECT id, content
FROM runbook_chunks
WHERE environment = 'production'
ORDER BY embedding <=> $1::vector
LIMIT 8;

COMMIT;
```

Strict ordering preserves distance order among the returned candidates. It does not turn approximate search into an exact search. More scanning also consumes work, so measure latency alongside the recovered passages.

There is another reasonable choice. If production documents form a small eligible subset, filtering through a conventional index and calculating exact distances may be fast enough. I would benchmark that path before accepting the tuning and monitoring burden of approximate retrieval.

Returning eight chunks is only a diagnostic milestone. All eight could be irrelevant. Keep the exact-search comparison separate from a human check that the retrieved passage actually explains the invoice recovery procedure.

For this assistant, I would add the failed-load question to a retrieval regression set, with the required runbook passage identified. Test it across environments with different document counts. Choose the simplest search plan that consistently retrieves that passage within the response-time budget. If none does, keep the assistant's insufficient-guidance response while repairing retrieval; an operational procedure should not be filled in from guesswork.

