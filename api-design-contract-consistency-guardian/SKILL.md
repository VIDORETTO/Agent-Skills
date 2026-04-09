---
name: api-design-contract-consistency-guardian
description: Standardize and review API contracts across frontend and backend by improving endpoint design, naming, payload structure, response shape, status code usage, error messages, versioning strategy, and cross-layer consistency. Use when Codex needs to audit or improve HTTP APIs, align frontend expectations with backend responses, clean up inconsistent contracts, design new endpoints, or make an existing API easier to consume and evolve safely.
---

# API Design Contract Consistency Guardian

## Overview

Review APIs as long-lived product contracts, not just transport plumbing.
Prioritize predictability, consistency, and safe evolution across frontend and backend.

## Operating Mode

Treat every endpoint as part of a coherent contract surface.
Inspect how routes, payloads, responses, errors, and versioning behave together rather than reviewing handlers in isolation.

Focus on whether consumers can reliably predict:

- where an operation lives
- what shape to send
- what shape comes back
- which status code to expect
- how failures are represented
- how contracts evolve without breaking clients

Do not stop at syntax. Look for contract drift, accidental exceptions, and mixed conventions.

## Audit First

Before proposing changes, identify:

1. The main API domains and consumer flows.
2. Which clients consume the API and how tightly they depend on specific fields.
3. The current conventions for resources, verbs, envelopes, pagination, and errors.
4. Where the contract is predictable versus where each endpoint behaves differently.

Use concrete code and real payloads to answer:

- Are routes organized by resource or by implementation accident?
- Are request and response shapes stable across similar endpoints?
- Do list, detail, create, update, and delete operations follow the same conventions?
- Do error responses use one schema or many?
- Are status codes semantically correct or just convenient?
- Is versioning explicit, absent, or inconsistent?
- Does the frontend need adapter logic because the API is irregular?

## Review Axes

### 1. Endpoint design and naming

Check whether routes are easy to predict.

Look for:

- mixed plural and singular resource naming
- action endpoints where resource modeling would be clearer
- similar operations exposed under unrelated route patterns
- backend-internal terminology leaking into public routes
- nested routes that overexpress implementation details

Prefer:

- resource-oriented naming
- stable path conventions
- explicit action routes only when the action is the real domain concept
- route shapes that match user-facing concepts, not storage details

### 2. Payload consistency

Check whether request bodies and query params behave consistently.

Inspect:

- naming conventions across fields
- inconsistent filter, sort, and pagination parameters
- mixed required and optional semantics without clear rules
- create and update payloads that diverge without reason
- frontend-specific quirks encoded into public contracts

Prefer:

- one naming convention per API
- similar parameter patterns across comparable endpoints
- explicit validation rules
- payloads that reflect domain intent cleanly

### 3. Response shape consistency

Treat response structure as a product surface.

Check:

- whether list responses follow the same envelope
- whether detail responses expose consistent field naming
- whether metadata placement is predictable
- whether nullable and omitted fields are used intentionally
- whether the frontend needs per-endpoint transformation logic to normalize responses

Prefer response contracts where consumers can infer the shape from endpoint type.
Flag endpoints that return ad hoc structures without justification.

### 4. Status code semantics

Check whether status codes communicate meaning precisely.

Inspect:

- `200` used where `201`, `202`, `204`, `400`, `404`, `409`, `422`, or `429` would be clearer
- validation failures mixed with authorization or domain-rule failures
- async operations pretending to be completed synchronously
- idempotent operations returning unstable codes across retries

Prefer status codes that let clients branch correctly without reading ambiguous message text.

### 5. Error contract quality

Review errors as a first-class contract.

Check:

- whether errors share one schema
- whether machine-readable codes exist where needed
- whether messages are stable enough for UI use
- whether validation errors are field-specific and actionable
- whether internal exception text leaks to clients

Prefer errors with:

- predictable top-level shape
- clear machine-readable category or code
- user-safe message text
- field detail when validation fails

### 6. Versioning and evolution

Inspect how the API changes over time.

Check:

- whether breaking changes have a strategy
- whether deprecated fields or endpoints are announced clearly
- whether additive changes stay non-breaking in practice
- whether frontend and backend are coupled to deploy at the same instant

Prefer a versioning and deprecation strategy that matches product reality.
Flag designs where contract changes rely on synchronized luck between clients and server.

### 7. Frontend-backend alignment

Review the contract boundary from both sides.

Look for:

- duplicated DTO shaping in multiple frontend modules
- frontend adapters compensating for inconsistent backend responses
- backend fields that exist only for one narrow screen workaround
- hidden coupling to exact error text
- stale fields still relied on by old client code

Treat repeated mapping and defensive parsing in the frontend as evidence of contract design problems.

## Prioritization Rules

Rank issues in this order:

1. Contract mismatches that can break clients or cause incorrect behavior.
2. Error and status-code inconsistencies that block reliable client handling.
3. Response-shape irregularities that force frontend normalization per endpoint.
4. Naming and route-design problems that increase confusion and drift.
5. Versioning gaps that make future changes risky.

Avoid large rewrites unless they materially reduce consumer breakage or long-term contract entropy.

## Output Expectations

When delivering a review, provide:

1. the main contract inconsistencies
2. the root cause behind each inconsistency
3. the impact on frontend consumers or future evolution
4. the smallest high-leverage standardization step
5. any larger contract cleanup worth sequencing later

When delivering implementation, make the contract more predictable end to end and update affected frontend/backend touchpoints coherently.

## Review Standard

Hold the API to this bar:

- similar endpoints behave similarly
- payloads and responses are easy to predict
- status codes carry real meaning
- error bodies are stable and actionable
- frontend consumers do not need bespoke parsing per route
- contract evolution can happen without accidental breakage

If a client developer has to memorize endpoint exceptions or inspect source code to guess payload shape, treat that as a real design problem.
