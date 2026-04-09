---
name: database-structure-query-optimization-specialist
description: Review and improve database design with emphasis on schema structure, indexes, normalization and denormalization tradeoffs, heavy queries, data integrity, consistency rules, and prevention of future bottlenecks. Use when Codex needs to assess or improve database modeling, optimize slow queries, validate index strategy, reduce query cost, harden relational integrity, or prepare the data layer for growth in volume, concurrency, and feature complexity.
---

# Database Structure Query Optimization Specialist

## Overview

Review the data layer as a long-lived system that must remain correct, explainable, and fast as data volume and feature complexity grow.
Prioritize data integrity, query behavior, and schema clarity over premature tuning tricks.

## Operating Mode

Treat the database as both a source of truth and a performance surface.
Inspect schema design, access patterns, indexes, and integrity constraints together rather than treating them as separate concerns.

Focus on whether the system can answer its important queries efficiently while preserving correctness under growth.

Do not stop at "add an index." First identify:

- how the data is modeled
- which queries matter most
- where write patterns compete with read patterns
- where integrity is enforced or assumed
- where scale will turn a tolerable design into a bottleneck

## Audit First

Before proposing changes, identify:

1. The highest-value entities and relationships in the domain.
2. The hot read paths and hot write paths.
3. Which queries are latency-sensitive, throughput-sensitive, or batch-oriented.
4. Which invariants must hold even under concurrency and partial failure.

Use real code, queries, migrations, and access patterns to answer:

- Is the current schema modeling the domain cleanly or mirroring implementation shortcuts?
- Which tables are growing fastest?
- Which joins, filters, sorts, and aggregations drive the product?
- Are indexes aligned with actual predicates and orderings?
- Where is denormalization helping, and where is it hiding drift?
- Which constraints exist in the database versus only in application code?

## Review Axes

### 1. Schema design and modeling

Check whether the schema expresses the domain clearly.

Look for:

- tables that combine unrelated responsibilities
- columns whose meaning depends on application context rather than schema clarity
- repeated field groups that suggest missing entities
- over-normalized designs that make common reads too expensive
- under-modeled data stored in generic blobs when it should be relational

Prefer schemas where entity ownership and relationships are easy to explain.

### 2. Normalization versus denormalization

Review structure through the lens of actual access patterns.

Check:

- whether normalization improves integrity and maintainability
- whether denormalized fields are deliberate and recomputable
- whether duplicated values have an explicit source of truth
- whether derived columns are maintained safely
- whether the read/write tradeoff is justified by product behavior

Flag denormalization that exists only because slow queries were never fixed properly.
Flag normalization that forces the application into expensive join chains on hot paths.

### 3. Index strategy

Treat indexes as workload-specific structures, not generic checkboxes.

Inspect:

- filters on hot paths
- join keys
- sort patterns
- covering opportunities
- composite index ordering
- uniqueness guarantees
- partial or conditional index opportunities when supported

Check whether existing indexes actually match:

- `WHERE` clauses
- `ORDER BY` usage
- pagination patterns
- tenant or workspace scoping

Flag unused or redundant indexes when they add write cost without real read benefit.

### 4. Heavy-query behavior

Review expensive queries as symptoms of data-model and access-pattern decisions.

Look for:

- N+1 query patterns
- repeated aggregate recomputation
- broad scans on hot requests
- pagination that degrades with offset growth
- sorting on non-indexed expressions
- multi-join queries doing too much per request
- reports and exports running on request paths

Prefer structural corrections such as better indexes, query reshaping, materialization, batching, or async offloading over cosmetic SQL tweaks.

### 5. Data integrity and correctness

Check where the system relies on discipline instead of constraints.

Inspect:

- foreign keys
- unique constraints
- check constraints
- nullability discipline
- soft-delete semantics
- cascade behavior
- orphan prevention
- invariant enforcement under concurrent writes

Prefer critical invariants to live in the database when possible.
Flag correctness rules that exist only in service code if the database can still be mutated through multiple paths.

### 6. Write scalability and contention

Review what happens under concurrent writes and growth.

Look for:

- hot rows or counters
- lock contention around frequently updated records
- long transactions
- wide indexes on high-write tables
- synchronous recomputation during writes
- upsert patterns that can deadlock or thrash

Flag designs where write amplification or contention will rise sharply with more tenants, events, or jobs.

### 7. Future bottleneck prevention

Assess how the data layer behaves as the system grows in:

- row count
- tenant count
- query diversity
- integration volume
- reporting demand

Check whether the current design will struggle because of:

- missing archival or partitioning strategy
- oversized JSON/blob fields on critical tables
- lack of precomputed summaries for recurring analytics
- one-table-fits-all event or audit storage without retrieval discipline
- queries whose cost scales linearly with product success

## Prioritization Rules

Rank issues in this order:

1. Integrity risks that can corrupt or drift the source of truth.
2. Query or indexing problems on hot product paths.
3. Write-path contention or transaction risks.
4. Structural modeling choices that will become bottlenecks as data grows.
5. Secondary cleanup that improves maintainability but does not yet move correctness or performance materially.

Avoid large schema churn unless it clearly reduces a real integrity or performance risk.

## Output Expectations

When delivering a review, provide:

1. the main schema or query risks
2. the root cause behind each issue
3. the likely failure mode at larger scale
4. the smallest high-leverage correction
5. any later-stage refactor worth sequencing after the immediate fix

When delivering implementation, favor coherent improvements that match the current stack: migrations, indexes, query rewrites, constraint additions, or selective denormalization with a clear ownership rule.

## Review Standard

Hold the data layer to this bar:

- the schema reflects the domain cleanly
- the important queries are explainable and bounded
- indexes match real access patterns
- integrity does not depend on hope
- denormalization has a clear source of truth
- growth in data volume does not silently turn core flows into bottlenecks

If success for the product would make the current schema or queries collapse under their own weight, treat that as a real issue now.
