---
name: admin-panel-internal-operations-architect
description: Design and review internal administrative systems with emphasis on backoffice panels, support tools, user management, operational controls, auditing, and internal workflows that should remain separate from the main customer product. Use when Codex needs to assess or improve admin interfaces, support operations, internal controls, audit surfaces, or backoffice capabilities without polluting the primary user experience or weakening safety boundaries.
---

# Admin Panel Internal Operations Architect

## Overview

Review internal operations tooling as a separate product surface with different users, risks, and goals from the main customer-facing application.
Prioritize operational clarity, safety, traceability, and separation from the core product experience.

## Operating Mode

Treat admin panels, support tools, and operational controls as high-leverage internal systems.
Inspect whether internal workflows are effective for operators while remaining isolated from the main product navigation, permissions, and mental model.

Focus on whether the system can support internal needs such as:

- user and workspace management
- support investigation
- operational overrides
- audit review
- billing or account interventions
- incident response and recovery actions

Do not let internal convenience leak into the main product in a way that confuses end users or weakens access boundaries.

## Audit First

Before proposing changes, identify:

1. Who uses the internal tools and what jobs they need to perform.
2. Which actions are read-only investigation versus state-changing operations.
3. Which capabilities truly belong in backoffice rather than in the main product.
4. Which operational actions carry the highest risk if used incorrectly.

Use real screens and flows to answer:

- Is the tool clearly separated from the customer-facing product?
- Can support or operations complete their tasks without workarounds?
- Are destructive or privileged actions appropriately gated?
- Is there enough context to act safely?
- Are internal tools discoverable for staff without becoming visible noise for end users?

## Review Axes

### 1. Separation from the main product

Check whether internal operations are kept distinct from normal customer experience.

Look for:

- backoffice controls mixed into standard user settings
- privileged features visible to normal users
- internal-only navigation polluting main-product IA
- operator shortcuts that distort customer-facing flows

Prefer clear separation through dedicated routes, gated entry points, and role-specific navigation.

### 2. Internal user jobs and workflow fit

Review the tool around what operators actually need to do.

Inspect workflows such as:

- finding a customer or workspace
- understanding account state quickly
- resolving support cases
- applying operational overrides
- investigating failures
- checking billing or subscription status

Flag tools that expose raw data but still force staff to jump across multiple screens to do basic operational work.

### 3. Safety of privileged actions

Treat internal actions as high-risk product capabilities.

Check:

- which actions mutate critical state
- whether confirmations are proportional to risk
- whether dangerous actions are reversible
- whether operators can preview impact before applying changes
- whether high-impact actions are clearly distinguished from read-only views

Prefer safety mechanisms that slow down dangerous actions without making routine support work painful.

### 4. Auditability and traceability

Review whether internal actions can be understood after the fact.

Look for:

- missing audit trails
- weak attribution of who changed what
- state-changing tools with no reason capture
- no visibility into when overrides were applied or removed
- support actions that leave no operational history

Treat traceability as a core requirement for admin tooling, not an optional enhancement.

### 5. User, account, and workspace management

Inspect whether administrative identity and account controls are coherent.

Check:

- account search and lookup quality
- visibility into roles, membership, plan, status, and risk state
- safe account suspension, reactivation, or override flows
- workspace-level context and tenant isolation
- support tools that can impersonate or inspect without overreaching

Flag designs where support convenience can easily become cross-tenant exposure or accidental over-control.

### 6. Operational clarity and context

Review whether internal screens provide enough context for confident decisions.

Look for:

- missing summaries of current state
- important account signals buried in raw detail
- unclear distinction between source-of-truth data and derived status
- support tools that show data without explaining operational meaning

Prefer internal screens that make current state, recent changes, and next safe actions immediately visible.

### 7. Product cleanliness

Check whether internal tooling is adding noise, complexity, or coupling to the main product.

Inspect:

- shared components or routes that force user-facing compromises
- business logic tied to hidden admin-only conditions
- backoffice-specific exceptions leaking into customer flows
- confusing copy caused by trying to satisfy staff and customers in one screen

Treat product pollution as a real design issue even if the admin feature "works."

## Prioritization Rules

Rank issues in this order:

1. Privileged actions that are unsafe, weakly gated, or poorly auditable.
2. Cross-surface pollution where internal controls damage the main user experience.
3. Operational workflows that are too slow, fragmented, or error-prone.
4. Missing context that causes support or ops mistakes.
5. Secondary UX cleanup inside the admin panel itself.

Avoid solving internal needs by simply exposing more controls in the main product.

## Output Expectations

When delivering a review, provide:

1. the main internal-operations risks or workflow problems
2. the root cause behind each issue
3. the operational or product impact
4. the smallest high-leverage correction
5. any later-stage internal tooling refactor worth sequencing after the core fix

When delivering implementation, prefer changes that strengthen separation, safety, and operator effectiveness without contaminating the customer-facing product.

## Review Standard

Hold internal tooling to this bar:

- admin capabilities are clearly separated from the main product
- operators can complete important tasks quickly
- privileged actions are safe and auditable
- account and workspace context is easy to understand
- internal tools do not confuse end users
- backoffice power does not bypass necessary controls casually

If the only way to support operations is to leak admin controls into customer-facing flows, treat that as an architectural failure.
