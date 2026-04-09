---
name: authentication-authorization-access-control-guardian
description: Review application identity and access-control design with emphasis on login flows, session management, permission checks, role models, multitenancy boundaries, route protection, and unintended data exposure between users or workspaces. Use when Codex needs to assess or improve authentication, authorization, RBAC or role enforcement, tenant isolation, protected routes, session safety, or access-control consistency across frontend and backend.
---

# Authentication Authorization Access Control Guardian

## Overview

Review access control as a system-wide correctness boundary, not as scattered middleware checks.
Prioritize who can authenticate, what they can access, how session state is maintained, and whether tenant and user boundaries hold under real usage.

## Operating Mode

Treat identity, session, and authorization behavior as one end-to-end control surface.
Inspect frontend guards, backend checks, data scoping, and session lifecycle together rather than evaluating isolated code paths.

Focus on whether the system can answer these questions reliably:

- who is this actor
- how was the actor authenticated
- what scope does the actor currently have
- what routes, records, and actions are allowed
- what happens when sessions change, expire, or are revoked
- can one user observe or affect another user's data

Do not stop at "there is middleware." Check whether access rules are actually enforced where data and mutations happen.

## Audit First

Before proposing changes, identify:

1. The supported authentication methods and login flows.
2. The session model used by web, API, and background surfaces.
3. The roles, permissions, and tenant boundaries in the domain.
4. The highest-risk read and write paths.

Use real code and flows to answer:

- Where is identity established and persisted?
- Which routes or endpoints require authentication?
- Which operations require role or permission checks?
- How is active tenant or workspace scope selected and enforced?
- Are object-level access checks performed on read and write paths?
- Can session or role state become stale across tabs, devices, or API clients?

## Review Axes

### 1. Login and identity flows

Check whether authentication flows are coherent and safe.

Look for:

- inconsistent login behavior across clients
- weak credential or token handling
- unsafe password reset or invite flows
- identity transitions that bypass normal verification
- externally authenticated users mapped ambiguously into local accounts

Prefer explicit identity lifecycle rules for signup, login, password change, reset, invite acceptance, and federated login.

### 2. Session management

Treat session handling as a core security and correctness mechanism.

Check:

- where session state lives
- cookie or token properties
- session rotation on login
- revocation on logout, password change, or reset
- TTL and refresh behavior
- cross-device and cross-tab consistency
- stale-session behavior after role or membership changes

Flag session designs that remain technically functional but are too permissive, too sticky, or too easy to misuse.

### 3. Authorization model

Review whether permissions are clear and enforceable.

Inspect:

- role definitions
- permission granularity
- default-deny versus implicit allow behavior
- how privileged actions are identified
- whether authorization logic is centralized or duplicated
- whether route-level checks are incorrectly treated as sufficient

Prefer models where authorization rules are explainable and enforced close to the business action and data access.

### 4. Object-level and tenant-level access control

Check whether the system protects records, not just screens.

Look for:

- ID-based access without ownership or workspace verification
- cross-tenant reads through shared query paths
- mutations scoped only by client-provided identifiers
- background jobs or exports missing tenant filters
- support or admin flows that can overreach without explicit gates

Treat repeated tenant filters and ownership checks as critical review points.
If one malformed request can cross workspace boundaries, treat that as a top-priority issue.

### 5. Route and screen protection

Review frontend protection, but do not confuse it with real enforcement.

Check:

- protected routes and redirects
- loading behavior while auth state resolves
- stale UI after logout or role downgrade
- hidden-but-accessible actions
- exposure of privileged data in cached client state

Flag UI guards that improve experience but are not backed by backend authorization.

### 6. Data exposure and contract leakage

Inspect whether APIs and pages reveal more than necessary.

Check:

- overbroad list/detail responses
- internal IDs or metadata exposed unnecessarily
- admin-only fields returned to normal users
- cross-user counts, summaries, or search results
- audit or support tools available through public contracts

Prefer least-privilege responses, not just least-privilege mutations.

### 7. Consistency across surfaces

Review whether the same rule applies consistently in:

- frontend route guards
- backend handlers
- background jobs
- webhooks
- exports
- support or backoffice paths

Treat policy drift across surfaces as a real access-control bug even if each piece looks reasonable in isolation.

## Prioritization Rules

Rank issues in this order:

1. Cross-user or cross-tenant data exposure.
2. Missing or bypassable authorization on sensitive mutations.
3. Session weaknesses that preserve access too long or revoke too late.
4. Role and permission ambiguity that will produce inconsistent enforcement.
5. Secondary UX or maintainability cleanup around auth flows.

Avoid shallow fixes that only hide controls in the UI while leaving the backend permissive.

## Output Expectations

When delivering a review, provide:

1. the main auth or access-control risks
2. the root cause behind each risk
3. the likely abuse or failure mode
4. the smallest high-leverage correction
5. any follow-up hardening justified after the immediate fix

When delivering implementation, prefer end-to-end corrections that align frontend behavior, backend enforcement, and data scoping.

## Review Standard

Hold the system to this bar:

- identity is established intentionally
- session lifecycle is bounded and explainable
- authorization is enforced on real business actions
- tenant boundaries hold on every read and write path
- route protection matches backend enforcement
- users only receive the data they are allowed to see

If access control depends on client honesty, remembered conventions, or one fragile middleware layer, treat that as a real design failure.
