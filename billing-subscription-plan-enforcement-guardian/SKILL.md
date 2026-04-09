---
name: billing-subscription-plan-enforcement-guardian
description: Review and improve billing systems with emphasis on subscriptions, plan definitions, entitlement enforcement, usage limits, trials, upgrades, downgrades, cancellations, payment-state handling, and commercial rule consistency across the product. Use when Codex needs to assess or improve how a system charges customers, enforces plan access, handles billing lifecycle changes, or keeps pricing and product behavior aligned without leaks or inconsistent exceptions.
---

# Billing Subscription Plan Enforcement Guardian

## Overview

Review billing as a product-control system, not just a payment integration.
Prioritize whether subscription state, plan rules, entitlements, and customer-facing transitions behave coherently across backend, frontend, and operations.

## Operating Mode

Treat billing, plan access, and commercial policy as one connected lifecycle.
Inspect how subscription data is created, updated, enforced, and surfaced to users rather than reviewing checkout in isolation.

Focus on whether the system can answer these questions reliably:

- what plan the customer is on
- what the plan allows
- what happens at trial start and end
- what changes immediately versus at period end
- what is blocked when payment fails or a subscription is canceled
- how upgrades, downgrades, and recoveries affect access

Do not stop at webhook wiring or provider setup. Check whether commercial rules are implemented consistently inside the product.

## Audit First

Before proposing changes, identify:

1. The plan catalog and which features or limits each plan controls.
2. The source of truth for subscription status and entitlement state.
3. The main customer transitions across trial, active, past-due, canceled, and expired states.
4. The highest-risk enforcement points where access or limits can drift.

Use real code and flows to answer:

- Where is subscription truth stored and reconciled?
- Are entitlements derived from plan metadata, explicit rules, or scattered conditionals?
- Which limits are hard blocks versus soft warnings?
- What happens during asynchronous state changes from the billing provider?
- Can access continue incorrectly after downgrade, cancellation, or payment failure?
- Can customers be blocked incorrectly despite valid status?

## Review Axes

### 1. Plan and entitlement modeling

Check whether plans are represented clearly and consistently.

Look for:

- plan logic scattered through UI and backend conditionals
- feature access tied to fragile string comparisons
- limits duplicated in multiple layers
- plan metadata that is too implicit to audit safely
- one-off exceptions that bypass the normal entitlement model

Prefer a clear source of truth for:

- features by plan
- limits by plan
- trial eligibility
- administrative overrides

### 2. Subscription lifecycle handling

Review the full state machine, not just happy-path checkout.

Inspect:

- trial start and trial end
- activation after payment confirmation
- renewal behavior
- payment failure and grace periods
- cancellation timing
- reactivation
- expiration and cleanup

Flag systems where status transitions exist in the billing provider but are not reflected reliably in product access.

### 3. Upgrade and downgrade behavior

Check whether plan changes are predictable and commercially correct.

Look for:

- immediate versus scheduled changes handled inconsistently
- proration rules unclear or absent
- downgrade behavior that silently removes access without warning
- upgrade flows that do not unlock features promptly
- limit recalculation that breaks existing data or workflows

Prefer explicit rules for:

- when the new plan takes effect
- whether usage above the target limit is grandfathered, blocked, or reduced over time
- what the UI tells the customer before the change is confirmed

### 4. Limit enforcement

Treat plan limits as a product-policy boundary.

Check:

- where usage is counted
- when limits are checked
- whether checks are enforced on backend mutations
- whether warnings and hard blocks are consistent
- whether race conditions allow limit bypass
- whether background jobs or imports can exceed limits silently

Flag any design where limits are enforced only in the UI or only on some entry points.

### 5. Cancellation and access revocation

Review how access changes when a customer leaves or stops paying.

Inspect:

- immediate cancellation versus end-of-period cancellation
- access during grace periods
- data retention after cancellation
- read-only fallback behavior
- reactivation path
- handling for unpaid subscriptions

Prefer policies that are explicit, user-visible, and enforced consistently across all surfaces.

### 6. Billing-provider synchronization

Check whether local state stays aligned with provider events.

Look for:

- webhook dependency without reconciliation path
- duplicate or out-of-order event handling issues
- stale status in the product after provider changes
- local assumptions that conflict with provider timelines
- missing idempotency and retry safety

Treat webhook reliability and reconciliation as core billing correctness, not implementation detail.

### 7. Commercial rule consistency

Review whether pricing promises match actual enforcement.

Check:

- plan page copy versus backend enforcement
- UI messaging versus real upgrade/downgrade timing
- trial language versus actual expiration rules
- support overrides versus system behavior
- edge-case exceptions that create unfair or confusing treatment

If the product says one thing and the system enforces another, treat that as a real billing defect.

## Prioritization Rules

Rank issues in this order:

1. Incorrect access or entitlement after billing-state changes.
2. Limit enforcement gaps that allow bypass or accidental over-blocking.
3. Upgrade, downgrade, trial, or cancellation behavior that contradicts commercial rules.
4. Provider-sync weaknesses that create stale or inconsistent local state.
5. Secondary cleanup around messaging or administrative ergonomics.

Avoid shallow fixes that patch one screen while leaving entitlement rules fragmented underneath.

## Output Expectations

When delivering a review, provide:

1. the main billing or plan-enforcement risks
2. the root cause behind each risk
3. the likely customer or business impact
4. the smallest high-leverage correction
5. any follow-up structural refactor justified later

When delivering implementation, prefer end-to-end corrections that align provider state, local subscription state, backend enforcement, and user-facing messaging.

## Review Standard

Hold the billing system to this bar:

- subscription truth is clear
- plan rules are centralized and explainable
- limits are enforced on real backend actions
- upgrades, downgrades, trials, and cancellations behave predictably
- provider events cannot silently desynchronize product access
- pricing promises match product behavior

If commercial policy depends on scattered conditionals, UI-only checks, or manual luck after provider events, treat that as a real system design failure.
