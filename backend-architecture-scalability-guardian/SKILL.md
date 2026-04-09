---
name: backend-architecture-scalability-guardian
description: Review backend systems with emphasis on modular architecture, separation of responsibilities, service boundaries, queue usage, cache strategy, concurrency safety, throughput, performance bottlenecks, and readiness for growth. Use when Codex needs to assess or improve a backend for maintainability and scale, audit controllers/services/workers/data flows, evaluate background job design, reduce coupling, prepare for higher traffic, or make a backend more robust under load.
---

# Backend Architecture Scalability Guardian

## Overview

Review existing backend systems as production software that must stay understandable while traffic, data volume, integrations, and team size grow.
Prioritize architecture, operational behavior, and scaling risks over abstract purity.

## Operating Mode

Treat this skill as a backend-structure and growth-readiness review.
Focus on whether the system can evolve safely, run predictably under load, and remain easy to change without spreading fragility.

Inspect the real execution model:

- request path
- service and domain boundaries
- database access patterns
- background workers and queues
- caching behavior
- concurrency and consistency risks
- integration boundaries
- failure handling and retries

Do not stop at naming or code style. Look for places where the current structure will become expensive, slow, or unsafe as usage grows.

## Audit First

Before proposing changes, identify:

1. The main runtime paths that matter most.
2. Which modules own which responsibilities.
3. Where state is created, mutated, cached, queued, or synchronized.
4. Which flows are synchronous today but should not remain synchronous as load grows.

Use concrete code and runtime behavior to answer:

- What is the unit of modularity here: route, controller, service, domain module, worker, repository?
- Are business rules centralized or duplicated across handlers and jobs?
- Where are cross-module dependencies muddy or cyclic?
- Which operations are latency-sensitive versus throughput-sensitive?
- Which writes need transactional consistency?
- Which tasks can be offloaded to background execution?
- Which reads should be cached, and at what invalidation cost?

## Review Axes

### 1. Modularity and boundaries

Check whether modules have clear ownership.

Look for:

- controllers or routes that contain business logic
- services that know too much about unrelated domains
- helpers that became hidden god-modules
- repository or ORM code leaking into many layers
- worker code duplicating request-path logic

Prefer:

- explicit domain ownership
- thin transport layers
- business logic concentrated in coherent services or domain modules
- shared logic extracted once with a clear home

### 2. Separation of responsibilities

Check whether each layer has a narrow reason to change.

Separate clearly:

- transport and validation
- orchestration and business rules
- persistence and query shaping
- background execution and scheduling
- external integration adapters

If one file or service is doing validation, policy, persistence, API composition, and retry logic together, flag it as structural debt.

### 3. Queue and async-job design

Inspect background processing critically.

Check:

- which tasks must leave the request path
- idempotency of jobs
- retry policy and poison-message behavior
- deduplication strategy
- backpressure handling
- visibility into stuck or failing jobs
- ownership of job payload schemas

Prefer background execution for:

- external syncs
- large exports/imports
- fan-out work
- expensive recalculation
- non-blocking notifications

Flag designs where request latency depends on work that should be queued.

### 4. Cache strategy

Treat caching as a consistency design problem, not just a speed trick.

Check:

- what is cached and why
- cache key shape and ownership
- invalidation path
- TTL realism
- stale-data tolerance
- cache stampede risk
- fallback behavior when cache is cold or unavailable

Prefer caches around expensive, repeated, low-volatility reads with clear invalidation ownership.
Flag caches that hide poor query design or create correctness drift.

### 5. Concurrency and consistency

Review what happens when the same entity is updated concurrently.

Look for:

- read-modify-write races
- duplicate job execution
- missing idempotency on webhooks or retries
- transaction boundaries that are too wide or too narrow
- lock contention
- out-of-order event handling
- eventually consistent flows with no reconciliation path

Call out where the system relies on luck rather than explicit concurrency control.

### 6. Performance and scale bottlenecks

Identify the real bottlenecks before recommending optimization.

Inspect:

- N+1 query patterns
- full-table scans on hot paths
- excessive serialization or payload size
- synchronous chaining of remote calls
- repeated computation that should be memoized, cached, or precomputed
- work executed per request that should be batched
- endpoints with unbounded fan-out or pagination weakness

Prefer removing structural bottlenecks over micro-optimizing local code.

### 7. Growth readiness

Assess whether the backend can support more:

- traffic
- tenants
- data volume
- integrations
- operators
- developers touching the code at once

Flag designs that will break down when:

- background volume spikes
- one tenant dominates throughput
- retries pile up
- a dependency slows down
- a new feature needs to reuse existing logic cleanly

## Prioritization Rules

Rank issues in this order:

1. Correctness or consistency risks under concurrency.
2. Structural coupling that blocks safe change.
3. Request-path work that should be async.
4. Data-access or integration bottlenecks on hot paths.
5. Cache, observability, or maintainability gaps that amplify incidents.

Avoid recommending large refactors unless they materially reduce a real operational or scaling risk.

## Output Expectations

When delivering a review, provide:

1. the main architectural risks
2. the root cause behind each risk
3. the likely failure mode at higher scale
4. the smallest high-leverage correction
5. any follow-up refactor that becomes justified later

When delivering implementation, change only the parts required to improve the architecture meaningfully and keep the design coherent with the existing stack.

## Review Standard

Hold the backend to this bar:

- module ownership is explainable
- business rules are not scattered
- request paths stay bounded
- background work is explicit and reliable
- caching has clear correctness rules
- concurrency behavior is intentional
- the next major feature does not require copy-paste architecture

If the design would become fragile under more traffic, more data, or more developers, treat that as a real issue even if the code works today.
