---
name: observability-logging-monitoring-guardian
description: Review and improve system observability with emphasis on useful logs, error tracking, metrics, alerts, health checks, and operational monitoring that make support, debugging, maintenance, and incident response easier. Use when Codex needs to assess or improve how a system exposes runtime behavior, detects failures, surfaces degradation, supports troubleshooting, or proves service health in production and staging environments.
---

# Observability Logging Monitoring Guardian

## Overview

Review observability as the system's ability to explain its own behavior under normal operation and failure.
Prioritize operational visibility that helps humans detect, understand, and resolve real issues quickly.

## Operating Mode

Treat logs, metrics, alerts, traces, and health checks as one connected operational feedback system.
Inspect whether the system emits the right signals at the right points and whether those signals are actionable for support and engineering.

Focus on whether operators can answer these questions without guesswork:

- is the system healthy right now
- what failed
- where it failed
- which users or tenants are affected
- whether the issue is transient, growing, or resolved
- what changed before the incident started

Do not stop at "we log errors." Check whether the output is structured, correlated, and useful in real support workflows.

## Audit First

Before proposing changes, identify:

1. The critical user-facing and internal flows that need operational visibility.
2. The main failure modes the system is likely to encounter.
3. Which runtime signals already exist and which are missing.
4. Who will consume the signals: support, developers, operations, or leadership.

Use real code and runtime paths to answer:

- Where are logs emitted and what context do they include?
- Are application errors captured centrally?
- Which metrics describe health, throughput, latency, and failures?
- Are alerts tied to symptoms that matter or just to noisy technical events?
- Do health endpoints reflect meaningful readiness and dependency state?
- Can one request, job, or incident be traced across system boundaries?

## Review Axes

### 1. Log quality and structure

Check whether logs help investigation instead of adding noise.

Look for:

- unstructured free-text logs
- missing correlation IDs or request context
- logs without tenant, user, job, or resource context where relevant
- duplicate logging at many layers
- sensitive data leaking into logs
- logs that announce activity but not outcome

Prefer logs that are:

- structured
- scoped to meaningful events
- rich in operational context
- safe for production use

### 2. Error tracking and failure visibility

Review how failures are surfaced and grouped.

Inspect:

- uncaught exception capture
- handled errors that disappear silently
- failure categorization
- stack trace availability
- contextual breadcrumbs around the failure
- distinction between user mistakes, dependency failures, and internal bugs

Flag systems where support cannot tell whether an issue is user error, product bug, or external dependency degradation.

### 3. Metrics and service indicators

Treat metrics as behavioral evidence, not vanity counters.

Check:

- request volume
- latency
- error rate
- queue depth and processing time
- job failures and retries
- integration success/failure rates
- business-critical operational counters

Prefer metrics that help answer:

- are users getting results
- where is the bottleneck
- is performance degrading
- which subsystem is unhealthy

### 4. Alerts and signal-to-noise ratio

Review alerts as an operational decision system.

Look for:

- alerts with no clear responder action
- thresholds detached from user impact
- alert storms from one root cause
- missing alerts for silent degradation
- alerts that trigger too late or too early

Prefer alerts that are:

- actionable
- tied to material symptoms
- routed to the right audience
- resilient against noise

### 5. Health checks and readiness

Inspect health endpoints critically.

Check:

- liveness versus readiness distinction
- dependency checks that matter to user-facing behavior
- false-green health endpoints
- unhealthy states that are not reflected externally
- startup and migration readiness handling

Treat health checks as contracts for orchestration and operations, not cosmetic endpoints.

### 6. Cross-system correlation

Review whether a single incident can be traced across boundaries.

Check:

- request IDs
- job IDs
- webhook/event IDs
- subscription or tenant identifiers
- propagation of trace context across services and workers

Flag systems where each component logs locally but incidents still require manual archaeology across disconnected outputs.

### 7. Support and maintenance usability

Inspect whether observability actually helps day-to-day support.

Look for:

- missing context to answer customer tickets
- no clear way to inspect recent failures for one tenant or user
- dashboards that show infrastructure but not product behavior
- no quick signal for whether a fix worked after deployment

Prefer observability that helps both incident response and routine support triage.

## Prioritization Rules

Rank issues in this order:

1. Failures or degradations that cannot be detected reliably.
2. Missing context that prevents fast diagnosis of user-impacting problems.
3. Alerting or health-check designs that mislead operators.
4. Logging and metrics gaps that make support slow or reactive.
5. Secondary cleanup around formatting, naming, or dashboard polish.

Avoid adding more telemetry volume if the real problem is poor signal design or lack of correlation.

## Output Expectations

When delivering a review, provide:

1. the main observability gaps
2. the root cause behind each gap
3. the operational or support impact
4. the smallest high-leverage correction
5. any later-stage monitoring refinement worth sequencing after the core fix

When delivering implementation, prefer changes that improve real visibility into system health, failure causes, and user impact rather than just adding more logs.

## Review Standard

Hold the system to this bar:

- important failures are visible
- logs are structured and useful
- metrics describe real health and behavior
- alerts are actionable and not noisy
- health checks reflect meaningful readiness
- incidents can be traced across boundaries

If support and engineering still have to guess what happened during a failure, treat that as a real system design problem.
